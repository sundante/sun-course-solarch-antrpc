# 06 · Drill 01 — Design an Eval on the Whiteboard

> **Status: Outline.** Body fills in Week 3. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** A timed whiteboard drill — given a use case description, design a complete eval framework in 15 minutes covering eval class, judge design, threshold, and monitoring.

> **What this file is NOT.** A fill-in-the-blank worksheet. Not a theory review. This is a performance drill.

---

## The prompt

> *"I'm going to give you a use case description. You have 15 minutes to design a complete eval framework for it on the whiteboard. I want to see: the eval class you'd use and why, the judge design if applicable, the acceptance threshold, and how you'd monitor it in production. Go."*

**Time box:** 15 minutes per card. In a real interview, you'll have one card. Practice all four.

**Interviewer probe patterns:** expect interruptions:
- "Why that threshold and not 95%?"
- "How would you calibrate the judge?"
- "What happens when this fails in production — how do you detect it?"
- "What about the edge case where retrieval returns nothing?"

---

## Use-case cards

---

### Card 1: Compliance Document Review

**Use case description:**
> A large Indian private-sector bank uses Claude to review loan sanction letters for compliance with RBI's Fair Practices Code for Lenders. The system retrieves relevant RBI circular excerpts (RAG) and generates a structured compliance assessment: PASS/FAIL per checklist item, with evidence citations. The assessment goes to a HITL compliance officer before any customer-facing action. Hallucination tolerance is near zero — a fabricated RBI citation reaching the regulator is a P0 incident.

**Strong Hire answer (Card 1):**

*Eval framework skeleton for the whiteboard:*

**Step 1 — Define dimensions (2 min):**
Three dimensions, each with its own eval class:
1. Citation validity: does every cited RBI circular exist in the retrieved corpus?
2. Assessment faithfulness: does the compliance verdict accurately characterize the cited clause?
3. Completeness: did the assessment cover all required checklist items for this document type?

**Step 2 — Assign eval classes (3 min):**
1. **Citation validity → exact-match.** Ground truth: the RBI corpus index. Check every cited clause number and circular ID against the corpus. Pass/fail per citation. Automated, deterministic, zero compute cost beyond string matching.
2. **Assessment faithfulness → LLM-as-judge.** Ground truth: anchored 5-point rubric calibrated by compliance SMEs. Judge: Sonnet 4.6. Calibration: 300-case human-labeled set, target ρ > 0.85 per criterion.
3. **Completeness → LLM-as-judge + rule-based hybrid.** Checklist items for each document type are enumerable — rule-based check for presence of required fields. Quality of coverage is LLM-as-judge.

**Step 3 — Set thresholds (3 min):**
- Citation validity: **100% pass required.** Any fabricated citation blocks deploy and triggers P0 alert.
- Assessment faithfulness: **≥4.4/5 mean on rolling 500-case sample.** Threshold derived from calibration: score of 4.4 corresponds to <0.5% error rate on the calibration set.
- Completeness: **≥4.2/5 mean + 100% required fields present.**

**Step 4 — Regression gate wiring (3 min):**
- Pre-deploy: run all three eval dimensions on a 200-case validation set in CI. Gate: all three must pass. Run time target: <10 min.
- Cache state: dedicated eval Bedrock account to avoid production cache contamination.
- Delta-based: block if any dimension drops >0.1 from prod baseline, even if absolute score is above threshold.

**Step 5 — Production monitoring (4 min):**
- Rate metrics: fabrication flag rate (alert: any non-zero, 4-hour window), escalation rate (alert: >1.5× baseline), latency p95 (alert: >5s).
- Quality metrics: sampled judge score on 5% of production traffic. Rolling 7-day mean on faithfulness. Alert: mean drops >0.2 from baseline.
- Incident response: every failed judge case → logged with full context → added to regression suite after root cause analysis.

**Whiteboard output structure:**
```
[DIMENSIONS]
  → Citation validity (exact-match, 100% pass)
  → Faithfulness (LLM-as-judge Sonnet 4.6, ≥4.4/5)
  → Completeness (hybrid, ≥4.2/5 + 100% fields)

[GATE]
  → CI eval run on 200-case set, <10 min
  → Delta-based: block if any dimension drops >0.1 from prod

[PRODUCTION MONITOR]
  → Fabrication flag rate: P0 alert on any non-zero
  → Escalation rate: alert at 1.5× baseline
  → Rolling judge score: alert at 0.2 drop
```

---

### Card 2: SOP Search Faithfulness

**Use case description:**
> The bank uses Claude to answer employee queries over 50,000 internal SOP and policy documents. The system returns an answer with citations (document name, section, page). The main failure mode is "grounded fabrication" — Claude synthesizes an answer that sounds like it's from the documents but subtly misrepresents the policy. Employees act on these answers for process-critical decisions.

**Strong Hire answer skeleton:**

**Dimensions:**
1. Retrieval relevance: did the retrieved chunks actually contain the answer? (recall@3 on held-out query set with labeled relevant chunks)
2. Citation accuracy: do the cited document/section/page references exist and contain the claimed information? (exact-match against corpus)
3. Answer groundedness: is every factual claim in the answer supported by a specific retrieved chunk? (LLM-as-judge, faithfulness rubric)
4. Refusal appropriateness: when the answer isn't in the corpus, does the system correctly say "I don't know"? (LLM-as-judge on held-out unanswerable queries)

**Thresholds:**
- Recall@3: ≥90% (retrieval must surface relevant document in top 3)
- Citation accuracy: 100% (zero fabricated citations)
- Answer groundedness: ≥4.2/5
- Refusal rate on unanswerable queries: ≥95% (system must decline when answer isn't in corpus)

**Monitoring:** re-query rate (user searches again immediately → implicit dissatisfaction), escalation to HR/compliance helpdesk as downstream failure signal.

---

### Card 3: RM Copilot Action Proposals

**Use case description:**
> A Relationship Manager (RM) copilot surfaces meeting preparation materials, customer interaction summaries, and next-best-action proposals before every RM-client meeting. Actions include: product recommendations, service requests to log, follow-up reminders. Customer data is pulled from CRM. The main risk: wrong product code or incorrect customer context in the proposal causes the RM to offer the wrong product, triggering a misselling complaint.

**Strong Hire answer skeleton:**

**Dimensions:**
1. Customer data accuracy: does the proposal correctly reflect the CRM record? (exact-match against CRM data at proposal time — test by injecting known CRM state)
2. Product code validity: are all recommended products in the current approved product catalogue for this customer's segment/region? (exact-match against product catalogue)
3. Action proposal quality: is the proposed action appropriate for the customer's context and risk profile? (LLM-as-judge with compliance-reviewed action rubric)
4. Context completeness: did the proposal include all required customer context fields? (rule-based: required fields checklist)

**Thresholds:**
- Customer data accuracy: 100% (wrong data = misselling risk)
- Product code validity: 100%
- Action proposal quality: ≥4.0/5 (lower tolerance than compliance review — actions are HITL-reviewed by RM)
- Context completeness: 100% required fields

**Monitoring:** action acceptance rate by RM (RM modifies the proposal substantially = implicit quality signal), misselling complaint rate as lagging downstream signal.

---

### Card 4: Exec Analytics SQL Generation

**Use case description:**
> A natural-language-to-SQL assistant lets the CFO's office query a Snowflake data warehouse (150+ tables, BFSI financial data) without writing SQL. The system generates SQL from natural language, shows the query for review, executes it, and returns results with a brief English interpretation. Main failure modes: wrong aggregation (sum of balances vs. sum of counts), wrong date filter, cross-tenant data exposure (joins that accidentally pull other business unit data).

**Strong Hire answer skeleton:**

**Dimensions:**
1. SQL execution correctness: does the SQL execute without errors? (run against Snowflake test environment with known schema)
2. Semantic correctness: does the SQL answer the question asked? (LLM-as-judge: given the NL question, the generated SQL, and the expected SQL, score semantic equivalence — or test against a golden answer set using test data)
3. Security boundary compliance: does the SQL contain any cross-tenant joins or accesses to unauthorized tables? (rule-based: static analysis of the SQL against the RBAC-permitted table list for the requesting user's role)
4. Result interpretation accuracy: does the English interpretation correctly describe the query result? (LLM-as-judge, factual consistency against the actual query result)

**Thresholds:**
- SQL execution: ≥97% (3% ambiguous queries expected to fail gracefully with re-prompt)
- Security boundary: 100% (zero unauthorized table access — P0 incident if cross-tenant data returned)
- Semantic correctness: ≥4.0/5 on golden query set
- Result interpretation: ≥4.2/5

**Monitoring:** query re-request rate (user requests clarification immediately), downstream data action errors (CFO team flags discrepancies in board reports as lagging signal).

---

## Rubric

### Strong Hire

Names all four dimensions of an eval framework (eval class, judge design, threshold, monitoring) without prompting. Assigns the right eval class to each dimension with a crisp rationale (not just "I'd use LLM-as-judge for everything"). Derives the threshold from the acceptable error rate, not from a gut feeling. Describes a production monitoring setup with both rate metrics and quality metrics. Handles probe questions on threshold justification, calibration, and failure detection with specific answers.

### Hire

Gets three of the four components right. May default to LLM-as-judge for a dimension that should be exact-match (e.g., citation validity). May state the threshold without deriving it from an error rate. May describe monitoring as "check the logs" rather than naming specific metrics and alert thresholds. Recovers well when probed — shows understanding when the dimension is named for them.

### Lean No

Opens with "I'd build a test set and score it" without naming eval class, judge design, or threshold. Cannot explain why a threshold is 95% vs. 99%. Treats LLM-as-judge as the only option for every dimension. Cannot describe a production monitoring setup beyond "track accuracy." Does not mention calibration.

### Strong No

Proposes a monitoring approach that requires full human review of every output (not scalable, misunderstands the problem). Sets a threshold of "100% accuracy" without acknowledging that this requires infinite HITL. Cannot explain what "faithfulness" means in the context of a RAG system. Proposes using the same model as both the production system and the judge without noting the circularity problem.

---

## Common traps

1. **LLM-as-judge for everything.** Exact-match is better for citation validity, SQL execution, product code validation. Applying LLM-as-judge adds noise and cost where determinism is available.

2. **Setting the threshold before deriving it from the error rate.** "95% is standard" is not a threshold justification. The threshold must trace to an acceptable error rate that traces to a business consequence.

3. **Forgetting calibration.** An LLM-as-judge eval without calibration is not an eval — it is a number. Stating calibration protocol (human-labeled set, Spearman ρ target) is part of the Strong Hire answer.

4. **Monitoring = "track the accuracy metric".** Online monitoring requires both rate metrics (refusal rate, latency, cache hit) and quality metrics (sampled judge scores). Missing either category is a Hire-level answer at best.

5. **Missing the cache state trap.** Eval runs must be cache-isolated. Candidates who design eval suites without mentioning this haven't thought through the infra design.

---

## Cross-references

- [Note 01 — Evals as Architecture Concern](../Notes/01-Evals-as-Architecture-Concern.md)
- [Note 02 — Eval Taxonomy](../Notes/02-Eval-Taxonomy.md)
- [Note 03 — LLM-as-Judge Design](../Notes/03-LLM-as-Judge-Design.md)
- [Note 04 — Online Monitoring & Regression Gates](../Notes/04-Online-Monitoring-and-Regression-Gates.md)
- [Case Study 01 — Eval Framework for BFSI Compliance Review](../Case-Study/01-Eval-Framework-for-BFSI-Compliance-Review.md)

!!! personal "Personal anchor"

    Card 1 is the one most likely to appear in the real interview — compliance document review is the highest-stakes BFSI use case. Run Card 1 under timed conditions at least 5 times before the loop. Practice saying the eval dimensions, class assignments, and thresholds in sequence without hesitation.

    The probe question you're most likely to get: "Why 4.4 out of 5 for the faithfulness threshold — why not 4.0?" Your answer: "4.4 is derived from the calibration set — at that threshold, the judge-rated pass rate corresponds to <0.5% fabrication rate on the human-labeled cases. 4.0 would correspond to a ~2% fabrication rate, which the business owner confirmed is not acceptable for regulator submission."

    Have the Card 1 Strong Hire answer memorialized as a whiteboard structure, not a script. Draw the three boxes (dimensions, gate, production monitor), fill them in, then explain as you draw.
