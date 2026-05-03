# 06 · Note 02 — Eval Taxonomy

> **Status: Outline.** Body fills in Week 3. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The four eval classes every architect must be able to name and design cold — with selection criteria, cost models, and BFSI application for each.

> **What this file is NOT.** A survey of eval tools or frameworks. Not an academic taxonomy. Not a benchmarking methodology.

---

## Why taxonomy before design

The single most common eval mistake in enterprise AI deployments is applying the wrong eval class to a problem. Teams default to LLM-as-judge for everything because it's easy to set up. They apply human eval to cases that could be automated. They skip online metrics entirely. Each mismatch creates a different failure:

- LLM-as-judge on extraction tasks introduces unnecessary noise — exact-match is cheaper and more reliable.
- Human eval on high-volume monitoring is financially unsustainable and too slow to catch drift.
- Missing online metrics means you're flying blind in production even when your offline evals are perfect.

The taxonomy is a decision tree, not a menu. For each eval criterion, ask: what is the ground-truth structure of the correct answer? That question determines the class.

---

## Class 1: Exact-match / rule-based

**Definition:** the correct answer is deterministic — there is one right answer, or the answer conforms to a verifiable rule, and correctness can be established by string comparison, regex, or structured data validation without human judgment.

**When to use:**
- Information extraction from structured documents (clause numbers, account IDs, dates)
- Format compliance (JSON schema validation, required field presence)
- Classification with a finite label set where labels are unambiguous
- Numeric answer accuracy (dollar amounts, rates, counts)

**When NOT to use:**
- Any output that requires semantic interpretation ("is this response faithful to the source?")
- Any output where multiple correct phrasings exist
- Outputs where quality matters more than correctness in the strict sense

**Cost model:** near-zero compute cost, no human time, infinitely scalable. The cheapest eval class. Use it whenever the problem structure permits.

**BFSI example — RBI clause extraction:**
> Query: "What is the minimum capital adequacy ratio under RBI Master Direction on Basel III Capital Regulations?"
>
> Ground truth: "9% (Tier 1 + Tier 2)" as structured output `{"ratio": "9%", "tier": "1+2"}`.
>
> Eval: JSON schema validation + exact-match on the ratio value. Pass/fail is deterministic. A 1000-case exact-match eval suite for RBI clause extraction can be run in under a minute at negligible cost.

**BFSI application points:**
- SOP search: extracted section headers and document IDs
- Compliance review: clause citations, regulatory reference numbers
- Exec analytics: SQL output correctness (run against a test database)
- RM copilot: product code validation, account number format

---

## Class 2: Model-based / LLM-as-judge

**Definition:** the correct answer requires semantic judgment — fluency, faithfulness to a source, adherence to a rubric, quality of reasoning — and a language model is used as the evaluator rather than a human.

**When to use:**
- Faithfulness evaluation (does the response accurately reflect the retrieved documents?)
- Quality evaluation (is this a coherent, helpful, well-structured response?)
- Safety evaluation (does this response violate a content policy?)
- Comparative evaluation (which of two responses is better on criterion X?)

**When NOT to use:**
- When exact-match is possible — LLM-as-judge adds noise to problems that have deterministic answers
- When the judge model has the same blind spots as the model being evaluated (avoid using the same model family for both)
- Without calibration against human ground truth — an uncalibrated judge is not an eval, it's an opinion

**Calibration requirement:** before trusting an LLM judge score, you must establish agreement between the judge and human evaluators on a calibration set. The standard: judge-human agreement rate on the calibration set must exceed the agreement rate you'd expect from random chance by a meaningful margin. If your judge agrees with human raters 70% of the time and chance is 50%, that's marginal. 85%+ is credible for production use.

**Judge model selection:** see [Note 03 — LLM-as-Judge Design](./03-LLM-as-Judge-Design.md) for full treatment.

**Cost model:** meaningful compute cost per eval case (one API call to the judge per case). At high volume, LLM-as-judge eval is expensive. Design for it: sample production traffic rather than evaluating every output; use exact-match as a first-pass filter; reserve LLM-as-judge for the cases that pass initial screening.

**BFSI example — compliance review faithfulness:**
> Query: "Is the submitted loan sanction letter compliant with RBI's Fair Practices Code for Lenders?"
>
> Judge task: given the loan sanction letter, the RBI Fair Practices Code excerpts (retrieved context), and the model's compliance assessment, score the assessment on faithfulness (did the model accurately characterize the relevant clauses?) and completeness (did it cover the required checks?).
>
> Judge outputs: `{"faithfulness_score": 0-5, "completeness_score": 0-5, "missing_checks": [...], "fabricated_references": [...]}`.

---

## Class 3: Human eval

**Definition:** a qualified human reviewer provides ground-truth labels or quality scores — either as the primary source of truth or as calibration for automated evals.

**When to use:**
- Calibration set construction: ground truth labels for calibrating an LLM-as-judge
- High-stakes edge cases: the 0.1% of outputs where the eval question is genuinely ambiguous and the cost of a wrong call is high
- Novel capability evaluation: new tasks where you don't yet have a reliable automated eval
- Adversarial case review: red-team outputs that are designed to evade automated detection

**When NOT to use:**
- High-volume routine monitoring — not scalable, not fast enough
- When the task is deterministic — human eval adds noise to exact-match problems
- As a substitute for automating the eval — human eval is an investment in building automated evals, not a permanent monitoring solution

**Cost model:** the most expensive eval class. Reserve for calibration and edge-case triage. A typical enterprise calibration set is 200–500 cases, reviewed once, updated quarterly. Ongoing human eval should be <5% of production volume.

**BFSI application:** in the BFSI compliance review workflow, human eval is the source of ground truth labels for the LLM-as-judge calibration set. A team of three compliance subject matter experts reviews 300 cases and assigns faithfulness + completeness scores. Those scores become the calibration target. After calibration, the LLM-as-judge runs on production volume; humans review only the cases the judge flags as borderline.

---

## Class 4: Online production metrics

**Definition:** metrics computed continuously from live production traffic — not quality scores on individual outputs but aggregate signals that indicate whether the system is behaving as expected at the population level.

**When to use:** always. Every production AI system should have online metrics. They are the only way to detect drift between eval runs.

**Core metrics to instrument from day one:**
- **Latency percentiles (p50, p95, p99):** SLA compliance; sudden p95 spike indicates infra or caching issue
- **Refusal rate:** fraction of queries where Claude declined to answer; sudden increase signals prompt regression or distribution shift
- **Escalation rate:** fraction of outputs flagged for human review; proxy for model confidence; leading indicator of quality degradation
- **Output length distribution:** median and variance; sudden compression or expansion signals format change
- **Cache hit rate:** cost indicator; sudden drop signals prompt structure change or traffic pattern shift
- **User feedback signals:** explicit thumbs-up/down or implicit (user re-queries immediately after response → likely dissatisfied)
- **Downstream action success rate:** for agent workflows, did the proposed action succeed? Did the Salesforce record get created correctly?

**Cost model:** instrumentation cost is engineering time (one sprint). Runtime cost is log storage and aggregation (low). Return is continuous visibility into production behavior. This is not optional.

**BFSI monitoring stack summary:** see per-use-case thresholds in the table below.

---

## The BFSI eval stack: which class for each use case

| Use Case | Exact-match | LLM-as-judge | Human eval | Online metrics |
|---|---|---|---|---|
| Compliance document review | RBI clause extraction, citation format | Faithfulness, completeness | Calibration set (quarterly), edge-case triage | Refusal rate, escalation rate, fabrication flag rate |
| Internal SOP search | Document ID retrieval, section header extraction | Answer groundedness, semantic relevance | Calibration set | Escalation rate, re-query rate, cache hit rate |
| RM copilot | Product code validation, account number format | Action proposal quality, customer context accuracy | Edge cases: novel products, unusual customer profiles | Action acceptance rate, escalation rate, latency p95 |
| Customer support agent assist | Knowledge article citation, escalation routing | Response quality, empathy, policy accuracy | Calibration set, complaint-adjacent cases | CSAT proxy, escalation rate, handle-time delta |
| Exec analytics (SQL generation) | SQL execution against test database, output schema | Explanation quality, business relevance of interpretation | Ambiguous query edge cases | Query success rate, re-query rate |
| Developer productivity (Claude Code) | Syntax validity, test pass rate | Code review quality, security antipattern detection | Security-sensitive code review | Acceptance rate of suggestions, rework rate |

The general principle: **exact-match gates the pipeline; LLM-as-judge scores the quality; human eval calibrates the judge; online metrics watch the whole thing.**

---

## Cross-references

- [Note 01 — Evals as Architecture Concern](./01-Evals-as-Architecture-Concern.md)
- [Note 03 — LLM-as-Judge Design](./03-LLM-as-Judge-Design.md) — judge design depth
- [Note 04 — Online Monitoring & Regression Gates](./04-Online-Monitoring-and-Regression-Gates.md) — online metrics implementation
- [Case Study 01 — Eval Framework for BFSI Compliance Review](../Case-Study/01-Eval-Framework-for-BFSI-Compliance-Review.md)

---

## Strong-Hire bar for this file

- Can name all four classes without prompting and state the selection criterion for each
- Can articulate why LLM-as-judge requires calibration and what "calibration" means operationally
- Can draw the BFSI eval stack table from memory with the right class assigned to each use case
- Understands that online metrics and offline evals are complementary, not redundant
- Knows the cost model for each class and can justify the investment

---

## The interview-room answer (60 sec)

> "I think about evals in four classes. Exact-match for anything deterministic — RBI clause extraction, JSON schema validation, SQL correctness. LLM-as-judge for semantic quality: faithfulness, completeness, response quality — but only after calibration against human ground truth on a held-out set, otherwise you're measuring the judge's opinions, not your system's quality. Human eval for calibration set construction and high-stakes edge cases — not as a monitoring solution. And online production metrics for everything, all the time — refusal rate, escalation rate, latency, output length distribution. Those four layers work together: exact-match gates the pipeline, LLM-as-judge scores quality, human eval calibrates the judge, and online metrics watch the whole stack in production. In a BFSI context with hallucination tolerance near zero, you need all four. Skipping any one of them creates a blind spot."
