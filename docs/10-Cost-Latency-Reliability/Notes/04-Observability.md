# 10 · Note 04 — Observability

> **Status: Outline.** Body fills in Week 4. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The metrics, traces, and logs that give operators visibility into a Claude deployment — what to instrument, what to alert on, and how it integrates with the BFSI customer's existing stack.

> **What this file is NOT.** A generic APM tutorial — every metric here is chosen because it answers a specific operational or governance question in a regulated BFSI context.

---

## The three observability layers

```
Layer 1: METRICS    — aggregated rates and distributions, per use case
                       → "Is the system healthy right now?"

Layer 2: TRACES     — per-request spans, tool call chains
                       → "Why did this specific request behave this way?"

Layer 3: LOGS       — structured audit events, immutable
                       → "Prove to the regulator that this is what happened."
```

The layers serve different audiences:
- **SRE / platform team:** metrics and traces (p99 TTFT alerts, error rate dashboards)
- **Product / use-case owners:** metrics (cost per use case, refusal rate trends)
- **CISO / compliance:** logs (audit trail, PII flag events, policy refusals)
- **Regulator (RBI inspection):** logs (immutable, signed, tamper-evident)

All three layers must be operational before go-live. The CISO will not sign PRR without seeing the audit log design.

---

## Key metrics

Instrument these at the application layer (not inferred from Bedrock billing — that lags 24 hours and is too coarse):

**Latency metrics:**

| Metric | Dimension | Alert threshold |
|---|---|---|
| `ttft_ms` (p50, p95, p99) | per use case, per model | p99 > 2× baseline for 5 min |
| `total_latency_ms` (p50, p95, p99) | per use case | p99 > SLA for 5 min |
| `streaming_first_token_ms` | per use case | p99 > 500ms |

**Volume and cost metrics:**

| Metric | Dimension | Alert threshold |
|---|---|---|
| `input_tokens_per_request` (p50, p99) | per use case | p99 > 2× baseline (prompt injection probe?) |
| `output_tokens_per_request` (p50, p99) | per use case | p99 > 3× baseline |
| `cache_hit_rate` | per use case | <60% for 15 min (cache cold start) |
| `estimated_cost_usd` | per use case, per business unit | >110% daily budget |
| `calls_per_minute` | per use case | >120% burst threshold |

**Quality and safety metrics:**

| Metric | Dimension | Alert threshold |
|---|---|---|
| `refusal_rate` | per use case | >5% (baseline: <1%) |
| `error_rate` (4xx, 5xx) | per use case | >5% over 60s |
| `retry_rate` | per use case | >10% (leading indicator) |
| `fallback_activation_rate` | per use case | >2% (leading indicator) |
| `pii_flag_rate` | per use case | Any spike — review immediately |

**Why refusal rate matters:** A spike in refusal rate on the support agent use case means either (a) your prompt is triggering a guardrail inadvertently — a prompt engineering problem, or (b) users are attempting to use the AI for off-topic or adversarial queries — a security signal. Both need investigation; only one needs a security response. You can't tell which without the metric.

**Why cache hit rate matters:** A drop in cache hit rate below 60% is usually a cache cold start (TTL expired under traffic surge) or a prompt change that invalidated the cache key. Both are cost and latency events. Alert fast.

---

## Trace design for agentic workflows

Metrics tell you the rate; traces tell you the path. For agentic workflows where Claude makes multiple tool calls, a trace is essential for debugging.

**Span model:**

```
Trace: user_request_id = "req-abc-123"
│
├─ Span: intent_classifier (Haiku)
│   ├─ input_tokens: 450
│   ├─ output_tokens: 30
│   ├─ ttft_ms: 180
│   └─ result: { intent: "account_balance_query" }
│
├─ Span: tool_call: fetch_account_data
│   ├─ tool: "crm_account_lookup"
│   ├─ duration_ms: 95
│   └─ result: { success: true, records: 1 }
│
├─ Span: response_synthesizer (Sonnet)
│   ├─ input_tokens: 1,200
│   ├─ output_tokens: 280
│   ├─ ttft_ms: 420
│   ├─ cache_hit: true
│   └─ result: { response_length: 280, refusal: false }
│
└─ Span: safety_filter (post-processing)
    ├─ pii_detected: false
    └─ response_approved: true
```

**Every span must carry:** trace ID, span ID, parent span ID, use case, model, input/output tokens, cache status, latency, error status.

**For BFSI:** The trace must be queryable by `user_id` (for DPDPA data subject access requests — "show me all AI processing for this customer") and by `request_id` (for incident investigation). Store traces for 90 days minimum; compliance review traces for 7 years (RBI audit requirement).

**Implementation:** AWS X-Ray (native with Bedrock) + manual span instrumentation using the AWS SDK. If the customer uses Datadog, the Datadog APM agent can ingest X-Ray traces with the Datadog-AWS integration. Both are proven; X-Ray has lower cost for high-volume use cases.

---

## The cost observability dashboard

One dashboard, one question: "How much is this costing and where?"

**Dashboard panels:**

1. **Daily cost by use case** (bar chart, last 30 days) — shows trends, budget vs. actual
2. **Cost per 1,000 calls by use case** (line chart) — shows unit economics, catches prompt bloat
3. **Cache hit rate by use case** (gauge, current day) — real-time efficiency signal
4. **Monthly cost projection** (number, based on current 7-day run rate) — CFO-facing
5. **Cost by business unit** (pie chart, MTD) — chargeback visibility
6. **Top 10 most expensive request templates** (table) — identifies optimization opportunities

**Who gets this dashboard:**
- Platform team: daily
- Use-case product owners: weekly
- CFO office: monthly summary (auto-generated PDF from the dashboard)
- CISO: on request (ad-hoc queries tied to specific incident investigation)

---

## Alert design

**Alerts that page (immediate action required):**

| Alert | Condition | Page channel |
|---|---|---|
| Error rate high | Error rate >25% for 2 min, any use case | PagerDuty (P1) |
| Latency SLA breach | p99 TTFT >2× SLA for 5 min | PagerDuty (P2) |
| PII flag spike | PII flag rate >5% in 5 min | CISO + PagerDuty (P1) |
| Cost anomaly | Projected daily cost >150% budget | Slack + PagerDuty (P2) |
| Auth failure | Any 401 | PagerDuty (P1) — potential key compromise |
| Refusal spike | Refusal rate >10% in 5 min | Slack (P3) — investigate next business day |

**Alerts that go to weekly review (no page):**

- Cache hit rate trending down over 7 days
- Output token p99 growing over 7 days (prompt bloat)
- Retry rate increasing over 7 days
- Cost per use case above quarterly trend

**Alert fatigue rule:** If an alert fires more than 3 times per week for 3 consecutive weeks without generating a fix, either fix the root cause or raise the threshold. Alerts that are routinely acknowledged and ignored are worse than no alerts — they train the team to ignore the paging channel.

---

## Integration with existing BFSI observability stack

The customer's existing stack: CloudWatch (AWS-native), Datadog (enterprise APM + SIEM), Splunk (security log aggregation), Grafana (dashboards).

**Integration map:**

| Observability layer | Where it lives | How Claude data gets there |
|---|---|---|
| Metrics | CloudWatch + Datadog | Custom metrics via CloudWatch PutMetricData API, forwarded to Datadog via AWS integration |
| Traces | AWS X-Ray + Datadog APM | X-Ray instrumentation in Lambda/ECS; Datadog agent picks up via X-Ray integration |
| Logs | CloudWatch Logs → Splunk | Structured JSON logs to CloudWatch; Splunk HTTP Event Collector for compliance logs |
| Cost dashboard | Grafana | Grafana CloudWatch datasource, custom cost metrics |
| Alerts | PagerDuty via Datadog | Datadog monitor → PagerDuty integration (existing) |

**The CISO's audit log requirement:**

Compliance-grade logs must be:
- Structured JSON (not free-text)
- Immutable (CloudWatch Logs with object lock, or S3 with WORM policy)
- Tamper-evident (hash chaining or AWS CloudTrail integration)
- Accessible for 7 years (RBI audit window)
- Queryable within 48 hours of regulator request

Minimum fields for every Claude audit log event:
```json
{
  "timestamp": "ISO8601",
  "request_id": "uuid",
  "use_case": "string",
  "user_id": "hashed or anonymized",
  "model": "string",
  "input_token_count": "integer",
  "output_token_count": "integer",
  "refusal": "boolean",
  "pii_flagged": "boolean",
  "fallback_activated": "boolean",
  "response_approved": "boolean"
}
```

Do NOT log: prompt text, response text, user PII (name, account number, phone). Log token counts and metadata only. The actual prompt/response may be stored in a separate encrypted store with a stricter access policy if the use case requires post-hoc review (e.g., compliance review outputs).

---

## Cross-references
- [Note 01 — Cost Architecture](./01-Cost-Architecture.md) — cost metrics design
- [Note 03 — Reliability Patterns](./03-Reliability-Patterns.md) — error rate and retry alerting
- [Note 05 — Production Readiness Review](./05-Production-Readiness-Review.md) — "monitoring live" PRR gate
- [Module 06 — Evaluation Architecture](../../06-Evaluation-Architecture/Notes/01-Evals-as-Architecture-Concern.md)
- [Module 09 — Security, Privacy, Governance](../../09-Security-Privacy-Governance/Notes/02-Data-Leakage-Surfaces.md)

---

## Strong-Hire bar for this file

- Structures observability into three layers (metrics, traces, logs) and maps each to a stakeholder
- Names the specific metrics that matter — not a generic list — with alert thresholds and rationale
- Designs a trace schema for agentic workflows with all required fields
- Specifies audit log fields and explicitly calls out what NOT to log (prompt text, PII)
- Integrates with the specific BFSI stack (CloudWatch, Datadog, Splunk) with concrete plumbing

---

## The interview-room answer (60 sec)

> "Observability for Claude has three layers. Metrics tell you if the system is healthy right now — key ones are TTFT p50/p99, cache hit rate, error rate, refusal rate, and cost per use case. Traces tell you why a specific request behaved the way it did — for agentic workflows, every tool call gets its own span. Logs satisfy the CISO and regulator — structured, immutable, 7-year retention per RBI requirements. The metric I'd alert on first that most teams miss: refusal rate. A spike in refusals is either a prompt engineering bug or a security signal — you can't tell which without the metric, and both need a different response. For this customer's stack, metrics route to CloudWatch plus Datadog, traces via X-Ray, audit logs to Splunk via HTTP Event Collector. The one thing I'd be explicit about: do not log prompt text or PII in the audit log — log token counts and metadata. The actual content goes to a separate encrypted store with a tighter access policy."
