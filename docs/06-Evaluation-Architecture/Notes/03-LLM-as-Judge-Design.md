# 06 · Note 03 — LLM-as-Judge Design

> **Status: Outline.** Body fills in Week 3. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** How to design an LLM-as-judge eval system that produces reliable, calibrated scores — and the specific failure modes (positional bias, length bias, judge-evaluated-by-judge circularity) that make most out-of-the-box judge setups unreliable.

> **What this file is NOT.** A prompt library. Not a comparison of eval frameworks. Not a research-style study of judge reliability.

---

## Why judge design matters

An LLM-as-judge eval is only as reliable as the judge prompt and calibration process. An uncalibrated judge with a poorly structured prompt is not a quality gate — it is a number generator that gives the appearance of rigor while masking the same failure modes it is supposed to detect.

The two failure modes that kill judge reliability in practice:

**Positional bias:** the judge systematically scores the first response higher when comparing two options, regardless of quality. Studies on GPT-4 and Claude as judges show 10–20 percentage point bias toward the first position in pairwise evaluation. If your pairwise eval isn't randomizing position across repeated runs, your results are contaminated.

**Length bias:** the judge systematically scores longer responses higher, independent of quality. This is pervasive. A 400-word response to a yes/no compliance question that adds hedges, context, and caveats will score higher than a 50-word precise answer if the judge prompt doesn't explicitly control for conciseness.

Both biases are invisible in a single-pass eval. They only surface when you compare judge scores to human ground truth.

---

## The judge prompt anatomy

A reliable judge prompt has four mandatory components and two optional components:

### Mandatory component 1 — Task description

Tell the judge exactly what it is evaluating, in the context of the original task. Do not assume the judge knows what the system under evaluation is supposed to do.

```
You are evaluating the quality of a compliance review assistant's response.
The assistant was given: (1) a loan sanction letter, (2) excerpts from RBI's
Fair Practices Code for Lenders, and (3) the task of identifying compliance gaps.

Your job is to evaluate the assistant's response on the criteria below.
You are NOT the compliance reviewer. You are evaluating the quality of the
assistant's review, not performing the review yourself.
```

The "you are NOT doing X" framing is important — without it, the judge sometimes performs the primary task rather than evaluating the response to it.

### Mandatory component 2 — Rubric with anchored levels

Each criterion must have explicit level anchors — not just a 1–5 scale. Level anchors prevent the judge from interpreting the scale differently on different examples.

```
CRITERION: Faithfulness to retrieved documents
Score 5 — Every factual claim in the response is supported by a specific quote
          or reference from the provided RBI document excerpts. No fabricated
          references. Clause numbers, if cited, are accurate.
Score 4 — All major claims are supported. One minor unsupported hedge or
          imprecise paraphrase present, but no fabricated information.
Score 3 — Most claims are supported but one claim lacks a clear source in
          the provided documents. No fabricated references.
Score 2 — Multiple claims lack clear sources. At least one claim appears to
          generalize beyond the provided documents.
Score 1 — One or more fabricated regulatory references. Claims directly
          contradict the provided documents, or sources cited do not exist
          in the provided excerpts.
```

This level of specificity is not optional in BFSI. A rubric that says "1 = poor, 5 = excellent" for a compliance faithfulness eval is not a rubric. It is a mood indicator.

### Mandatory component 3 — Isolation of criteria

Score each criterion independently. Do not ask the judge to give an "overall quality" score — overall scores collapse criteria together, making it impossible to diagnose which dimension is failing.

```
Please score ONLY the Faithfulness criterion above. Do not consider
response length, formatting, or any other quality dimension in this score.
Your Faithfulness score should be independent of your Completeness score,
which you will provide separately.
```

### Mandatory component 4 — Structured output format

The judge must output scores in a structured format you can parse programmatically. Prose explanations are valuable for debugging but cannot be the primary signal.

```json
{
  "faithfulness_score": <1-5>,
  "faithfulness_rationale": "<one sentence explaining the score>",
  "fabricated_references": ["<list any fabricated citations here, or empty list>"],
  "completeness_score": <1-5>,
  "completeness_rationale": "<one sentence>",
  "missing_checks": ["<list any required checks not addressed, or empty list>"]
}
```

### Optional component 1 — Reference answer

For cases where a ground-truth answer exists, include it. The reference answer makes the judge's task easier and more reliable. Do not include it when you want to evaluate generative quality independent of a reference.

### Optional component 2 — Anti-bias instructions

For pairwise evals, add explicit anti-bias instructions and implement position randomization at the orchestration layer:

```
IMPORTANT: Do not let response length influence your quality scores. A concise,
accurate response should score equally to or higher than a verbose response with
the same factual content. If you notice yourself scoring the longer response
higher purely due to length, revise your score.
```

---

## Calibration: ground truth alignment before trusting judge scores

The calibration process is non-negotiable. Without it, you do not know whether your judge is measuring what you think it is measuring.

**Standard calibration protocol:**

1. **Build a calibration set.** 200–500 cases where human subject matter experts have assigned scores using the same rubric. For BFSI compliance review: compliance officers score faithfulness and completeness on a diverse sample of 300 synthetic and real-anonymized cases.

2. **Run the judge on the calibration set.** Score every case with the judge prompt.

3. **Compute agreement metrics.** The primary metric: Pearson or Spearman correlation between judge scores and human scores on each criterion. For production use, target ρ > 0.75 on each criterion. For high-stakes compliance use cases, target ρ > 0.85.

4. **Diagnose disagreement patterns.** Where the judge and humans disagree, look for systematic patterns. Common findings:
   - Judge is more lenient on hedged language ("the document suggests...") than humans
   - Judge misses subtle factual errors that humans catch with domain expertise
   - Judge over-penalizes concise answers (length bias confirmed)

5. **Iterate the judge prompt.** Address the systematic disagreements. Re-calibrate. Repeat until agreement is acceptable.

6. **Document the calibration result.** The calibration ρ score is part of the eval framework documentation. It is what you show the CISO when they ask "how reliable is your eval?"

---

## Judge model selection for BFSI

### Why Sonnet 4.6 is the default judge model

- **Capability:** Sonnet 4.6 (`claude-sonnet-4-6`) has sufficient reasoning capability to apply multi-criterion rubrics reliably. It understands regulatory language and can distinguish faithful from fabricated citations.
- **Cost:** judge model calls are paid at Sonnet rates, not Opus rates. At production eval volume (sampled ~5% of production traffic), Sonnet cost is manageable. Opus at the same volume is 3–5× more expensive with marginal quality uplift for judge tasks.
- **Latency:** the judge pipeline adds latency. Sonnet p95 latency is acceptable for async eval pipelines. Opus adds unnecessary latency when the bottleneck is not reasoning depth.
- **Residency:** Bedrock in ap-south-1 (Mumbai) serves Sonnet 4.6. Data residency constraint is satisfied.

### Why not Opus 4.7

Opus 4.7 (`claude-opus-4-7`) is overkill for judge tasks as designed above. The rubric anchors do the reasoning work — the judge's job is to apply the rubric, not to reason from first principles. Spend Opus budget on the primary task (compliance review) where reasoning depth matters. Use Sonnet for the eval layer.

Exception: if calibration reveals that Sonnet 4.6 disagrees with human raters on a specific criterion (ρ < 0.75), test Opus 4.7 on that criterion. In practice, this is rare when the rubric is well-designed.

### Why not Haiku 4.5

Haiku 4.5 (`claude-haiku-4-5-20251001`) is appropriate for exact-match and rule-based evals (format validation, schema checking). It is not reliable for multi-criterion semantic evaluation with anchored rubrics. The failure mode: Haiku applies level anchors less consistently than Sonnet, producing noisier scores. The calibration ρ is systematically lower for Haiku on semantic criteria.

### The circularity problem: don't evaluate Claude with Claude

When the production model is `claude-sonnet-4-6`, the judge should be a different model or a different version. Evaluating a Sonnet 4.6 output with a Sonnet 4.6 judge is circular — the judge is likely to share the same blind spots as the model being evaluated.

**Options:**
- Use Sonnet 4.6 as the production model and as the judge only if the tasks are clearly distinct (the judge is evaluating a domain the production model is not expert in — which is rare).
- Preferred: use a model from a different provider as the judge, or a version of Claude that has been calibrated specifically for the eval task.
- Practical default for BFSI: when `claude-sonnet-4-6` is the production model, use `claude-sonnet-4-6` as the judge only after explicit calibration on the compliance domain where you've confirmed the judge is not blind to the same failure modes as the production model. Document this explicitly in the eval framework.

---

## Positional bias and length bias: the two most common failures

### Positional bias

**Detection:** run pairwise evals with randomized position (A vs B, then B vs A). If the win rate for the same response is >55% in position 1 across 100+ cases, positional bias is present.

**Mitigation:**
1. Randomize position at the orchestration layer — every pairwise comparison is run twice with swapped positions; take the average.
2. Add explicit anti-bias instructions to the judge prompt (above).
3. Prefer single-output scoring over pairwise when the rubric permits — avoids the positional bias problem entirely.

### Length bias

**Detection:** scatter plot of judge score vs. output length for a calibration set. If there is a positive correlation (ρ > 0.3) between length and judge score independent of human score, length bias is present.

**Mitigation:**
1. Anchor each rubric level with examples of concise high-quality responses — make explicit that conciseness is not penalized.
2. Add anti-length-bias instructions (above).
3. In the structured output, ask the judge to separately flag "would removing [X] words improve this response?" — surfaces verbosity as a dimension separate from quality.

---

## The judge-as-gatekeeper design for compliance review

For the BFSI compliance document review use case, the judge does not just score — it gates. The architecture:

```
[Compliance review assistant output]
         ↓
[LLM-as-judge: Sonnet 4.6]
         ↓
{faithfulness_score, completeness_score, fabricated_references}
         ↓
Gate: faithfulness_score >= 4 AND completeness_score >= 4 AND fabricated_references == []
         ↓                           ↓
     [PASS → HITL queue]    [FAIL → automatic reject + flag for review]
```

The HITL reviewer sees only the cases that passed the judge gate. This keeps HITL volume manageable and prevents alert fatigue (see [Note 01 — Evals as Architecture Concern](./01-Evals-as-Architecture-Concern.md)).

The reject path is not silent: every failed case is logged with the judge's structured output, the failure reason, and the original model response. This log feeds the regression suite.

---

## Cross-references

- [Note 02 — Eval Taxonomy](./02-Eval-Taxonomy.md) — when to use LLM-as-judge vs. other classes
- [Note 04 — Online Monitoring & Regression Gates](./04-Online-Monitoring-and-Regression-Gates.md) — wiring the judge into production monitoring
- [Case Study 01 — Eval Framework for BFSI Compliance Review](../Case-Study/01-Eval-Framework-for-BFSI-Compliance-Review.md) — full judge design for compliance

---

## Strong-Hire bar for this file

- Can write a production-quality judge prompt skeleton from memory: task description, anchored rubric, criteria isolation, structured output
- Knows the calibration protocol and can name the target agreement metric (Spearman ρ > 0.85 for high-stakes)
- Can articulate the Sonnet-vs-Opus-vs-Haiku tradeoff for judge model selection with cost and residency rationale
- Understands the circularity problem and can name a mitigation
- Knows both positional bias and length bias, can describe detection and mitigation for each

---

## The interview-room answer (60 sec)

> "LLM-as-judge reliability depends entirely on two things: the judge prompt design and calibration against human ground truth. On the prompt side: the prompt needs four things — a task description that tells the judge what it's evaluating (not performing), an anchored rubric where each level has specific examples, isolation of criteria so you can diagnose which dimension is failing, and structured output you can parse. On calibration: you run the judge on a human-labeled set and compute Spearman correlation per criterion. You don't use it in production until you're above 0.85 on high-stakes dimensions. For BFSI compliance review, I'd use Sonnet 4.6 as the judge — capable enough to apply the rubric, cheap enough to run at production volume, and in-region on Bedrock ap-south-1. The two biases to watch for: positional bias in pairwise evals (randomize position, run twice) and length bias (anchor the rubric with concise examples, add anti-length instructions). If you don't control for those, your judge is generating noise with a credibility veneer."
