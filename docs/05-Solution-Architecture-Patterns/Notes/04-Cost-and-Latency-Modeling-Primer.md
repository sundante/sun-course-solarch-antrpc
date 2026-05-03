# 05 · Note 04 — Cost & Latency Modeling Primer

> **Status: Outline.** Body fills in Week 3. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** Back-of-envelope cost and latency modeling for a Claude deployment. The skill that separates principal architects from engineers — and the skill that separates a pre-sales win from a lost deal when the CFO asks "what will this cost?"

> **What this file is NOT.** A billing documentation summary. This is the methodology for building credible cost models that hold up to scrutiny in a customer meeting.

---

## Why cost modeling is an architecture skill

Most candidates treat cost as something you look up after the architecture is designed. Principal architects treat cost as a first-class constraint that shapes the architecture. The right model choice, caching strategy, and latency tier are all cost decisions dressed up as technical decisions.

In BFSI pre-sales specifically, the CFO is often in the room. They will ask: "What is the total cost of ownership over 12 months?" If your answer is "it depends on usage," you have lost the room. The right answer is a model with a central estimate and a sensitivity analysis.

---

## Token cost formula

```
Call Cost = (input_tokens × input_rate) + (output_tokens × output_rate)
           + (cache_write_tokens × cache_write_rate)
           + (cache_read_tokens × cache_read_rate)

Monthly Cost = Call Cost × calls_per_month
```

**April 2026 reference rates for Sonnet 4.6** (verify against current Apic pricing; Bedrock rates differ):

| Token class | Rate (USD per million tokens) |
|---|---|
| Input | $3.00 |
| Output | $15.00 |
| Cache write | $3.75 (1.25× input) |
| Cache read | $0.30 (0.10× input) |

**For Opus 4.7** (use for compliance review, complex planning):
- Input: $15.00 / MTok
- Output: $75.00 / MTok
- Cache rates scale proportionally.

**For Haiku 4.5** (use for routing, classification, eval judging):
- Input: $0.80 / MTok
- Output: $4.00 / MTok

**The 5× rule of thumb:** Sonnet input is ~5× Haiku input. Opus input is ~5× Sonnet input. Use this for quick mental estimates.

---

## Latency decomposition

End-to-end latency for a Claude API call is not just inference time. Decompose it:

```
Total latency = client_to_orch + auth + tool_calls + inference_ttfb + generation + orch_to_client

Where:
  client_to_orch  = network from client to orchestration service
  auth            = IAM/SSO token validation (typically 10–50ms)
  tool_calls      = external API calls, potentially in parallel
  inference_ttfb  = time-to-first-byte from Claude API (network + model startup)
  generation      = (output_tokens / tokens_per_second) × streaming_factor
  orch_to_client  = network back to client
```

**Typical values for Bedrock ap-south-1 (April 2026):**

| Component | P50 | P95 |
|---|---|---|
| Client → Orchestration (in-VPC) | 5ms | 20ms |
| Auth (cached IAM) | 10ms | 30ms |
| Salesforce REST API call | 200ms | 600ms |
| Claude inference TTFB (Sonnet, 1K input) | 250ms | 600ms |
| Claude generation (Sonnet, 200 output tokens) | 800ms | 2000ms |
| Total (no tools, non-streaming) | ~1100ms | ~2700ms |
| Total (streaming, TTFB shown to user) | ~300ms | ~700ms |

**Key insight:** streaming transforms the user experience without changing the total generation time. Always stream interactive use cases.

---

## BFSI Support Agent Assist: worked cost model

**Assumptions:**
- 200,000 calls per day (200K agent-assist queries across a mid-size BFSI contact centre)
- Operating hours: 8am–10pm IST (14 hours/day)
- System prompt: 30K-token BFSI policy bundle (cached)
- Average user turn: 150 tokens
- Average response: 300 tokens
- Model: Sonnet 4.6
- Cache hit rate: 95% (one cache write per 5-min window; 168 windows in 14 hours)

**Token budget per call:**
- Input: 150 tokens (user turn, dynamic)
- Cache read: 30,000 tokens (system prompt, cached) — 95% of calls
- Cache write: 30,000 tokens — 5% of calls (first call per window)
- Output: 300 tokens

**Cost per call:**
```
Cache hit (95% of calls):
  Input:       150 × $3.00/MTok  = $0.000000450
  Cache read:  30000 × $0.30/MTok = $0.000009000
  Output:      300 × $15.00/MTok  = $0.000004500
  Total (hit): $0.000013950 ≈ $0.000014

Cache miss (5% of calls):
  Input:       150 × $3.00/MTok  = $0.000000450
  Cache write: 30000 × $3.75/MTok = $0.000112500
  Output:      300 × $15.00/MTok  = $0.000004500
  Total (miss): $0.000117450 ≈ $0.000117
```

**Daily cost:**
```
Calls with hit:  200,000 × 0.95 = 190,000 × $0.000014 = $2.66
Calls with miss:  200,000 × 0.05 = 10,000 × $0.000117  = $1.17
Total daily:                                              $3.83
Monthly (22 working days):                                $84.26
Annual:                                                  ~$1,010
```

**Without caching (full input cost every call):**
```
Per call: (30,150 × $3.00/MTok) + (300 × $15.00/MTok) = $0.0905 + $0.0045 = $0.0950
Daily:    200,000 × $0.095 = $19,000
Monthly:  $418,000
Annual:   ~$5,016,000
```

**Caching saves ~$5M/year on this use case.** This is the number that gets a CFO's attention.

**Sanity check:** this seems surprisingly low for 200K calls/day. That's because the workload is high-volume low-value queries (Tier 1 support). The token costs are dominated by the cached system prompt, not the user turn. The math holds — verify against current Bedrock pricing.

---

## Latency budget for interactive vs. batch

### Interactive (agent assist, RM copilot)

| Component | Budget |
|---|---|
| End-to-end latency (P95) | < 2,000ms |
| Time-to-first-token (streaming, P95) | < 500ms |
| Tool call chain (2 parallel calls, P95) | < 800ms |
| Claude generation (streaming, 200 tokens) | < 1,200ms streaming |

**Implication:** on the interactive path, you have ~800ms for tool calls and ~500ms for TTFB. This means tool calls must be parallelized. It also means Opus is off-limits for the interactive hot path — Sonnet or Haiku only.

### Batch (compliance review, overnight analytics)

| Component | Budget |
|---|---|
| End-to-end latency | < 30 min per document |
| Documents per hour | ≥ 50 |
| Cost per document | < ₹500 (~$6 at 30K input + 2K output, Opus) |

**Implication:** for batch, Opus is appropriate — the higher quality on complex compliance reasoning justifies the 5× cost premium. Use the Apic Batch API for throughput and 50% cost reduction.

---

## Sensitivity analysis template

Always include a sensitivity table in customer-facing cost models:

| Scenario | Daily calls | Cache hit rate | Daily cost (Sonnet) |
|---|---|---|---|
| Low (pilot, 10 agents) | 5,000 | 90% | ~$0.11 |
| Base (mid-size, 200 agents) | 200,000 | 95% | ~$3.83 |
| High (large bank, 2,000 agents) | 2,000,000 | 97% | ~$35 |
| Disaster (no caching, base volume) | 200,000 | 0% | ~$19,000 |

The disaster row is the "what if caching breaks?" scenario. It's not hypothetical — it informs the circuit breaker design. If caching fails, the fallback must not send 30K-token uncached prompts at 200K/day scale.

---

## BFSI cost governance implications

- **RBI IT Outsourcing Guidelines** require BFSI entities to maintain records of all cloud service costs. The cost model must be documented and auditable.
- **Budget approval process:** a ₹5 lakh/month spend (even from API costs) may require CFO or board sign-off in some BFSI entities. Size the pilot accordingly.
- **Bedrock vs. direct API:** Bedrock pricing includes AWS infrastructure costs but may be eligible for Enterprise Discount Programs (EDP). For a BFSI customer already committed to AWS, Bedrock is often cheaper in total cost than the direct Apic API.

---

## Interview-room answer (60 sec)

*"My cost modeling approach is back-of-envelope first, sensitivity analysis second. I start with the token budget per call: system prompt + user turn + response. For a BFSI support assistant with a 30K-token policy bundle, that's roughly $0.095 per call on Sonnet without caching — multiply by 200K calls/day and you're at $19K/day, or $5M/year. That number doesn't pass a CFO review. With prompt caching at a 95% hit rate, the same workload costs about $3.83/day — $1K/year. The $5M-vs-$1K difference is prompt caching. That's the architecture decision that pays for itself in week one of production. I always present the sensitivity table — the CFO will ask 'what happens if usage doubles?' — and I always include the disaster row: what does it cost if caching fails? Because the circuit breaker that kicks in when caching fails is a business continuity decision, not just an ops task."*

---

## Cross-references
- Note: [01 — Reference Architecture Anatomy](01-Reference-Architecture-Anatomy.md) (Section 10 — Cost Model)
- Note: [05 — Rollout Patterns](05-Rollout-Patterns.md) (cost gates at each phase)
- CodeLab 04: [Prompt Caching](../../04-Claude-Platform-Architecture/CodeLabs/04-Prompt-Caching.md) (the benchmark data for this model)
- Module 10: Cost, Latency, Reliability (deeper treatment)
- System Design 01: Customer Support Agent Assist (uses this cost model directly)

## Strong-Hire bar for this file
- Can compute the BFSI support assistant cost model in a whiteboard interview without notes.
- Knows the $5M-vs-$1K prompt caching delta and can defend it.
- Distinguishes interactive latency budget (streaming + parallelism) from batch latency budget (throughput).
- Knows the "5× rule" for quick mental model comparison.
- Can explain the disaster row in the sensitivity table and what architecture decision it informs.
