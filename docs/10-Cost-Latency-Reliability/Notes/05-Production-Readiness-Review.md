# 10 · Note 05 — Production Readiness Review

> **Status: Outline.** Body fills in Week 4. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The checklist a principal architect runs before giving the green light for BFSI production go-live — the complete PRR template with seven sections and five hard no-go criteria.

> **What this file is NOT.** A generic launch checklist — every item here is calibrated to regulated BFSI deployment of a Claude-based AI system.

---

## Why a formal PRR matters in BFSI

Banks don't ship fast and fix later. RBI's IT governance guidelines (Circular RBI/2023-24/73) require documented risk assessment and sign-off before production deployment of AI systems that touch customer data or customer-impacting processes. The CISO is not a stakeholder who reviews and provides input — the CISO is a gatekeeper with veto power.

The PRR is your evidence package. It proves to the CISO, the CTO, and the regulator that you have:

1. Validated the system does what it claims
2. Designed for failure
3. Controlled cost
4. Met latency commitments
5. Built the safety and audit infrastructure
6. Planned for rollback

Without a formal PRR, you do not go to production. With a PRR, you have a documented artifact that survives personnel changes and satisfies audit.

---

## PRR Section 1: Eval gate passed

**The question:** Has the system been evaluated against defined success criteria, and do those results hold on the production data distribution?

**Evidence required:**

- [ ] Faithfulness score ≥ defined threshold (e.g., >85% on RAG eval set for SOP search)
- [ ] Hallucination rate < threshold (e.g., <2% fabricated citations in compliance review)
- [ ] Safety eval passed: red-team adversarial prompts tested; refusal rate on disallowed queries >99%
- [ ] Regression suite defined and passing: adding new features does not degrade existing behavior
- [ ] Eval set is drawn from real production-like data (not synthetic only) and reviewed by domain SMEs
- [ ] Eval results reviewed and signed off by use-case product owner + CISO reviewer

**Common failure mode:** Eval is done on synthetic/clean data that doesn't reflect how real users interact with the system. A support agent that scores 92% faithfulness on curated test queries and 71% on real production queries is not ready. The eval distribution must match the production distribution.

**BFSI-specific requirement:** For compliance review (UC4), the eval must include a "false negative" rate — cases where the system misses a compliance violation. This is more dangerous than a false positive. The threshold for false negatives in compliance review must be agreed with the Compliance team and documented in the PRR.

---

## PRR Section 2: CISO review signed off

**The question:** Has the security and privacy design been reviewed and approved by the CISO's office?

**Evidence required:**

- [ ] Threat model documented and reviewed (see Module 09): prompt injection, data exfiltration, privilege escalation, adversarial misuse vectors identified
- [ ] Data flow diagram approved: shows every path that customer/employee data can take, confirms no data leaves India
- [ ] RBAC design implemented and tested: roles mapped to Claude tool access, tested with role-specific test accounts
- [ ] Audit log design approved by CISO: fields defined, retention confirmed (7 years), immutability mechanism in place
- [ ] PII handling reviewed: no PII in prompt text that exits to Claude without classification and approval; masking implemented where required
- [ ] Incident response playbook for AI-specific incidents documented (prompt injection attempt, unexpected refusal spike, data exfiltration attempt)
- [ ] DPDPA 2023 data processing notice reviewed by Legal: users informed that AI processes their data where legally required

**The CISO sign-off is a physical signature (or DocuSign equivalent) on the PRR document.** Not an email. Not a Slack acknowledgment. A documented approval. This is what survives the audit.

---

## PRR Section 3: Cost model validated

**The question:** Is the projected cost based on real pilot data, and have we tuned caching to the level we claimed?

**Evidence required:**

- [ ] Pilot cost actuals: compare pilot cost per call (actual) vs. projected cost per call (model)
- [ ] Variance explained: if actual > projected by >20%, root cause identified (e.g., cache hit rate lower than assumed, output tokens higher than assumed)
- [ ] Caching tuned: cache hit rate in pilot ≥ target (e.g., >80%) for all use cases using prompt caching
- [ ] Monthly cost projection reviewed by Finance: sign-off on projected spend and budget allocation
- [ ] Budget cap controls implemented: per-use-case daily budget caps in place, alert at 80%, hard stop at 110%
- [ ] Batch vs. synchronous split validated: all eligible offline workloads (UC4 nightly review, UC6 reports) confirmed on Batches API

**The cost model validation is not theoretical.** Run the pilot for at least 2 weeks with representative traffic before PRR. Model projections can be off 2–3× if the cache hit rate assumption was wrong.

---

## PRR Section 4: Latency budget verified

**The question:** At expected production load, does the system meet the latency SLA specified for each use case?

**Evidence required:**

- [ ] Load test executed at 150% of expected peak traffic (use case volumes from capacity plan)
- [ ] p50 and p99 TTFT measurements match design targets (see Note 02) at peak load
- [ ] Streaming enabled and tested: first visible token time verified in UI under realistic network conditions (not localhost)
- [ ] Cache cold-start latency measured: TTFT at cache miss acceptable even if higher than cache-hit case
- [ ] Regional latency verified: load test run from ap-south-1, not from developer laptop in a different region
- [ ] Latency at Bedrock confirmed: not just first-party API; Bedrock-specific TTFT captured (there is variance)

**Common failure:** Load test is run from the development environment (same region, low latency, warm cache). Production latency is 2× worse. Always test from the production environment or a faithful replica.

---

## PRR Section 5: Reliability runbook

**The question:** Does the team know what to do when Claude fails, and have they practiced it?

**Evidence required:**

- [ ] Fallback hierarchy documented and tested in staging: all five levels (retry, downgrade, cache, human escalation, degradation message)
- [ ] Circuit breaker tested: manually injected failures at rate limit to confirm circuit opens at defined threshold
- [ ] Graceful degradation UX verified: agent UI tested in "Claude unavailable" state; no error messages exposed to end customers
- [ ] Human escalation path tested end-to-end: mock incident where Claude unavailable → Genesys routing fires → human agent queue receives volume spike → confirm capacity
- [ ] On-call rotation defined: who gets paged for a P1 Claude API outage (not just "the infra team")
- [ ] Runbook page in internal wiki: step-by-step incident response, specific commands, escalation path, Apic/Bedrock support contact

**The runbook test is not theoretical.** Run a game day: inject failure into staging, have the on-call team run the runbook from scratch without guidance. Document what broke. Fix it. Re-run.

---

## PRR Section 6: Rollback plan

**The question:** If we need to revert this deployment in the first 30 days, how do we do it and what triggers the decision?

**Evidence required:**

- [ ] Rollback trigger criteria defined: specific metrics that trigger a rollback decision (e.g., refusal rate >15% sustained, false negative rate in compliance review >5%, p99 latency >3× SLA for 24 hours)
- [ ] Rollback mechanism documented: feature flag toggle, blue/green switch, or manual reversion — which one and how long does it take
- [ ] Pre-Claude fallback path validated: the system that existed before Claude integration still works; it has not been decommissioned
- [ ] Data state after rollback: are there any Claude-generated data artifacts that need to be invalidated or reviewed post-rollback?
- [ ] Communication plan: who is notified when rollback is triggered (users, business units, CISO)

**The most common rollback mistake:** Decommissioning the pre-Claude system (the old rule-based SOP search, the human-only compliance queue) before the Claude integration is proven stable. Keep the previous system on life support for 90 days post go-live. The cost of maintaining it for 90 days is negligible compared to having no rollback path.

---

## PRR Section 7: Monitoring live

**The question:** Are dashboards, alerts, and audit logs active and verified before the first production traffic hits?

**Evidence required:**

- [ ] All metrics listed in Note 04 are actively collected in production environment (not just staging)
- [ ] All P1 and P2 alerts have been test-fired and confirmed to reach PagerDuty
- [ ] Cost dashboard shows data from the pre-launch smoke test
- [ ] Audit log in Splunk: smoke test log events appear, query confirmed by CISO team
- [ ] Datadog APM traces visible for smoke test requests
- [ ] On-call team has been briefed on the dashboard; they can navigate it without assistance
- [ ] First 48-hour post-launch: dedicated monitoring shift (rotation) watching the dashboards in real time

**The "monitoring live before go-live" rule is absolute.** Never launch and then set up monitoring. If the monitoring isn't live, the go-live date moves.

---

## The five hard no-go criteria

These five conditions auto-block production regardless of timeline pressure, business urgency, or seniority of the person asking you to proceed:

| # | No-go condition | Why it's absolute |
|---|---|---|
| 1 | Eval gate not passed | Deploying an unvalidated system into a regulated environment is a compliance violation, not a calculated risk |
| 2 | CISO sign-off not obtained | RBI IT governance requires it; proceeding without it creates personal liability for the approver |
| 3 | Monitoring not live | You cannot know if the production system is behaving correctly without observability in place |
| 4 | Rollback plan has not been tested | An untested rollback plan is not a rollback plan |
| 5 | Audit log not operational | DPDPA 2023 requires demonstrable data processing records from the first transaction; there is no retroactive fix |

**How to hold the line when pressured:**

The business will push. A go-live date will be on a slide. An executive will say "we'll sort the monitoring later." The principal architect's job is to say no without losing the relationship.

Script: "I understand the date is important. If we go live today without the audit log, the first RBI inspection creates a material finding that could delay or suspend the entire program. The 5-day delay to complete the logging setup protects the program. I can help scope what's needed this week — but I can't sign the PRR without it."

---

## Cross-references
- [Note 01 — Cost Architecture](./01-Cost-Architecture.md) — cost model and caching
- [Note 02 — Latency Architecture](./02-Latency-Architecture.md) — latency budget verification
- [Note 03 — Reliability Patterns](./03-Reliability-Patterns.md) — fallback and circuit breaker
- [Note 04 — Observability](./04-Observability.md) — monitoring and audit log design
- [Module 06 — Evaluation Architecture](../../06-Evaluation-Architecture/Notes/01-Evals-as-Architecture-Concern.md)
- [Module 09 — Security, Privacy, Governance](../../09-Security-Privacy-Governance/Notes/05-CISO-Trust-Building-and-Honest-Hallucination.md)

---

## Strong-Hire bar for this file

- Articulates all seven PRR sections with concrete evidence requirements, not vague checklist items
- Names the five hard no-go criteria and has a credible script for holding the line under pressure
- Specifies BFSI-specific requirements in each section (RBI, DPDPA, CISO sign-off format)
- Understands that the CISO's sign-off is a formal approval, not an informal acknowledgment
- Recommends keeping the pre-Claude system alive for 90 days as the rollback path — practical engineering judgment
