# 10 · Note 03 — Reliability Patterns

> **Status: Outline.** Body fills in Week 4. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** Patterns for making Claude deployments reliable — retry logic, fallbacks, circuit breakers, and graceful degradation — applied to a regulated BFSI environment where "Claude is down" has a specific, designed answer.

> **What this file is NOT.** Generic distributed systems theory — every pattern here is calibrated to Claude API failure modes specifically.

---

## Failure taxonomy

Not all failures are equal. Before designing retry and fallback logic, name what you're retrying from:

| Failure type | HTTP status | Retryable? | Notes |
|---|---|---|---|
| API timeout (no response) | — (timeout) | Yes, with backoff | Check idempotency first |
| Rate limit exceeded | 429 | Yes, with backoff | Respect `retry-after` header |
| Inference error (model error) | 500, 529 | Yes, 1–2 times | Not always transient |
| Overloaded (API capacity) | 529 | Yes, aggressive backoff | Apic-specific status |
| Streaming interruption | — (mid-stream) | Yes, from checkpoint | Requires resumption logic |
| Invalid request | 400 | No | Fix the prompt, not the retry |
| Auth failure | 401 | No | Rotate credentials, alert |
| Content policy refusal | 200 (stop_reason: "max_tokens" or refusal) | No | Handle in application logic, not retry |

**The critical distinction:** A 529 (overloaded) is a temporary capacity signal — retry with aggressive backoff. A 400 is your bug — retrying wastes quota and time. A content policy refusal is not a failure; it is the model working correctly. Retrying a refusal is a product logic error, not a reliability pattern.

---

## Retry strategy

**The parameters:**

- **Max retries:** 3 for synchronous; 5 for async/batch
- **Initial backoff:** 1s for rate limits; 500ms for inference errors
- **Backoff multiplier:** 2× (exponential)
- **Jitter:** Full jitter (random(0, backoff)) — prevents thundering herd
- **Max backoff:** 30s (cap to prevent extreme delays)
- **Timeout per attempt:** Synchronous: 8s; Async: 60s; Batch: no per-call timeout

**Pseudocode:**

```python
def call_with_retry(request, max_retries=3):
    delay = 1.0
    for attempt in range(max_retries + 1):
        try:
            return claude_client.messages.create(**request)
        except RateLimitError as e:
            if attempt == max_retries:
                raise
            wait = e.retry_after or (delay + random.uniform(0, delay))
            time.sleep(wait)
            delay = min(delay * 2, 30)
        except OverloadedError:
            if attempt == max_retries:
                raise
            time.sleep(delay + random.uniform(0, delay))
            delay = min(delay * 2, 30)
        except APITimeoutError:
            if attempt == max_retries:
                raise
            time.sleep(delay * 0.5 + random.uniform(0, delay * 0.5))
            delay = min(delay * 2, 30)
        except BadRequestError:
            raise  # Never retry — fix the request
```

**BFSI-specific constraint:** Retry budgets must be visible in observability. A system that is silently retrying 20% of requests looks fine from the outside but is burning 20% extra quota and adding 1–3s to p99 latency. Alert on retry rate >5% as a leading indicator of API health degradation.

---

## Fallback hierarchy

Design fallbacks before deployment. The question "what does the system do when Claude fails?" must have a documented, tested answer.

**The BFSI fallback hierarchy (ordered, stop at first success):**

```
Level 1: Retry with same model (exponential backoff, max 3 attempts)
Level 2: Downgrade model tier (Sonnet → Haiku, or Opus → Sonnet)
Level 3: Return cached static response (if use case supports it)
Level 4: Human escalation / queue for later processing
Level 5: Graceful degradation message to end user / agent
```

**Level 2 (model downgrade) details:**

Model downgrade is the most powerful and underused fallback. If Sonnet returns a 529, switch to Haiku. The response quality drops, but the interaction completes. For the support agent assist use case, Haiku's response quality is acceptable for most queries — the agent will flag low-confidence responses for human review anyway.

Implementation: maintain two client configurations (primary tier, fallback tier). The fallback is transparent to the caller — the response has the same schema, just with a `model_used: haiku` flag added for observability.

**Level 3 (cached static response) details:**

For high-frequency, low-specificity queries, a pre-generated response cache is appropriate. In SOP search, if Claude is unavailable, return the last generated answer for that query if it was generated within the past 24 hours. Cache at the query-hash level, not the request level.

Not appropriate for: anything time-sensitive, anything where the query is unique (RM copilot, compliance review).

**Level 4 (human escalation):**

For the support agent use case: if Claude fails after all retries, route the conversation to a human queue. The agent UI shows "Transferring to a specialist" — no error message to the end customer. The bank's existing Genesys routing handles this. The integration contract must be agreed with the Contact Centre Operations team before go-live.

---

## Circuit breaker pattern

The circuit breaker prevents a failing API from being hammered with retries — protecting both your application and the upstream service.

**Three states:**

1. **Closed (normal):** Requests flow through. Error rate tracked.
2. **Open (failing):** Requests fail fast without calling Claude. Periodic probe after timeout.
3. **Half-open (recovering):** One test request allowed. If it succeeds, close the circuit; if it fails, reopen.

**Thresholds for BFSI:**

| Use case | Open threshold | Open timeout | Notes |
|---|---|---|---|
| Support agent assist | Error rate >25% over 60s | 30s | High volume; open quickly |
| SOP search | Error rate >40% over 120s | 60s | Lower volume; more tolerance |
| Compliance review | Error rate >50% over 300s | 120s | Async; tolerant |
| RM copilot | Error rate >30% over 90s | 45s | Higher stakes; moderate tolerance |

**When the circuit is open:**

The system should immediately route to the fallback path (human escalation, cached response) without waiting for the retry budget. This is the difference between a 3-second degraded experience and a 30-second failed experience.

**The CISO's question:** "What does your system do in the 30 seconds after Claude goes down?" The circuit breaker answer: "After 25% error rate over 60 seconds, the circuit opens. All calls fail fast to fallback in <50ms, no retries. The Genesys queue gets a load spike, and we alert on-call within 2 minutes. We have capacity headroom in the human queue for a 30-minute outage."

---

## Graceful degradation: what the support agent does when Claude is unavailable

This is the specific scenario the CISO will probe. Have a concrete answer.

**The degradation design for UC1 (support agent assist):**

```
When Claude is unavailable (circuit open or all retries exhausted):

1. The agent UI banner: "AI assistance temporarily limited — 
   suggested responses may be slower or unavailable."
   
2. Existing rule-based suggestions continue (pre-Claude playbook — 
   these run in the bank's existing CRM layer, no dependency on Claude).

3. If agent needs help beyond rule-based: one-click escalation to 
   senior agent / specialist queue.

4. All affected interactions are flagged in the audit log for 
   post-incident review.

5. Recovery: when circuit closes, the UI banner disappears automatically.
   No agent action required.
```

**What you do NOT do:**

- Show an error message to the end customer ("AI is down")
- Let the agent see a blank response with no fallback
- Let the application spin indefinitely (timeout must be enforced)
- Re-enable Claude automatically without a smoke test ("half-open" state in circuit breaker handles this)

---

## SLA design: 99.9% vs. 99.95%

| SLA | Allowed downtime/month | Equivalent |
|---|---|---|
| 99.9% | ~43 minutes | Acceptable for internal tools (SOP search, RM copilot) |
| 99.95% | ~22 minutes | Appropriate for customer-facing (support agent assist) |
| 99.99% | ~4 minutes | Reserved for core banking — not appropriate for AI layer |

**Reality check:** The Apic API / Bedrock SLA is 99.9% by contractual terms. Your application-level SLA cannot exceed the provider's SLA unless you build multi-provider redundancy (Bedrock + Vertex, with failover). For most BFSI deployments, 99.9% is achievable; 99.95% requires active-active multi-region or multi-provider.

**The BFSI recommendation:**

- UC1, UC2, UC3 (customer/employee-facing, synchronous): target 99.9% application SLA; document that 99.95% requires multi-provider investment
- UC4, UC6 (async, batch-tolerant): 99.5% is acceptable (retry windows absorb downtime)
- UC5 (developer productivity): 99.0% is acceptable (developer tolerance for tool outages is higher)

The CISO will ask about the SLA. The honest answer: "Claude's hosted SLA is 99.9%. Our application adds retry, circuit breaker, and fallback to get our effective application-level availability above that. We've tested for the 43-minute scenario. If you need 99.95%, we need to scope a multi-provider architecture and the additional cost."

---

## Cross-references
- [Note 02 — Latency Architecture](./02-Latency-Architecture.md) — latency budget and timeout design
- [Note 04 — Observability](./04-Observability.md) — error rate alerting, circuit breaker metrics
- [Note 05 — Production Readiness Review](./05-Production-Readiness-Review.md) — reliability section of PRR
- [Drill 03 — Blast Radius Conversation](../Drills/03-Blast-Radius-Conversation.md)
- [Module 09 — Security, Privacy, Governance](../../09-Security-Privacy-Governance/Notes/05-CISO-Trust-Building-and-Honest-Hallucination.md)

---

## Strong-Hire bar for this file

- Names the failure taxonomy and distinguishes retryable from non-retryable failures
- Specifies retry parameters with jitter and explains why jitter prevents thundering herd
- Describes the five-level fallback hierarchy with specific BFSI fallback behavior at each level
- Explains circuit breaker states and gives concrete thresholds for the BFSI use cases
- Directly answers the CISO question: "what happens in the 30 seconds after Claude goes down?"

---

## The interview-room answer (60 sec)

> "Reliability for Claude deployments has three layers. First: retry strategy — exponential backoff with full jitter, max three attempts for synchronous, separate retry budgets for 429 rate limits versus 529 overload. Never retry a 400 — that's your bug. Second: fallback hierarchy — retry same model, downgrade to cheaper tier, serve cached static response, escalate to human. The model downgrade is underused and very effective; Haiku on fallback completes the interaction at lower quality rather than failing. Third: circuit breaker — when error rate exceeds 25% over 60 seconds, fail fast to fallback without retrying. This is what answers the CISO's question: 30 seconds after Claude goes down, every support agent call is already routing to the Genesys human queue via the open circuit, not timing out after 8 seconds each. The difference is whether your customers experience a 3-second degraded experience or a 30-second failure."
