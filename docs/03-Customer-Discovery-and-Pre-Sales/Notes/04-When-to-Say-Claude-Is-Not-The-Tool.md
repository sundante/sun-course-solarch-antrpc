# 03 · Note 04 — When to Say Claude Is Not the Right Tool

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The judgment call Apic interviewers grade highest on: when to tell a customer Claude is *not* the right tool — and how to do it without losing the deal or the relationship.

> **What this file is NOT.** A list of Claude's weaknesses. Not a pitch for fine-tuning. Not a competitor recommendation.

---

## Why this is a Strong-Hire signal

Apic's mission is "safe and beneficial AI." Telling a customer "you don't need an AI here" is *the most concretely mission-aligned thing an architect can do.* Interviewers who ask this question are looking for one specific thing: the candidate's willingness to leave a deal on the table when the use case shouldn't exist.

The candidate who can't say no is a vendor. The candidate who can is a trusted advisor.

---

## The four "not Claude" patterns

### Pattern 1 — Deterministic problem in disguise
Customer's "AI need" is actually a deterministic problem (regex, structured query, business-rules engine).

- **Telltale:** the answer is verifiable, the inputs are bounded, and stakeholders want zero variance.
- **Strong-Hire response:** *"This looks like a rules engine, not a model. We could use Claude to draft the rules, but the production system shouldn't have inference in the loop."*
- **Example:** "Validate this transaction matches RBI compliance rules" — rules engine, not Claude.

### Pattern 2 — Wrong cost / latency profile
Customer's volume × latency × cost shape doesn't fit a frontier model.

- **Telltale:** millions of decisions/day at sub-100ms — costs blow up, latency budget breaks.
- **Strong-Hire response:** *"At this volume and latency, the cost-per-decision math doesn't work even at Haiku. The right move is a smaller specialized model with Claude as the eval-and-supervise layer."*
- **Example:** real-time fraud scoring on every card swipe. Not Claude in the hot path.

### Pattern 3 — Deployment posture incompatible
Customer's residency, air-gap, or compliance posture rules out current Claude deployment surfaces.

- **Telltale:** "we can't have data leaving our datacenter ever, even encrypted, even ephemerally."
- **Strong-Hire response:** *"This use case won't fit any of the deployment surfaces we have today. Let's either re-scope to a use case that fits Bedrock-Mumbai-or-Vertex-Mumbai, or wait until the deployment posture you need is GA."*
- **Example:** an agency-style customer where the data physically cannot leave the secure enclave.

### Pattern 4 — Use case shouldn't be automated
The use case is high-stakes enough that automating it is the wrong choice — even if it would technically work.

- **Telltale:** the failure mode is irreversible (unauthorized credit decision, automated medical recommendation without HITL).
- **Strong-Hire response:** *"This shouldn't be Claude-driven. The blast radius of a single failure outweighs the productivity gain. Claude can assist a human, but shouldn't be in the decision loop."*
- **Example:** automated loan rejection. Strong No, even if accuracy is high.

---

## How to say no without losing the deal

### Move 1 — Name the constraint that fails
Don't say "Claude can't do this." Say "*this constraint* fails for this use case." Constraints are objective; capabilities are arguable.

### Move 2 — Offer a re-scope, not a refusal
Don't end on "we can't help." End on "the right shape for this is X — let's scope it that way."

### Move 3 — Stay specific about the safety frame
*"This is the kind of use case where we'd rather lose the deal than ship something that could fail badly. Apic's bar is the same as yours — we're not optimizing to close every deal."*

### Move 4 — Document the no in writing
Send the customer a written summary of which use case you de-scoped and why. The CISO will remember it and trust you more.

---

## What's *not* a "not Claude" response

- "It's hard." → That's a pitch failure, not a tool-fit failure.
- "It would take longer than you'd like." → That's a planning conversation.
- "Our competitors might do this differently." → Refuses the actual question.
- "Let me check with engineering." → If true, fine. If a stalling tactic, Strong No.

---

## The interview-room version

When asked *"give me a use case where you'd tell a customer Claude isn't right"*, the Strong-Hire shape is:

1. Pick **one** of the four patterns (don't list all four).
2. Name a **specific** customer profile (regulated, BFSI, the one in the running case study).
3. Name **the constraint that bit** — not Claude's capability gap.
4. Offer the **re-scope** that does fit.
5. Close on the **safety frame** — *"this is where Apic's bar and the customer's bar are the same."*

Total length: 60–90 sec. Anything longer and it reads as defensive.

---

## Cross-references
- Predecessor: [Note 02 — Discovery to Production Motion](02-Discovery-to-Production-Motion.md).
- Module 04: [Model Selection Matrix](../../04-Claude-Platform-Architecture/Notes/02-Model-Selection-Matrix.md).
- Module 09: [Should This Be Claude-Driven](../../09-Security-Privacy-Governance/Drills/03-Should-This-Be-Claude-Driven.md) — the safety-lens version of this same call.
- Drill: [Drills/03 — Three Discovery Objections](../Drills/03-Three-Discovery-Objections.md).

## Strong-Hire bar for this file
- All four "not Claude" patterns recognizable in 30 sec from a customer description.
- The four "saying no without losing the deal" moves are reflex.
- The interview-room version is a 60–90 sec answer, not a list.
- "Lose the deal rather than ship a bad use case" is a posture, not a hedge.
