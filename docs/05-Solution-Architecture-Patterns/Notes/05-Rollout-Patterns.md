# 05 · Note 05 — Rollout Patterns

> **Status: Outline.** Body fills in Week 3. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** How to take a Claude deployment from demo to production at a regulated BFSI enterprise — the 4-phase rollout pattern with eval gates at each phase transition.

> **What this file is NOT.** A DevOps deployment guide. This is the organizational and architectural governance pattern for earning trust at every phase transition in a risk-averse institution.

---

## Why phased rollout is non-negotiable in BFSI

A BFSI institution cannot roll out a Claude deployment the way a startup ships a feature. The reasons are structural:

1. **RBI IT Outsourcing Guidelines** require a Board-approved IT outsourcing policy that covers third-party AI vendors. A production rollout without this policy is a regulatory violation.
2. **Change management:** a contact center with 2,000 agents cannot be retrained in a single sprint. Phased rollout is operationally necessary.
3. **CISO approval gates:** the CISO will not approve production access without evidence from a controlled environment. Shadow mode provides that evidence.
4. **Failure blast radius:** in Phase 1, a prompt injection attack or a hallucination failure affects zero customers. In Phase 4, it affects millions. The eval gate at each transition is what justifies the expanded blast radius.

---

## The 4 phases

### Phase 1: Shadow Mode

**Duration:** 4–8 weeks

**What happens:**
- Claude is deployed behind the production interface but its responses are not shown to end users or customers.
- Every production request is sent to Claude in parallel with the existing system (or with no response at all, if there is no existing AI system).
- Claude's responses are logged to the eval harness.
- Human reviewers score a sample of Claude responses against the eval criteria.

**What is not enabled:**
- No Claude output reaches customers.
- No Claude output reaches agents (they operate without AI assistance).
- No tool calls that mutate state (read-only tools only, if any).

**HITL intensity:** 100% — every response reviewed, or a statistically significant sample with manual spot-checking.

**Eval gate to exit Phase 1:**
- Pass rate on eval harness ≥ 90% across all `fail_hard` criteria.
- Hallucination rate < 1% on sampled responses.
- No critical_failure events (prompt injection attempts handled correctly).
- CISO sign-off on security controls review.

**CISO approval checkpoint:** before Phase 2 begins, the CISO (or their delegate) reviews the Phase 1 eval report and formally approves the transition. This is not optional — it is documented in the deployment plan and in the RBI IT outsourcing dossier.

---

### Phase 2: Closed Pilot

**Duration:** 4–6 weeks

**What happens:**
- Claude responses are shown to a controlled group of internal users (agents, compliance analysts, RMs) — typically 5–20 people.
- The pilot users know they are in a pilot. Their feedback is explicitly solicited.
- Human agents can see Claude's suggestions but are not required to follow them.
- All customer-impacting actions (account changes, compliance submissions) still require explicit human approval — HITL is mandatory.

**What is now enabled:**
- Claude output visible to pilot users.
- Read-only tool calls to Salesforce, ServiceNow, Snowflake.
- Claude suggestions in the agent desktop side panel.

**What is still not enabled:**
- Claude suggestions to customers (all customer-facing text is written by humans).
- Write tool calls without explicit agent confirmation.
- Autonomous multi-step actions.

**HITL intensity:** 100% on customer-impacting actions; sampling on non-impacting suggestions.

**Eval gate to exit Phase 2:**
- Pass rate ≥ 92% on eval harness.
- Pilot user satisfaction score ≥ 3.5/5 (measured via structured feedback form, not NPS).
- Escalation rate within expected range (baseline from Phase 1 ± 10%).
- No critical failures in the pilot cohort.
- Handle time delta measurably positive (even if statistically marginal at N=20 agents).

**CISO approval checkpoint:** written sign-off before Phase 3 access to expanded user population.

---

### Phase 3: Controlled Rollout

**Duration:** 6–12 weeks

**What happens:**
- Rollout expands to 10–20% of the target user population (e.g., 200 agents across 3 branches in one region).
- Geographic or organizational segmentation — not random user selection — to contain blast radius.
- Claude suggestions are now trusted enough that agents can act on them directly, but customer-facing text must still be reviewed by the agent before sending.
- Write tool calls enabled for low-risk mutations (e.g., adding a note to a Salesforce case record after agent confirms).

**What is now enabled:**
- Expanded user population.
- Low-risk write tool calls with agent confirmation.
- Performance metrics now available at statistically significant scale.

**Eval gate to exit Phase 3:**
- Pass rate ≥ 93% on eval harness.
- Handle time reduction ≥ target delta (e.g., 0.8 minutes per call).
- Escalation rate within SLA.
- No more than 1 P1/P2 incident in the pilot population over the rollout period.
- Cost per call within ±20% of the modeled estimate.
- Board-level presentation of results (for BFSI institutions where AI deployment requires board awareness).

**Rollback trigger:** if pass rate drops below 88% or a P1 incident occurs, automatic rollback to Phase 2 for the affected population. Rollback procedure must be documented and tested before Phase 3 begins.

---

### Phase 4: Production

**Duration:** Ongoing

**What happens:**
- Rollout completes to full target population.
- Customer-facing text or actions may be handled by Claude (with appropriate HITL gates for regulated actions).
- Continuous eval monitoring in place: 5–10% of production traffic sampled by the judge model daily.
- Regular eval report reviewed at monthly operating review.

**What is now enabled:**
- Full user population.
- Production-grade write tool calls for approved mutation types.
- Direct customer communication (for use cases where this is in scope — e.g., SOP search responses delivered to the customer).

**Ongoing governance:**
- Quarterly model upgrade review: run eval harness on new model versions before upgrading.
- Quarterly corpus audit (for RAG use cases): review document freshness, remove superseded circulars.
- Annual CISO review of security controls and access permissions.
- Continuous eval sampling with drift alert threshold (> 2% pass rate drop = alert, > 5% = incident).

---

## Eval gate summary

| Phase transition | Pass rate | Hallucination | Critical failures | CISO sign-off |
|---|---|---|---|---|
| Phase 1 → 2 | ≥ 90% | < 1% | Zero | Required |
| Phase 2 → 3 | ≥ 92% | < 0.5% | Zero | Required |
| Phase 3 → 4 | ≥ 93% | < 0.5% | Zero | Required + Board |
| Production continuous | ≥ 90% | < 0.5% | Zero | Quarterly review |

---

## HITL intensity by phase

| Phase | Customer-impacting actions | Agent-visible suggestions | Customer-visible text |
|---|---|---|---|
| Shadow | N/A (not visible) | N/A (not visible) | N/A |
| Closed Pilot | 100% human approval | Shown, not enforced | Human-written only |
| Controlled Rollout | 100% human approval for high-risk; sampling for low-risk | Trusted default | Human-reviewed before sending |
| Production | High-risk: 100% human; Low-risk: HITL sampling | Trusted default | Varies by use case scope |

---

## Rollback triggers and procedure

Every phase transition plan must include a documented rollback procedure. Rollback should take < 15 minutes and require no code change — it is a configuration toggle.

**Automatic rollback triggers:**
- Eval pass rate drops > 5% from phase entry baseline.
- Any single P1 incident (data leak, compliance violation, safety failure).
- Cost per call exceeds 2× the model estimate for >1 hour.
- Latency P95 exceeds 2× the SLA for >15 minutes.

**Manual rollback triggers (judgment call):**
- CISO or CTO request.
- Regulator inquiry that creates reputational risk.
- User feedback indicating systemic quality issue not yet captured by eval metrics.

**Rollback procedure:**
1. Toggle feature flag in the orchestration service (Claude disabled, fallback enabled).
2. Alert the team and log the rollback event with timestamp and reason.
3. Send notification to CISO and deployment sponsor.
4. Run root cause analysis before re-enabling.

---

## Interview-room answer (60 sec)

*"My rollout pattern for BFSI is four phases with hard eval gates between them. Shadow mode first — Claude runs in parallel with production but no one sees its output. We use this phase to build the eval evidence the CISO needs to approve the next phase. Closed pilot second — a small cohort of internal users sees Claude suggestions, but every customer-impacting action still requires human approval. Controlled rollout third — 10–20% of the user population, one region, statistically significant performance data. Production fourth — full population, continuous eval sampling. The CISO approves each phase transition — this isn't bureaucracy, it's the evidence trail that makes the RBI IT outsourcing dossier defensible. The eval gate thresholds are hard stops: below 90% pass rate or a single critical failure, we don't advance. And the rollback procedure is a feature flag toggle, not a deployment — it takes 15 minutes, not a week."*

---

## Cross-references
- Note: [01 — Reference Architecture Anatomy](01-Reference-Architecture-Anatomy.md) (Section 11 — Rollout Plan)
- Note: [03 — Failure Modes Catalog](03-Failure-Modes-Catalog.md) (rollback triggers)
- CodeLab 05: [Custom Eval Harness](../../04-Claude-Platform-Architecture/CodeLabs/05-Custom-Eval-Harness.md)
- Module 06: Evaluation Architecture
- Module 09: Security, Privacy, Governance (CISO approval documentation)
- Every SystemDesign in this module includes a Rollout Plan section referencing this pattern.

## Strong-Hire bar for this file
- Can name all four phases, their durations, and what is and is not enabled in each.
- Knows the HITL intensity at each phase and how it changes.
- Can state specific eval gate thresholds (not vague "quality must be good enough").
- Understands why Shadow Mode is mandatory in regulated environments.
- Knows the 15-minute rollback requirement and can explain it as a design constraint on the orchestration architecture.
