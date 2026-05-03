# 08 · Drill 01 — Should This Be an Agent?

> **Status: Outline.** Body fills in Week 4. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this drill is.** Given six use-case descriptions, decide: full agentic loop, single LLM call, RAG, or human workflow. Defend the decision in under 60 seconds per card. This is the "before you build the agent" gate that separates principal-level architects from engineers who reach for agents reflexively.

> **What this drill is NOT.** A debate drill. The decisions are defensible — there are Strong Hire answers, and "it depends" without naming the deciding axis is Lean No.

---

## Prompt

> *Partner reads a use-case card. You have 60 seconds to: (a) name the architecture pattern — Full Agent, Single LLM Call, RAG, or Human Workflow, (b) name the one deciding factor that drove the choice, (c) name the one failure mode if you got it wrong.*

---

## Time box

- **Per card:** 60 sec.
- **Full drill:** 6 cards = ~8 min total (allow transitions).
- **Hard cap:** 75 sec — beyond is Lean No on this drill. The decision should be reflex.
- **Rotation:** Run all 6 cards in order first. Then randomise for speed training.

---

## The 6 scenario cards

### Card 1
*"The bank's customer support team wants to automatically triage incoming chat messages into one of six categories: general enquiry, complaint, fraud report, account service request, product enquiry, or escalation. Volume is 12,000 messages per day. Each triage decision feeds a routing rule. No customer-visible output from the AI — only the routing label."*

### Card 2
*"An RM wants a tool that, given a customer's name, retrieves the customer's current portfolio allocation, compares it against their stated risk profile, and drafts a one-paragraph note flagging any misalignment for the RM's review before the next call. The retrieval is from two systems: Snowflake (portfolio data) and Salesforce (risk profile)."*

### Card 3
*"The compliance team needs to verify that all 1,200 loan agreement documents signed in the last quarter contain the mandatory RBI-required disclosures. The check is purely presence/absence of 12 specific clause types. Output is a CSV: document name, present clauses, missing clauses."*

### Card 4
*"A senior executive wants to ask natural-language questions about the bank's disbursement data in Snowflake: 'What was our home loan growth in Maharashtra Q3 vs Q2?' and 'Which branch had the highest NPA rate last month?' The system should answer conversationally and cite the figures."*

### Card 5
*"The bank wants to build a system that monitors incoming regulatory circulars from RBI, identifies which internal policies are affected, flags the specific policy clauses that need updating, drafts proposed amendment language for each clause, and creates ServiceNow change tickets for the policy owners to review and approve."*

### Card 6
*"An RM wants an assistant that, during a live customer call, listens to the conversation summary (manually typed by the RM mid-call), looks up relevant product information, checks if the customer has any pending requests or compliance flags, and provides the RM with a suggested next-best action in under 5 seconds."*

---

## Strong Hire answers

### Card 1 — Single LLM Call with structured output
**Decision:** Single LLM call. Classification into a fixed 6-class schema. Structured output (JSON with `category: enum`).

**Deciding factor:** The output is a routing label from a closed set. There is no tool use required, no multi-step reasoning, no session state. A classification prompt + structured output enforcement handles this completely.

**Failure mode if you got it wrong (chose "full agent"):** 3–5x latency per message, 3–5x cost, 12K agentic loops per day with no capability benefit. The CISO also has more surface area to review.

**Model choice:** Haiku 4.5 (`claude-haiku-4-5-20251001`). Sub-200ms, high-volume, closed-set output. No reasoning depth needed.

!!! personal "Personal anchor"
    This is the question that filters junior thinking. At the healthcare AI consultancy, the first instinct on a triage task was always "let's make an agent." The Strong Hire answer is: don't. Ship the single call first.

### Card 2 — Full Agent (ReAct, two-tool loop)
**Decision:** Full Agent — specifically a short ReAct loop with two tools: `get_portfolio_snapshot` (Snowflake) and `get_risk_profile` (Salesforce).

**Deciding factor:** The task requires tool use across two systems, and the output of the first call (portfolio data) is needed to evaluate the second (risk profile comparison). The sequence is dynamic: if Snowflake is down, the agent should surface that, not silently fail.

**Failure mode if you got it wrong (chose "single LLM call"):** You'd have to pre-fetch all data and stuff it into the prompt before every call, even when it's not needed. The agent handles "what data do I actually need?" cleanly; a single call requires the application to know upfront.

**Important constraint:** This agent's output is a draft for RM review — it never writes to CRM autonomously. Category B HITL applies.

**Model choice:** Sonnet 4.6 (`claude-sonnet-4-6`). Agentic reasoning, two-tool loop, sub-3s streaming budget.

### Card 3 — Single LLM Call (batch, structured output) — NOT RAG, NOT Agent
**Decision:** Single LLM call in batch mode. Not RAG (no retrieval needed — the document *is* the input). Not agent (fixed checklist, deterministic structure check).

**Deciding factor:** The task is document-to-structured-output with a fixed checklist. For each document: pass the full text + the 12-clause checklist → get back a JSON object with 12 boolean fields. Repeat 1,200 times in batch.

**Failure mode if you got it wrong (chose "agent"):** An agentic loop adds no value here. There are no decisions to make between steps — every document gets the same 12-clause check. You've added state management complexity and latency with no capability gain.

**Failure mode if you chose "RAG":** RAG retrieves relevant passages; it doesn't classify presence/absence across a structured checklist in a single document. Wrong primitive.

**Model choice:** Sonnet 4.6 in Apic Batch API or Bedrock Batch. 1,200 documents, weekly run, cost matters more than latency. With prompt caching on the system prompt + checklist: significant cost reduction.

### Card 4 — RAG with structured query layer (not full agent for most queries)
**Decision:** RAG-style with a named-query tool — not a full agentic loop for typical queries, but needs to handle multi-step analytical questions as a short agent.

**Deciding factor:** Most exec questions are "fetch data from Snowflake + generate answer." That's one named query call + generation. Full agentic overhead not warranted for the majority case. However, some questions ("compare Q3 vs Q2 by region") require two named query calls + comparison reasoning — that's a short 2-turn ReAct loop.

**The right design:** Default to single query call + generation. If the query planner detects a multi-step question (keyword patterns + intent classification), route to a short 2-turn ReAct loop. Cap at 3 tool calls before escalating to "I need to break this into a more structured request."

**Model choice:** Sonnet 4.6 default. Haiku for query routing classification.

**The exec-specific constraint:** Responses must cite exact figures with source and timestamp. Hallucinated numbers from an exec assistant are a Board-level problem. Structured output + citation enforcement required.

### Card 5 — Full Multi-Step Agent (plan-and-execute)
**Decision:** Full Agent, specifically plan-and-execute. This is genuinely the most complex card.

**Deciding factor:** The task has an emergent sequence: the steps for one regulatory circular depend on which internal policies it touches. The "identify affected policies" step determines the scope of all subsequent steps. An agent that plans before executing is the right shape. 

**Architecture sketch:**
- Planner (Opus 4.7): reads the circular, produces an execution plan listing which policies are affected and which clauses need review.
- Executor (Sonnet 4.6): works through each policy-clause pair, calls the SOP knowledge base, drafts amendment language.
- HITL interrupt before every ServiceNow ticket is created. The policy owner, not Claude, submits the change.
- Category C HITL: compliance officer reviews and approves before any policy submission. Claude's amendments are proposals, not approvals.

**Failure mode if you underbuilt (chose RAG):** RAG retrieves similar-sounding policy passages. It doesn't identify all affected policies, draft amendments, AND create tickets. Those require tool use + sequenced reasoning.

**Model choice:** Opus 4.7 for planning phase (complex cross-document reasoning), Sonnet 4.6 for execution.

### Card 6 — Full Agent (short ReAct, latency-critical)
**Decision:** Full Agent — a latency-optimised ReAct loop. The three lookups (product info, pending requests, compliance flags) run in parallel tool calls.

**Deciding factor:** The task requires tool use from three systems AND synthesis into a recommendation in <5 seconds. The parallel tool call capability of the ReAct loop handles this: Claude emits three tool_use blocks simultaneously; the application dispatches them in parallel; results come back; Claude synthesises.

**The latency engineering:** 
- Parallel tool calls: the three fetches run concurrently, not sequentially.
- Pre-warmed context: the customer CIF is loaded into session state the moment the call starts (not after the first user message).
- Haiku for the sub-second lookups; Sonnet for the synthesis.
- Streaming: the RM sees the next-best-action start to appear within 2s of the last tool result arriving.

**Failure mode if you got it wrong (chose "single LLM call"):** You'd have to pre-fetch all three data sources before passing them to Claude. That means three sequential API calls before the LLM even starts — likely exceeding the 5-second budget. The agent's parallel dispatch is what makes the latency target achievable.

---

## Rubric

### Strong Hire
- Pattern named in <15 sec.
- Deciding factor is specific: cites task characteristics (fixed vs dynamic sequence, tool use required or not, parallelism available, latency budget).
- Failure mode named for the alternative choice — shows awareness of what goes wrong when you pick the wrong pattern.
- Model choice given for at least 3 cards with rationale.
- "Single LLM call" is seriously considered for Cards 1 and 3 — not everything is an agent.

### Hire
- Pattern named correctly.
- Deciding factor is generic ("it's complex" / "it needs tool use") without the specific architectural reason.
- Failure mode for the wrong choice not named.

### Lean No
- Chooses "full agent" for Card 1 or Card 3 without acknowledging the simpler alternative.
- "It depends" without naming the deciding axis.
- Deciding factor is a use-case description restatement, not an architectural reason.
- No model selection — treats model choice as irrelevant.

### Strong No
- Claims RAG can handle Card 5's multi-system plan-and-execute without tool use.
- Claims Card 1 requires an agentic loop because "classification is complex."
- Proposes free SQL execution in the analytics assistant without noting the injection risk.
- Doesn't mention HITL for Card 5 write actions.

---

## Common Lean No traps

### Trap 1 — Agent reflex
The instinct to reach for agents for every multi-step task. Cards 1 and 3 are both multi-step in some sense but neither needs an agent. If the sequence is fixed and no reasoning is required between steps, it's a pipeline.

### Trap 2 — "RAG" as a synonym for "LLM with retrieval"
RAG is a specific pattern: retrieve, then generate, grounded in retrieved passages. It's not the right pattern when the input *is* the document (Card 3) or when the output requires live structured data (Cards 2, 6).

### Trap 3 — Ignoring latency constraints
Card 6 has a hard 5-second budget. Picking a single sequential call architecture ignores this. Picking Opus for the live lookup ignores this. Latency is an architectural constraint, not a preference.

### Trap 4 — Forgetting HITL on write actions
Cards 2 and 5 both involve writing (CRM note, ServiceNow ticket). Proposing autonomous writes without HITL is a regulatory failure. The Strong Hire answer names the HITL gate before the interviewer asks about it.

### Trap 5 — Model name errors
Using old model IDs (`claude-3-opus`, `claude-instant`) or getting the Haiku model name wrong. Strong No.

---

## How to run this drill
1. Partner randomises cards after the first pass.
2. Timer starts on last word of the card.
3. 60 sec: pattern + deciding factor + failure mode.
4. Score: pattern correct (1 pt) + deciding factor specific (1 pt) + failure mode named (1 pt) = 3 pts per card.
5. Strong Hire threshold: 16/18 across all 6 cards.
6. Log to [Drill Tracker](../../Drill-Tracker.md).

---

## Cross-references
- Note: [08 · Note 02 — Agentic Loop Patterns](../Notes/02-Agentic-Loop-Patterns.md)
- Note: [08 · Note 01 — Tool Schema Design Doctrine](../Notes/01-Tool-Schema-Design-Doctrine.md)
- Note: [08 · Note 04 — MCP Integration](../Notes/04-MCP-Integration.md)
- Drill: [08 · Drill 02 — Design a Safe Tool Schema](02-Design-a-Safe-Tool-Schema.md)
- [Drill Tracker](../../Drill-Tracker.md)

## Strong-Hire bar for this drill
- 16/18 across two randomised runs.
- "Single LLM call" named unprompted for Cards 1 and 3.
- HITL mentioned unprompted on every card with write actions.
- Parallel tool calls cited as the latency solution for Card 6.
- All model IDs correct (Haiku 4.5, Sonnet 4.6, Opus 4.7).
