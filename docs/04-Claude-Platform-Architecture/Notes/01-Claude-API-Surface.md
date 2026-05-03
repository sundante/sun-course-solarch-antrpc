# 04 · Note 01 — Claude API Surface

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The architect's working knowledge of the Claude API surface — endpoints, SDK shape, request/response model, streaming, system prompts, message format. The depth needed to whiteboard a Claude integration cold.

> **What this file is NOT.** A copy of Apic's public API reference. Not an SDK tutorial. Not exhaustive — calibrated to what you'll actually be asked.

---

## What the architect must know cold

- The shape of a request (system + messages + parameters).
- The shape of a response (content blocks, stop reason, usage).
- Streaming events and when streaming is the right default.
- Where prompt caching, tool use, and structured outputs sit relative to the basic surface.
- Rate limits and how they map to capacity planning.
- The official SDKs (Python, TypeScript) and when to call the REST endpoint directly.

---

## Anatomy of a request

### Top-level fields
- `model` — the canonical model ID. Current: `claude-opus-4-7`, `claude-sonnet-4-6`, `claude-haiku-4-5-20251001`.
- `max_tokens` — hard ceiling on output. Architectural decision, not a default.
- `messages` — list of role-tagged turns: user / assistant.
- `system` — system prompt(s); supports cache breakpoints.
- `temperature` — typically 0–1; load-bearing for evals (set explicitly, don't rely on defaults).
- `tools` / `tool_choice` — for tool use. → [Note 04](04-Tool-Use-and-Structured-Outputs.md).
- `stream` — boolean; streaming changes the response shape (events, not a single body).

### Message format
- Each message has `role` and `content`.
- `content` can be a string or a list of blocks (text, image, tool_use, tool_result).
- Tool-use loops carry tool_use blocks from assistant + tool_result blocks back from user.

### System prompt as architecture, not afterthought
- The system prompt is *not* a tip-jar for instructions.
- It carries: persona, behavior boundaries, required output shape, tool-use guidance, the eval-defining constraints.
- For caching, the system prompt is typically the *first* cache breakpoint.

---

## Anatomy of a response

### Top-level fields
- `id` — the unique response id.
- `model` — echoed back; useful for logging.
- `content` — list of content blocks. Most common shapes: a single text block; or text + tool_use blocks.
- `stop_reason` — `end_turn`, `max_tokens`, `tool_use`, `stop_sequence`. Each maps to different downstream handling.
- `usage` — `input_tokens`, `output_tokens`, plus cache-related counters when caching is on.

### Stop-reason handling
- `end_turn` — normal.
- `max_tokens` — ceiling hit; output may be truncated. Architectural concern (don't silently treat as success).
- `tool_use` — handle the tool-use loop. → [Note 04](04-Tool-Use-and-Structured-Outputs.md).
- `stop_sequence` — your custom stop hit.

### Why this matters for evals
The eval framework needs to score responses *partitioned by stop_reason*. A `max_tokens` truncation is a different failure mode than a hallucinated answer.

---

## Streaming

### Event types
- `message_start` — the message envelope arrives.
- `content_block_start` / `content_block_delta` / `content_block_stop` — for each content block as it streams.
- `message_delta` — incremental updates to top-level fields.
- `message_stop` — the response is done.

### When to stream
- User-facing chat-shaped UX. Default on.
- Customer-support agent assist (latency-perceived UX dominates actual latency).

### When *not* to stream
- Programmatic consumers downstream of the API call (no UX to perceive).
- Tool-use loops where the next step depends on the full response — streaming buys nothing.
- Eval harnesses (deterministic scoring, batch shapes).

---

## SDK shape (Python, TypeScript)

### Python — `apic` package
- `client = Apic()`; `client.messages.create(...)`.
- Streaming: `client.messages.stream(...)` returns an event iterator.
- Tool use: tool definitions are dicts with name + description + input_schema (JSON Schema).
- Async client available (`AsyncApic`) — non-trivial for high-concurrency workloads.

### TypeScript — `@apic-ai/sdk`
- Mirrors Python shape. Important if the customer's stack is JS/TS-first.

### When to call REST directly
- Edge functions where the SDK weight matters.
- Custom infrastructure that doesn't fit the SDK's retry/timeout assumptions.
- Almost never in BFSI — SDK is the right default.

---

## Rate limits and capacity planning

- Limits are tier-based: per-minute requests + per-minute input tokens + per-minute output tokens.
- For Indian BFSI customers, capacity planning starts from business volume (tickets/day, documents/day) and translates *backwards* through token estimates per call. → [Module 10 Drill 01](../../10-Cost-Latency-Reliability/Drills/01-Estimate-Tokens-from-Business-Volume.md).
- Rate-limit hits should be treated as architectural events (queueing, fallback to smaller model), not as exceptions to retry-and-hope.

---

## What is *not* in this note (intentional split)

- Model selection between Opus/Sonnet/Haiku → [Note 02](02-Model-Selection-Matrix.md).
- Prompt caching mechanics → [Note 03](03-Prompt-Caching-Architecture.md).
- Tool use + structured outputs → [Note 04](04-Tool-Use-and-Structured-Outputs.md).
- Bedrock vs Vertex vs first-party API → [Note 05](05-Deployment-Surfaces.md).

---

## Cross-references
- Predecessors: [Module 03](../../03-Customer-Discovery-and-Pre-Sales/INDEX.md).
- Successors: this is the entry note for Module 04. Notes 02–05 build on it.
- CodeLab: [01 — First Claude Call](../CodeLabs/01-First-Claude-Call.md).
- Drill: [02 — Whiteboard Prompt Cache ROI](../Drills/02-Whiteboard-Prompt-Cache-ROI.md).

## Strong-Hire bar for this file
- Whiteboard the request/response cold without notes.
- Stop-reason handling discipline articulated, not implicit.
- Streaming-vs-non-streaming decision rule reflex.
- Capacity planning grounded in business volume, not token theatrics.
