# 04 · CodeLab 02 — Tool Use

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, hands-on, BFSI-themed.

> **What this lab is.** A working tool-use loop: define a tool schema, handle the tool_use → tool_result loop, run multiple tools in parallel, surface the eval-relevant fields. The second commit of the GitHub artifact.

> **What this lab is NOT.** A LangChain/LangGraph wrapper. Not a multi-agent crew.

---

## Goal

By end of lab:

- A Python module that:
    - Defines 2–3 tools with proper JSON Schema.
    - Implements the tool_use loop end-to-end.
    - Handles parallel tool calls correctly.
    - Records tool_use blocks + tool_result blocks for eval consumption.
    - Demonstrates an idempotent vs non-idempotent tool side by side.

---

## Prerequisites

- CodeLab 01 complete.
- Familiarity with the response shape (content blocks, stop_reason).

---

## Step-by-step outline

### Step 1 — Define the tool schema for a BFSI customer-balance lookup
- Tool name: `get_customer_balance`.
- Input schema: `{account_id: str (regex: account-format), as_of_date: str (ISO date)}`.
- Description: precise, behavioral. *"Returns the available account balance as of the given date. Read-only. Does not initiate any transactions."*

### Step 2 — Define a second tool: a deterministic FX rate lookup
- Tool name: `get_fx_rate`.
- Schema: `{base: str (3-letter code), quote: str (3-letter code), as_of_date: str}`.
- Description: read-only, deterministic.

### Step 3 — Define a third tool: a non-idempotent operation (for the contrast)
- Tool name: `submit_alert_to_compliance` (mutation).
- Schema includes a `client_request_id` field for idempotency-key passing.
- Description explicitly notes it's mutation and idempotent-by-key.

### Step 4 — Implement the tool_use loop
- Start a conversation with `tools=[...]`.
- On `stop_reason == "tool_use"`, parse `tool_use` blocks.
- Run the tools (mock implementations are fine for the lab).
- Construct `tool_result` blocks with the outputs.
- Continue the conversation; loop until `stop_reason == "end_turn"`.

### Step 5 — Parallel tool calls
- Test a prompt that invites two independent tools (e.g. "what's customer X's USD balance in INR as of today?").
- Verify Claude returns multiple `tool_use` blocks in one response.
- Run them in parallel (asyncio.gather or threadpool).
- Return both `tool_result` blocks in one user-message turn.

### Step 6 — Error handling
- One tool deliberately raises (invalid `account_id`).
- Return the error as a `tool_result` with `is_error: true` and a clear message.
- Verify Claude reasons over the error rather than papering over it.

### Step 7 — Logging hooks
- Each tool call (request + response) goes to `logs/tool_calls.jsonl`.
- Used in CodeLab 05 to score tool-call correctness.

---

## Code skeleton (placeholder — fill in Week 2)

```python
TOOLS = [
    {
        "name": "get_customer_balance",
        "description": "Returns the available account balance as of the given date. Read-only.",
        "input_schema": {
            "type": "object",
            "properties": {
                "account_id": {"type": "string", "pattern": "^acct-[0-9]{8}$"},
                "as_of_date": {"type": "string", "format": "date"},
            },
            "required": ["account_id", "as_of_date"],
        },
    },
    # ... fx_rate, submit_alert_to_compliance
]

def run_tool_use_loop(initial_prompt: str):
    messages = [{"role": "user", "content": initial_prompt}]
    while True:
        resp = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=2048,
            tools=TOOLS,
            messages=messages,
        )
        messages.append({"role": "assistant", "content": resp.content})
        if resp.stop_reason == "end_turn":
            return resp
        tool_results = handle_tool_uses(resp.content)
        messages.append({"role": "user", "content": tool_results})
```

---

## Acceptance criteria

- [ ] Three tools defined with clean schemas.
- [ ] Tool-use loop runs end-to-end.
- [ ] Parallel tool calls demonstrated and verified.
- [ ] Error case demonstrated; Claude reasons over the error.
- [ ] Idempotency-key pattern visible on the mutation tool.
- [ ] Structured logging captures tool_use + tool_result pairs.

---

## Stretch goals

- Wrap the tool loop as an `mcp` server (preview of CodeLab integration with MCP).
- Add a "tool budget" — fail the conversation if Claude calls >N tools.

---

## Cross-references
- Note: [04 — Tool Use and Structured Outputs](../Notes/04-Tool-Use-and-Structured-Outputs.md).
- Module 08: [Tool Schema Design Doctrine](../../08-Agentic-and-Tool-Use-Architecture/Notes/01-Tool-Schema-Design-Doctrine.md).
- Sibling: [03 — Structured Output](03-Structured-Output.md).
- Sibling: [05 — Custom Eval Harness](05-Custom-Eval-Harness.md).

## Strong-Hire bar for this lab
- Tool schemas are *behavioral contracts*, not field lists.
- Parallel tool calls work first try; not added as an afterthought.
- Error path doesn't paper over failure.
- Mutation tool wears its idempotency key on its sleeve.
