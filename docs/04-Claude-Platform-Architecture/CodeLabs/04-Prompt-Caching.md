# 04 · CodeLab 04 — Prompt Caching

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, hands-on, BFSI-themed.

> **What this lab is.** Add cache breakpoints to the system-prompt call from CodeLab 01, instrument hit/miss through the usage headers, benchmark cost before vs. after, and demonstrate breakpoint placement for a 30K-token BFSI policy bundle.

> **What this lab is NOT.** A magic cost-reduction switch. Caching has a write cost, a minimum token threshold, and a TTL that requires your architecture to respect. The lab is about understanding the mechanics precisely enough to explain trade-offs to a CISO.

---

## Goal

By end of lab:

- Instrument `system_prompt_call.py` (from CodeLab 01) with `cache_control` breakpoints.
- Prove hit vs. miss via `usage.cache_read_input_tokens` and `usage.cache_creation_input_tokens`.
- Run a before/after cost benchmark on the BFSI policy bundle scenario.
- Document the breakpoint placement doctrine for a BFSI-scale system prompt.

---

## Prerequisites

- CodeLab 01 complete — specifically `system_prompt_call.py` and the JSONL logging.
- CodeLab 03 complete (the compliance review scenario provides a natural 30K-token policy bundle).
- Apic API key with caching enabled (default for most tiers; confirm via usage response).

---

## Background: how Claude prompt caching works

### The mechanics

Prompt caching in the Apic API stores a prefix of the prompt context on Apic's infrastructure between API calls. On a cache hit, the stored prefix is not re-processed — you pay `cache_read_input_tokens` (1/10th of normal input token price) instead of full input tokens.

**Key constraints:**
- Minimum cacheable prefix: **1,024 tokens** (below this, cache writes are rejected silently).
- Maximum cache entries: **4 concurrent breakpoints** per request.
- TTL: **5 minutes** (refreshed on each cache hit, so active conversations stay warm).
- Cache is **per model** — switching from `claude-sonnet-4-6` to `claude-opus-4-7` invalidates.
- Cache is **per API key** — not shared across accounts.

### Pricing differential (April 2026 — confirm current rates)

| Token class | Relative cost |
|---|---|
| Cache write (first call) | 1.25× normal input |
| Cache read (subsequent calls) | 0.10× normal input |
| Normal input (no cache) | 1.00× |

**Break-even:** cache write is profitable after ~2 reads of the same prefix within the TTL window. For a BFSI system with hundreds of calls per minute against the same policy bundle, break-even occurs in seconds.

### The BFSI policy bundle scenario

- BFSI compliance team has a 30K-token system prompt: persona + 15 RBI circulars + DPDPA section.
- Without caching: every one of 200K daily calls pays full 30K input tokens.
- With caching: first call per 5-min window pays 1.25× write; all subsequent pay 0.10×.
- At 200K calls/day across a 14-hour workday → ~238 calls/minute → one write + 237 reads per window.

---

## Step-by-step outline

### Step 1 — Understand the request structure with `cache_control`

The `cache_control` field is added to content blocks inside the `system` array (multi-block form) or the `messages` array. You cannot add it to a plain string `system=` parameter — you must use the block form.

```python
# BEFORE: plain string system prompt (no caching)
resp = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system="You are a BFSI compliance analyst...",
    messages=[{"role": "user", "content": prompt}],
)

# AFTER: block form with cache breakpoint
resp = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": POLICY_BUNDLE_TEXT,   # 30K tokens — the expensive part
            "cache_control": {"type": "ephemeral"},
        },
        {
            "type": "text",
            "text": PERSONA_INSTRUCTIONS,  # ~500 tokens — not worth caching separately
        },
    ],
    messages=[{"role": "user", "content": prompt}],
)
```

**Rule of thumb on breakpoint placement:** put the breakpoint at the boundary between the *stable* (expensive, infrequently-changing) content and the *dynamic* (per-request, per-user) content. The stable part gets cached; the dynamic part runs fresh every call.

---

### Step 2 — Instrument hit/miss detection

```python
# src/sun_claude_artifact/cached_call.py
from apic import Apic
from pathlib import Path
import json
from datetime import datetime, timezone

client = Apic()
POLICY_BUNDLE_PATH = Path("prompts/bfsi_policy_bundle.txt")
LOG_PATH = Path("logs/cache_runs.jsonl")

def load_policy_bundle() -> str:
    """Loads the 30K-token BFSI policy bundle from disk."""
    return POLICY_BUNDLE_PATH.read_text(encoding="utf-8")

def call_with_caching(
    user_prompt: str,
    policy_bundle: str,
    model: str = "claude-sonnet-4-6",
) -> dict:
    resp = client.messages.create(
        model=model,
        max_tokens=1024,
        system=[
            {
                "type": "text",
                "text": policy_bundle,
                "cache_control": {"type": "ephemeral"},
            },
            {
                "type": "text",
                "text": (
                    "You are a BFSI compliance assistant. "
                    "Answer questions using only the policy bundle above. "
                    "If a question cannot be answered from the bundle, say so explicitly."
                ),
            },
        ],
        messages=[{"role": "user", "content": user_prompt}],
    )

    usage = resp.usage
    cache_hit = usage.cache_read_input_tokens > 0
    
    log_entry = {
        "ts": datetime.now(timezone.utc).isoformat(),
        "model": model,
        "cache_hit": cache_hit,
        "input_tokens": usage.input_tokens,
        "output_tokens": usage.output_tokens,
        "cache_creation_tokens": getattr(usage, "cache_creation_input_tokens", 0),
        "cache_read_tokens": getattr(usage, "cache_read_input_tokens", 0),
    }
    
    LOG_PATH.parent.mkdir(exist_ok=True)
    with LOG_PATH.open("a") as f:
        f.write(json.dumps(log_entry) + "\n")
    
    return {
        "text": resp.content[0].text,
        "cache_hit": cache_hit,
        "usage": log_entry,
    }
```

---

### Step 3 — Build the policy bundle fixture

The lab needs a ~30K-token document to demonstrate meaningful cost savings. Use synthetic but realistic content:

```
prompts/bfsi_policy_bundle.txt
├── Section 1: RBI Master Direction — KYC (2016, updated 2023) — ~8K tokens
├── Section 2: RBI Circular — Outsourcing IT Services (2023) — ~5K tokens
├── Section 3: DPDPA 2023 — Chapter III (Obligations of Data Fiduciaries) — ~6K tokens
├── Section 4: Internal BFSI Credit Policy — Chapter 4-7 — ~7K tokens
└── Section 5: SEBI LODR Regulations — Disclosure requirements — ~4K tokens
```

**Why synthetic is fine for the lab:** you need token count realism, not legal accuracy. Use Markdown headers, numbered clauses, and paragraph text. Generate it via Claude itself (one call, no caching needed, discard after).

Token counting tip:
```python
count = client.messages.count_tokens(
    model="claude-sonnet-4-6",
    system=[{"type": "text", "text": policy_bundle}],
    messages=[{"role": "user", "content": "ping"}],
)
print(f"System tokens: {count.input_tokens}")
```

---

### Step 4 — Benchmark script

```python
# scripts/benchmark_caching.py
"""
Run N calls against the same policy bundle.
Print a before/after cost comparison table.
"""
import time
from src.sun_claude_artifact.cached_call import call_with_caching, load_policy_bundle

PROMPTS = [
    "What is the re-KYC frequency for high-risk customers?",
    "Summarize the data residency obligations under DPDPA 2023.",
    "What disclosures are required before outsourcing IT to a third party?",
    "What triggers a SAR filing under AML guidelines?",
    "Can video KYC be used for NRI account opening?",
]

# Token cost assumptions (April 2026 Sonnet 4.6 — verify against current pricing)
INPUT_COST_PER_MTok = 3.00      # USD per million tokens (normal input)
CACHE_WRITE_PER_MTok = 3.75     # 1.25x
CACHE_READ_PER_MTok = 0.30      # 0.10x
OUTPUT_COST_PER_MTok = 15.00    # USD per million tokens

def compute_cost(log_entry: dict) -> float:
    normal = (log_entry["input_tokens"] / 1_000_000) * INPUT_COST_PER_MTok
    write = (log_entry["cache_creation_tokens"] / 1_000_000) * CACHE_WRITE_PER_MTok
    read = (log_entry["cache_read_tokens"] / 1_000_000) * CACHE_READ_PER_MTok
    output = (log_entry["output_tokens"] / 1_000_000) * OUTPUT_COST_PER_MTok
    return normal + write + read + output

policy_bundle = load_policy_bundle()
results = []

print(f"{'Call':>4}  {'Hit?':>6}  {'In':>8}  {'CacheR':>8}  {'CacheW':>8}  {'Cost $':>10}")
print("-" * 60)

for i, prompt in enumerate(PROMPTS * 3):   # 15 calls, cycling prompts
    result = call_with_caching(prompt, policy_bundle)
    cost = compute_cost(result["usage"])
    results.append((result["usage"], cost))
    u = result["usage"]
    print(
        f"{i+1:>4}  {str(result['cache_hit']):>6}  "
        f"{u['input_tokens']:>8}  {u['cache_read_tokens']:>8}  "
        f"{u['cache_creation_tokens']:>8}  {cost:>10.6f}"
    )
    time.sleep(0.2)   # avoid rate-limit on test tier

total_cost = sum(c for _, c in results)
# Compute uncached baseline: all calls pay full input tokens
uncached_cost = sum(
    ((u["input_tokens"] + u["cache_read_tokens"]) / 1_000_000) * INPUT_COST_PER_MTok
    + (u["output_tokens"] / 1_000_000) * OUTPUT_COST_PER_MTok
    for u, _ in results
)

print("-" * 60)
print(f"Cached total:   ${total_cost:.4f}")
print(f"Uncached est.:  ${uncached_cost:.4f}")
print(f"Savings:        {(1 - total_cost/uncached_cost)*100:.1f}%")
```

---

### Step 5 — Breakpoint placement for multi-layer prompts

For a production BFSI system with a layered system prompt, use up to 4 breakpoints ordered by stability:

```
Layer 1 — Regulatory corpus (30K tokens)           ← breakpoint 1 (most stable)
Layer 2 — Product configuration (2K tokens)         ← breakpoint 2
Layer 3 — User-role context (200 tokens, per-call)  ← NO breakpoint (changes per call)
Layer 4 — Dynamic instructions (50 tokens)          ← NO breakpoint
```

**The rule:** a breakpoint saves money only if the tokens above it are identical on subsequent calls. If the content changes, the cache is busted and you pay the write cost again. Role-injected context (e.g., "This is the relationship manager's client portfolio summary: ...") must always sit below all breakpoints.

---

## Expected benchmark results

For a 30K-token policy bundle across 15 calls (first call cold):

| Metric | Expected range |
|---|---|
| Call 1 (cache write) | ~30K cache_creation_tokens |
| Calls 2–15 (cache hit) | ~30K cache_read_tokens, ~0 input_tokens |
| Cost reduction vs. uncached | ~85–92% on input token cost |
| Cache hit rate in logs | 14/15 (93%) |

> **Note:** The 5-minute TTL means if you run the benchmark across sessions the cache is cold again. In a production system with continuous traffic, effective hit rate is much higher.

---

## BFSI-specific trade-offs to know cold

| Question | Answer |
|---|---|
| "Can you cache conversation history for multi-turn?" | Yes — append `cache_control` to the last user turn in history to freeze the conversation prefix. |
| "Does caching work on Bedrock?" | Yes — Amazon Bedrock's Claude offering supports prompt caching; confirm the region (ap-south-1) supports the model version you're targeting. |
| "What if the policy bundle updates?" | Cache is busted on any content change. Plan for a nightly bundle refresh — acceptable since policy documents change on a weekly-to-monthly cadence. |
| "Is cached content visible to Apic?" | Cache storage is on Apic infrastructure; the same data handling terms apply as for non-cached inputs. For DPDPA 2023 data residency, use Bedrock ap-south-1 where the cache is co-located in-region. |

---

## Acceptance criteria

- [ ] `call_with_caching()` function using block-form `system` with `cache_control`.
- [ ] Hit/miss instrumentation via `usage.cache_read_input_tokens` and `usage.cache_creation_input_tokens`.
- [ ] Policy bundle fixture of ≥28K tokens confirmed via `count_tokens`.
- [ ] Benchmark script runs 15 calls and prints the cost comparison table.
- [ ] Cache hit rate ≥ 80% in the benchmark run (excluding the cold first call per TTL window).
- [ ] `cache_runs.jsonl` log produced with per-call cache metrics.
- [ ] Documented breakpoint placement rationale in code comments.

---

## Stretch goals

- Simulate TTL expiry by sleeping >5 minutes mid-benchmark; observe the cache-cold write cost spike.
- Add multi-breakpoint variant: separate breakpoints for regulatory corpus + product config layer.
- Extend the benchmark to compare Sonnet 4.6 vs. Haiku 4.5 cache economics (Haiku is cheaper but less capable for compliance reasoning).
- Wire to the compliance review from CodeLab 03: cache the regulation text, benchmark against CodeLab 03's uncached baseline.

---

## What this lab feeds

- CodeLab 03 (compliance review) — regulation text is the prime caching target.
- Module 05 SystemDesign 04 (Compliance Document Review) — cost model section uses this benchmark.
- Module 10 (Cost, Latency, Reliability) — the BFSI policy bundle cost model is built on this data.

---

## Cross-references
- Note: [03 — Prompt Caching Architecture](../Notes/03-Prompt-Caching-Architecture.md)
- Sibling: [03 — Structured Output](03-Structured-Output.md)
- Sibling: [05 — Custom Eval Harness](05-Custom-Eval-Harness.md)
- System Design: [Compliance Document Review](../../05-Solution-Architecture-Patterns/SystemDesigns/04-Compliance-Document-Review.md)
- Module 10: Cost & Latency Modeling

## Strong-Hire bar for this lab
- Token counts are verified via `count_tokens` — not estimated.
- Hit/miss is proved via usage fields, not inferred from timing.
- Breakpoint placement is documented with the stability-ordering rationale.
- The benchmark produces a real cost delta, not a hypothetical projection.
- The 5-minute TTL implication is understood and discussed in the code comments.
