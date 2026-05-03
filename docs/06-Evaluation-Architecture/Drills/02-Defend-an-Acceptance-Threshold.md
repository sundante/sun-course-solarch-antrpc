# 06 · Drill 02 — Defend an Acceptance Threshold

> **Status: Outline.** Body fills in Week 3. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** A debate drill — the interviewer names a use case and proposes an eval threshold; you defend it, counter-propose with a business justification, or accept it with documented reasoning.

> **What this file is NOT.** A fill-in-the-blank worksheet. Not a theory exercise. This is an adversarial probe drill.

---

## The prompt

> *"For this use case, the team has proposed a 95% faithfulness threshold. Defend it or counter-propose — but give me the business justification, not just a technical opinion."*

**Time box:** 5 minutes per scenario. The interviewer will probe your justification.

**What interviewers are testing:** whether you can derive a threshold from business consequences, not pull a number from intuition. A candidate who says "95% is standard" has not answered the question. A candidate who says "95% is appropriate because at our production volume of X documents per day, 5% errors means Y reviews per day reaching HITL, which the HITL team can absorb without fatigue" has answered the question.

---

## Threshold scenario 1: Compliance document review at 95% faithfulness

**Setup:** The product team proposes a 95% faithfulness threshold for the compliance document review assistant. "Ninety-five percent seems like a high bar."

**Your defense or counter-proposal:**

Counter-propose a higher threshold with business justification.

**The argument:**

95% faithfulness sounds high. At production volume, it isn't. The bank processes 200 compliance document reviews per day. A 95% faithfulness rate means 10 unfaithful assessments reach the HITL queue per day. The HITL compliance officer's job is to review borderline cases, not to re-review every AI output. If 10 of 200 submissions have quality issues, the compliance officer is spending 30–40 minutes per day re-working AI errors on top of their normal review load. That eliminates a large portion of the efficiency gain.

More critically: "unfaithful" in this context includes fabricated RBI citations. A 5% error rate on a faithfulness criterion that includes fabrication means some fabrications will reach the HITL queue. Some will be caught. Some won't. The acceptable error rate for fabricated citations reaching the regulator is functionally zero — not "95% of citations are real."

**Counter-proposal:** 
- Faithfulness criterion split into two sub-criteria: citation validity (100% required, exact-match) and assessment quality (≥4.4/5 on the rubric, which corresponds to ~0.5% assessment-level errors).
- At 200 docs/day, 0.5% = 1 borderline assessment per day reaching HITL. Manageable. HITL reviews it. Error rate to regulator approaches the current human-only baseline.

**Why not 99%:** 99% is operationally equivalent to "near-perfect" and would require either (a) a much stricter system prompt that increases refusal rate significantly, reducing coverage, or (b) a larger HITL team, which eliminates the efficiency gain. 99.5% on citation validity (exact-match) is free — it costs nothing beyond the corpus lookup. 99.5% on semantic faithfulness requires a different tradeoff.

---

## Threshold scenario 2: SOP search at 90% groundedness

**Setup:** The ML team proposes a 90% groundedness threshold for the SOP search assistant. "We think 90% is realistic given the document corpus."

**Your defense or counter-proposal:**

Challenge the threshold with a volume argument, then accept a modified version.

**The argument:**

90% groundedness on 500 SOP queries per day means 50 ungrounded answers per day. Employees are acting on these answers for process-critical decisions — onboarding procedures, KYC requirements, branch operations. 50 wrong answers per day is not a quality bar; it is a liability generator.

The operational question: what is an employee supposed to do when an ungrounded answer is acted on? If the answer is "the system should say 'I don't know' when the answer isn't in the corpus," then the 90% groundedness threshold needs to be accompanied by a high unanswerable-query refusal rate (≥95%) — the system must decline rather than fabricate.

**Modified acceptance:**
- Groundedness: ≥4.2/5 on grounded queries (those where the answer exists in the corpus)
- Refusal rate on unanswerable queries: ≥95% (the system must refuse when it doesn't have a grounded answer)
- These two together are more meaningful than a single 90% groundedness score across all queries

**Why the ML team's framing is incomplete:** "90% is realistic" is a capability statement, not a business threshold. The question is not what the model can achieve — it is what the business can accept. Start with the acceptable wrong-answer rate (derived from the employee action consequence), then work back to the technical threshold.

---

## Threshold scenario 3: RM copilot at 80% action proposal acceptance rate

**Setup:** The product owner proposes that 80% of RM copilot action proposals should be accepted by the RM without modification. "That seems like a good target for adoption."

**Your defense or counter-proposal:**

Defend the 80% target but clarify what it measures.

**The argument:**

80% acceptance rate is a reasonable adoption target, but "accepted without modification" is not an eval criterion — it is a behavior signal. The eval design must unpack what drives the 20% modification rate:

- If 20% of proposals are modified because the RM adds context or personalizes the approach → quality signal: proposals are a useful starting point, not wrong
- If 20% of proposals are modified because the product code is wrong → quality signal: data accuracy failure, needs an exact-match eval on product catalogue
- If 20% of proposals are modified because the RM disagrees with the strategic recommendation → quality signal: reasonable disagreement, acceptable
- If 20% of proposals are modified because the customer data is wrong → quality signal: CRM data accuracy failure, P0 issue

The 80% acceptance rate should be tracked as a production metric, but the eval framework must have separate dimensions for data accuracy (100% threshold) and proposal quality (≥4.0/5). The acceptance rate is the outcome; the eval dimensions are the causes.

**What you're accepting:** 80% adoption target as a north star product metric. What you're adding: decompose the 20% modification rate into root causes, and set separate eval thresholds for each cause.

---

## Threshold scenario 4: Exec analytics SQL at 97% execution success

**Setup:** The data engineering team proposes 97% SQL execution success as the primary eval criterion. "If the SQL runs, it's a success."

**Your defense or counter-proposal:**

Reject "execution success" as the primary criterion. Counter-propose a semantic correctness criterion as the primary.

**The argument:**

SQL that executes is not the same as SQL that answers the question asked. A query that returns a result without error but aggregates on the wrong column, filters on the wrong date range, or joins the wrong tables is worse than a query that fails — because it fails silently. The CFO makes a decision based on a wrong number with no indication that the number is wrong.

The eval must have:
1. **Execution success (97%):** necessary but not sufficient. A query that fails is caught immediately — the user re-queries. A query that succeeds with wrong results is a hidden failure.
2. **Semantic correctness (≥4.0/5):** the primary quality criterion. Does the SQL answer the question asked? Evaluated against a golden SQL set on test data.
3. **Security boundary (100%):** does the SQL access only tables the requesting user's role is permitted to access? Any unauthorized table access is a P0 incident — DPDPA and RBI data governance implications.

**Why execution success alone is insufficient:** "it ran" is the SQL equivalent of "Claude returned a response." It tells you nothing about quality. The right analogy: you wouldn't accept "Claude didn't crash" as a faithfulness eval for compliance review.

---

## Threshold scenario 5: Customer support agent assist at 85% CSAT

**Setup:** The CX team proposes 85% CSAT (customer satisfaction score) as the success criterion for the support agent assist. "Our current CSAT is 78% — 85% is ambitious."

**Your defense or counter-proposal:**

Accept CSAT as the north star metric, but flag that it cannot be the eval criterion.

**The argument:**

85% CSAT is a valid business target and a good north star metric. It is not an eval criterion for the AI model, because:

1. CSAT is an outcome that aggregates many factors: agent performance, resolution time, channel experience, IVR routing, product quality. The AI model's contribution is one of several drivers. A 7-point CSAT improvement target may require improvements on factors the AI model doesn't control.

2. CSAT is a lagging indicator — you find out about a quality failure weeks after it happens, not in time to prevent the next one.

3. CSAT is collected for a minority of interactions — most customers don't rate the support interaction. The sample is biased toward extreme experiences (very satisfied or very dissatisfied).

**What to use instead:**
- **Pre-eval:** response relevance (LLM-as-judge on sample: does the agent assist response correctly address the query type?) ≥4.0/5
- **Pre-eval:** escalation routing accuracy (exact-match: was the escalation route correct for the query type?) ≥95%
- **Production metric:** escalation rate (fraction of interactions requiring senior agent involvement) — target: reduce from current baseline
- **Production metric:** re-contact rate (customer calls back within 7 days about the same issue) — proxy for first-contact resolution

**What you're accepting:** 85% CSAT as the business target that the eval framework must ultimately serve. The eval dimensions are the leading indicators that predict whether CSAT will improve.

---

## Rubric

### Strong Hire

Doesn't accept or reject thresholds on gut feel. Derives every threshold from (a) production volume, (b) acceptable error count per day/week, and (c) the consequence of each error type. Distinguishes between production metrics (CSAT, acceptance rate) and eval criteria (faithfulness score, execution success). Knows when to split a single threshold into multiple sub-criteria (citation validity vs. assessment quality in scenario 1). Can name the asymmetry between "SQL runs" and "SQL is correct."

### Hire

Gets the direction right (too lenient / too strict) but can't fully derive the threshold from business consequences. May accept "97% execution success" without flagging the semantic correctness gap. May agree that CSAT is a good eval criterion without identifying the lag/bias problems. Recovers when probe questions reveal the gap — shows understanding when it's surfaced.

### Lean No

Accepts proposed thresholds as reasonable without justification. "95% sounds right." "80% is a good adoption target." Cannot explain why 95% faithfulness for compliance review might be too low at production volume. Treats all use cases as equivalent — does not adjust threshold reasoning based on consequence severity.

### Strong No

Proposes 100% as the threshold for every dimension without acknowledging the HITL-vs-efficiency trade-off. Cannot distinguish execution success from semantic correctness. Treats CSAT as a model eval criterion. Does not connect threshold to production volume at any point.

---

## Common traps

1. **"X% is standard."** There is no standard threshold. Every threshold derives from volume, consequence, and business tolerance. If you're citing a standard number, you're guessing.

2. **Single threshold for compound criteria.** Faithfulness has at least two sub-criteria (citation validity + assessment quality) with different thresholds and eval classes. Collapsing them into one number loses diagnostic value.

3. **Conflating production metrics with eval criteria.** CSAT, adoption rate, and action acceptance rate are production outcomes. They are useful north star metrics but not eval criteria — they're too lagged and too aggregated.

4. **Ignoring the volume argument.** A threshold that sounds high (95%) can represent a large absolute error count at production volume. Always translate the threshold to an error count per day.

5. **Accepting a threshold without noting its monitoring counterpart.** A threshold without an online monitoring signal is a snapshot, not a gate. Every accepted threshold must have a production metric that tracks whether it's holding.

---

## Cross-references

- [Note 02 — Eval Taxonomy](../Notes/02-Eval-Taxonomy.md) — eval class selection
- [Note 04 — Online Monitoring & Regression Gates](../Notes/04-Online-Monitoring-and-Regression-Gates.md) — threshold-to-monitoring translation
- [Note 05 — Eval Kickoff Script](../Notes/05-Eval-Kickoff-Script.md) — the Question 3/Question 5 pair for threshold derivation

!!! personal "Personal anchor"

    The scenario most likely to appear in the Apic interview is scenario 1 (compliance review at 95% faithfulness). The answer pattern: counter-propose, split the criterion, use volume math. Memorize: "200 docs/day × 5% = 10 ungrounded assessments per day in the HITL queue. That's the number that makes the threshold concrete."

    The scenario you're most likely to fumble: scenario 4 (SQL execution success). The trap is agreeing that "97% execution success is a good eval." Practice the counter: "Execution success is necessary but not sufficient — it tells you the SQL ran, not that it answered the question."
