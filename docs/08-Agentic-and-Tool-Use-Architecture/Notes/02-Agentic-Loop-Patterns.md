# 08 · Note 02 — Agentic Loop Patterns

> **Status: Outline.** Body fills in Week 4. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The three agentic loop patterns — ReAct, plan-and-execute, multi-agent orchestration — with the decision rule for when each fits BFSI workflows and when none of them should be reached for at all.

> **What this file is NOT.** A framework survey. Not an academic treatment of agent architectures. Not a replacement for hands-on loop design in CodeLabs.

---

## The "is this actually agentic?" check — first

Before choosing a loop pattern, ask whether the task warrants agentic architecture at all. Most tasks that look agentic don't need to be.

A task is genuinely agentic when ALL of the following are true:
1. **The sequence of steps is not fully known upfront** — if you can enumerate every step before the request arrives, a pipeline is simpler and more reliable.
2. **At least one step depends on the output of a previous step** in a way that requires model reasoning to mediate (not just chaining fixed functions).
3. **At least one step involves tool use** — if the task is purely generation, it's a multi-turn conversation, not an agent.
4. **The task cannot be broken into independent sub-tasks that run in parallel** — if it can, parallel function calls beat a loop.

For the BFSI running case:
- **Customer support triage:** NOT agentic. Fixed classification → fixed response template. Single call with structured output.
- **SOP search:** NOT agentic. RAG + generation. No tool-call loop needed.
- **RM copilot:** AGENTIC. The sequence varies per customer. Tool calls depend on what prior calls return. Model must reason about which tool to call next.
- **Compliance document review:** AGENTIC. Plan-and-execute — the review sequence is derived from the document, not hardcoded.
- **Exec analytics assistant:** BORDERLINE. Most queries are single Snowflake calls. Becomes agentic only for multi-step analytical questions.

The cost of choosing agentic when you don't need to: higher latency, higher cost, more failure modes, harder evals.

---

## Pattern 1: ReAct (Reason → Act → Observe)

### What it is
The model iterates through a loop: reason about what to do next → call a tool → observe the result → reason again → call another tool → ... → produce final answer when done.

The "reason" step is implicit in Claude (it's the model's inference pass). The "act" is a `tool_use` block. The "observe" is the `tool_result` block the application inserts. The loop terminates when Claude emits `stop_reason: end_turn` without a tool call.

### When ReAct is right for BFSI
- **RM copilot call prep:** The RM says "prep me for my 3pm with Rohan Mehta." Claude reasons: need customer 360 first. Calls `get_customer_360`. Observes the profile — sees a compliance flag. Calls `check_compliance_flags`. Observes — AML flag active. Reasons: should not proceed to portfolio review before surfacing this. Returns a prep summary with the flag prominent.
- **Exec analytics question:** "What is our Q4 disbursement growth by product category vs Q3?" Claude calls `run_snowflake_query` for Q4, then Q3, then reasons about the comparison. Dynamic query construction based on what the prior call returned.

### When ReAct is NOT right
- When the step sequence is fully fixed (use a pipeline).
- When the task requires a long planning horizon — ReAct is greedy (next-step reasoning only). A 20-step compliance review done as ReAct accumulates error at every hop.
- When you need HITL between steps — ReAct's loop structure makes it hard to pause cleanly. Use plan-and-execute instead.

### Failure modes in ReAct
- **Infinite loop:** Claude keeps calling tools without a stopping condition. Mitigations: max_turns limit in the loop controller; Claude's constitution-trained stopping behavior; explicit "stop" signal in the tool schema.
- **Hallucinated tool arguments:** Claude infers an argument value it should have retrieved. Mitigation: required fields + server-side validation.
- **Wrong tool sequence:** Claude calls `draft_proposal` before `check_compliance_flags`. Mitigation: `check_compliance_flags` must be called before `draft_proposal` — enforce this in the system prompt as a hard rule, OR in the tool description of `draft_proposal` as a prerequisite statement.

---

## Pattern 2: Plan-and-Execute

### What it is
Two-phase loop:
1. **Planning phase:** The orchestrating model (Opus 4.7 for complex plans) receives the full task and produces a structured execution plan — a sequence of tool calls or sub-tasks, with dependencies.
2. **Execution phase:** A lighter model (Sonnet 4.6) works through the plan step by step. It executes, reports results, and the planner may revise the plan in light of what execution found.

### When plan-and-execute is right for BFSI

**Compliance document review** is the canonical BFSI case:
- The review sequence is derived from the document type (a product approval note requires a different checklist than an outsourcing agreement).
- The planner (Opus) reads the document header + metadata and produces a review plan: "Step 1: extract key dates. Step 2: cross-reference against RBI circular XYZ. Step 3: check counterparty against AML list. Step 4: identify clauses requiring legal sign-off. Step 5: draft finding summary. HITL interrupt before Step 5."
- The executor (Sonnet) works through the plan, calling tools at each step.
- Each HITL interrupt is a named checkpoint in the plan — clean pause-and-resume semantics.

### Why plan-and-execute beats ReAct for compliance review
- The plan is **auditable** — a compliance team can read the planned steps and verify the review was thorough, even before execution starts.
- HITL is **structurally clean** — interrupt at named checkpoints, not between arbitrary tool calls.
- **Error recovery is local** — if Step 3 fails, the planner can revise only Step 3, not restart from scratch.
- **Model routing** — Opus does the hard reasoning (planning), Sonnet does the execution (tool calls, extraction). Cost-effective and quality-optimal.

### Failure modes in plan-and-execute
- **Overconfident planner:** Opus produces a plan that assumes tools will return data that may not exist. Mitigation: the executor must surface tool failures back to the planner, not silently skip.
- **Plan staleness:** The document changes between planning and execution. Mitigation: version-stamp the document at plan time; abort if hash changes.
- **Over-planning:** Opus produces a 40-step plan for a 5-step task. Mitigation: token budget on the planning phase; max steps constraint.

---

## Pattern 3: Multi-Agent Orchestration

### What it is
Multiple Claude instances running concurrently or in coordination. Common topologies:
- **Orchestrator + subagents:** One orchestrator model manages task decomposition and dispatches to specialist subagents (retrieval agent, calculation agent, drafting agent).
- **Parallel subagents:** Multiple agents work independent sub-tasks in parallel, results aggregated by the orchestrator.
- **Critic + generator:** One agent generates a draft; a second agent critiques it; the first revises. (Relevant for evals and proposal drafting.)

### When multi-agent is the right call for BFSI
The bar is high. Add a second agent only when:
1. **Specialization genuinely helps** — a retrieval subagent optimized for Salesforce calls is faster and cheaper than a general orchestrator doing everything.
2. **Parallelism provides latency benefit** — the RM copilot fetching customer 360 + portfolio snapshot + compliance flags *in parallel* via three subagent calls is faster than doing it sequentially in a single loop.
3. **Trust boundary separation is required** — the compliance review agent should not have access to the CRM write tool. Split into a read-only compliance subagent and a separate write agent behind a HITL gate.

### BFSI multi-agent topology for RM copilot
```
User Utterance
     │
     ▼
Orchestrator (Sonnet 4.6)
     │   Parallel dispatch
     ├──► Retrieval Agent (Haiku 4.5): get_customer_360, get_portfolio_snapshot, check_compliance_flags
     │        └── Returns: customer context bundle
     ├──► SOP Agent (Haiku 4.5): search_sop_knowledge_base
     │        └── Returns: relevant policy passages
     └──► [waits for both]
          │
          ▼
     Orchestrator synthesizes → draft response or next reasoning step
          │
          ▼ (if action needed)
     Action Agent (Sonnet 4.6) — has write tools
          │
          ▼ HITL interrupt
     RM approval
          │
          ▼
     log_crm_interaction / create_service_request
```

### When NOT to add a second agent
- The task is sequential — parallelism adds no benefit, orchestration adds latency.
- The two agents share all the same tools — you've just added an extra inference hop with no capability gain.
- The first agent already completes the task in <3 turns — multi-agent overhead isn't worth the complexity.
- You haven't written evals for the single-agent version yet. Eval the simple thing before you add orchestration complexity.

The Apic-aligned framing: **the minimal agentic footprint principle**. Give the agent the least power needed to accomplish the task. Each added agent is an added failure surface.

### Failure modes in multi-agent
- **Cascading hallucination:** Subagent A returns a hallucinated customer ID. Subagent B uses it. Orchestrator commits. Mitigation: validate tool results at each hand-off; don't pass unvalidated subagent output forward.
- **Cross-agent PII leakage:** Orchestrator bundles customer PII into a prompt to a subagent that doesn't need it. Mitigation: design prompts with data minimization — pass only what the subagent needs for its specific function.
- **Orchestrator paralysis:** Orchestrator receives conflicting results from two subagents and loops. Mitigation: explicit conflict-resolution instruction in the orchestrator system prompt; max_turns limit.
- **Trust level confusion:** A subagent receives a prompt that claims to be from the orchestrator but is injected via a retrieved document. Mitigation: orchestrator messages should be distinguished from user/retrieved content at the application level, not just by prompt position.

---

## Decision tree: which pattern to use?

```
Is the task fully enumerable (all steps known before the request)?
  └── YES → Pipeline (not agentic). Stop here.
  └── NO ↓

Does the task require a long planning horizon (>5 steps, with dependencies)?
  └── YES → Plan-and-execute (Opus planner + Sonnet executor)
  └── NO ↓

Is parallelism or specialization genuinely needed?
  └── YES + trust boundary needed → Multi-agent orchestration
  └── YES but same tools → Parallel tool calls in single ReAct loop
  └── NO → Single-agent ReAct

In all agentic cases: is there a customer-impacting action?
  └── YES → HITL gate required before the action tool fires
  └── NO → Confirm with CISO before asserting this is true
```

---

## Cross-references
- Predecessor: [08 · Note 01 — Tool Schema Design Doctrine](01-Tool-Schema-Design-Doctrine.md)
- Sibling: [08 · Note 03 — State and Memory](03-State-and-Memory.md)
- HITL: [08 · Note 05 — Human-in-the-Loop & Decision Rights](05-Human-in-the-Loop-and-Decision-Rights.md)
- Eval discipline: [06 · Notes — Evaluation Architecture](../../06-Evaluation-Architecture/INDEX.md)
- Case Study: [08 · Case Study 01 — BFSI Agentic Workflow Walkthrough](../Case-Study/01-BFSI-Agentic-Workflow-Walkthrough.md)
- SystemDesign: [08 · SystemDesign 02 — BFSI Compliance Review](../SystemDesigns/02-BFSI-Compliance-Review-Agentic-with-HITL.md)

## Strong-Hire bar for this file
- The "is this agentic?" gate articulated before any pattern discussion — not an afterthought.
- ReAct vs plan-and-execute distinction: "ReAct is greedy next-step; plan-and-execute is horizon-aware." Deliverable cold.
- Compliance review as canonical plan-and-execute case — explained with the Opus/Sonnet routing rationale.
- Multi-agent topology for RM copilot explained with the trust-boundary justification.
- Minimal agentic footprint principle named and credited to Apic's guidance.
- Decision tree deliverable from memory.

## The interview-room answer (60 sec)

> "Before I pick a loop pattern I ask whether the task is actually agentic. Most things that look agentic are pipelines or single calls. For the BFSI cases: support triage is a single structured-output call; SOP search is RAG. The RM copilot is genuinely agentic because the tool-call sequence varies per customer. For that I use ReAct — reason, call a tool, observe, reason again. For compliance document review I use plan-and-execute: Opus plans the review sequence from the document header, Sonnet executes step by step, with clean HITL interrupt points named in the plan. Multi-agent I reach for when I need either parallelism — fetching customer context in parallel across three systems — or trust boundary separation, like keeping the compliance read agent separate from the CRM write agent. The rule is minimal footprint: don't add an agent unless it removes a constraint the single-agent can't remove."
