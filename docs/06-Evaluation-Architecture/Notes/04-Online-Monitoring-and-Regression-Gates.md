# 06 · Note 04 — Online Monitoring & Regression Gates

> **Status: Outline.** Body fills in Week 3. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** How to detect eval drift in production before users notice it — and how to wire regression gates into the deployment pipeline so bad prompts and model updates never reach production without passing a quality bar.

> **What this file is NOT.** A guide to APM tooling. Not a comparison of monitoring platforms. Not a cloud-infrastructure tutorial.

---

## Why offline evals alone are insufficient

Offline evals are run on a fixed dataset. Production traffic is not a fixed dataset. The following cannot be caught by offline evals alone:

- **Distribution shift:** users ask questions the eval set never anticipated. As the product matures, the query distribution changes. A compliance assistant trained and evaluated on Q1 regulatory queries will encounter Q2 regulatory updates that the eval set doesn't cover.
- **Integration failures:** the model's output format changes subtly. The downstream Salesforce integration starts silently misrouting. The eval set tested the model's output but not the integration's parsing of it.
- **User behavior patterns:** users learn the system and start gaming it. Prompt injection attempts increase. The refusal rate changes in ways the offline eval set never anticipated.
- **Infrastructure drift:** cache hit rate drops when a deployment change introduces prompt variability. Latency p95 spikes when traffic patterns change. These are not model quality signals — but they degrade user experience and mask quality signals.

The production monitor is the only thing that catches these.

---

## The two monitoring signal categories

### Category 1 — Rate metrics

Rate metrics are aggregate signals computed over a time window. They are cheap to compute, easy to alert on, and are the first line of defense against system-level failures.

**Refusal rate:** fraction of requests where Claude declined to answer (returned a refusal response). Baseline refusal rates vary by use case:
- Compliance review: ~5% expected (some queries are out of scope)
- SOP search: ~2% expected
- RM copilot: ~3% expected

A sudden spike in refusal rate (e.g., 2% → 15% over a 4-hour window) is the most reliable early-warning signal for prompt regressions. The leading cause: a prompt update changed the instruction framing in a way that makes Claude over-refuse.

**Escalation rate:** fraction of outputs routed to human review (HITL). For use cases with a judge-as-gatekeeper design, escalation rate tracks how often the judge fails cases. A rising escalation rate means quality is degrading. A *falling* escalation rate that wasn't driven by a prompt improvement is suspicious — investigate whether the judge threshold was accidentally loosened.

**Output length distribution (p50, p95):** sudden compression (median drops 30%+) signals format regression. Sudden expansion signals the model has changed its response strategy. Neither is necessarily bad, but both require investigation before accepting as expected behavior.

**Latency (p50, p95, p99):** baseline these per use case. Set alert thresholds at 1.5× baseline p95. Latency spikes are usually infra (cache miss rate change, model endpoint issue) but can signal prompt changes that increased token generation load.

**Cache hit rate:** from [Note 03 — Prompt Caching Architecture](../../04-Claude-Platform-Architecture/Notes/03-Prompt-Caching-Architecture.md): a sudden drop in cache hit rate without a code deploy signals that something in the prompt construction pipeline is introducing variability. Common cause: a datetime or session ID is accidentally included in the cached prompt zone.

**Error rate:** HTTP 5xx, timeouts, malformed JSON outputs. This is baseline SRE monitoring — but wire it into the AI monitoring dashboard, not just the infra dashboard, so the on-call engineer sees it in context.

### Category 2 — Quality metrics

Quality metrics require computation on individual outputs — either by an LLM-as-judge or by analysis of downstream signals. They are more expensive to compute and more informative about what is going wrong.

**Sampled LLM-as-judge score:** run the LLM-as-judge (Sonnet 4.6, calibrated) on a 5% sample of production traffic. Compute rolling 7-day mean faithfulness and completeness scores. Alert when the rolling mean drops below the acceptance threshold established at launch.

**Downstream action success rate:** for agent use cases (RM copilot, exec analytics), track whether the proposed action succeeded in the downstream system. RM copilot: did the proposed meeting notes get accepted by the RM? Exec analytics: did the generated SQL execute without errors? Action failure rate is a lagging indicator but a high-confidence one.

**User re-query rate:** if a user re-queries within 30 seconds of receiving a response (same or similar query), they were probably unsatisfied. This is an implicit negative feedback signal. Track it per use case and compare to baseline.

**Explicit feedback rate:** thumbs-up/down, 5-star rating, whatever explicit feedback mechanism exists. Useful but biased toward the tails (users only bother when very satisfied or very dissatisfied).

---

## Regression gate: eval score threshold before model update ships

A regression gate is a required pass condition in the deployment pipeline. No model update, prompt change, or configuration change ships unless it passes the gate.

**Gate architecture:**

```
[Change candidate: model update / prompt update / config change]
         ↓
[Regression eval run]
  ├── Exact-match suite (RBI clause extraction, format validation)
  ├── LLM-as-judge suite (faithfulness, completeness) on calibrated judge
  └── Comparison to baseline: delta scores, not absolute
         ↓
Gate conditions (ALL must pass):
  ├── Exact-match pass rate >= 98% (was: 98.3% in prod)
  ├── Judge faithfulness mean >= 4.2 (was: 4.3 in prod) [max -0.1 regression allowed]
  ├── Judge completeness mean >= 4.0 (was: 4.1 in prod) [max -0.1 regression allowed]
  └── Refusal rate on held-out edge cases <= 5% (was: 4.8%)
         ↓                  ↓
    [PASS → deploy]   [FAIL → block + investigation required]
```

The delta-based gate is more robust than absolute thresholds because it catches regressions even when the absolute score is still acceptable. A drop from 4.3 → 4.1 is within the "acceptable" range absolutely but represents a 5% regression in a direction you don't want.

**Gate cadence:** run before every deploy. This means the eval suite must be fast enough to run in CI. Design for it: the exact-match suite must run in under 2 minutes; the LLM-as-judge suite must run in under 10 minutes on a 200-case validation set. Achieve this by running evals in parallel and keeping the validation set sized for speed, not comprehensiveness.

---

## Cache state isolation in eval runs

This is the trap from [Note 03 — Prompt Caching Architecture](../../04-Claude-Platform-Architecture/Notes/03-Prompt-Caching-Architecture.md), restated in the eval context.

**The trap:** your production system has a warm cache. When you run your regression eval suite, it runs against the warm cache and gets cached responses for many cases. This means you are not actually evaluating the new model or prompt — you are evaluating the cache. The eval passes. The deploy ships. The new model or prompt hasn't been tested on the cached cases.

**The mitigation:** eval runs must target a clean cache state. Two options:

1. **Dedicated eval API key / account:** use a separate Apic API account (or Bedrock account) for eval runs, with no production cache state. All eval calls are cache misses by design.
2. **Cache-busting prefix:** add a random nonce to the beginning of the system prompt for eval runs only. This breaks the cache key and forces fresh inference. Strip the nonce before comparing outputs — it must not affect the model's response to the actual prompt.

The dedicated account approach is cleaner and is the recommended pattern for BFSI deployments where audit trails matter. The eval account's usage log is a clean record of every eval run, separate from production traffic.

---

## Alert design: what pages oncall vs. what goes to weekly review

Not all metrics are created equal. Paging oncall for a 0.2% change in refusal rate is alert fatigue. Not paging for a 15% spike in fabrication flags is a missed incident. The decision matrix:

| Signal | Page oncall | Weekly review | Trigger level |
|---|---|---|---|
| Refusal rate | Yes | Yes | >2× baseline over 1-hour window |
| Latency p95 | Yes | Yes | >1.5× baseline over 30-min window |
| Error rate | Yes | — | >1% over 15-min window |
| Escalation rate | Yes | Yes | >1.5× baseline over 4-hour window |
| Judge faithfulness score | — | Yes | Rolling 7-day mean drops >0.2 from baseline |
| Cache hit rate | — | Yes | <70% of baseline over 24-hour window |
| User re-query rate | — | Yes | >1.5× baseline over 7-day window |
| Fabrication flag rate | Yes | Yes | Any non-zero fabrication in compliance review over 4-hour window |

The "fabrication flag rate" row is BFSI-specific. For compliance review, any confirmed fabricated citation is a P1 incident — zero tolerance. The judge returns `fabricated_references: []` or `fabricated_references: [...]`. Any non-empty list triggers an immediate alert, not a weekly review.

---

## BFSI monitoring stack: per-use-case alert thresholds

| Use Case | Refusal Rate Baseline | Escalation Rate Baseline | Latency p95 Target | Quality Signal |
|---|---|---|---|---|
| Compliance document review | 5% (out-of-scope queries) | 8% (judge gate rejections) | <5s | Fabrication flag rate: 0; Judge faithfulness: ≥4.2 |
| Internal SOP search | 2% | 4% | <2s | Judge groundedness: ≥4.0; Re-query rate: <15% |
| RM copilot | 3% | 6% | <3s | Action acceptance rate: ≥80%; Judge action quality: ≥4.0 |
| Customer support agent assist | 4% | 10% | <2s | CSAT proxy: ≥4.0/5; Escalation rate |
| Exec analytics (SQL) | 1% (ambiguous queries) | 3% | <5s | SQL execution success: ≥97%; Re-query rate: <10% |
| Developer productivity | 2% | 5% | <3s | Suggestion acceptance rate: ≥65%; Security flag rate: 0 |

These baselines are set at launch based on the pre-launch eval suite pass rates. They are living targets — updated quarterly after reviewing the weekly trend reports.

---

## Cross-references

- [Note 01 — Evals as Architecture Concern](./01-Evals-as-Architecture-Concern.md) — the three-layer eval architecture
- [Note 02 — Eval Taxonomy](./02-Eval-Taxonomy.md) — online metrics as Class 4 evals
- [Note 03 — LLM-as-Judge Design](./03-LLM-as-Judge-Design.md) — production judge design
- [04 · Note 03 — Prompt Caching Architecture](../../04-Claude-Platform-Architecture/Notes/03-Prompt-Caching-Architecture.md) — cache state isolation
- [Case Study 01 — Eval Framework for BFSI Compliance Review](../Case-Study/01-Eval-Framework-for-BFSI-Compliance-Review.md)

---

## Strong-Hire bar for this file

- Can distinguish rate metrics from quality metrics and name at least three of each without prompting
- Can describe the regression gate architecture (delta-based threshold, CI integration, fail-to-block behavior)
- Knows the cache state isolation trap and can name both mitigations (dedicated eval account, cache-busting nonce)
- Can draw the alert escalation matrix — what pages oncall vs. weekly review and why
- Can name the BFSI-specific zero-tolerance alert (fabrication flag rate in compliance review)

---

## The interview-room answer (60 sec)

> "Online monitoring has two signal categories: rate metrics and quality metrics. Rate metrics — refusal rate, escalation rate, latency, cache hit rate — are cheap, continuous, and catch system-level failures fast. Quality metrics — sampled LLM-as-judge scores, downstream action success rates, user re-query rate — are more expensive but tell you what's actually going wrong. Regression gates sit between those and deployment: any prompt or model update must pass a delta-based eval suite in CI before it ships. Delta-based, not absolute — a drop from 4.3 to 4.1 on faithfulness is a block even if 4.1 is 'acceptable.' One trap: evals must run against a clean cache state, not the production cache — otherwise you're testing the cache, not the change. In BFSI compliance review, the zero-tolerance alert is fabrication flag rate — any non-empty fabricated references list is a P1, not a weekly review item. Everything else follows the standard escalation matrix."
