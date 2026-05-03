# 03 · Note 01 — Discovery Call Structure

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The doctrine for the first 30 minutes with a CIO/CDO/CISO before proposing anything. The motion the JD describes verbatim: "guide customers from initial technical discovery through successful deployment."

> **What this file is NOT.** A pitch playbook. Not a qualification framework (BANT/MEDDIC). Not a sales script.

---

## Why discovery is the highest-leverage 30 minutes in pre-sales

The first 30 min with a customer dictates whether the rest of the engagement is *trusted advisory* (you're chosen as the architect for their hardest problems) or *vendor evaluation* (you're one of three, lowest-price wins). The shape of that 30 min is the difference.

### The diagnostic the customer is running on you
- Did this person come to listen, or to pitch?
- Do they understand our regulatory environment without me explaining it?
- Will they tell me when Claude is *not* the right tool?

If all three answers are yes by minute 25, you're the architect they call back.

### Why most discovery calls fail
- The architect treats the customer's *requested solution* as the requirement.
- The architect proposes architecture before understanding the business problem.
- The architect over-claims Claude's capabilities to keep the deal warm.

---

## The 30-minute anatomy

The shape of a Strong-Hire discovery call:

| Time | Phase | Goal |
|---|---|---|
| 0:00–0:03 | Architect intro (handed off from AE) | Land identity. See [Module 02 Case Study](../../02-Personal-Narrative-and-Role-Fit/Case-Study/01-Introducing-Yourself-to-the-BFSI-Customer.md). |
| 0:03–0:08 | Customer context | What does this team do? Who are their customers? What's in production today? |
| 0:08–0:18 | Business problem surfacing | The slow, patient question loop. Goal: customer says the *underlying business problem*, not the requested AI feature. |
| 0:18–0:24 | Constraint enumeration | Regulatory, residency, integration, talent, timeline, budget. |
| 0:24–0:28 | Pattern recognition (cautious) | Architect names 1–2 customer-pattern parallels — without proposing solution. |
| 0:28–0:30 | Hand-off to next step | Recap what you heard. Propose next-step shape (eval design, architecture sprint, POC scoping). |

**Key principle:** the architect speaks ≤30% of the time across the 30 min. If you've spoken more than 9 minutes, you've taken too much air.

---

## The question taxonomy

Three classes of question; deploy in this order.

### Class 1 — Context questions (open, slow)
*"Walk me through what your team does today."* / *"What's in production?"* / *"Who are the users?"*

Goal: get the customer talking. The architect's job here is to listen and take notes.

### Class 2 — Problem-surfacing questions (probing, careful)
*"You mentioned X — what's the business outcome you're chasing there?"* / *"If the AI piece weren't part of this, how would you solve this problem?"* / *"What did the previous attempt at this teach you?"*

Goal: get *behind* the requested AI feature to the underlying business pain.

### Class 3 — Constraint-enumeration questions (specific, technical)
*"Where does the data live?"* / *"What's your stance on cross-border processing?"* / *"Who has to sign off before any of this goes live?"*

Goal: surface the constraints that will dictate architecture *before* you architect.

→ Sibling: [Note 02 — Discovery to Production Motion](02-Discovery-to-Production-Motion.md) — what happens after discovery.

---

## The "underlying business problem" reframe

The customer rarely says the actual problem. They say the *requested solution*. The architect's job is to slow them down enough to surface the underlying problem.

### Example — request: "We want a chatbot for our support team"
- Surface question: "What outcome would the chatbot improve?"
- Customer: "Resolution time."
- Probe: "What's slowing resolution time today?"
- Customer: "Agents can't find policy documents fast enough."
- Real problem: **enterprise search, not chat.** Architecture changes accordingly.

### Example — request: "We want Claude to write our compliance reports"
- Surface question: "What's the bottleneck in compliance reporting today?"
- Customer: "Drafting takes too long; review takes too long."
- Probe: "Where do reports go wrong today?"
- Customer: "Auditor finds inconsistencies between sections."
- Real problem: **consistency-checking + structured drafting with review gates**, not free-form generation.

### The reframe rule
*The customer's requested AI feature is a hypothesis about the solution. Treat it as a hypothesis, not a requirement.*

→ Drill: [Drills/02 — Surface the Real Problem](../Drills/02-Surface-the-Real-Problem.md).

---

## The "when to bring engineering in" decision

Some discovery calls reveal that the architect is enough; others reveal that the customer's problem needs Apic engineering attention. The decision rule:

- **Architect alone** — when the use case fits standard Claude API patterns, the integrations are well-trodden, and the eval design is the unknown.
- **Engineering joins** — when the customer's bind requires a non-standard residency posture, custom MCP server design, or capability unlocks that aren't yet GA.

Bring engineering in *no earlier than call 3*. Earlier than that signals dependency.

---

## The "tell me what I should ask you" close

Most discovery calls end with an awkward hand-off. The Strong-Hire close is to invert the question:

> *"Before we wrap — what should I have asked you that I didn't? And what's a good answer to it?"*

Two effects:
1. The customer surfaces the constraint they were waiting for you to ask about.
2. The architect signals that they know the discovery wasn't exhaustive — and is honest about it.

---

## What discovery is *not*

- Not a vendor evaluation. Don't compare Claude to OpenAI/Gemini unless asked.
- Not a pitch. Don't reach for slides.
- Not a feasibility check on a pre-decided architecture. The customer's pre-decided architecture is itself a hypothesis.
- Not a free consulting session. If it's drifting that way, signal it: *"this is great context — let's get to a follow-up where we co-design."*

---

## Cross-references
- Sibling: [Note 02 — Discovery to Production Motion](02-Discovery-to-Production-Motion.md).
- Sibling: [Note 03 — Stakeholder Mapping](03-Stakeholder-Mapping.md) — who else should be in the room.
- Sibling: [Note 04 — When Claude Is Not the Tool](04-When-to-Say-Claude-Is-Not-The-Tool.md).
- Drill: [Drills/01 — Discovery Call Simulation](../Drills/01-Discovery-Call-Simulation.md).
- Case study: [BFSI Discovery Script](../Case-Study/01-BFSI-Discovery-Script.md).

## Strong-Hire bar for this file
- Architect-speaking-time ≤30% rule internalized.
- Three question classes deployable in correct order.
- "Underlying business problem" reframe runs cleanly on at least three customer asks.
- "What should I have asked you?" close internalized.
