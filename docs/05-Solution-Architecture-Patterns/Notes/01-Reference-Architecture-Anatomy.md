# 05 · Note 01 — Reference Architecture Anatomy

> **Status: Outline.** Body fills in Week 3. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The anatomy of a defensible architecture document — the 11 sections every BFSI architecture needs, ordered by how a skeptical senior engineer reads them.

> **What this file is NOT.** A list of tools to include. Architecture documents are not tool inventories — they are argument documents that answer a specific question under specific constraints.

---

## Why reference architectures fail review

Most reference architectures fail at the review stage for one of three reasons:

1. **They start with the solution, not the problem.** A document that opens with "we will use RAG + Claude Sonnet" before stating the business goal tells the reviewer the author worked backward from a preferred tool stack. This signals low maturity.
2. **They omit the failure modes.** A CISO skips to the "what happens when this breaks?" section within 30 seconds. If it's missing or vague, the architecture is rejected before the benefits are read.
3. **They treat NFRs as a checkbox.** "99.9% uptime, <500ms latency" without a supporting cost model or latency decomposition is not an NFR — it is a wish list. Reviewers who have built production systems know the difference immediately.

The 11-section structure below addresses all three failure modes systematically.

---

## The 11 sections

### Section 1: Business Goal + Success Metrics

**What it contains:** One paragraph stating the specific business problem and the quantitative metrics that define success. For BFSI: not "improve customer experience" but "reduce average handle time for Tier 1 support calls from 4.2 minutes to <3.0 minutes at the same quality score."

**Why it's first:** Everything downstream — model choice, latency budget, eval criteria — must be traceable to this section. If someone challenges a decision later, you answer by pointing back to Section 1.

**What separates Strong-Hire:** Success metrics are measurable before the system launches (baseline exists) and after (measurement instrumentation is specified).

---

### Section 2: Non-Functional Requirements (NFRs)

**What it contains:** Latency (P50/P95/P99), throughput (calls/day, peak QPS), availability (SLA), data residency (in-India for BFSI), RBAC requirements, and audit log retention.

**Why it matters:** NFRs constrain every downstream decision. A <500ms P95 latency requirement with a 30K-token system prompt and no caching is physically impossible with current API latency. Documenting the NFRs early forces that conversation.

**BFSI-specific NFRs that are always present:**
- Data residency: in-India (ap-south-1 on AWS, ap Vertex region) — required by RBI IT Outsourcing Guidelines and DPDPA 2023.
- Audit log retention: minimum 8 years (RBI records management requirement).
- RBAC granularity: branch / region / role / department (minimum 4-level hierarchy).
- HITL gate: required before any customer-impacting action (account changes, credit decisions, compliance submissions).

---

### Section 3: High-Level Design (HLD)

**What it contains:** A boxes-and-arrows diagram (described in text if the doc is Markdown) showing the major system components and the primary data flows between them. No implementation detail — just the component boundaries.

**The right level of abstraction:** A senior engineer who has never seen the system should be able to draw the HLD from memory after reading Section 3 once. If it takes more than one whiteboard, the HLD is too granular.

**For Claude deployments, the canonical HLD components are:**
- Client surface (agent desktop, web app, mobile, API consumer)
- Orchestration layer (the component that manages prompts, tools, conversation state)
- Claude API / Bedrock gateway
- Retrieval layer (if RAG) — vector store, document store
- External system integrations (Salesforce, ServiceNow, Genesys)
- Audit and observability stack
- RBAC / IAM layer (crosscuts everything)

---

### Section 4: Low-Level Design (LLD)

**What it contains:** Interface definitions, data models, prompt structures, and the decision logic within each HLD component. Where the HLD shows *what* communicates, the LLD shows *how*.

**For Claude deployments:**
- Prompt template structure (system / human / tool blocks)
- Tool schemas (input/output contract, idempotency, error handling)
- Conversation state model (what is stored, where, for how long)
- Structured output schema (the Pydantic models)
- Cache breakpoint placement and justification

---

### Section 5: Data Flow

**What it contains:** The end-to-end journey of a single request — from the user action to the Claude API call to the response delivery — with every data transformation, latency budget, and security boundary labeled.

**BFSI requirement:** the data flow must show exactly where PII/PCI data enters, where it is redacted or masked, and where it exits. For DPDPA 2023, this must be traceable to a data processor agreement.

---

### Section 6: Integration Boundaries

**What it contains:** For each external system (Salesforce, ServiceNow, Genesys, Snowflake, etc.), a one-page mini-design covering: integration pattern, authentication, data exchanged, error handling, and the owner team.

See Note 02 (Integration Boundary Doctrine) for the full taxonomy.

---

### Section 7: Security Controls

**What it contains:** Authentication, authorization, data-in-transit encryption, data-at-rest encryption, prompt injection mitigations, and audit logging. The CISO reads this section second (after failure modes).

**The non-negotiables in BFSI:**
- TLS 1.2+ on all API traffic (TLS 1.3 preferred).
- API key rotation policy — not "keys are stored in Secrets Manager" but "keys rotate every 90 days, rotation is automated, a revocation procedure exists."
- Prompt injection defense: input sanitization + output validation + anomaly detection on tool calls.
- No PII in logs — log structure references must be tokenized identifiers, not raw customer data.

---

### Section 8: Eval Framework

**What it contains:** The criteria by which the system is evaluated before go-live and in production. This is not a QA section — it is an architecture section. The eval framework defines the system's quality contract.

**Minimum for BFSI:**
- Named eval criteria per use case (e.g., factual accuracy, escalation correctness, hallucination rate).
- Measurement method (LLM-as-judge, human review, deterministic checks).
- Pass/fail threshold.
- Regression policy: "no deploy if pass rate drops > 2% from baseline."
- Production monitoring: continuous sampling of live traffic.

---

### Section 9: Failure Modes and Fallbacks

**What it contains:** The catalog of failure modes and the specific fallback behavior for each. Not "we handle errors gracefully" but "if Claude returns a refusal on a sensitive topic, the system responds with a templated empathy message and escalates to a human agent within 30 seconds."

This is the section the CISO reads first. See Note 03 (Failure Modes Catalog) for the 6-class taxonomy.

**Required fallbacks in BFSI:**
- API timeout / unavailability: deterministic fallback response + human escalation.
- Hallucination detection trigger: low-confidence flag → HITL routing.
- Prompt injection detected: drop the turn, log the event, alert security team.
- Cost spike: circuit breaker on token consumption anomalies.

---

### Section 10: Cost and Latency Model

**What it contains:** Back-of-envelope token cost calculation, latency decomposition (network + inference + streaming), and a projection at target scale. For BFSI, this is the CFO's section.

See Note 04 (Cost & Latency Modeling Primer) for worked examples.

**Key discipline:** the cost model must be based on actual token counts measured from the system prompt + expected user turn length + expected response length — not estimated from marketing figures.

---

### Section 11: Rollout Plan

**What it contains:** The 4-phase progression from demo to production with eval gate criteria at each phase transition, HITL intensity at each phase, and rollback triggers.

See Note 05 (Rollout Patterns) for the 4-phase model.

---

## The CISO read-path

A CISO at a BFSI institution does not read an architecture document from Section 1 to Section 11. They read in this order:

1. **Section 9 (Failure Modes)** — "what happens when this breaks, and how bad is it?"
2. **Section 7 (Security Controls)** — "what's the attack surface and who controls it?"
3. **Section 2 (NFRs)** — "what did you promise, and can I hold you to it?"
4. **Section 8 (Eval Framework)** — "how do you know it works before I put it in front of customers?"
5. Everything else.

If Sections 9 and 7 are weak, the CISO stops reading. This is not a hypothetical — it happens in every regulated enterprise pre-sales cycle. Design your architecture documents for the CISO read-path, not the engineer read-path.

**In a pre-sales presentation:** lead with the failure modes and security controls, not the capabilities. The capabilities are assumed; the controls are the question.

---

## Interview-room answer (60 sec)

*"When I write or review an architecture document for a BFSI Claude deployment, I'm asking eleven questions in order: what business problem is this solving and how will we measure it? What are the constraints we cannot violate — latency, residency, RBAC? What are the components and how do they communicate at a high level? How do the critical interfaces work in detail? Where does data go and where does it stop? How do we integrate without ripping out the existing stack? What's the security posture — not just TLS but rotation, audit, injection defense? How do we know the model is giving the right answers before we go live? What are the failure modes and what happens when each one fires? Can we afford to run this at scale? And how do we roll it out without blowing up production? Most architects skip to the component diagram. I don't — because in regulated enterprises, the sections that matter most to the buyer's gatekeeper are failure modes and security controls, and those come last in a naive architecture doc but should be designed first."*

---

## Cross-references
- Note: [02 — Integration Boundary Doctrine](02-Integration-Boundary-Doctrine.md)
- Note: [03 — Failure Modes Catalog](03-Failure-Modes-Catalog.md)
- Note: [04 — Cost & Latency Modeling Primer](04-Cost-and-Latency-Modeling-Primer.md)
- Note: [05 — Rollout Patterns](05-Rollout-Patterns.md)
- Case Study: [Six Use-Case Meta-Pattern](../Case-Study/01-Six-Use-Case-Meta-Pattern.md)
- All SystemDesigns in this module follow the 11-section structure.

## Strong-Hire bar for this file
- Can recite the 11 sections in order without notes.
- Can articulate the CISO read-path and why it matters.
- Distinguishes between NFRs-as-architecture-constraints and NFRs-as-wishes.
- Knows that the eval framework is an architecture section, not a QA section.
- Can explain why the failure modes section is Section 9 but gets read first.
