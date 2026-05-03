# Case Study — BFSI Customer Intro

> The reference customer that threads every subsequent module. By Module 12 you will have a complete enterprise architecture portfolio for one customer — far more memorable in an interview than five disconnected toy examples.

---

## The customer

A **large Indian BFSI enterprise** evaluating Claude. Realistic profile (composite, but each detail is true to actual Indian BFSI):

| Dimension | Profile |
|---|---|
| Industry | Universal Bank — retail banking, corporate banking, wealth management, life insurance, general insurance under a single group |
| Employees | ~50,000 (Bank: ~30,000; Insurance: ~15,000; GCC: ~5,000) |
| Customers | ~80M retail + ~500K corporate |
| Branches | ~6,000 across India + a small APAC presence (Singapore, UAE, UK) |
| Regulator | RBI (banking), IRDAI (insurance), SEBI (wealth management) |
| Existing AI | Some legacy ML (credit risk scoring, fraud detection); pilot work on GPT-4 via OpenAI API and Gemini via Vertex; no production GenAI at scale yet |
| Cloud posture | Hybrid — Snowflake + Databricks for analytics, GCP and AWS for select workloads, significant on-prem (core banking, payments) |
| Strategic ambition | "Become a Tier 1 AI-enabled bank by 2028" — board-level commitment, CIO-led transformation programme |

This profile is representative of the Apic India Applied AI Architect's likely first 12 months of customer pipeline.

---

## The six use cases (in scope for Claude evaluation)

The customer is evaluating Claude for six concrete use cases — each has its own architecture, eval framework, security posture, and rollout. By Module 12 you will have designed and defended all six.

| # | Use case | Primary user | Apic value lens |
|---|---|---|---|
| 1 | **Customer support agent assist** | Branch + contact-center support agents | Steerability + low hallucination tolerance — the agent-side copilot that doesn't fabricate policy |
| 2 | **Internal SOP / policy search** | All employees, RBAC-scoped | Long-context + citation discipline — the right answer with the source attached |
| 3 | **Relationship manager copilot** | Wealth management + corporate banking RMs | Tool-use + customer 360 context — the assistant that operates inside the RM's workflow |
| 4 | **Compliance document review** | Compliance officers + legal | Steerability + refusal patterns + human-in-the-loop — the highest-stakes use case, full HITL approval before any action |
| 5 | **Developer productivity with Claude Code** | The bank's internal engineering team (~2,500 developers) | Coding capability + enterprise governance — productivity uplift with audit |
| 6 | **Executive analytics assistant** | Board, C-suite, business unit heads | Long-context + structured outputs + Snowflake/Databricks integration — natural-language analytics with cost guardrails |

---

## The constraints (hard, named at discovery)

These are the constraints the customer's CIO and CISO will surface in the first discovery call. Every architecture in this course must satisfy them — or explicitly de-scope a use case that can't.

### Regulatory + governance

- **PII + financial data handling** — strict; under DPDPA 2023 + RBI Master Directions on Outsourcing, IT Risk, IT Governance
- **RBI-style governance expectations** — board-level oversight, documented risk assessment per use case, regulatory-acceptable audit trail
- **Auditability** — every model interaction logged with tamper-resistance; queryable for regulator within 24 hours
- **Data residency** — preference for in-India data residency for production data; APAC-region acceptable for non-production workloads

### Access + identity

- **Branch + region + role + department-level access controls** — a relationship manager in Mumbai-Andheri should not see customer data from Mumbai-Bandra; a compliance officer should see redacted personal details by default
- **IAM/SSO integration** — existing Azure AD + Okta; no separate identity for Claude
- **Audit log integrity** — write-once / append-only logging, immutable, queryable

### Connectivity + deployment

- **Private connectivity preference** — VPC-attached deployment; no public-internet API calls from production paths
- **Hybrid cloud + on-prem** — production paths must support both; the wealth management copilot runs in a workload that's GCP-native; the compliance review system must work over an on-prem core banking integration
- **Cost visibility by business unit** — every Claude API call attributable to a BU for chargeback; cost dashboards must surface per-BU spend daily

### Existing systems (integration surface)

- **Salesforce** (corporate banking + wealth CRM)
- **ServiceNow** (internal IT + branch operations ticketing)
- **Genesys** (contact center telephony + IVR)
- **SharePoint + Confluence** (internal knowledge bases)
- **Snowflake + Databricks** (analytics warehouse + lakehouse)
- **Kafka** (event streaming, customer 360, transaction events)
- **Internal core banking + policy admin systems** (on-prem, REST APIs)
- **IAM/SSO** — Azure AD + Okta

### Operational

- **Low hallucination tolerance** — "approaching zero" for customer-facing and compliance use cases; evaluated via custom eval harness, not vendor benchmarks
- **Human approval before customer-impacting actions** — any agentic workflow that would email a customer, update a policy, submit a regulatory filing, etc. must pause for human review with full audit
- **Latency** — p95 ≤ 2s for support agent assist; p95 ≤ 5s for SOP search; p95 ≤ 10s for compliance review (longer because it's higher-stakes and reviewed by humans anyway)
- **Cost ceilings per use case** — defined at sponsor approval; the architect's job is to deliver under the ceiling

### Stakeholder dynamics

- **Executive demand for measurable ROI** — every use case must have a defined KPI tied to a P&L or risk metric within 6 months of go-live
- **Engineering demand for maintainability** — the customer's own engineering team will own the system after handover; architecture must be readable, evolvable, not vendor-locked at the orchestration layer
- **CISO concern about prompt injection, data leakage, audit logs** — the CISO will be a *gatekeeper*, not a stakeholder. Any architecture that doesn't pass the CISO check doesn't ship.

---

## The stakeholder map

Who you'll meet across the engagement, and what each cares about:

| Role | Primary concern | What earns trust |
|---|---|---|
| **CEO / MD** | Strategic narrative, board-defensibility, brand | Executive clarity in 5 minutes; tied to the 2028 strategy |
| **CIO** | Technology coherence, vendor discipline, multi-year roadmap | Architecture that fits the existing stack; honest trade-offs |
| **CTO / Head of Engineering** | Maintainability, build-vs-buy, internal team capability | Patterns the bank's engineers can own post-handover |
| **CISO** | Risk, audit, regulatory exposure, prompt injection, data leakage | Detailed safety lens, named threat models, refusal patterns, audit posture |
| **Head of Compliance** | Regulator narrative, RBI/IRDAI/SEBI defensibility, refusal accuracy | Eval frameworks tied to regulatory rule sets; HITL design |
| **Head of Customer Support** | Agent productivity, CSAT, AHT (average handle time) | Operational metrics with a clear before/after |
| **Head of Wealth (RM business)** | RM productivity, cross-sell uplift, client experience | Workflow integration; tool-use patterns that don't disrupt |
| **Data Platform Lead** | Snowflake/Databricks integration, governance, cost | RAG architectures that respect the existing data plane |
| **Legal / Privacy Officer** | DPDPA, contractual exposure, BAA-equivalent for AI | Clear data-handling architecture; documented retention |
| **Procurement** | Commercial terms, comparison vs alternatives, lock-in | Honest model-portability story; cost predictability |
| **Product / Business Sponsor (per use case)** | Their use case shipping, on time, with measurable value | Discovery rigor; honest scope; clear hand-off |
| **Hands-on engineering team** | Working code, working APIs, debuggability | CodeLabs-equivalent material; clear integration patterns |

---

## Why this customer is representative

For the Apic India Applied AI Architect role, this customer profile is not arbitrary — it's a synthesis of where the role's first-12-month pipeline most likely lives:

1. **Mumbai-located** — short flight from Apic India HQ, frequent in-person visits feasible
2. **Tier 1 enterprise** — the kind of account that justifies a multi-quarter Pre-Sales Architect engagement
3. **Multiple use cases in one customer** — exactly the kind of land-and-expand motion that pre-sales architects are evaluated on
4. **Regulated** — Apic's safety properties commercially differentiate here, more than in unregulated industries
5. **Hybrid cloud reality** — matches the actual Indian enterprise deployment posture, not a clean cloud-native fantasy
6. **Has stakeholder complexity** — exercises the multi-register communication skill the role demands
7. **Has explicit CISO gating** — exercises the safety lens the role rewards

If you can architect for this customer, you can architect for most of the customer pipeline you'll inherit. Internalizing this customer is one of the highest-leverage investments in this course.

---

## How this customer threads the rest of the course

| Module | What this customer adds in that module |
|---|---|
| 02 | The customer profile sharpens your "why this seat" answer — *these* are the customers you want to serve |
| 03 | The discovery call structure is built around running discovery with this customer's CIO + CISO + business sponsors |
| 04 | The Claude platform choices (Bedrock vs Vertex vs first-party, model selection per use case) are made for this customer |
| 05 | All six SystemDesigns are for this customer's six use cases |
| 06 | The eval frameworks are designed for this customer's regulator-acceptable acceptance bar |
| 07 | The RAG architecture is designed for this customer's data plane (Snowflake + Databricks + SharePoint + Confluence + Kafka) |
| 08 | The agentic and tool-use patterns are designed for this customer's HITL approval requirements |
| 09 | The CISO-ready architecture is *exactly* this customer's CISO's questions |
| 10 | The cost / latency / reliability model is for this customer's actual workloads and ceilings |
| 11 | The executive pitch is for this customer's CEO; the workshop agenda is for this customer's engineering team |
| 12 | The full loop simulation can use this customer as the live whiteboard scenario |

---

## Working memory

Three things to have at instant recall about this customer:

1. **Six use cases** — support agent assist, SOP search, RM copilot, compliance review, Claude Code, executive analytics
2. **Hardest constraints** — RBI governance, in-India residency, RBAC at branch/region/role/department level, hallucination tolerance approaching zero, HITL before customer-impacting actions
3. **The CISO is the gatekeeper, not a stakeholder** — every architecture is gated by the CISO check; design for that posture from day one
