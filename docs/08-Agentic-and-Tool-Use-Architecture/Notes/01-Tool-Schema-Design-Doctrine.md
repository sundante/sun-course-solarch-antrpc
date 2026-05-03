# 08 · Note 01 — Tool Schema Design Doctrine

> **Status: Outline.** Body fills in Week 4. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** How to design tool schemas that make Claude reliable in agentic workflows — the decisions that determine whether the agent calls the right tool with the right args every time.

> **What this file is NOT.** A JSON Schema tutorial. Not Module 04's primitives overview ([see 04 · Note 04](../../04-Claude-Platform-Architecture/Notes/04-Tool-Use-and-Structured-Outputs.md)).

---

## The tool schema as a contract (not just a JSON blob)

A tool schema is not documentation. It is a behavioral contract between the architect and the model. Claude reads the tool name, description, and parameter descriptions at inference time and decides: (a) whether to call this tool, (b) which arguments to pass. If any part of that contract is ambiguous, Claude will fill the gap with inference — and inference introduces variance.

The principal-level reframe: **schema quality is agent reliability**. The engineer ships JSON. The architect ships behavior.

Three things Claude reads from a tool schema, in order of weight:
1. **Tool name** — the primary signal for tool selection. Claude pattern-matches the intent of the user utterance against tool names.
2. **Tool description** — the *when to use this tool* contract. If two tools could both plausibly answer the intent, the description resolves the ambiguity.
3. **Parameter descriptions** — the *what value belongs here* contract. Missing descriptions cause Claude to guess. Guessed values in BFSI are regulatory events.

The design principle: **write schemas as if the model has never seen your codebase**. Because it hasn't.

---

## Schema design principles

### Principle 1 — Name for intent, not implementation

Bad: `crm_db_read`, `sf_object_fetch`, `api_call_customer`
Good: `get_customer_360`, `get_account_balance`, `list_open_service_requests`

The name should answer: *"what does calling this tool accomplish?"* not *"which system does this hit?"*.

### Principle 2 — One tool, one job

A tool with a `mode` or `action` parameter is a god-tool in disguise. Claude will hallucinate the mode value or pick the wrong action. Split it.

Bad: `customer_data_tool(mode: "balance" | "portfolio" | "kyc" | "history")`
Good: four distinct tools, each named and described for its one job.

### Principle 3 — Every parameter has a description

Parameter names alone are not enough. `as_of_date` is better than `date`, but the description should say: *"ISO 8601 date for the portfolio snapshot. Defaults to today. Pass the date the customer mentioned if they referenced a historical period."*

Without that last sentence, Claude will either omit the parameter or pass today's date on a historical query.

### Principle 4 — Make enums explicit

If a parameter accepts a finite set of values, enumerate them. Claude will not hallucinate values that aren't in the enum. It will hallucinate values it has to invent.

### Principle 5 — Flag mutating tools explicitly

Read tools and write tools look identical to Claude at schema-read time. If a tool mutates state, the description must say so: *"Writes an interaction log to the CRM. This action is not reversible."* This gives Claude (and the human reviewer) the signal to treat it differently.

### Principle 6 — Required vs optional is a design decision

Everything marked optional will sometimes be omitted. If omitting it produces a wrong answer, mark it required. If the default is safe and sensible, optional is fine.

---

## The BFSI RM copilot tool catalog (8 tools)

The RM copilot for the running BFSI case operates with the following tool set. These are design contracts, not implementation specs.

### Tool 1: `get_customer_360`
```json
{
  "name": "get_customer_360",
  "description": "Retrieve the full customer profile for a relationship manager's client, including demographics, relationship history, product holdings, and recent interaction summary. Use at the start of any session where the RM is preparing for a customer interaction.",
  "input_schema": {
    "type": "object",
    "properties": {
      "customer_id": {
        "type": "string",
        "description": "Unique CRM customer identifier. Format: CIF-XXXXXXXX."
      },
      "include_sections": {
        "type": "array",
        "items": {"type": "string", "enum": ["demographics", "products", "interactions", "risk_profile", "compliance_flags"]},
        "description": "Sections to include. Omit to return all sections. Pass specific sections to reduce latency."
      }
    },
    "required": ["customer_id"]
  }
}
```

**Why this shape:** `include_sections` is optional to allow a lean call for quick prep and a full call for deep review. The enum prevents Claude from inventing section names.

### Tool 2: `get_portfolio_snapshot`
```json
{
  "name": "get_portfolio_snapshot",
  "description": "Retrieve the customer's current investment and deposit portfolio from the wealth management platform. Returns holdings, allocation percentages, and current market value. Read-only. Does not trigger any trade or recommendation.",
  "input_schema": {
    "type": "object",
    "properties": {
      "customer_id": {"type": "string", "description": "CIF identifier."},
      "as_of_date": {
        "type": "string",
        "description": "ISO 8601 date (YYYY-MM-DD). Pass today's date for current snapshot. Pass a historical date if the customer asked about a past period."
      }
    },
    "required": ["customer_id", "as_of_date"]
  }
}
```

**Why `as_of_date` is required:** Omitting it forces a default that may not match the customer's question. Making it required forces Claude to surface the date decision to the RM.

### Tool 3: `search_sop_knowledge_base`
```json
{
  "name": "search_sop_knowledge_base",
  "description": "Search the internal SOP and policy knowledge base for procedures relevant to a customer situation or product. Use when the RM needs to verify procedure before advising. Returns top-k relevant passages with document ID and section reference.",
  "input_schema": {
    "type": "object",
    "properties": {
      "query": {"type": "string", "description": "Natural language description of the procedure or policy you need."},
      "top_k": {"type": "integer", "description": "Number of passages to return (1–10). Default 5.", "minimum": 1, "maximum": 10}
    },
    "required": ["query"]
  }
}
```

### Tool 4: `log_crm_interaction`
```json
{
  "name": "log_crm_interaction",
  "description": "Write a structured interaction note to the Salesforce CRM for the specified customer. THIS TOOL WRITES TO PRODUCTION CRM. The note will be visible to all RMs and compliance teams. Do not call without RM approval of the note content.",
  "input_schema": {
    "type": "object",
    "properties": {
      "customer_id": {"type": "string", "description": "CIF identifier."},
      "interaction_type": {
        "type": "string",
        "enum": ["call", "meeting", "email", "portal_message"],
        "description": "Channel of the interaction."
      },
      "summary": {"type": "string", "description": "Plain-text summary of the interaction (100–500 words). Will be stored verbatim."},
      "action_items": {
        "type": "array",
        "items": {"type": "string"},
        "description": "List of follow-up actions the RM has committed to. Each item should start with a verb."
      },
      "approved_by_rm": {
        "type": "boolean",
        "description": "Must be true. Set to true only after the RM has reviewed and approved the note content."
      }
    },
    "required": ["customer_id", "interaction_type", "summary", "approved_by_rm"]
  }
}
```

**Security design:** `approved_by_rm: true` is a schema-level HITL gate. The application enforces it; the schema names it so Claude will not silently log without surfacing the approval requirement.

### Tool 5: `draft_proposal`
```json
{
  "name": "draft_proposal",
  "description": "Generate a structured product proposal document for the customer based on their profile and the RM's stated objective. Returns a draft — does not submit or share it. RM must review before any submission.",
  "input_schema": {
    "type": "object",
    "properties": {
      "customer_id": {"type": "string"},
      "objective": {"type": "string", "description": "What the RM is trying to achieve for this customer (e.g. 'retirement corpus, 20-year horizon, moderate risk')."},
      "products_to_include": {
        "type": "array",
        "items": {"type": "string"},
        "description": "List of product codes to include in the proposal. Use get_eligible_products first if unsure."
      }
    },
    "required": ["customer_id", "objective"]
  }
}
```

### Tool 6: `get_eligible_products`
```json
{
  "name": "get_eligible_products",
  "description": "Return the list of products the customer is currently eligible for, given their KYC, risk profile, and regulatory classification. Required before any product recommendation. Returns product codes, names, and eligibility rationale.",
  "input_schema": {
    "type": "object",
    "properties": {
      "customer_id": {"type": "string"},
      "product_category": {
        "type": "string",
        "enum": ["mutual_funds", "fixed_deposits", "insurance", "structured_products", "equities", "all"],
        "description": "Category to filter. Pass 'all' for the full eligible set."
      }
    },
    "required": ["customer_id", "product_category"]
  }
}
```

### Tool 7: `check_compliance_flags`
```json
{
  "name": "check_compliance_flags",
  "description": "Check whether the customer has any active compliance flags, AML alerts, KYC expiry notices, or regulatory holds that the RM must be aware of before any transaction or recommendation. Call before draft_proposal or log_crm_interaction.",
  "input_schema": {
    "type": "object",
    "properties": {
      "customer_id": {"type": "string"},
      "flag_types": {
        "type": "array",
        "items": {"type": "string", "enum": ["aml", "kyc_expiry", "pep", "regulatory_hold", "dormancy"]},
        "description": "Flag categories to check. Omit to check all."
      }
    },
    "required": ["customer_id"]
  }
}
```

### Tool 8: `create_service_request`
```json
{
  "name": "create_service_request",
  "description": "Open a ServiceNow service request on behalf of the customer for a non-advisory action (e.g. address change, statement request, dispute). THIS TOOL CREATES A LIVE TICKET. Do not call without confirming the request details with the RM.",
  "input_schema": {
    "type": "object",
    "properties": {
      "customer_id": {"type": "string"},
      "request_type": {
        "type": "string",
        "enum": ["address_change", "statement_request", "cheque_book", "dispute_initiation", "kyc_renewal", "account_closure_request"],
        "description": "Type of service request. Do not pass free text — pick the closest enum value."
      },
      "details": {"type": "string", "description": "Additional context the service team needs to fulfill the request."},
      "urgency": {"type": "string", "enum": ["normal", "urgent"], "description": "Urgency level. Default normal. Set to urgent only if customer explicitly states time sensitivity."}
    },
    "required": ["customer_id", "request_type", "details", "urgency"]
  }
}
```

---

## What makes a schema fail

### Failure mode 1: Ambiguous tool names
Two tools named `get_customer_data` and `fetch_customer_info`. Claude cannot distinguish them by name. It will pick one semi-randomly. Result: wrong data surface, potential RBAC violation.

### Failure mode 2: Missing descriptions on discriminating parameters
`account_type` with no description and no enum. Claude passes `"savings"` when the system expects `"SB"`. Tool call fails. Claude retries or hallucinates a recovery.

### Failure mode 3: Over-broad parameters
`query: string` on a SQL-executing tool with no constraint. Claude writes SQL. That is a prompt injection waiting to happen — a Salesforce note containing `'; DROP TABLE interactions; --` now executes via Claude.

### Failure mode 4: Conflated read/write in one tool
A tool named `manage_customer_data` that both reads and writes depending on a `mode` parameter. Claude calls it in a read context and accidentally passes `mode: "update"`.

### Failure mode 5: Missing HITL gate naming
A tool that writes to CRM with no language in the description indicating it is a write. The RM expects a draft; the CRM now has an unreviewed note. Audit flag.

---

## The CISO angle: tool schema as the attack surface

From the CISO's perspective, every tool is an attack surface. The schema is the attack surface specification.

The CISO's question for every tool:
1. **What is the worst thing Claude could do if this tool is called with attacker-controlled arguments?** If the answer is "write to production CRM / execute a transaction / open a service request," that tool needs a HITL gate in the schema *and* in the application.
2. **Can an indirect prompt injection in a retrieved document cause this tool to fire?** Yes, unless the architecture separates the retrieval trust layer from the tool-call trust layer. ([See 09 · Note 01](../../09-Security-Privacy-Governance/Notes/01-Prompt-Injection-Classes-and-Defenses.md))
3. **What is the blast radius of a hallucinated argument?** Narrow enums + required fields + server-side validation = blast radius constrained to the enum set.

The principal-level posture: **design the schema for the failure, not the happy path**.

---

## Cross-references
- Predecessor: [04 · Note 04 — Tool Use and Structured Outputs](../../04-Claude-Platform-Architecture/Notes/04-Tool-Use-and-Structured-Outputs.md)
- Sibling: [08 · Note 02 — Agentic Loop Patterns](02-Agentic-Loop-Patterns.md)
- Security: [09 · Note 01 — Prompt Injection Classes](../../09-Security-Privacy-Governance/Notes/01-Prompt-Injection-Classes-and-Defenses.md)
- Case Study: [08 · Case Study 01 — BFSI Agentic Workflow Walkthrough](../Case-Study/01-BFSI-Agentic-Workflow-Walkthrough.md)
- SystemDesign: [08 · SystemDesign 01 — BFSI RM Copilot](../SystemDesigns/01-BFSI-RM-Copilot.md)

## Strong-Hire bar for this file
- Schema-as-contract framing articulated without prompting — not "it's just JSON."
- Six schema design principles delivered as a reflex, not a list to be read.
- All 8 BFSI tools understood well enough to explain the *design choices* (why `approved_by_rm` is a required boolean, why `as_of_date` is required, why enums over free-text).
- CISO attack-surface framing for tool schemas (blast radius language, indirect injection risk).
- Failure modes named cold.

## The interview-room answer (60 sec)

> "Tool schemas are behavioral contracts, not documentation. Claude reads the name, description, and parameter descriptions at inference time to decide whether and how to call the tool. So schema quality directly determines agent reliability. The principles I follow: name for intent not implementation, one tool one job, every parameter has a description, enums on finite value sets, and write tools must declare themselves as write tools. In BFSI that last one is critical — the RM copilot's `log_crm_interaction` tool has `approved_by_rm: boolean, required: true` in the schema itself. Not just in the application — in the schema, so the model sees it. From the CISO angle, every tool is an attack surface. The schema definition determines blast radius. Narrow enums plus required fields plus server-side validation keeps the blast radius bounded even if an injected prompt tries to abuse the tool."
