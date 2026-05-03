# 04 · Note 02 — Model Selection Matrix

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The decision matrix for picking between Opus 4.7, Sonnet 4.6, and Haiku 4.5 — by use case, by cost shape, by latency budget, by reasoning depth. The 30-second answer for the customer call, with the depth behind it.

> **What this file is NOT.** A benchmark recital. Not a marketing-page summary.

---

## The current lineup (as of build)

| Model | Model ID | Best for | Cost shape | Latency shape |
|---|---|---|---|---|
| **Opus 4.7** | `claude-opus-4-7` | Hardest reasoning, agentic orchestration, hard-to-eval reasoning chains | Highest | Highest |
| **Sonnet 4.6** | `claude-sonnet-4-6` | Default workhorse — agent loops, RAG, structured drafting | Mid | Mid |
| **Haiku 4.5** | `claude-haiku-4-5-20251001` | High-volume classify/route/extract, judge models, low-latency UX | Lowest | Lowest |

→ Knowledge cutoff for Claude in this repo: per Apic docs at the time of writing.

---

## The decision matrix

Three axes — capability requirement × volume × latency budget. Pick the model where all three constraints are simultaneously satisfied.

### Axis 1 — Capability requirement
- **High-stakes reasoning, multi-step plans, hard-to-eval correctness:** Opus.
- **Mid-complexity reasoning, RAG over structured docs, agentic tool loops with bounded depth:** Sonnet.
- **Bounded classification, extraction with clear schema, fast triage:** Haiku.

### Axis 2 — Volume
- **<1K calls/day:** model choice is dominated by capability; cost is rounding error.
- **10K–100K/day:** model choice is dominated by per-call cost. Sonnet or Haiku unless capability requires Opus.
- **>1M/day:** model choice is constrained by *both* per-call cost and rate-limit headroom. Haiku for the bulk; Opus only on selected paths.

### Axis 3 — Latency budget
- **>30s tolerable:** any model.
- **5–30s:** any model with streaming.
- **<5s P95:** Sonnet or Haiku, with caching, with parallel tool calls.
- **<2s P95:** Haiku, with prompt caching, ideally with prefetch.

---

## The "tiered architecture" pattern

Most BFSI use cases benefit from a **tiered model architecture** — different requests routed to different models based on complexity. The pattern:

```
User request
    │
    ▼
Haiku — triage classifier
    │
    ├── Simple request → Haiku response (fast, cheap)
    ├── Standard request → Sonnet (RAG, drafting, agent loop)
    └── Hard request → Opus (multi-step reasoning, supervision)
```

The triage classifier is *itself* an eval-gated component — too aggressive on routing-up costs the customer money; too aggressive on routing-down costs accuracy.

---

## Six BFSI use cases × model selection (preview)

| Use case | Hot path | Triage / supervision |
|---|---|---|
| Support agent assist | Sonnet 4.6 | Haiku for triage classifier |
| Internal SOP search | Sonnet 4.6 | Haiku for query rewriter |
| RM copilot | Sonnet 4.6 | Opus for plan-and-execute occasionally |
| Compliance doc review | Opus 4.7 (hot path) | Sonnet for first-pass draft |
| Developer productivity (Claude Code) | Sonnet 4.6 | Opus for hardest tasks |
| Exec analytics | Sonnet 4.6 | Haiku for query routing |

→ Full SystemDesigns: [Module 05 SystemDesigns](../../05-Solution-Architecture-Patterns/SystemDesigns/).

---

## The 30-second answer (interview-room version)

> *"Three models, three slots. Opus for the hardest reasoning — orchestration, multi-step plans, hard-to-eval correctness. Sonnet is the workhorse — agent loops, RAG, drafting, the bulk of production traffic. Haiku is high-volume — classify, route, extract, and as the LLM-as-judge in eval frameworks. The pattern that compounds is tiered architecture: Haiku triages, Sonnet handles standard, Opus handles hard. For the BFSI support-agent-assist case I worked on, that pattern dropped per-decision cost ~70% vs naive-Opus routing."*

---

## Anti-patterns

### Anti-pattern 1 — "Always use Opus to be safe"
Reads as not having done the math. Lean No. Opus on bulk traffic is a budget hole, and capability-wise often unnecessary.

### Anti-pattern 2 — "Always use Haiku for cost"
Strong No on use cases where reasoning depth matters. Compliance review on Haiku is a deployment failure waiting to happen.

### Anti-pattern 3 — One-model architectures
Every request routed to the same model. Wastes capability on simple requests and starves on hard ones. Lean No.

### Anti-pattern 4 — Selection without evals
"We picked Sonnet." Why? "It seemed about right." Lean No. Selection without an eval scorecard is opinion, not architecture.

### Anti-pattern 5 — Locking selection in production code
Hard-coded model IDs across the codebase. Strong No. Selection is a config decision; the eval framework should make substitution a change-config-and-rerun-evals operation.

---

## Cross-references
- Sibling: [Note 03 — Prompt Caching](03-Prompt-Caching-Architecture.md) — caching changes the cost shape and shifts the matrix.
- Sibling: [Note 05 — Deployment Surfaces](05-Deployment-Surfaces.md) — model availability per surface differs.
- Module 06: [Eval Taxonomy](../../06-Evaluation-Architecture/Notes/02-Eval-Taxonomy.md) — selection without evals is opinion.
- Drill: [01 — 30-Second Model Selection](../Drills/01-30-Second-Model-Selection.md).

## Strong-Hire bar for this file
- Three-model matrix internalized; can be spoken in 30 sec.
- Tiered-architecture pattern is reflex.
- Selection is always paired with eval framework — never opinion.
- The six BFSI use case selections defended cold.
