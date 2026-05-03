# 04 · CodeLab 03 — Structured Output

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, hands-on, BFSI-themed.

> **What this lab is.** Extend the artifact from CodeLab 02 to produce typed, validated structured responses — using Pydantic models and the tool-schema trick — building a compliance gap analysis schema for the BFSI compliance document review use case.

> **What this lab is NOT.** A JSON prettifier. Not a schema-validation library wrapper. The goal is a repeatable pattern for converting unstructured Claude responses into strongly-typed Python objects that downstream systems can consume without defensive parsing.

---

## Goal

By end of lab:

- A Python module that:
    - Defines a Pydantic schema for `ComplianceGapAnalysis`.
    - Uses the tool-schema trick to coerce Claude into returning structured JSON.
    - Deserializes the response into the Pydantic model with validation.
    - Handles partial / malformed outputs gracefully (retry with repair prompt).
    - Logs the typed result alongside the raw response for eval consumption.

The BFSI framing: you are reviewing a draft circular from the compliance team against RBI Master Direction on KYC. Claude must identify each gap, its severity, and a remediation action — all in a typed schema.

---

## Prerequisites

- CodeLab 01 complete (SDK initialized, JSONL logging in place).
- CodeLab 02 complete (tool_use loop understood).
- `pydantic>=2.0` added to `pyproject.toml`.

---

## Background: the two structured-output paths

### Path A — `response_format` (not yet on Claude; OpenAI-only as of April 2026)
The `response_format: {type: "json_schema", ...}` parameter is an OpenAI-ism. Apic's SDK does not expose this on `messages.create`. Do not use it. This is a common candidate trap in pre-sales demos.

### Path B — Tool-schema trick (the correct path for Claude)
Define a single tool whose `input_schema` is the exact Pydantic model structure you want. Set `tool_choice={"type": "tool", "name": "submit_gap_analysis"}`. Claude is forced to produce a tool call with your typed payload as its input. Deserialize with `model.model_validate(tool_input)`.

This is not a hack — it is the documented approach and the one Apic recommends for structured extraction with validation guarantees.

---

## Step-by-step outline

### Step 1 — Define the Pydantic schema

```python
# src/sun_claude_artifact/schemas/compliance_gap.py
from enum import Enum
from typing import Optional
from pydantic import BaseModel, Field

class GapSeverity(str, Enum):
    CRITICAL = "critical"      # regulatory penalty likely if not remediated
    HIGH = "high"              # material control gap
    MEDIUM = "medium"          # process deviation, reportable internally
    LOW = "low"                # cosmetic / documentation gap

class ComplianceGap(BaseModel):
    clause_reference: str = Field(
        description="The specific clause or section of the governing regulation "
                    "(e.g. 'RBI Master Direction KYC 2016, Section 18.3')."
    )
    gap_description: str = Field(
        description="Precise statement of the gap: what the regulation requires "
                    "versus what the document states or omits."
    )
    severity: GapSeverity
    remediation_action: str = Field(
        description="Concrete action required to close the gap. "
                    "Start with an imperative verb (Add, Remove, Amend, Escalate)."
    )
    owner_function: Optional[str] = Field(
        default=None,
        description="The BFSI business function responsible for remediation "
                    "(e.g. Compliance, Legal, Operations, IT)."
    )
    timeline_days: Optional[int] = Field(
        default=None,
        description="Suggested remediation timeline in calendar days."
    )

class ComplianceGapAnalysis(BaseModel):
    document_title: str
    governing_regulation: str = Field(
        description="Primary regulation being assessed against."
    )
    review_summary: str = Field(
        description="2–3 sentence executive summary of the overall compliance posture."
    )
    gaps: list[ComplianceGap]
    overall_risk_rating: GapSeverity = Field(
        description="Highest severity gap found, rolled up to document level."
    )
    reviewer_confidence: float = Field(
        ge=0.0, le=1.0,
        description="Model's self-reported confidence in the analysis (0–1). "
                    "Low values (<0.7) should trigger human review."
    )
```

**Why these fields matter in a BFSI context:**
- `clause_reference`: makes the output auditable — auditors need the precise reg cite, not paraphrases.
- `reviewer_confidence`: explicitly flags uncertainty for HITL routing. A score < 0.7 must go to a human compliance officer before acting.
- `timeline_days`: forces the model to produce actionable output, not just diagnosis.

---

### Step 2 — Generate the tool definition from the schema

```python
# src/sun_claude_artifact/structured_call.py
import json
from apic import Apic
from pydantic import ValidationError
from .schemas.compliance_gap import ComplianceGapAnalysis

client = Apic()

# Derive the tool JSON Schema from the Pydantic model.
# Pydantic v2: model_json_schema() returns a JSON Schema dict.
GAP_ANALYSIS_TOOL = {
    "name": "submit_gap_analysis",
    "description": (
        "Submit the completed compliance gap analysis. "
        "Call this exactly once with the full analysis result. "
        "Do not call any other tools."
    ),
    "input_schema": ComplianceGapAnalysis.model_json_schema(),
}
```

**Important:** `model_json_schema()` produces valid JSON Schema. Verify it contains no `$defs` references that the tool input schema doesn't resolve — Pydantic v2 inlines nested models correctly, but check before shipping.

---

### Step 3 — The `parse` helper

```python
def analyze_compliance_document(
    document_text: str,
    regulation_name: str,
    regulation_text: str,
    model: str = "claude-opus-4-7",  # Opus on hot path — near-zero hallucination tolerance
    max_retries: int = 2,
) -> ComplianceGapAnalysis:
    """
    Run a compliance gap analysis against a regulation.
    Returns a validated ComplianceGapAnalysis or raises after max_retries.
    """
    system = f"""You are a senior compliance analyst reviewing BFSI documents 
against regulatory requirements. You must identify every material gap with 
precision. You are NOT generating a summary — you are producing a structured 
audit artifact that will be reviewed by a licensed compliance officer before 
any action is taken.

Governing regulation:
<regulation>
{regulation_text}
</regulation>

Be conservative: if uncertain whether something is a gap, flag it at LOW severity 
rather than omitting it. Set reviewer_confidence below 0.7 if the document is 
ambiguous or the regulation is unclear."""

    messages = [
        {
            "role": "user",
            "content": (
                f"Analyze the following document against {regulation_name}.\n\n"
                f"<document>\n{document_text}\n</document>\n\n"
                "Submit your findings using the submit_gap_analysis tool."
            ),
        }
    ]

    for attempt in range(max_retries + 1):
        resp = client.messages.create(
            model=model,
            max_tokens=4096,
            system=system,
            tools=[GAP_ANALYSIS_TOOL],
            tool_choice={"type": "tool", "name": "submit_gap_analysis"},
            messages=messages,
        )

        # Extract the tool_use block
        tool_use_block = next(
            (b for b in resp.content if b.type == "tool_use"), None
        )
        if tool_use_block is None:
            raise RuntimeError(f"No tool_use block in response (attempt {attempt})")

        try:
            result = ComplianceGapAnalysis.model_validate(tool_use_block.input)
            return result
        except ValidationError as e:
            if attempt == max_retries:
                raise
            # Repair: feed the validation error back as a user turn
            messages.append({"role": "assistant", "content": resp.content})
            messages.append({
                "role": "user",
                "content": (
                    f"Your previous response failed schema validation:\n{e}\n\n"
                    "Fix the errors and call submit_gap_analysis again with a valid payload."
                ),
            })
```

**Why `claude-opus-4-7` here?** Compliance document review is the one use case in the BFSI stack with near-zero hallucination tolerance. The cost premium is justified. Sonnet 4.6 is appropriate for the RM copilot hot path; Opus is appropriate here.

---

### Step 4 — Logging the typed result

```python
# Extend the existing JSONL logging from CodeLab 01
import json, hashlib
from datetime import datetime, timezone
from pathlib import Path

LOG_DIR = Path("logs")
LOG_DIR.mkdir(exist_ok=True)
STRUCTURED_LOG = LOG_DIR / "structured_outputs.jsonl"

def log_gap_analysis(
    document_title: str,
    result: ComplianceGapAnalysis,
    usage,
    model: str,
):
    entry = {
        "ts": datetime.now(timezone.utc).isoformat(),
        "model": model,
        "document_title": document_title,
        "input_tokens": usage.input_tokens,
        "output_tokens": usage.output_tokens,
        "gap_count": len(result.gaps),
        "overall_risk": result.overall_risk_rating,
        "reviewer_confidence": result.reviewer_confidence,
        "result": result.model_dump(),
    }
    with STRUCTURED_LOG.open("a") as f:
        f.write(json.dumps(entry) + "\n")
```

---

### Step 5 — End-to-end smoke test

```python
# scripts/run_compliance_review.py
from src.sun_claude_artifact.structured_call import analyze_compliance_document

SAMPLE_DOC = """
Customer Identification Policy v2.3 (Internal Draft)
...
Section 4: Account Opening
New accounts require submission of PAN card and Aadhaar. 
Video KYC may be used for remote onboarding.
Re-KYC is required every 10 years for low-risk customers.
...
"""

SAMPLE_REGULATION = """
RBI Master Direction — Know Your Customer (KYC) Direction, 2016
(Updated Oct 2023)
...
Section 18: Periodic Updation of KYC
18.1 Low-risk customers: re-KYC every 10 years.
18.2 Medium-risk customers: re-KYC every 8 years.
18.3 High-risk customers: re-KYC every 2 years.
18.4 Re-KYC must capture updated Aadhaar + OTP confirmation.
...
"""

result = analyze_compliance_document(
    document_text=SAMPLE_DOC,
    regulation_name="RBI Master Direction KYC 2016",
    regulation_text=SAMPLE_REGULATION,
)

print(f"Risk: {result.overall_risk_rating} | Gaps: {len(result.gaps)} | Confidence: {result.reviewer_confidence:.2f}")
for gap in result.gaps:
    print(f"  [{gap.severity}] {gap.clause_reference}: {gap.gap_description[:80]}...")
```

---

## Common traps (interview-room traps too)

| Trap | Why it fails | Fix |
|---|---|---|
| Using `response_format: json_schema` | Not supported on Claude as of April 2026 | Use tool-schema trick |
| Asking Claude to "return JSON in your response" | Output is a string; parse errors are silent | Force via `tool_choice` |
| Using `tool_choice: "auto"` | Claude may choose not to call the tool | Use `tool_choice: {type: tool, name: ...}` |
| Deriving schema with Pydantic v1 `.schema()` | Returns different shape than v2 | Use `model_json_schema()` (Pydantic v2) |
| Putting 30K-token policy bundle in the user turn | Token cost balloons every call | Cache the regulation text (→ CodeLab 04) |

---

## Acceptance criteria

- [ ] `ComplianceGapAnalysis` Pydantic schema defined with field descriptions.
- [ ] Tool definition derived programmatically from the schema (no hand-written JSON Schema duplication).
- [ ] `analyze_compliance_document()` function returns a validated Pydantic object or raises.
- [ ] Retry-with-repair loop handles one ValidationError.
- [ ] Structured log captures result, usage, and confidence.
- [ ] Smoke test runs end-to-end on the sample document.
- [ ] `reviewer_confidence < 0.7` path is observable in the logs (even if not wired to HITL yet).

---

## Stretch goals

- Add a second schema: `PolicyChangeImpactAssessment` for comparing two versions of a circular.
- Wire `reviewer_confidence < 0.7` to a stub HITL notification (print + log entry with `hitl_required: true`).
- Benchmark Opus vs. Sonnet on the same document; record gap count and confidence delta.
- Cache the regulation text with `cache_control` (→ CodeLab 04, the next lab).

---

## What this lab feeds

- CodeLab 04 (caching) — the regulation text in `system` is the prime caching candidate.
- CodeLab 05 (eval harness) — `structured_outputs.jsonl` is a second input stream for the judge.
- Module 05 SystemDesign 04 (Compliance Document Review) — this schema is the typed payload in the production design.

---

## Cross-references
- Note: [04 — Tool Use and Structured Outputs](../Notes/04-Tool-Use-and-Structured-Outputs.md)
- Note: [03 — Prompt Caching Architecture](../Notes/03-Prompt-Caching-Architecture.md)
- Sibling: [02 — Tool Use](02-Tool-Use.md)
- Sibling: [04 — Prompt Caching](04-Prompt-Caching.md)
- Sibling: [05 — Custom Eval Harness](05-Custom-Eval-Harness.md)
- System Design: [Compliance Document Review](../../05-Solution-Architecture-Patterns/SystemDesigns/04-Compliance-Document-Review.md)

## Strong-Hire bar for this lab
- Schema is derived from Pydantic — not hand-written JSON Schema duplication.
- `tool_choice` forced — no ambiguity about whether Claude will call the tool.
- `reviewer_confidence` field is present and wired to a HITL signal (even if stub).
- Retry-with-repair works; the error message fed back is precise, not generic.
- The Opus choice is justified in a comment, not arbitrary.
