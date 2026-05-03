# 10 · Note 02 — Latency Architecture

> **Status: Outline.** Body fills in Week 4. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** How latency decomposes and where architectural decisions land in each component — the latency budget as an architecture constraint that must be specified before model selection.

> **What this file is NOT.** A benchmark comparison table — this is about designing a latency budget and then architecting to meet it.

---

## Latency decomposition

Every Claude API call has four measurable latency components:

```
Total latency = Network (client → API endpoint)
              + Queue wait (if rate-limited or burst)
              + Inference: TTFT (time-to-first-token)
              + Generation: tokens × (1 / tokens-per-second)
```

**Component 1: Network (client → API endpoint)**

For an AWS-hosted application calling the Apic API or Amazon Bedrock (ap-south-1 / Mumbai):
- Intra-AWS (Lambda/EC2 in ap-south-1 → Bedrock ap-south-1): 5–20ms typical
- Cross-region (ap-south-1 → us-east-1 Apic API): 120–180ms typical
- Mobile/edge client → AWS: 30–80ms typical (Mumbai urban, LTE/5G)

**Architecture implication:** Use Bedrock in ap-south-1 for synchronous customer-facing use cases. The 120–180ms cross-region penalty is paid on every call and adds up to 10–25% of your total latency budget for fast interactions.

**Component 2: Queue wait**

Under normal load: ~0ms. Under burst (rate limit exceeded): 200ms–2s. Rate limit events are latency spikes, not just errors. Design your rate limit tiers to give burst headroom above your p99 traffic volume.

**Component 3: TTFT (time-to-first-token)**

TTFT is the time from sending the request to receiving the first token in the response. It is dominated by:
- Input token count (larger prompts = longer prefill time)
- Model size (Haiku TTFT < Sonnet TTFT < Opus TTFT)
- Cache status (cache hit dramatically reduces TTFT by skipping prefill for cached portion)

Approximate TTFT ranges (p50, Bedrock ap-south-1, April 2026):
- Haiku 4.5: 150–300ms (small-medium prompt)
- Sonnet 4.6: 300–700ms (small-medium prompt)
- Opus 4.7: 600–1,500ms (small-medium prompt)

With prompt caching and a large system prompt: TTFT for the non-cached portion only. A 5,000-token system prompt that is cached reduces TTFT by ~300–500ms on Sonnet.

**Component 4: Generation rate**

After the first token arrives, tokens stream at a rate measured in tokens/second:
- Haiku 4.5: ~120–160 tokens/sec
- Sonnet 4.6: ~80–110 tokens/sec
- Opus 4.7: ~40–60 tokens/sec

For a 300-token response on Haiku: generation adds ~2s after TTFT. On Opus: ~5–7s. For short responses (50–100 tokens), generation time is negligible relative to TTFT.

**Perceived vs. actual latency:**

Streaming changes user experience. With streaming enabled, the user sees the first token at TTFT and reads as generation proceeds. Perceived latency ≈ TTFT. Without streaming, perceived latency = total round-trip. For conversational UI, always stream.

---

## TTFT as the user experience metric

TTFT is the number that matters for interactive use cases. Users perceive:
- <200ms: instant
- 200–500ms: responsive
- 500–1000ms: noticeable but acceptable
- 1–3s: slow (user may second-guess the product)
- >3s: broken (support tickets start)

This is why model tier selection is a latency decision, not just a capability decision. Putting Opus on a synchronous customer-facing workflow is an architecture mistake even if Opus produces better output.

---

## The BFSI latency budget by use case

| Use case | Acceptable TTFT | Acceptable total | Notes |
|---|---|---|---|
| Support agent assist (UC1) | <500ms | <3s | Streaming to agent UI; 300-token avg output |
| SOP search (UC2) | <1s | <2s | Employee-facing; synthesis response |
| RM copilot (UC3) | <1s | <5s | Generation-heavy; longer synthesis |
| Compliance review (UC4) | <3s | <30s | Async-tolerant; Opus-tier; batch eligible |
| Claude Code dev productivity (UC5) | <1s | <10s | IDE inline; 30% longer prompts |
| Exec analytics assistant (UC6) | <2s | <15s | Dashboard query; complex multi-step |

**How to read this table in an interview:**

The latency budget drives model selection. UC4 (compliance review) has a 30s total budget — that unlocks Opus with large context. UC1 (support agent) has a 500ms TTFT budget — that mandates Haiku on the hot path. UC3 (RM copilot) generates longer synthesis — build in buffer for generation time, not just TTFT.

---

## Architectural latency levers

**Lever 1: Streaming**

Enable streaming on every synchronous use case. Non-negotiable. The engineering effort is 2 days; the perceived latency improvement is 60–70% on user satisfaction.

**Lever 2: Prompt caching**

Caching reduces effective input for the TTFT calculation. If a 4,000-token system prompt is cached, the prefill phase processes only the 200-token user turn. TTFT improvement: 40–60% on Sonnet for large system prompts.

**Lever 3: Model tier (hot-path Haiku)**

The single highest-impact latency lever. Putting Haiku on the hot path for extraction/routing/classification and Sonnet or Opus only for the generation step creates a tiered pipeline:

```
User query → Haiku (50ms: intent classification + retrieval query) 
           → Retrieval (vector search: 80ms)
           → Sonnet (400ms TTFT: answer synthesis)
           → Streamed to UI
```

Total TTFT to first synthesis token: ~530ms. This beats a single Sonnet call on the full pipeline because the Sonnet call has shorter input (retrieval results only, not the full query routing logic).

**Lever 4: Parallel tool calls**

For agentic workflows, fan out tool calls in parallel rather than sequential. Claude 3+ models support requesting multiple tool calls simultaneously. A workflow that calls three tools sequentially at 200ms each takes 600ms. Fanned out in parallel: 200ms.

**Lever 5: Context window management**

Sending more tokens than needed is the most common latency mistake. Audit input token count per use case. Common waste:
- Sending full conversation history when only the last 3 turns matter (truncate)
- Sending a 10,000-token policy document when retrieval should extract 500 relevant tokens (RAG)
- Sending verbose tool schemas when only 2 of 15 tools are relevant to this query (dynamic tool loading)

---

## Latency at Bedrock vs. first-party API vs. Vertex

**Bedrock (ap-south-1):**
- Lowest network latency for AWS-hosted BFSI workloads
- In-India residency — required for DPDPA 2023 compliance
- p50 TTFT: ~300–500ms (Sonnet)
- p99 TTFT: ~700–1200ms (Sonnet) — higher variance than p50
- Burst limits: per-account, can be raised via AWS support

**Apic first-party API (us-east-1):**
- 120–180ms additional network latency from Mumbai
- Higher rate limits at equivalent tier
- No in-India residency — non-starter for most BFSI data categories
- Better for dev/test from non-India environments

**Vertex AI (asia-south1 / Mumbai):**
- Google Cloud; comparable in-India residency posture to Bedrock
- Generally higher p99 variance than Bedrock in early 2026
- Appropriate for GCP-primary customers; not the primary recommendation for this customer (AWS-primary)

**The interview answer:** For this BFSI customer (AWS-primary, RBI/DPDPA data residency required), Bedrock ap-south-1 is the only option for production. First-party API is used in development/sandbox. Vertex is not in scope unless the customer has a GCP workload.

**Variance is the hidden cost:** p50 latency sells the product. p99 latency is what the CISO sees in the incident report. Design your SLA on p99, present your demo on p50, and be honest about the difference.

---

## Cross-references
- [Note 01 — Cost Architecture](./01-Cost-Architecture.md) — caching's dual role in cost and latency
- [Note 03 — Reliability Patterns](./03-Reliability-Patterns.md) — latency timeouts and retry strategy
- [Note 04 — Observability](./04-Observability.md) — TTFT p50/p99 as primary metric
- [Case Study 01 — BFSI Cost & Latency Model](../Case-Study/01-BFSI-Cost-and-Latency-Model.md)
- [Drill 02 — Defend a Latency Budget](../Drills/02-Defend-a-Latency-Budget.md)

---

## Strong-Hire bar for this file

- Decomposes latency into four components and explains which architectural decisions affect each
- Names TTFT as the UX metric that drives model selection on synchronous use cases
- Specifies the BFSI latency budget per use case with explicit reasoning (not arbitrary numbers)
- Explains caching's latency benefit (TTFT reduction) separately from its cost benefit
- Addresses p50 vs. p99 variance and commits to designing SLAs on p99

---

## The interview-room answer (60 sec)

> "Latency decomposes into four components: network, queue wait, TTFT, and generation time. The one that dominates user experience on interactive use cases is TTFT — time to first token — because with streaming, users see output immediately and perceived latency is mostly TTFT. For this BFSI customer, the support agent must hit sub-500ms TTFT, which mandates Haiku on the hot path — Sonnet simply can't hit that budget reliably. Prompt caching reduces TTFT by 40–60% by shrinking the prefill computation for repeated system prompts. The two things most architects get wrong: they benchmark p50 and promise SLAs on it, when they should be designing on p99 which can be 2–3x higher. And they pick model tier on capability alone without checking if the latency budget is even achievable with that model. Latency budget should be the first constraint, model selection comes after."
