# 05 · Note 02 — Integration Boundary Doctrine

> **Status: Outline.** Body fills in Week 3. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** Where Claude sits relative to existing enterprise systems — the architectural posture for pre-sales. How to define clean integration boundaries that don't require the customer to rip out their existing stack.

> **What this file is NOT.** An API integration tutorial. This is the doctrine for the conversation you have with the BFSI integration architect on day one of the proof-of-concept scoping call.

---

## The single most important pre-sales insight on integrations

BFSI enterprises have existing systems they have spent years and tens of millions of rupees implementing. Salesforce for CRM. Genesys for contact centre. ServiceNow for ITSM. Confluence and SharePoint for documents. These systems are not going away. The question is never "will you replace X with Claude?" — it is "where does Claude sit relative to X?"

The answer that wins deals: **Claude owns language; existing systems own state.**

Claude does not replace Salesforce as a customer-of-record system. Claude reads from Salesforce (via tool calls) to produce language, and writes back to Salesforce (via tool calls) only after HITL approval. The existing system is the source of truth. Claude is the intelligence layer.

This posture keeps the integration boundary clean, reduces the enterprise's risk perception, and gives the CISO a model they can approve: "Claude sits on top; our systems of record stay intact."

---

## The three integration postures

### Posture 1: Bolt-on

Claude is added as an overlay to an existing workflow without modifying the underlying system's data model or process flow. The existing system remains the primary user interface; Claude augments it.

**Example:** Agent Assist in Genesys Contact Centre. The call center agent's desktop is still the Genesys interface. Claude listens to the conversation (via a real-time transcript feed), surfaces relevant KB articles and policy summaries in a side panel, and suggests responses. The agent acts; Claude advises.

**When to recommend:** the enterprise wants fast time-to-value, minimal change management, and the existing workflow is defensible (just slow or inconsistent).

**Risk:** the "side panel" that nobody uses. Adoption requires workflow integration — if Claude's output requires a context switch, it won't be used.

---

### Posture 2: Embedded

Claude is embedded inside an existing system's workflow as a processing step, not an overlay. The existing system owns the UX; Claude is called at a specific decision point.

**Example:** Compliance document review via SharePoint. When a document transitions to "Compliance Review" status in SharePoint, a workflow trigger calls Claude (via the compliance review service), which produces a structured gap analysis. The output is attached to the SharePoint document as a review artifact. A compliance officer views it in SharePoint — the same interface they already use.

**When to recommend:** the enterprise has a mature existing workflow with clear trigger points, and the Claude output needs to be durable (stored, versioned, auditable) rather than ephemeral.

**Risk:** the workflow trigger is owned by a system team with a long change queue. Plan for this in the rollout timeline.

---

### Posture 3: Replace

Claude becomes the primary interface for a workflow, replacing an existing tool. This is the riskiest posture and the one that generates the most CISO scrutiny.

**Example:** replacing a legacy document search tool with a semantic search interface. Users stop typing keywords into a SharePoint search box and start asking natural language questions to a Claude-powered SOP search interface.

**When to recommend:** the existing tool is demonstrably broken (poor adoption, low satisfaction scores, active user complaints), the data migration is manageable, and the enterprise has organizational appetite for change. Do not recommend Replace as a starting point — start with Bolt-on or Embedded, prove value, then propose Replace.

**Risk:** you own the quality of the replacement. If Claude's search is worse than the broken SharePoint search in any dimension, the rollout fails.

---

## The boundary doctrine for BFSI

These are the architectural rules that hold across all six BFSI use cases:

| Rule | Rationale |
|---|---|
| Claude reads from systems of record via tools; it does not write directly | Preserves auditability and single source of truth. Mutations require HITL gate. |
| Claude does not store conversation history in its own database | Conversation state lives in the existing CRM/ITSM/ticketing system, where the enterprise has established retention policies and access controls. |
| Claude's inputs and outputs are logged to the enterprise's existing audit infrastructure | The BFSI enterprise already has a SIEM, a log aggregation system, and a data retention policy. Claude logs go there. |
| Integration is mediated by a thin orchestration service, not direct API calls from the UI | The UI calls the orchestration service; the orchestration service calls Claude and the integration backends. This decouples UI changes from model changes. |
| Every tool call that mutates state in an external system requires an explicit approval step | Applies to: Salesforce record updates, ServiceNow ticket creation/closure, any compliance submission. |

---

## Integration points: per-system doctrine

### Salesforce (CRM — RM Copilot, Agent Assist)

- **Read:** customer 360 (account data, interaction history, open cases, portfolio summary) via Salesforce REST API or connected app.
- **Write:** post-call summary, recommended next action, opportunity updates — after human agent confirms.
- **Authentication:** OAuth 2.0 connected app with scoped permissions (read-only default, write requires separate approval).
- **RBAC:** Salesforce sharing rules map to BFSI branch/region/role hierarchy — Claude only receives data the logged-in user is permitted to see.
- **Key concern:** Claude must never receive data from Salesforce records that the current user is not authorized to access. Filter at the Salesforce API query layer, not in the Claude prompt.

### ServiceNow (ITSM — Internal SOP Search, Developer Productivity)

- **Read:** incident, change, knowledge base articles.
- **Write:** incident creation (developer productivity use case — file a bug from Claude Code output), knowledge base article drafts (requires human approval before publish).
- **Authentication:** Basic auth or OAuth 2.0 (ServiceNow supports both; OAuth preferred).
- **Key concern:** ServiceNow KB articles are often the ground truth for internal SOPs. If Claude references a KB article, it must cite the article ID and version — not paraphrase without attribution.

### Genesys (CTI — Customer Support Agent Assist)

- **Real-time feed:** Genesys provides a real-time transcript stream via Genesys Cloud SDK / Event Streaming. Claude processes transcript deltas.
- **Integration point:** the orchestration service subscribes to the transcript stream; Claude is called on each complete customer utterance.
- **Latency constraint:** the agent must see the Claude suggestion within 2 seconds of the customer utterance ending. This drives the model selection (Sonnet or Haiku, not Opus) and the prompt design (no large context windows; keep system prompt cached).
- **No Genesys write path in the first phase** — agent sees suggestions in a side panel; Genesys records are not modified by Claude in shadow mode or closed pilot.

### SharePoint / Confluence (Document Management — SOP Search, Compliance Review)

- **Read:** document indexing for RAG. The integration layer runs periodic crawls; documents are chunked, embedded, and stored in a vector store.
- **Write:** compliance review artifacts (gap analysis) attached to source documents — requires human submission.
- **Access control passthrough:** SharePoint and Confluence both support group-based access control. The RAG retrieval layer must apply the same access control as the source system. Do not return document chunks to a user who cannot access the source document.
- **Key concern (CISO):** if the vector store does not enforce access control, a user can ask a natural language question and receive content from documents they should not have access to. This is a critical failure mode. The retrieval layer must be access-control-aware.

### Snowflake / Databricks (Analytics — Executive Analytics Assistant)

- **Read only** — no write path exists for the executive analytics use case. Claude generates SQL; the SQL is executed by the orchestration service using a read-only service account.
- **Row-level security:** Snowflake row access policies and Databricks Unity Catalog row filters must be applied at query time using the requesting user's identity — not bypassed by the service account.
- **Query validation:** Claude-generated SQL must be validated (no DDL, no DML, no data export functions) before execution. The validation is deterministic — regex or AST parse — not delegated to Claude.
- **Key concern:** prompt injection via data. An attacker who can insert data into a Snowflake table can potentially inject SQL into Claude's context. Parameterized query generation and query validation are mandatory.

### IAM / SSO (Crosscuts all use cases)

- **Pattern:** all Claude-facing services authenticate users via the enterprise's existing SSO (SAML 2.0 or OIDC). The JWT/assertion carries the user's role, branch, region, and department.
- **The orchestration service validates the token** before constructing any Claude prompt. The user's role context is injected into the system prompt — not as user-controllable input but as a system-controlled injection.
- **Claude cannot be manipulated into ignoring the role context** by a user prompt that claims a different identity. The role context is in the system prompt (above the human turn), and the prompt instructions explicitly state that the human turn cannot override it.

---

## Interview-room answer (60 sec)

*"My integration doctrine for Claude in BFSI has one axiom: Claude owns language, existing systems own state. That means Claude reads from Salesforce, ServiceNow, and Snowflake via tool calls — it doesn't replace them as systems of record. It writes back only after a human approves the action. Every integration is mediated by a thin orchestration service, not direct API calls from the UI — that decoupling is what lets you upgrade the model without touching 12 integration points. For access control, the rule is: whatever permission the user has in the source system, they have in Claude. We don't reimplement RBAC in the Claude layer — we pass through the enterprise's existing controls. The CISO loves this design because it means their existing governance model extends to Claude rather than being replaced by it."*

---

## Cross-references
- Note: [01 — Reference Architecture Anatomy](01-Reference-Architecture-Anatomy.md)
- Note: [03 — Failure Modes Catalog](03-Failure-Modes-Catalog.md)
- Case Study: [Six Use-Case Meta-Pattern](../Case-Study/01-Six-Use-Case-Meta-Pattern.md)
- System Design: [Customer Support Agent Assist](../SystemDesigns/01-Customer-Support-Agent-Assist.md)
- System Design: [Internal SOP & Policy Search](../SystemDesigns/02-Internal-SOP-Policy-Search.md)
- System Design: [Executive Analytics Assistant](../SystemDesigns/06-Executive-Analytics-Assistant.md)
- Module 08: Agentic and Tool Use Architecture

## Strong-Hire bar for this file
- Can articulate the three integration postures and when to recommend each.
- Knows the "Claude owns language; existing systems own state" axiom and can defend it.
- Can explain access-control passthrough for SharePoint/Confluence RAG — and why re-implementing RBAC in Claude is wrong.
- Knows why query validation for NL-to-SQL must be deterministic, not delegated to Claude.
- Can explain the CISO appeal of the thin orchestration layer pattern.
