# 04 · Note 04 — Tool Use and Structured Outputs

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** When to give Claude tools, when to constrain output via structured outputs / JSON mode, and how the two patterns differ in production. The tool-use loop, parallel calls, schema discipline, error handling.

> **What this file is NOT.** Module 08's full agentic-architecture treatment ([see there for ReAct/plan-execute/etc](../../08-Agentic-and-Tool-Use-Architecture/INDEX.md)). Not a copy of the SDK docs.

---

## Two distinct primitives

### Tool use
- Claude *decides* whether to call a tool, which tool, with what arguments.
- Architect provides a *menu* of tools (JSON Schema for inputs).
- Multi-turn loop: Claude → tool_use block → application runs tool → tool_result block → Claude → ... → final answer.
- Claude can run multiple tools in parallel when independent.

### Structured outputs
- Architect *constrains* the model's output to a fixed shape (JSON, schema-validated).
- Single-turn: send prompt + schema, get parseable response.
- No tool calling; pure shape enforcement.

### When each fits
- **Tool use:** the answer requires *acting* (querying a system, calculating something deterministically, fetching data).
- **Structured outputs:** the answer is shape-constrained (extract these 12 fields from this document, classify into one of these categories with reasoning).

---

## Tool use — the architect's bar

### Tool schema design (the part that matters most)
Tool schemas are *behavioral contracts*. Bad schemas produce confused agents that loop or hallucinate args.

- **Name the tool by what it does, not what it operates on.** `get_customer_balance` > `customer_db_read`.
- **One tool, one job.** Don't pack four behaviors into a `do_customer_thing` god-tool.
- **Argument names should be self-evident.** `account_id` > `id`; `as_of_date` > `date`.
- **Descriptions are part of the contract.** The architect writes these, not the engineer.
- **Idempotency matters.** Tools that mutate state should be named, schema'd, and used as if they could be called twice.

→ Module 08: [Tool Schema Design Doctrine](../../08-Agentic-and-Tool-Use-Architecture/Notes/01-Tool-Schema-Design-Doctrine.md).

### The tool-use loop
```
Claude (assistant): "I'll call tool X with args Y" → tool_use block
Application: runs tool, produces result
Application (user): tool_result block, content = output
Claude (assistant): integrates result, either answers or calls another tool
[loop until stop_reason = end_turn]
```

### Parallel tool calls
When tools are independent, Claude can return multiple tool_use blocks in one response. Architect's job: design tools to be parallel-safe (no implicit ordering, no shared mutable state across tools in one batch).

### Tool-use error handling
- Tool errors become tool_result blocks with `is_error: true` (or app convention).
- Don't silently retry — let Claude see the error and decide.
- Don't paper over errors with fake successful results — Claude will commit to a wrong answer.

---

## Structured outputs — the architect's bar

### When to reach for it
- Output shape is fixed and downstream systems will parse it.
- Schema is non-trivial (10+ fields, nested objects).
- You want validation discipline (Pydantic in Python, Zod in TS).

### When *not* to reach for it
- A small, free-text answer with two numbers — the JSON wrapper costs more than it saves.
- The output structure depends on the input (e.g. extract whichever fields the document has). Tool use is closer to right.

### Schema design
- Use the most specific types possible (enums, regex-validated strings, ranges).
- Validate Claude's output against the schema downstream — never trust the parse.
- Errors in validation should re-prompt Claude with the validation error, not silent-retry.

---

## When tool use beats RAG (and vice-versa)

A common architecture choice. Pattern:

- **Use RAG** when: the relevant content lives in a vector store and the answer is "summarize this" or "answer the question grounded in these documents."
- **Use tool use** when: the relevant content lives in *systems* (databases, ticketing tools, calendars) and the answer requires looking it up live.
- **Use both** when: the answer requires both retrieval *and* live action — most BFSI agentic use cases.

---

## Anti-patterns

### Anti-pattern 1 — "Just prompt-engineer it to output JSON"
Without structured-output enforcement, this works *most* of the time and fails 1% of the time. In BFSI 1% is a regulatory issue.

### Anti-pattern 2 — God-tool
One tool with 14 arguments and a `mode` parameter. Claude will misuse it.

### Anti-pattern 3 — Tools without idempotency thinking
Mutation tools that fire twice on retry corrupt customer state. Architect's responsibility, not engineering's.

### Anti-pattern 4 — Silent fallback on tool error
Tool fails → app fakes success → Claude commits to a hallucinated path. Strong No.

### Anti-pattern 5 — Structured output for free-text answers
Wrapping a 2-sentence answer in `{"reasoning": ..., "answer": ...}` adds latency and cost without structure benefit.

---

## Cross-references
- Predecessor: [Note 01 — Claude API Surface](01-Claude-API-Surface.md).
- Sibling: [Note 02 — Model Selection Matrix](02-Model-Selection-Matrix.md).
- Module 08: full agentic treatment.
- CodeLabs: [02 — Tool Use](../CodeLabs/02-Tool-Use.md), [03 — Structured Output](../CodeLabs/03-Structured-Output.md).

## Strong-Hire bar for this file
- Tool use vs structured outputs as a *primitive distinction*, not a phrasing variation.
- Tool schema rules (name, single-job, idempotency) reflex.
- Parallel-tool-call design discipline articulable cold.
- The "RAG vs tool use vs both" decision rule clean.
