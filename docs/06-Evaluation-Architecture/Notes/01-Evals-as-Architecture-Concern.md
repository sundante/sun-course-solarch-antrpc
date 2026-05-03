# 06 · Note 01 — Evals as Architecture Concern

> **Status: Outline.** Body fills in Week 3. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The argument for treating evals as a first-class architectural concern — not a QA gate bolted on after the fact — with real failure modes and the BFSI stakes that make this non-negotiable.

> **What this file is NOT.** A tutorial on how to write test cases. Not a list of eval tools. Not a benchmarking guide.

---

## The eval gap: what happens when you ship without them

The most common failure mode in enterprise LLM deployment is not a model that behaves badly in development — it is a model that behaved fine in development and then failed silently in production. The eval gap is the space between "Claude answered our test questions correctly" and "Claude is making reliable decisions at scale."

Three real-world patterns from the gap:

**Pattern 1 — The undiscovered hallucination.** A compliance review assistant is deployed. It answers with high confidence. No eval gate exists. Six weeks in, a compliance officer notices the system cited an RBI circular that doesn't exist. Post-mortem: the hallucination rate was 4% on structured regulatory queries. With no monitoring, 4% of submissions had a fabricated citation. It was discovered by accident, not by design.

**Pattern 2 — The prompt regression.** A prompt is updated to improve latency (shorter system prompt, fewer examples). The change ships. Two weeks later, the refusal rate on ambiguous queries doubles — users are escalating at twice the pre-change rate. No regression gate existed. The eval that would have caught it — refusal rate on a held-out ambiguous-query set — was never built.

**Pattern 3 — The model-update blind spot.** Apic ships a new Claude version with improved reasoning. The customer upgrades. Output format changes in subtle ways: JSON field names differ slightly, confidence language softens. The downstream Salesforce integration starts failing silently — fields unmapped, tickets misrouted. No format-stability eval existed. The integration degraded for three weeks before anyone noticed.

In all three patterns, the problem was not the model. The problem was the absence of a regression gate, a monitoring signal, or a format-stability eval. These are architecture failures, not model failures.

---

## Evals as architecture: the three places evals live

A well-designed eval system has three distinct layers, each with a different function:

### Layer 1 — Pre-deploy gate

**Purpose:** block the bad deploy before it happens.

**Mechanism:** a suite of offline eval cases — golden inputs with expected outputs — that must pass before any model update, prompt change, or configuration change ships to production. The gate is binary: pass rate above threshold → deploy; below threshold → block and investigate.

**What it covers:** quality metrics (faithfulness, format stability, task completion), safety metrics (refusal correctness on edge cases), and regression checks (known failure modes from prior incidents).

**Who owns it:** the Applied AI Architect, in collaboration with the customer's ML ops or platform team. Not QA. Not the vendor.

**BFSI application:** before any prompt change ships to the compliance document review assistant, it must pass a suite of ~200 eval cases covering RBI clause extraction accuracy, hallucination rate on synthetic regulatory queries, and format stability on the JSON output schema the Salesforce integration depends on.

### Layer 2 — Production monitor

**Purpose:** detect drift before users notice it.

**Mechanism:** continuous sampling of live production traffic with automated quality scoring (typically LLM-as-judge on sampled outputs) plus rate metrics (refusal rate, escalation rate, latency p95, output length distribution). Alerts fire when metrics cross thresholds; weekly review for trends below alert threshold.

**What it covers:** anything that can change without a code deploy — traffic distribution shifts, novel query patterns, user behavior changes that expose new failure modes.

**BFSI application:** for the RM copilot, the production monitor tracks action proposal acceptance rate (low acceptance → model is proposing wrong actions), escalation rate (high escalation → confidence calibration is off), and response latency (p95 > 3s → cache or infra issue).

### Layer 3 — Post-incident forensics

**Purpose:** understand why something went wrong after it happened, and build a regression test from it.

**Mechanism:** structured incident review that includes: (a) replay of the failing input through the current and prior system; (b) root cause analysis tracing back through retrieval, prompt construction, and model output; (c) addition of the failure case to the regression suite before the fix ships.

**What it covers:** the tail of the distribution — the cases you didn't anticipate. Every incident is a new data point for the eval suite.

**BFSI application:** when the compliance review assistant hallucinates an RBI circular reference, the forensic layer traces: was the circular in the retrieval corpus? Did retrieval return it? Did the model ignore it? Was the system prompt ambiguous? Each question is answered with logged data, not speculation.

---

## The BFSI eval imperative

Hallucination tolerance approaching zero plus HITL (Human-in-the-Loop) before customer-impacting actions creates a specific eval design requirement: **the eval must flag errors before they reach HITL, not after.**

This is counterintuitive. Teams often argue: "we have HITL, so the human will catch it." The problem: HITL humans are expert reviewers, not eval testers. If the error rate reaching HITL is 10%, reviewers develop alert fatigue. If it is 2%, reviewers stay sharp. The eval system is the mechanism that keeps the error rate at HITL below the fatigue threshold.

For the six BFSI use cases, this translates to:

| Use Case | Hallucination Tolerance | Eval Must Catch Before HITL |
|---|---|---|
| Compliance document review | <0.5% | Fabricated citations, wrong clause numbers |
| RM copilot action proposals | <1% | Wrong product codes, incorrect customer data |
| Internal SOP search | <2% | Answers not grounded in retrieved documents |
| Customer support agent assist | <3% | Policy misstatements, escalation misroutes |
| Exec analytics (SQL generation) | <2% | Wrong aggregations, hallucinated metrics |
| Developer productivity (Claude Code) | <5% (lower stakes) | Incorrect API usage, security antipatterns |

The eval design must be calibrated to these tolerances, not to a generic "LLM eval benchmark."

---

## Why the Applied AI Architect owns eval design

This is not the ML team's job. Here is why:

**ML teams build models.** They evaluate whether the model improved on a benchmark. They do not know what a 2% hallucination rate means to an RBI compliance officer. They do not know that a format regression in the JSON output will break the Salesforce integration at 3am. They are not in the room with the CISO when the incident question is "how did this happen?"

**QA teams test software.** They verify that features work as specified. LLM output is probabilistic — there is no "passes/fails" against a deterministic spec. QA methodology is necessary but not sufficient.

**The Applied AI Architect is the person who:**
- Was in the room when the business goal was defined
- Designed the system the model sits inside
- Knows what failure modes the customer cannot tolerate
- Will be presenting the eval framework to the CISO
- Will be accountable when the post-incident question is "did you know this could happen?"

Eval design is therefore an extension of architecture work, not a handoff item. If you design a RAG system for compliance review without also designing the eval framework, you have not finished the architecture. You have built a system with no feedback loop.

The specific architect responsibility: translate business tolerance for error into measurable eval criteria, select the eval class appropriate for each criterion, set the acceptance threshold, wire the gate into the deployment pipeline, and design the monitoring that proves the gate is working.

---

## Cross-references

- [Note 02 — Eval Taxonomy](./02-Eval-Taxonomy.md)
- [Note 03 — LLM-as-Judge Design](./03-LLM-as-Judge-Design.md)
- [Note 04 — Online Monitoring & Regression Gates](./04-Online-Monitoring-and-Regression-Gates.md)
- [Note 05 — Eval Kickoff Script](./05-Eval-Kickoff-Script.md)
- [Case Study 01 — Eval Framework for BFSI Compliance Review](../Case-Study/01-Eval-Framework-for-BFSI-Compliance-Review.md)
- [04 · Note 03 — Prompt Caching Architecture](../../04-Claude-Platform-Architecture/Notes/03-Prompt-Caching-Architecture.md) — cache state isolation in eval runs

---

## Strong-Hire bar for this file

- Can articulate the three-layer eval architecture (pre-deploy, production monitor, post-incident) without prompting
- Can explain *specifically* why HITL does not substitute for evals in a BFSI context — the alert fatigue argument
- Can name at least two concrete BFSI failure modes that would not have been caught without an eval framework
- Owns the "why the architect, not ML team" argument with a crisp justification
- Treats the eval system as part of the architecture deliverable, not a follow-on

---

## The interview-room answer (60 sec)

> "Evals are an architecture concern because they are the mechanism that turns a probabilistic model output into a production decision you can stand behind. Without them, you have a system that works in demos but fails silently at scale. In a BFSI context — hallucination tolerance approaching zero, HITL before customer-impacting actions — the eval framework has to do three things: gate deploys before bad prompts or model updates ship, monitor production for drift before users discover it, and generate regression cases from every incident. The HITL layer doesn't substitute for this — it depends on it. If your eval isn't filtering errors before they reach the human reviewer, you get alert fatigue and your 'safety net' stops working. Designing that three-layer system is part of the architecture, not a QA afterthought. If I hand off an architecture without an eval framework, I haven't finished the job."
