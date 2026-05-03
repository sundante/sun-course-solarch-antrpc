# 10 · Drill 01 — Estimate Tokens from Business Volume

> **Status: Outline.** Body fills in Week 4.

> **What this file is.** Back-of-envelope token cost estimation from a business volume description — the skill of deriving a monthly cost figure with explicit assumptions in under 3 minutes.

> **What this file is NOT.** A calculator replacement — the goal is judgment about order of magnitude and the ability to name the top cost lever without a spreadsheet.

---

## Prompt

> You are in a pre-sales meeting. A BFSI CDO says: "We're considering Claude for five use cases. Before we talk architecture, give me a rough monthly cost estimate for use case 1 — our contact centre, 200,000 agent assist calls per day. Average conversation is about 8 turns." Produce your estimate on a whiteboard in 3 minutes. State your assumptions explicitly. Name the single highest-impact cost lever.

**Time box:** 3 minutes. No calculator. Write the math.

---

## Scenario cards

**Scenario 1 (Worked — see Strong Hire below):**
Contact centre agent assist. 200K calls/day. Average 8-turn conversation. Customer queries are account/transaction related (short). Agent responses are 1–3 sentences (short).

**Scenario 2:**
Internal SOP search. 15K employee queries/day. Employees search policy documents. Avg query: 50 words. System retrieves 3 policy chunks (avg 400 tokens each). Response: 2–4 paragraphs.

**Scenario 3:**
Compliance document review. 5,000 documents/month. Avg document: 12 pages / 8,000 tokens. Review output: structured JSON, ~600 tokens. Nightly batch.

**Scenario 4:**
RM copilot. 3,000 RM sessions/day. Each session: 5 interactions. Avg interaction: 150-token query + 400-token CRM context + 500-token response. Model is Sonnet (relationship complexity).

**Scenario 5:**
Executive analytics assistant. 500 queries/day. Each query: 200-token question + 2,000-token data context. Response: 800 tokens. Low volume, high reasoning — Sonnet.

---

## Strong Hire answer: Scenario 1

**Step 1: Establish token counts per call (state assumptions):**

```
Assumption A: System prompt (role, guardrails, tool defs) = 3,000 tokens. 
              This is cacheable and stable.

Assumption B: Conversation context per call = last 4 turns × avg 200 tokens/turn 
              = 800 tokens. (8-turn conversation; send half as context)

Assumption C: Current user query = 80 tokens (short, account-related)

Assumption D: Model output = 250 tokens (1–3 sentence agent suggestion)

Assumption E: Model = Haiku 4.5 (customer-facing, latency-sensitive, 
              short extractive generation)

Assumption F: Cache hit rate on system prompt = 85%
```

**Step 2: Per-call cost:**

```
Input (uncached portion): 800 + 80 = 880 tokens
  → 880 × $0.25/M = $0.00022

Input (cache read, system prompt): 3,000 × $0.03/M × 0.85 = $0.0000765
Input (cache write, misses): 3,000 × $0.30/M × 0.15 = $0.000135

Output: 250 × $1.25/M = $0.0003125

Total per call: ~$0.00083
```

**Step 3: Monthly cost:**

```
200,000 calls/day × 30 days = 6,000,000 calls/month
6,000,000 × $0.00083 = $4,980/month ≈ $5,000/month
```

**Step 4: Top cost lever:**

Model tier selection. If we used Sonnet instead of Haiku on this use case:
```
Sonnet input rate: $3/M vs Haiku $0.25/M (12× more expensive)
Sonnet output rate: $15/M vs Haiku $1.25/M (12× more expensive)
Monthly cost at Sonnet: ~$60,000/month vs $5,000/month
```

The top lever is model tier. Haiku is appropriate for this use case (short, extractive, latency-sensitive). Sonnet would be a $55K/month mistake.

**The assumption log (speak this aloud):**

> "I'm assuming Haiku is the right tier — short, extractive, no complex reasoning. I'm assuming 85% cache hit rate; if the system prompt changes frequently, that drops and cost rises. I'm assuming we send the last 4 turns of context, not the full 8 — if the full conversation is always sent, input tokens double and cost goes up ~30%. I'll validate all three with the engineering team."

---

## Rubric

### Strong Hire
- Derives cost from first principles (formula), not rate card lookup
- States ≥5 explicit assumptions before calculating
- Gets the model tier decision right (Haiku, not Sonnet) and explains why
- Names the top cost lever correctly (model tier) and quantifies the delta
- Completes in 3 minutes with legible whiteboard math
- Proactively flags what could make the estimate wrong (cache hit rate, context window size)

### Hire
- Correct method and correct model tier, but:
  - Assumptions are vague (e.g., "typical input" without a number)
  - Or: names caching as the top lever instead of model tier (partially right, wrong magnitude)
  - Or: takes 5+ minutes
- Fixable with one pass of the formula and more explicit assumption discipline

### Lean No
- Uses a round number without deriving it ("probably around $10K/month")
- Cannot name the model tier or justify why Haiku vs. Sonnet
- Does not state assumptions, or states them after the estimate rather than before
- Names "batching" as the top lever (only applicable to offline workloads — wrong use case)
- Cannot articulate what would change the estimate

### Strong No
- Quotes rates incorrectly (e.g., confuses input and output rates by a factor of 5)
- Recommends Opus for a 200K/day customer-facing use case
- Does not know what TTFT or cache hit rate means
- Claims the estimate is "exact" without acknowledging the assumption dependency

---

## Common traps

**Trap 1: Forgetting output tokens cost 5× input.**
Most candidates correctly estimate input cost and underestimate output cost. For a generation-heavy use case (RM copilot, compliance review), output tokens can be 40–60% of total cost.

**Trap 2: Assuming 100% cache hit rate.**
It's a common optimism. Real cache hit rates for well-designed prompts: 75–90%. For poorly designed (prompt changes frequently): 30–50%. The estimate should have cache hit rate as a sensitivity variable.

**Trap 3: Using Sonnet for everything because "it's better."**
Better at what? For extractive, short-output, customer-facing tasks, Haiku is architecturally appropriate. Defaulting to Sonnet without justification is a cost architecture failure.

**Trap 4: Not stating assumptions before calculating.**
In a pre-sales meeting, assumptions stated upfront = professional. Assumptions stated after a number = backpedaling. Train the habit: "Here are my assumptions, then the math."

**Trap 5: Forgetting to mention the sensitivity.**
The estimate is only as good as the assumptions. A strong candidate closes with: "If volume is 2× higher, cost doubles. If cache hit rate drops to 60%, add 15%. If we switch to Sonnet for any use case, multiply by 12." This is what the CDO actually needs to budget.

---

!!! personal "Personal anchor"
    Practice this drill with Scenario 2 (SOP search) and Scenario 4 (RM copilot) — the harder cases because SOP search has retrieval context to account for and RM copilot is Sonnet-tier. Time yourself. The 3-minute constraint is real — Apic interviewers will stop you.

---

## Cross-references
- [Note 01 — Cost Architecture](../Notes/01-Cost-Architecture.md)
- [Case Study 01 — BFSI Cost & Latency Model](../Case-Study/01-BFSI-Cost-and-Latency-Model.md)
- [Drill Tracker](../../Drill-Tracker.md)
