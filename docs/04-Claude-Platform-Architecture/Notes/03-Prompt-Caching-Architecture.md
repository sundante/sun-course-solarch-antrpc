# 04 · Note 03 — Prompt Caching Architecture

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** Prompt caching as an architectural concern, not a "tip" — TTL, hit-rate optimization, cost impact, breakpoint placement, eval implications.

> **What this file is NOT.** A copy of Apic's caching docs. Not a benchmark.

---

## Why caching is architecture

Most engineers see prompt caching as "save tokens." Architects see it as the single largest lever to fit Claude into a customer's cost envelope without sacrificing capability. Treating caching as a tip leaves 50–80% of the savings on the table.

### The three architectural questions caching answers
1. *Where do you place cache breakpoints?* — wrong placement burns the budget.
2. *What hit-rate are you designing for?* — sets your cost model.
3. *How does cache invalidation interact with your eval regression gates?* — wrong answer means you ship a model change with cache pollution skewing the evals.

---

## Mechanics (the part you must know cold)

- Cache breakpoints sit at chosen positions in the prompt (system prompts, document contexts, tool definitions, prior conversation turns).
- Tokens before a breakpoint can be cached and reused on later calls.
- Cache hits are billed at a *fraction* of the input-token rate; cache writes carry a small premium.
- TTL is bounded — typically minutes — so cache strategy is sensitive to call cadence.
- Cache scope is per-account / per-organization, not cross-customer.

→ Verify exact pricing/TTL numbers against current Apic docs before quoting in interview.

---

## Where to place breakpoints

The single highest-leverage decision. The pattern that works in regulated enterprise:

```
[ system prompt ] ◄── breakpoint 1: large, stable content
[ tool definitions ]
[ retrieved context (RAG) ] ◄── breakpoint 2 if context is reused
[ conversation history ] ◄── breakpoint 3 in long-lived sessions
[ current user turn ]
```

### Rules of thumb
- **Stable content first, volatile content last.** Anything that changes per-request must come *after* the last breakpoint.
- **Big-and-static beats small-and-dynamic.** Cache the 50K-token policy bundle once; don't try to cache the 200-token user turn.
- **Tool definitions go in the cached zone** — they are large and rarely change.

---

## Hit-rate optimization

The cost model depends on hit-rate:

```
effective_input_cost = (cache_miss_cost × miss_rate)
                     + (cache_hit_cost × hit_rate)
                     + (cache_write_premium × first_call_rate)
```

For a 50K-token system prompt with 90% hit-rate, effective input cost can drop 5–8× vs naive uncached. For a 5K-token system prompt with 30% hit-rate, savings are marginal.

### Architectural moves to lift hit-rate
- **Co-locate calls in time.** Burst-then-quiet patterns kill hit-rate. Steady-state traffic is cheap.
- **Pin one user-session to one worker** when sticky-session is acceptable.
- **Pre-warm caches** for predictable burst traffic (market open, daily-batch start).
- **Refuse to put random IDs in the cached zone.** Even one varying byte breaks the cache.

---

## Eval implications (the bit most teams miss)

Caching can mask regressions if you don't design for it. Two specific traps:

### Trap 1 — Cache pollution across model versions
You cached against `claude-sonnet-4-6` last week. You're testing `claude-sonnet-4-7-preview` this week. Cache misses → cost spikes → you blame the new model when it's actually fine.

**Mitigation:** evals run against a *cleared cache state* (deterministic), not the production cache state.

### Trap 2 — Eval scores skew with cache hot/cold
The first call in an eval batch is cold; the next 99 are hot. Latency numbers from a cached run won't match production. **Mitigation:** report eval latency at both states (cold p99, hot p99).

→ See [Module 06 Note 04 — Online Monitoring & Regression Gates](../../06-Evaluation-Architecture/Notes/04-Online-Monitoring-and-Regression-Gates.md).

---

## BFSI customer caching pattern (preview)

For the running BFSI customer's six use cases:

| Use case | Cached zone | Why |
|---|---|---|
| Support agent assist | Persona + policy bundle + escalation rules | ~30K tokens, stable per shift |
| Internal SOP search | Tool defs + retrieval rerank prompt | Stable across all queries |
| RM copilot | Persona + customer-360 schema + tool defs | Stable per RM |
| Compliance review | Reg-framework reference + drafting style guide | Stable per regulator |
| Dev productivity | (handled by Claude Code defaults) | — |
| Exec analytics | Schema + safety guardrails | Stable per dashboard |

---

## The interview-room answer (60 sec)

> *"Caching is architecture, not a tip. Three decisions: where to place breakpoints — stable content first, volatile last, tool definitions inside the cached zone; what hit-rate you design for — that sets your cost model; and how cache state interacts with eval regression gates — evals run cleared-cache, latency reported at both cold and hot. For BFSI support agent assist with a 30K-token policy bundle and steady-state traffic, hit-rate of ~90% drops effective input cost roughly 5–8× vs naive uncached. The trap most teams miss is cache pollution across model versions — that's why evals must run with a deterministic cache state, not the prod state."*

---

## Cross-references
- Predecessor: [Note 01 — Claude API Surface](01-Claude-API-Surface.md).
- Sibling: [Note 02 — Model Selection Matrix](02-Model-Selection-Matrix.md) — caching shifts the matrix.
- CodeLab: [04 — Prompt Caching](../CodeLabs/04-Prompt-Caching.md).
- Drill: [02 — Whiteboard Prompt Cache ROI](../Drills/02-Whiteboard-Prompt-Cache-ROI.md).
- Module 10: [Cost Architecture](../../10-Cost-Latency-Reliability/Notes/01-Cost-Architecture.md).

## Strong-Hire bar for this file
- Breakpoint placement rules reflex.
- Cost-model formula at fingertips.
- Eval implications (cache pollution + cold/hot reporting) called out without prompting.
- BFSI per-use-case caching plan defended cold.
