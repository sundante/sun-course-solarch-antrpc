# 04 · Note 05 — Deployment Surfaces

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The architect's working knowledge of where Claude is deployable — first-party Apic API, AWS Bedrock, GCP Vertex — plus Claude for Work, plus MCP-style integration. Trade-offs in regions, pricing, residency, support, IAM, audit. Calibrated for an Indian BFSI customer.

> **What this file is NOT.** Marketing comparisons. Not a hyperscaler endorsement.

---

## The four surfaces

### 1. First-party Apic API
- Direct integration with Apic.
- Latest models first; broadest feature surface; fastest access to new capabilities (caching, tool use, etc.).
- Billing through Apic.
- Region availability and residency posture: verify against current Apic docs.

### 2. AWS Bedrock
- Claude available as a Bedrock-hosted model. Region-specific availability — for Indian BFSI, **Bedrock Mumbai (ap-south-1)** is load-bearing.
- IAM-native. Audit logs through CloudTrail. KMS-managed keys.
- Billing rolls into AWS contract — often a procurement win for AWS-shop customers.
- Feature lag possible — newest Apic features may arrive on first-party first.

### 3. GCP Vertex AI
- Claude available as a Vertex-hosted model. **Vertex Mumbai (asia-south1)** is the equivalent residency anchor.
- IAM via GCP IAM. Audit logs through Cloud Audit Logs.
- Billing rolls into GCP contract.
- Same feature-lag caveat.

### 4. Claude for Work
- The enterprise *product*, not the API. Claude Projects, Claude Desktop, organizational governance, end-user productivity.
- Admin controls: SSO, role-based access, retention controls, data-handling posture.
- Coexists with the API — most Tier-1 enterprise customers run *both*: Claude for Work for end-user productivity, the API for custom workflows.

→ Module 01 Q5 has the elevator-pitch version of API vs Work.

---

## The decision matrix for Indian BFSI

| Concern | First-party API | Bedrock Mumbai | Vertex Mumbai | Claude for Work |
|---|---|---|---|---|
| Data residency in Mumbai | Verify | Yes | Yes | Verify |
| IAM integration with customer's existing cloud | No (Apic-direct) | Strong (AWS-native) | Strong (GCP-native) | SSO/SAML |
| Audit logging tied to customer's existing pipeline | API logs only | CloudTrail-native | Cloud Audit Logs-native | Admin console + exports |
| Newest features first | **Yes** | Lag possible | Lag possible | Product-pace, not API-pace |
| Procurement story | Apic contract | Rolls into AWS spend | Rolls into GCP spend | Apic contract |
| Default for BFSI POC | If customer is multi-cloud | If AWS-shop | If GCP-shop | For end-user productivity layer |

---

## The three "pick the surface" patterns

### Pattern 1 — AWS-shop BFSI customer, residency-driven
**Pick:** Bedrock Mumbai. **Why:** IAM, KMS, CloudTrail are already integrated; residency is solved; procurement is a single-vendor add. **Trade-off:** feature lag on bleeding-edge Claude capabilities. Architect designs around it.

### Pattern 2 — Multi-cloud / new GenAI buyer
**Pick:** First-party API for the API workloads, Claude for Work for end-user productivity. **Why:** newest features first; cleanest experience. **Trade-off:** customer adds an Apic contract to their procurement; verify residency.

### Pattern 3 — Hybrid (most common in mature BFSI)
**Pick:** Bedrock or Vertex for the API workloads (residency + procurement), Claude for Work for end-user productivity. **Why:** segregates "developers integrating Claude" (cloud-native shape) from "knowledge workers using Claude" (admin-console shape).

---

## MCP-style integration patterns

The Model Context Protocol (MCP) is the integration pattern for connecting Claude to existing enterprise tool ecosystems without bespoke per-tool plumbing.

### What MCP gives the architect
- A standard protocol for *capability advertisement* — tools, resources, prompts.
- A standard protocol for *invocation* — your tools become callable from Claude (and other MCP-aware clients).
- A standard *security model* — capability negotiation and scoping.

### When MCP is the right call
- The customer has 5+ internal tools they want Claude to integrate with.
- The customer wants those tools reusable across Claude Code, Claude Desktop, and custom agents.
- Without MCP, you'd be building 5 bespoke tool-use integrations.

### When MCP is overkill
- One use case, one tool, one workflow. Direct tool-use loop is simpler.
- Latency-critical hot path where the MCP indirection adds round-trips.

→ Module 08 [Note 04 — MCP Integration](../../08-Agentic-and-Tool-Use-Architecture/Notes/04-MCP-Integration.md).

---

## The 30-second answer (interview-room version)

> *"Four surfaces. First-party Apic API for newest features. Bedrock Mumbai for AWS-shop BFSI customers — IAM-native, residency-solved, procurement clean, feature-lag traded for it. Vertex Mumbai for GCP-shop equivalent. Claude for Work for the end-user productivity layer — admin console, SSO, retention. Most mature BFSI customers run a hybrid: Bedrock or Vertex for API workloads, Claude for Work for knowledge workers. MCP shows up when the customer has multiple tools and wants them reusable across Claude clients."*

---

## Cross-references
- Predecessors: Notes [01](01-Claude-API-Surface.md), [02](02-Model-Selection-Matrix.md).
- Sibling: [Note 04 — Tool Use](04-Tool-Use-and-Structured-Outputs.md).
- Module 08: [MCP Integration](../../08-Agentic-and-Tool-Use-Architecture/Notes/04-MCP-Integration.md).
- Module 09: [PII / DPDPA / RBI Posture](../../09-Security-Privacy-Governance/Notes/03-PII-DPDPA-RBI-Posture.md).
- Drill: [03 — Bedrock vs Vertex vs First-Party](../Drills/03-Bedrock-vs-Vertex-vs-First-Party.md).

## Strong-Hire bar for this file
- All four surfaces named with concrete trade-offs, not adjective.
- BFSI residency answer specific (Bedrock Mumbai / Vertex Mumbai), not "we have regions."
- Hybrid pattern (API + Claude for Work) reflex for mature enterprises.
- MCP pitched only when the integration count actually justifies it.
