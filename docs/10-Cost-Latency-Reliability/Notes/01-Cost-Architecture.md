# 10 · Note 01 — Cost Architecture

> **Status: Outline.** Body fills in Week 4. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** How to design token economics into an architecture — not as an afterthought but as a first-class constraint that shapes model selection, caching strategy, and tier design.

> **What this file is NOT.** A pricing page summary — this is about architectural decisions that change unit costs by an order of magnitude.

---

## The cost formula

Token cost is deterministic at design time. The formula:

```
Monthly cost = (input_tokens × input_rate + output_tokens × output_rate) × call_volume
             − (cache_read_tokens × cache_discount × cache_hit_rate × call_volume)
```

**April 2026 reference rates (Claude API, USD per million tokens):**

| Model | Input | Output | Cache write | Cache read |
|---|---|---|---|---|
| `claude-opus-4-7` | $15 | $75 | $18.75 | $1.50 |
| `claude-sonnet-4-6` | $3 | $15 | $3.75 | $0.30 |
| `claude-haiku-4-5-20251001` | $0.25 | $1.25 | $0.30 | $0.03 |

Key insight: output tokens cost 5× input tokens. A prompt that produces long verbose output is expensive in a different category from a prompt that reads large context. Optimize input and output separately.

Cache read costs ~10% of input cost. A 90% cache hit rate on a 2,000-token system prompt saves roughly $2.70 per million calls on Sonnet. At 200K calls/day, that's ~$16K/month saved on that one prompt alone.

---

## The three cost levers

**Lever 1: Model tier selection**

The cost ratio between Opus 4.7 and Haiku 4.5 is 60:1 on input, 60:1 on output. Choosing the wrong tier is the single biggest architectural error in cost design.

Decision framework:
- **Haiku on the hot path** — anything customer-facing, high-volume, latency-sensitive, extractive (classify, extract, route)
- **Sonnet for the reasoning layer** — generation, synthesis, structured drafting, moderate complexity
- **Opus for the judgment layer** — compliance review, adversarial edge cases, audit-grade reasoning, tasks where wrong output has legal/regulatory consequences

The BFSI SOP search use case: Haiku retrieves and ranks, Sonnet synthesizes the answer, Opus reviews if the query touches regulatory text. Three tiers, one workflow.

**Lever 2: Prompt caching**

Prompt caching converts repeated context into cheap cache reads. Anything that appears verbatim at the top of every call is a caching candidate:

- System prompts (role definition, safety guardrails, tool definitions)
- Static reference text (RBI guidelines, internal policy documents, product catalogs)
- Few-shot examples

Caching requires: content is ≥1024 tokens (Sonnet/Opus) or ≥2048 tokens (Haiku), content appears at the start of the prompt (or in `cache_control: ephemeral` blocks), and the cache TTL (5 minutes default, extendable) covers your call pattern.

The cache hit rate is not guaranteed. Burst traffic after a TTL expiry can cause a "cache cold start" — every request pays full input cost until the cache warms. Design for this; see the cost cliff section.

**Lever 3: Batching (Message Batches API)**

For offline workloads — nightly compliance review, document classification, report generation — the Message Batches API offers a 50% cost reduction with 24-hour SLA.

Not appropriate for: anything synchronous, anything customer-facing, anything with a latency requirement.

BFSI use cases suitable for batching:
- Nightly compliance document review (use case 4)
- Monthly executive analytics summarization (use case 6, historical data)
- Bulk SOP re-indexing after policy updates

---

## Cost-per-use-case: BFSI worked estimate

**Use case 1: Customer support agent assist at 200K calls/day**

Assumptions:
- Avg system prompt: 3,000 tokens (role, guardrails, tool defs) — cacheable
- Avg conversation context sent per call: 800 tokens (last 4 turns)
- Avg user query: 120 tokens
- Avg model output: 300 tokens
- Model: Haiku 4.5 (hot path, customer-facing)
- Cache hit rate: 85% on system prompt

Per-call token math:
- Input uncached: 800 + 120 = 920 tokens at $0.25/M
- Input cache read: 3,000 tokens at $0.03/M (85% of calls)
- Input cache write: 3,000 tokens at $0.30/M (15% of calls — cache miss)
- Output: 300 tokens at $1.25/M

Per-call cost:
```
Uncached input:    920 × $0.25/M  = $0.000230
Cache read:      3,000 × $0.03/M × 0.85 = $0.0000765
Cache write:     3,000 × $0.30/M × 0.15 = $0.000135
Output:            300 × $1.25/M  = $0.000375
                                  ──────────
Total per call:                    $0.000817
```

Monthly cost (200K/day × 30 days = 6M calls):
```
6,000,000 × $0.000817 = $4,900/month
```

**Without caching** (same volume, paying full input for system prompt):
```
(920 + 3,000) × $0.25/M × 6M + 300 × $1.25/M × 6M
= 3,920 × $0.25/M × 6M + 300 × $1.25/M × 6M
= $5,880 + $2,250 = $8,130/month
```

Caching saves ~40% on this use case: $3,230/month.

---

## Business unit cost attribution

At scale, "Claude costs X per month" is not useful. You need cost per team, per use case, per product SKU.

**Attribution model:**

1. Tag every API call with metadata: `use_case`, `business_unit`, `user_role`, `channel`
2. Ship these tags as request metadata; capture them in your observability layer (CloudWatch, Datadog)
3. Aggregate by dimension in your cost dashboard

**Per-use-case budget ownership:**
- Support agent assist → Contact Centre Operations budget
- SOP search → HR / Internal Operations budget
- RM copilot → Retail Banking / Wealth budget
- Compliance review → Compliance & Legal budget
- Developer productivity → Engineering / CTO office budget
- Exec analytics → Corporate Affairs / CEO office budget

**Showback vs. chargeback:**

Start with showback (show teams what they spend, don't bill them). Once patterns are established (typically 3 months), move to chargeback with per-use-case budgets and alerting. The CISO will want to know if a single team can generate runaway costs — budget caps per business unit are a governance control, not just a finance one.

**The CFO question you will get:** "What's my cost per customer interaction?" Answer: $0.0008 for Haiku-tier agent assist. For context, a human agent handles ~50 interactions/day at a fully-loaded cost of ~$80/day = $1.60/interaction. Claude is 2,000× cheaper at this tier.

---

## The cost cliff: what happens when caching fails

The cost cliff is a specific failure mode: cache TTL expires during a traffic surge, every concurrent request pays full input cost, the cost spike is 3–5× normal for 5–15 minutes, and if your rate limits are set based on normal cost, you may also hit throttling.

**Scenario: Monday 9am surge on SOP search**

- Normal traffic: 500 req/min, 85% cache hit rate
- Monday morning: 2,000 req/min, cache just expired at 8:58am
- All 2,000 req/min pay full input cost for 5 minutes (cache warms by 9:05am)
- Cost spike: 5× normal for 5 minutes ≈ 1.7% monthly cost blown in 5 minutes

**Mitigation strategies:**

1. **Cache warming job:** A scheduled Lambda/cron job that sends one request per use case at TTL-5 minutes to keep the cache warm. Simple and effective.
2. **Sticky cache via API tier:** Use Apic's extended cache TTL (available at higher usage tiers) for critical system prompts.
3. **Cost anomaly alert:** CloudWatch metric alert on `TokensIn > 2× baseline for 5 min` → pages on-call, not just weekly review.
4. **Graceful degradation budget:** Reserve 15% monthly budget headroom for cache cold starts. Don't budget to the penny.

The CISO cares about this because a cost cliff at scale can be induced by a targeted traffic spike — it's a cost-based denial-of-service vector. Budget caps per use case are a defense.

---

## Cross-references
- [Note 02 — Latency Architecture](./02-Latency-Architecture.md)
- [Note 04 — Observability](./04-Observability.md) — cost observability dashboard
- [Case Study 01 — BFSI Cost & Latency Model](../Case-Study/01-BFSI-Cost-and-Latency-Model.md)
- [Drill 01 — Estimate Tokens from Business Volume](../Drills/01-Estimate-Tokens-from-Business-Volume.md)
- [Module 04 — Claude Platform Architecture](../../04-Claude-Platform-Architecture/Notes/01-Claude-API-Surface.md)

---

## Strong-Hire bar for this file

- Can derive per-call cost from first principles (formula, not a rate card lookup)
- Names the three levers in order of impact magnitude and explains why caching > batching for synchronous workloads
- Produces the BFSI 200K/day worked estimate with explicit assumptions in under 3 minutes
- Connects cost attribution to governance (CISO, budget caps, chargeback)
- Explains the cost cliff and has at least two mitigations ready

---

## The interview-room answer (60 sec)

> "Cost architecture starts with the formula: input tokens times rate plus output tokens times rate, minus caching savings. The three levers in order of impact are model tier selection — a 60x cost ratio between Opus and Haiku means the wrong model choice dominates everything else — prompt caching, which converts your static system prompt from expensive input to cheap cache reads, and batching for offline workloads at 50% discount. For the BFSI support agent at 200K calls per day, Haiku plus prompt caching lands at roughly $5K/month. Without caching, that's $8K. The design decision that unlocks this is putting the cacheable system prompt first and keeping it stable. The one risk I'd flag proactively: cache cold starts during traffic surges are a 3–5x cost spike — I always build a cache warming job and reserve 15% budget headroom for it."
