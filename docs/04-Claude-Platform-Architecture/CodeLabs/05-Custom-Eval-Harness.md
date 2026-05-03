# 04 · CodeLab 05 — Custom Eval Harness

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, hands-on, BFSI-themed.

> **What this lab is.** Build a minimal eval harness that reads JSONL logs from CodeLab 01 (and 03), runs LLM-as-judge scoring on response quality, and outputs a PASS/FAIL report with score distribution. This harness is the "eval framework" deliverable that the application trigger checklist requires.

> **What this lab is NOT.** A replacement for production eval infrastructure (Weights & Biases, LangSmith, Apic's eval tooling). It is the minimal artifact that proves you understand eval architecture from first principles — and that you write evals before shipping, not after.

---

## Goal

By end of lab:

- A Python harness that:
    - Reads `logs/runs.jsonl` from CodeLab 01 and `logs/structured_outputs.jsonl` from CodeLab 03.
    - Scores each response using an LLM-as-judge prompt (Claude Haiku 4.5 as judge — cheap, consistent).
    - Produces a `reports/eval_report.json` with per-entry scores and a summary PASS/FAIL.
    - Prints a score distribution table to stdout.
- An eval that a skeptical senior engineer could read and trust — explicit criteria, observable scores, documented judge prompt.

---

## Prerequisites

- CodeLab 01 complete — `logs/runs.jsonl` in place.
- CodeLab 03 complete — `logs/structured_outputs.jsonl` in place.
- `pydantic>=2.0` (already added in CodeLab 03).

---

## Background: LLM-as-judge design principles

### Why LLM-as-judge for a pre-sales artifact?

In production, you would use reference-based evals (golden answers), human raters, or specialized frameworks. For a demo artifact, LLM-as-judge is the right tradeoff: it scales to your log volume, produces comparable scores across runs, and is transparent (the judge prompt is visible and auditable).

**The BFSI context makes this more pointed:** a compliance officer reading your system design will ask "how do you know the model is giving correct answers?" Your eval harness is the answer. A harness with an explicit judge prompt and documented scoring criteria is far more credible than "we tested it manually."

### Judge design rules

1. **One criterion per score.** Do not ask the judge to score "quality" — score "factual accuracy" and "citation precision" as separate dimensions.
2. **Anchored scale.** A 1–5 scale without anchor definitions produces noisy scores. Define what 1 and 5 look like for each criterion.
3. **Chain-of-thought before score.** Ask the judge to reason before emitting the score. Scores without reasoning are unreliable.
4. **Return structured output.** Use the tool-schema trick from CodeLab 03 — enforce a typed score response.
5. **Use a cheaper/faster model for judging.** `claude-haiku-4-5-20251001` is 10× cheaper than Sonnet. Judge calls at scale must not cost more than the original calls.

---

## Step-by-step outline

### Step 1 — Define the scoring schema

```python
# src/sun_claude_artifact/evals/schemas.py
from pydantic import BaseModel, Field
from enum import Enum
from typing import Optional

class PassFail(str, Enum):
    PASS = "PASS"
    FAIL = "FAIL"
    BORDERLINE = "BORDERLINE"

class EvalScore(BaseModel):
    criterion: str
    reasoning: str = Field(
        description="Step-by-step reasoning for this score. Must cite specific "
                    "evidence from the response. Do not score without reasoning."
    )
    score: int = Field(
        ge=1, le=5,
        description="Score 1–5. 5=fully satisfies criterion. 1=completely fails."
    )
    verdict: PassFail = Field(
        description="PASS if score >= 4, FAIL if score <= 2, BORDERLINE otherwise."
    )

class ResponseEvaluation(BaseModel):
    response_id: str
    eval_scores: list[EvalScore]
    overall_verdict: PassFail = Field(
        description="PASS only if all criterion scores >= 4. "
                    "FAIL if any score <= 2. Otherwise BORDERLINE."
    )
    critical_failure: Optional[str] = Field(
        default=None,
        description="If overall_verdict is FAIL, name the specific failure. "
                    "Required when verdict is FAIL."
    )
```

---

### Step 2 — Define the eval criteria

BFSI-specific criteria for the support agent assist use case (from `runs.jsonl`):

```python
# src/sun_claude_artifact/evals/criteria.py

SUPPORT_AGENT_CRITERIA = [
    {
        "name": "factual_accuracy",
        "description": (
            "Does the response contain only factually correct statements "
            "grounded in the system prompt? "
            "Score 5: every claim is verifiable from the context. "
            "Score 1: one or more invented facts (hallucinations)."
        ),
        "pass_threshold": 4,
        "fail_hard": True,   # A score <= 2 on this criterion auto-fails the response
    },
    {
        "name": "policy_adherence",
        "description": (
            "Does the response follow the behavioral instructions in the system prompt "
            "(escalation triggers, forbidden topics, tone)? "
            "Score 5: fully compliant. Score 1: violates one or more explicit instructions."
        ),
        "pass_threshold": 4,
        "fail_hard": True,
    },
    {
        "name": "response_completeness",
        "description": (
            "Does the response address all parts of the user question? "
            "Score 5: complete and actionable. Score 1: partial or evasive."
        ),
        "pass_threshold": 3,
        "fail_hard": False,
    },
    {
        "name": "escalation_correctness",
        "description": (
            "When the question involves a regulated action (account closure, loan, "
            "dispute), does the response correctly route to a human agent? "
            "Score 5: correct routing. Score 1: incorrect routing or no routing. "
            "Score N/A (encode as 5) if no regulated action in the question."
        ),
        "pass_threshold": 4,
        "fail_hard": True,
    },
]

COMPLIANCE_REVIEW_CRITERIA = [
    {
        "name": "gap_citation_precision",
        "description": (
            "Are all identified gaps cited with a specific regulatory clause reference? "
            "Score 5: every gap has a precise clause. Score 1: vague or missing citations."
        ),
        "pass_threshold": 4,
        "fail_hard": True,
    },
    {
        "name": "severity_calibration",
        "description": (
            "Are severity ratings proportionate to actual regulatory risk? "
            "Score 5: calibrated. Score 1: clearly miscalibrated (e.g., missing section 18.3 = LOW)."
        ),
        "pass_threshold": 4,
        "fail_hard": False,
    },
    {
        "name": "remediation_actionability",
        "description": (
            "Are remediation actions concrete and implementable? "
            "Score 5: each action starts with an imperative verb and is specific. "
            "Score 1: vague ('review as needed', 'improve compliance')."
        ),
        "pass_threshold": 3,
        "fail_hard": False,
    },
]
```

---

### Step 3 — The judge call

```python
# src/sun_claude_artifact/evals/judge.py
import json
from apic import Apic
from .schemas import ResponseEvaluation
from .criteria import SUPPORT_AGENT_CRITERIA

client = Apic()

EVAL_TOOL = {
    "name": "submit_evaluation",
    "description": (
        "Submit the completed evaluation. Call exactly once with all criterion scores."
    ),
    "input_schema": ResponseEvaluation.model_json_schema(),
}

JUDGE_SYSTEM = """You are a rigorous evaluator of AI assistant responses in a BFSI 
(banking, financial services, insurance) context. You score responses against explicit 
criteria. You are adversarial — you look for failure modes, not reasons to pass.

Rules:
1. Reason step-by-step before assigning any score.
2. Cite specific phrases from the response as evidence.
3. A response that invents a fact fails factual_accuracy at score 1 regardless of other quality.
4. HITL escalation failures (incorrect routing on regulated actions) are critical failures.
5. Do not award benefit of the doubt. If a claim is unverifiable, mark it low."""

def judge_response(
    response_id: str,
    system_prompt: str,
    user_prompt: str,
    assistant_response: str,
    criteria: list[dict],
) -> ResponseEvaluation:
    
    criteria_text = "\n".join([
        f"**{c['name']}**: {c['description']}"
        for c in criteria
    ])
    
    user_message = f"""Evaluate the following AI assistant response.

SYSTEM PROMPT (what the assistant was told):
<system_prompt>
{system_prompt[:2000]}  
</system_prompt>

USER QUESTION:
<user_question>
{user_prompt}
</user_question>

ASSISTANT RESPONSE:
<assistant_response>
{assistant_response}
</assistant_response>

EVALUATION CRITERIA:
{criteria_text}

Score each criterion using the submit_evaluation tool.
Response ID: {response_id}"""

    resp = client.messages.create(
        model="claude-haiku-4-5-20251001",   # Judge model: cheap + consistent
        max_tokens=2048,
        system=JUDGE_SYSTEM,
        tools=[EVAL_TOOL],
        tool_choice={"type": "tool", "name": "submit_evaluation"},
        messages=[{"role": "user", "content": user_message}],
    )
    
    tool_block = next(b for b in resp.content if b.type == "tool_use")
    return ResponseEvaluation.model_validate(tool_block.input)
```

---

### Step 4 — The harness runner

```python
# src/sun_claude_artifact/evals/harness.py
import json
from pathlib import Path
from datetime import datetime, timezone
from .judge import judge_response
from .criteria import SUPPORT_AGENT_CRITERIA, COMPLIANCE_REVIEW_CRITERIA
from .schemas import PassFail

RUNS_LOG = Path("logs/runs.jsonl")
STRUCTURED_LOG = Path("logs/structured_outputs.jsonl")
REPORT_PATH = Path("reports/eval_report.json")

def load_jsonl(path: Path) -> list[dict]:
    if not path.exists():
        return []
    return [json.loads(line) for line in path.read_text().splitlines() if line.strip()]

def run_eval_harness(max_entries: int = 50) -> dict:
    """
    Reads JSONL logs, runs LLM-as-judge, writes eval_report.json.
    Returns summary dict.
    """
    runs = load_jsonl(RUNS_LOG)[:max_entries]
    
    results = []
    pass_count = fail_count = borderline_count = 0
    
    for entry in runs:
        if not entry.get("response") or not entry.get("prompt"):
            continue
        
        system_prompt = entry.get("system_prompt", "(not logged — update CodeLab 01 logging)")
        
        evaluation = judge_response(
            response_id=entry.get("ts", "unknown"),
            system_prompt=system_prompt,
            user_prompt=entry["prompt"],
            assistant_response=entry["response"],
            criteria=SUPPORT_AGENT_CRITERIA,
        )
        
        if evaluation.overall_verdict == PassFail.PASS:
            pass_count += 1
        elif evaluation.overall_verdict == PassFail.FAIL:
            fail_count += 1
        else:
            borderline_count += 1
        
        results.append({
            "entry": entry,
            "evaluation": evaluation.model_dump(),
        })
    
    total = len(results)
    summary = {
        "generated_at": datetime.now(timezone.utc).isoformat(),
        "total_evaluated": total,
        "pass": pass_count,
        "fail": fail_count,
        "borderline": borderline_count,
        "pass_rate": pass_count / total if total > 0 else 0,
        "overall_verdict": "PASS" if (pass_count / total) >= 0.90 else "FAIL",
        "threshold": 0.90,
        "results": results,
    }
    
    REPORT_PATH.parent.mkdir(exist_ok=True)
    REPORT_PATH.write_text(json.dumps(summary, indent=2))
    
    return summary

def print_score_distribution(summary: dict):
    total = summary["total_evaluated"]
    print(f"\n{'='*60}")
    print(f"  EVAL REPORT — {summary['generated_at'][:10]}")
    print(f"{'='*60}")
    print(f"  Total evaluated:  {total}")
    print(f"  PASS:             {summary['pass']} ({summary['pass']/total*100:.1f}%)")
    print(f"  BORDERLINE:       {summary['borderline']} ({summary['borderline']/total*100:.1f}%)")
    print(f"  FAIL:             {summary['fail']} ({summary['fail']/total*100:.1f}%)")
    print(f"  Pass rate:        {summary['pass_rate']*100:.1f}%  (threshold: {summary['threshold']*100:.0f}%)")
    print(f"  Overall verdict:  {summary['overall_verdict']}")
    print(f"{'='*60}\n")
    
    print("Score breakdown by criterion:")
    criterion_scores: dict[str, list[int]] = {}
    for r in summary["results"]:
        for score_entry in r["evaluation"]["eval_scores"]:
            c = score_entry["criterion"]
            criterion_scores.setdefault(c, []).append(score_entry["score"])
    
    for criterion, scores in criterion_scores.items():
        avg = sum(scores) / len(scores)
        fails = sum(1 for s in scores if s <= 2)
        print(f"  {criterion:<30} avg={avg:.2f}  fails={fails}/{len(scores)}")
```

---

### Step 5 — CLI entrypoint

```python
# scripts/run_evals.py
from src.sun_claude_artifact.evals.harness import run_eval_harness, print_score_distribution

summary = run_eval_harness(max_entries=50)
print_score_distribution(summary)

if summary["overall_verdict"] == "FAIL":
    print("⚠  Eval harness: FAIL — review eval_report.json for failures before shipping.")
    exit(1)
else:
    print("✓  Eval harness: PASS")
```

---

## Eval report schema

```json
{
  "generated_at": "2026-04-30T10:00:00Z",
  "total_evaluated": 42,
  "pass": 38,
  "fail": 2,
  "borderline": 2,
  "pass_rate": 0.905,
  "overall_verdict": "PASS",
  "threshold": 0.90,
  "results": [
    {
      "entry": { "ts": "...", "prompt": "...", "response": "..." },
      "evaluation": {
        "response_id": "...",
        "eval_scores": [
          {
            "criterion": "factual_accuracy",
            "reasoning": "The response states X, which is directly supported by...",
            "score": 5,
            "verdict": "PASS"
          }
        ],
        "overall_verdict": "PASS",
        "critical_failure": null
      }
    }
  ]
}
```

---

## Why this matters in the interview room

When an interviewer asks "how do you ensure quality in a Claude deployment?" the expected answer is: **evals are an architecture concern, not a QA afterthought**. This harness demonstrates:

1. You ship evals before you ship features.
2. You understand that judge prompts are first-class artifacts — they must be versioned, reviewed, and tested like code.
3. You can name the failure modes you're evaluating against (hallucination, escalation misrouting) — not just "accuracy."
4. In BFSI, the harness is also a compliance artifact — the eval report is evidence that the model was tested before going live.

See Module 06 (Evaluation Architecture) for the full theory of eval frameworks at scale.

---

## Acceptance criteria

- [ ] `run_eval_harness()` reads from both JSONL logs and scores entries without crashing.
- [ ] Judge uses `claude-haiku-4-5-20251001` — not Sonnet or Opus.
- [ ] Judge returns typed `ResponseEvaluation` via tool-schema trick.
- [ ] Score distribution table printed to stdout.
- [ ] `eval_report.json` written with per-entry scores and overall verdict.
- [ ] Harness exits non-zero on FAIL (for CI integration).
- [ ] At least 4 criteria defined with explicit 1/5 anchor descriptions.

---

## Stretch goals

- Add a regression test: compare the current eval report against a stored baseline; alert on score drops > 0.1.
- Add criteria for the compliance review log (`structured_outputs.jsonl`) — gap citation precision, severity calibration.
- Add a `--fail-hard` flag that fails the harness if any single `critical_failure` is non-null.
- Add a prompt variation tester: run the same 10 prompts through two system prompts and compare score distributions.

---

## What this lab feeds

- Module 06 (Evaluation Architecture) — this harness is the concrete artifact for the eval-as-architecture discussion.
- Every System Design in Module 05 has an "eval framework" section — this lab is the implementation reference.
- The BFSI application trigger checklist requires an eval framework before go-live; this harness satisfies it.

---

## Cross-references
- Note: [04 — Tool Use and Structured Outputs](../Notes/04-Tool-Use-and-Structured-Outputs.md)
- Sibling: [01 — First Claude Call](01-First-Claude-Call.md) (provides the JSONL logs)
- Sibling: [03 — Structured Output](03-Structured-Output.md) (provides compliance review logs)
- Module 06: [Evals as Architecture Concern](../../06-Evaluation-Architecture/Notes/01-Evals-as-Architecture-Concern.md)
- System Design: [Customer Support Agent Assist](../../05-Solution-Architecture-Patterns/SystemDesigns/01-Customer-Support-Agent-Assist.md)

## Strong-Hire bar for this lab
- Judge prompt has anchored scale definitions — not just "score 1–5."
- Criteria are BFSI-specific, not generic "helpfulness/harmlessness."
- `fail_hard` criteria are distinguished from soft criteria.
- Harness exits non-zero on failure — it's CI-ready.
- The choice of Haiku as judge model is justified in a comment.
- The link to Module 06 is present: this lab is the artifact, Module 06 is the theory.
