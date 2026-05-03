# 04 · Drill 02 — Whiteboard Prompt-Cache ROI

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this drill is.** Stand at a whiteboard, given a use case + traffic shape, build the prompt-cache cost model live and defend the ROI. The drill that proves caching is architecture, not a tip.

> **What this drill is NOT.** A spreadsheet-driven exercise. Live whiteboard math, with rounded numbers.

---

## Prompt

> *"You're at the customer's whiteboard. The use case is the BFSI support agent assist — 50K calls/day, ~30K-token system+policy bundle, ~500-token user turns, ~1.5K-token responses. Walk me through the prompt-caching cost-impact analysis for this customer. I want to see the math, the assumptions, and the architectural moves."*

---

## Time box

- **Whiteboard:** 10 min.
- **Defense:** 5 min Q&A.
- **Hard cap:** 18 min total.

---

## The Strong-Hire walk-through (placeholder — fill in Week 2)

### Step 1 — Name the cost components
- Input tokens (cache miss) at $X/M tokens.
- Input tokens (cache hit) at ~$Y/M tokens (significantly cheaper).
- Cache write premium at ~$Z/M tokens (small surcharge on the first call).
- Output tokens at $W/M tokens.

(Use current published Apic prices when the body lands.)

### Step 2 — Lay out the per-call shape
- 30K input (most of which is cacheable: system prompt + policy bundle + tool defs).
- 500 input that is *not* cacheable (the volatile user turn).
- 1.5K output.

### Step 3 — Frame the hit-rate
- 50K calls/day, steady-ish during business hours, bursty around peak.
- Cache TTL bounds eligibility. Calls within the TTL window from the previous burst hit cache.
- Realistic hit-rate: ~85–92% with steady traffic + sticky-session worker pinning.

### Step 4 — Compute naive cost (no caching)
- Per-call input cost = (30K + 500) × cache-miss rate = ~30.5K × $X/M.
- Per-call output cost = 1.5K × $W/M.
- Daily total = 50K × per-call cost.

### Step 5 — Compute cached cost
- First call of each cache window: 30.5K input at miss rate + write premium.
- Subsequent calls: 30K input at *hit* rate (Y/X cheaper) + 500 input at miss rate + 1.5K output.
- Weighted by hit-rate: hit-rate × hit-cost + miss-rate × miss-cost + first-call premium.

### Step 6 — Compare
- Cached vs naive — typically 4–7× reduction on input cost, ~2–3× reduction on total cost in this shape.
- Breakeven: caching pays from the *first* repeated call within TTL.

### Step 7 — Architectural moves to lift hit-rate
- Sticky-session worker pinning.
- Pre-warm at shift start.
- Co-locate calls in time (avoid sub-second sparseness within a worker).

---

## Rubric

### Strong Hire
- All 7 steps walked, with rounded numbers.
- Cost components named, including the cache-write premium (most candidates skip this).
- Hit-rate is *justified* by traffic shape, not assumed.
- Architectural moves named to lift hit-rate.
- Eval implication called out (cache pollution across model versions).
- 10 min on the whiteboard, 5 min in Q&A, calm.

### Hire
- 5 of 7 steps; cache-write premium skipped; hit-rate stated without justification.
- Eval implication missed unless prompted.

### Lean No
- "Caching saves about 70%" — no math.
- Single-line answer; no whiteboard work.
- Hit-rate quoted as a number without traffic-shape context.

### Strong No
- Caching positioned as a "tip" rather than architecture.
- Wrong direction on cost (claims caching makes things more expensive).

---

## Q&A defense — likely follow-ups

### Q — *"What if the system prompt changes weekly?"*
A — Cache invalidates on the new version. First calls in the new window are misses; hit-rate recovers within the TTL. Architectural move: roll out new prompts during low-traffic windows.

### Q — *"What's the trap most teams fall into?"*
A — Cache pollution across model versions during evals. Evals must run with cleared cache; production cache state will skew latency and cost numbers if used as eval baseline.

### Q — *"What if hit-rate drops to 50%?"*
A — Diagnose first: traffic-shape change, TTL too short, pinning broken, or the cacheable zone has dynamic content. Math still works at 50% — just at lower savings.

### Q — *"How do I sell this to a CFO?"*
A — Convert the input-cost reduction into a per-business-unit monthly number. CFO cares about predictability + total spend, not multipliers. Module 10 has the BFSI cost model.

---

## Common Lean No traps

### Trap 1 — Skipping the cache-write premium
Most candidates do. Strong Hire calls it out.

### Trap 2 — Treating hit-rate as a constant
Real systems have variable hit-rate. The architect must say what they're designing *for* and what triggers a re-tune.

### Trap 3 — Forgetting the eval implications
Caching can hide regressions if eval state is wrong. Strong Hire surfaces this without prompting.

### Trap 4 — Using nominal prices, not effective prices
The effective cost is `hit-rate × hit-cost + miss-rate × miss-cost`. Quoting only nominal is incomplete.

---

## How to run this drill

1. Whiteboard cold (paper, tablet, or actual board).
2. Use a stopwatch.
3. Record yourself defending against the 4 Q&A follow-ups.
4. Log to Drill Tracker.
5. Application trigger: 2 Strong Hires across 2 sessions, with different traffic-shape variants.

---

## Cross-references
- Note: [03 — Prompt Caching Architecture](../Notes/03-Prompt-Caching-Architecture.md).
- CodeLab: [04 — Prompt Caching](../CodeLabs/04-Prompt-Caching.md).
- Module 10: [Cost Architecture](../../10-Cost-Latency-Reliability/Notes/01-Cost-Architecture.md).
- [Drill Tracker](../../Drill-Tracker.md).

## Strong-Hire bar for this drill
- All 7 steps walked, including cache-write premium.
- Hit-rate justified by traffic shape, not assumed.
- Eval cache-pollution called out unprompted.
- Q&A follow-ups all answered cleanly.
