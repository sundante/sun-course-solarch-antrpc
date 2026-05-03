# 04 · Drill 01 — 30-Second Model Selection

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this drill is.** Cold delivery of the Claude lineup decision matrix in 30 seconds — pick the right model for a given use case description, defended on capability + cost + latency. Ten use-case cards, randomized.

> **What this drill is NOT.** A capability comparison drill. Not benchmark recital.

---

## Prompt

> *Partner picks one of the 10 use-case cards (below) and reads it aloud. You have 30 seconds to: (a) name the model, (b) name the tier (Haiku / Sonnet / Opus), (c) name the *one* trade-off you accepted.*

---

## Time box

- **Per card:** 30 sec.
- **Drill:** all 10 cards = ~6 min.
- **Hard cap:** 35 sec — beyond is Lean No on this drill (the matrix isn't reflex).

---

## The 10 use-case cards

1. *"Triage classifier in front of a support agent assist — 200K calls/day, sub-200ms latency budget, 5-class output."*
2. *"Compliance document review for a public-sector regulator submission. Failure = retract submission."*
3. *"RM copilot at a wealth-management desk — multi-turn agentic, customer 360, occasional plan-and-execute."*
4. *"Internal SOP search RAG over a 50K-document corpus. ~10K queries/day."*
5. *"LLM-as-judge for an eval harness scoring faithfulness on 5K candidate responses per run."*
6. *"Exec analytics natural-language interface over Snowflake — 100 queries/day from C-suite users."*
7. *"Developer productivity copilot inside an internal IDE plugin — streaming, sub-3s latency budget."*
8. *"Customer-facing chatbot on a public site for product Q&A — 50K calls/day."*
9. *"Multi-document compliance gap analysis across 200-page regulatory bundles. Run weekly, ~100 runs."*
10. *"Tool-use planner that decides which of 12 tools to call on a Salesforce-integrated workflow."*

---

## Strong Hire answers (placeholder — fill in Week 2)

| Card | Answer | Trade-off accepted |
|---|---|---|
| 1 | Haiku 4.5, hot path | Capability ceiling — escalate complex cases to Sonnet |
| 2 | Opus 4.7 with eval gates | Cost — but the failure mode justifies it |
| 3 | Sonnet 4.6 default, Opus on hard | Two-model stack adds eval complexity |
| 4 | Sonnet 4.6 with caching | Cache pollution risk on doc updates — design for it |
| 5 | Haiku 4.5 as judge | Judge calibration must be tighter on a smaller model |
| 6 | Sonnet 4.6 + Haiku for query routing | Latency on the routing hop |
| 7 | Sonnet 4.6 streaming | Cost vs Haiku — but capability wins for code |
| 8 | Sonnet 4.6 with caching | Per-call cost; volume justifies caching investment |
| 9 | Opus 4.7 batch | Latency — but it's a batch run |
| 10 | Sonnet 4.6 for the planner, called tools per their needs | Planner-vs-executor split adds an eval layer |

---

## Rubric

### Strong Hire
- Model named in <10 sec.
- One specific trade-off named (not "it's a balance").
- Tier-architecture pattern surfaces when the use case justifies it.
- No "always Opus" or "always Haiku" defaults.

### Hire
- Model named, but trade-off is generic.
- Two-model stack mentioned only when prompted.

### Lean No
- "It depends" without the deciding axis named.
- Model picked without naming a trade-off.

### Strong No
- Wrong model ID (e.g. "claude-3-opus" — old).
- Single-model hammer for every card.

---

## Common Lean No traps

### Trap 1 — Always picking Opus
Conservative, but reads as "I haven't done the math."

### Trap 2 — Marketing-speak
"Sonnet is a great workhorse" — without the *why* (capability fit, cost shape, latency).

### Trap 3 — Skipping the trade-off
The rubric requires a trade-off named. Skipping it = Lean No.

### Trap 4 — Wrong/stale model IDs
Using `claude-3-opus`, `claude-2`, etc. Strong No.

### Trap 5 — Ignoring the latency or volume signal
Card 1 says "sub-200ms latency, 200K calls/day." Picking Sonnet ignores both signals.

---

## How to run this drill

1. Partner shuffles cards. You get them cold.
2. Timer starts when card is read.
3. 30 sec to answer. You get scored on model + trade-off.
4. Log to Drill Tracker.
5. Application trigger: 9 of 10 cards at Strong Hire across 2 sessions.

---

## Cross-references
- Note: [02 — Model Selection Matrix](../Notes/02-Model-Selection-Matrix.md).
- Note: [03 — Prompt Caching](../Notes/03-Prompt-Caching-Architecture.md).
- Module 06: [Eval Taxonomy](../../06-Evaluation-Architecture/Notes/02-Eval-Taxonomy.md).
- [Drill Tracker](../../Drill-Tracker.md).

## Strong-Hire bar for this drill
- 9/10 cards delivered in <30 sec.
- Trade-off named on every card.
- No stale model IDs.
- Tier-architecture pattern reflex when justified.
