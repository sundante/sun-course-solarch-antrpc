# 08 · Note 04 — MCP Integration

> **Status: Outline.** Body fills in Week 4. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** Model Context Protocol — what it is, when to use it, how to design the BFSI MCP server catalog, and why the security model matters more than the capability model.

> **What this file is NOT.** An MCP SDK tutorial. Not a complete spec reproduction — only the architectural decisions that matter for a pre-sales conversation.

---

## MCP in one sentence

MCP (Model Context Protocol) is a standardised protocol for giving Claude structured access to tools, resources, and prompts across multiple surfaces — so that the same integration logic doesn't get reimplemented for every app that wants to use it.

The longer version: MCP defines how a server exposes capabilities (tools, resources, prompts) and how a client (Claude, or a Claude-powered app) discovers and calls them. The protocol separates *what Claude can do* from *how that capability is implemented* — the same MCP server that powers an internal Claude desktop can power the RM copilot web app without rebuilding the integration.

---

## When MCP is the right pattern

MCP earns its complexity cost when:

1. **Multiple Claude surfaces need the same tools.** If the Salesforce integration is built directly into the RM copilot, it has to be rebuilt for the compliance review agent and the exec analytics assistant. An MCP server for Salesforce means all three surfaces share one implementation.

2. **The tool catalog is large (>5 tools) and evolving.** Individual tool definitions hardcoded into system prompts don't scale. An MCP server provides capability discovery — the client fetches the current tool list at runtime rather than having it baked into a deploy.

3. **Non-Claude applications already use the integration.** An MCP server sits between Claude and the system-of-record. Other consumers (existing apps) may continue to use the system-of-record directly; MCP adds Claude access without changing the existing integration.

4. **Security and auditing must be centralised.** All Claude tool calls routed through MCP pass through a single authentication, authorisation, and logging point. Without MCP, each Claude-integrated app has its own auth logic.

### When MCP is NOT the right pattern
- One Claude app, two or three tools, stable capability set. Bespoke tool use is simpler and has less moving parts.
- The integration requires synchronous real-time processing that the MCP server's request/response model doesn't fit.
- You're at POC stage — do not over-engineer the integration layer before you've validated the use case.

---

## The BFSI MCP server catalog

For the running BFSI case, the following MCP servers are designed:

### MCP Server 1: Salesforce CRM
**What it exposes:**
- `get_customer_360` (read)
- `log_crm_interaction` (write, HITL-gated)
- `create_service_request` (write, HITL-gated)
- `list_open_service_requests` (read)
- `get_account_manager_portfolio` (read, RM-scoped)

**Auth:** OAuth 2.0 with Salesforce Connected App. Token scoped to the RM's Salesforce identity, not a service account — RBAC enforcement at the Salesforce layer, not the MCP layer.

**HITL gate:** Write tools return a `pending_approval` response that includes a draft payload. The application presents this to the RM for approval. The MCP server does not commit the write until a second confirmation call arrives.

### MCP Server 2: ServiceNow
**What it exposes:**
- `create_service_request` (write)
- `get_ticket_status` (read)
- `list_pending_approvals` (read)
- `update_ticket_resolution` (write)

**Auth:** ServiceNow OAuth with role-based scopes. The RM copilot receives `itil_user` scope; the compliance review agent receives `compliance_user` scope. Separate tokens, separate permissions.

### MCP Server 3: Snowflake (Read-only analytics)
**What it exposes:**
- `run_named_query` (executes a pre-approved parameterised query by name — not free SQL)
- `get_portfolio_snapshot` (read)
- `get_disbursement_summary` (read)
- `get_product_performance` (read)

**Critical design decision:** The Snowflake MCP server exposes **named queries only**, not a SQL execution interface. This is a deliberate constraint. Allowing Claude to write free SQL creates a prompt injection attack surface (see [09 · Note 01](../../09-Security-Privacy-Governance/Notes/01-Prompt-Injection-Classes-and-Defenses.md)). Named parameterised queries are pre-approved by the data team and tested for performance and security.

### MCP Server 4: SharePoint (Document retrieval)
**What it exposes:**
- `search_documents` (semantic search over SharePoint index)
- `get_document` (retrieve a specific document by ID)
- `list_recent_policy_updates` (read)

**Auth:** Microsoft Graph API with SharePoint delegated permissions. Documents retrieved are filtered to the requesting user's SharePoint access groups — the MCP server validates permissions before returning content.

### MCP Server 5: Internal APIs (Composite)
**What it exposes:**
- `check_compliance_flags` (read, from compliance system)
- `get_eligible_products` (read, from product eligibility engine)
- `get_aml_status` (read, from AML platform)
- `submit_compliance_review` (write, mandatory HITL)

---

## MCP vs bespoke tool use: the decision rule

| Dimension | MCP | Bespoke tool use |
|---|---|---|
| Number of surfaces using the tools | Many (3+) | One |
| Tool catalog size and stability | Large, evolving | Small, stable |
| Centralised auth and audit requirement | Yes | No requirement |
| POC / early-stage | No | Yes |
| System-of-record has existing API | Yes (reuse) | No (new integration) |
| Real-time latency budget | > 500ms acceptable | Sub-100ms required |

In practice: start with bespoke tool use in the POC. Move to MCP servers in production when you have 2+ surfaces sharing tools.

---

## Security model: the MCP server as trust boundary

The most important architectural point about MCP for BFSI: **the security boundary is the MCP server, not Claude**.

Claude sees tool names and descriptions. Claude does not see the implementation, the credentials, or the underlying API. The MCP server is the last line of enforcement for:
- **Authentication:** Is the requesting Claude instance authorised to call this tool? (OAuth token, not trust-me-I'm-Claude)
- **Authorisation:** Does the RM have permission to read this customer's data? (Delegated identity, not service account)
- **Input validation:** Is the `customer_id` a valid CIF format? Does the `as_of_date` fall within allowed range? (Server-side, not prompt-side)
- **Rate limiting:** The Snowflake server enforces query rate limits per user. Claude cannot overwhelm the data warehouse regardless of how many tool calls it attempts.
- **Output sanitisation:** PII fields not relevant to the requesting context are redacted before the response reaches Claude.

The CISO-facing framing: "Claude is untrusted from the MCP server's perspective. Every call is validated as if it came from an external client."

---

## Staged rollout: MCP in 3 phases

### Phase 1 — Pilot (Weeks 1–6)
- 2 MCP servers: Salesforce (read-only only) + internal compliance API (read-only only).
- 1 Claude surface: RM copilot desktop.
- 5 RMs in pilot cohort.
- Goal: validate auth flow, measure latency, validate RBAC enforcement with synthetic test users.
- No write tools in Phase 1.

### Phase 2 — Controlled expansion (Weeks 7–16)
- 4 MCP servers: add ServiceNow (read) + Snowflake (named queries).
- Enable write tools on Salesforce with HITL gate. No auto-commit.
- Expand to 50 RMs. Add compliance review agent surface.
- Goal: validate HITL gate under real usage; measure approval latency; tune tool schema based on observed Claude tool-call failures.

### Phase 3 — Production (Week 17+)
- Full catalog (5 MCP servers, all tools).
- All 6 use cases active.
- Eval harness running continuously against production logs.
- CISO dashboard: tool call volume, approval rates, anomaly flags.

---

## Cross-references
- Predecessor: [08 · Note 02 — Agentic Loop Patterns](02-Agentic-Loop-Patterns.md)
- Sibling: [08 · Note 05 — HITL & Decision Rights](05-Human-in-the-Loop-and-Decision-Rights.md)
- Security: [09 · Note 01 — Prompt Injection](../../09-Security-Privacy-Governance/Notes/01-Prompt-Injection-Classes-and-Defenses.md)
- Governance: [09 · Note 04 — Audit Logging](../../09-Security-Privacy-Governance/Notes/04-Audit-Logging-and-Decision-Rights.md)
- SystemDesign: [08 · SystemDesign 01 — BFSI RM Copilot](../SystemDesigns/01-BFSI-RM-Copilot.md)

## Strong-Hire bar for this file
- MCP defined as a standardisation/reuse layer, not just "a way to give Claude tools."
- "When MCP is NOT right" delivered before the catalog — shows restraint.
- Snowflake named-query-only design choice explained on security grounds, not convenience.
- MCP server as trust boundary, not Claude, is the security framing — this is the CISO answer.
- Three-phase rollout pattern shows deployment discipline.

## The interview-room answer (60 sec)

> "MCP is the integration protocol that lets Claude access tools and resources without rebuilding the integration for every Claude surface. The killer use case is when three Claude-powered apps — RM copilot, compliance agent, exec assistant — all need the same Salesforce integration. One MCP server, three consumers, one auth and audit path. The critical security point: the MCP server is the trust boundary, not Claude. Claude is untrusted from the server's perspective — every call validates credentials, enforces RBAC, validates inputs server-side, and redacts PII before the response returns to the model. For BFSI I also make the Snowflake server expose named parameterised queries only, not free SQL — that closes the prompt injection attack surface at the data layer. On rollout: start with bespoke tool use in the POC, move to MCP in production when you have two or more surfaces sharing the same tools."
