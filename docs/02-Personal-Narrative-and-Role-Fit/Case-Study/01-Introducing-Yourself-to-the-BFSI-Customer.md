# 02 · Case Study 01 — Introducing Yourself to the BFSI Customer

> **Status: Outline.** Body fills in Week 1. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The first case-study chapter that exercises the Module 02 narrative material against the running BFSI customer. How would the Apic Applied AI Architect introduce themselves on call 1 with this customer's CIO/CDO/CISO?

> **What this file is NOT.** The discovery call itself (that's Module 03). Not the architecture proposal. Not a pitch deck.

---

## The customer (recap)

A large Indian BFSI institution — multiple regulated business units (retail banking, wealth management, compliance), ~50K employees, RBI scrutiny, DPDPA-2023 data-residency obligations, prior burned by a generic GenAI POC that hallucinated regulatory facts during a CISO demo.

→ Full customer profile: [Module 01 Case Study](../../01-Apic-Calibration/Case-Study/01-BFSI-Customer-Intro.md).

This chapter advances the thread by introducing the *architect* into the room. The customer has not yet seen any architecture from Apic. They have a fresh CISO scar from the previous vendor.

---

## The setup

**Call 1, joint AE + Architect, 30-min slot.** The customer brings: CIO (host), CDO (champion candidate), CISO (skeptic, recently burned). The AE has done the meeting setup; the architect's job in the first 5 min is to land identity. The next 25 min is discovery (Module 03).

**The introduction window:** ~3 minutes after the AE hands off. Anything longer than 3 minutes signals you're using customer time for self-marketing. Anything under 90 sec signals you didn't think this through.

---

## What the introduction must do (in 3 min)

Four jobs, in order:

1. **Land identity** — *who you are, in customer terms*.
2. **Earn the right to ask questions** — *why they should believe you can advise them*.
3. **Signal safety posture** — *how you think about the CISO scar without naming it*.
4. **Hand back to discovery** — *open the door for them to talk*.

If all four happen in 3 minutes without flattery and without name-dropping, the architect has succeeded.

---

## The Strong-Hire 3-minute introduction (placeholder)

> *Body to be filled in Week 1. Will load with the BFSI-tuned register of the Module 02 narrative material.*

### Job 1 — Identity (30 sec)
*"I'm the Applied AI Architect on Apic's India team. My day-job is being the technical voice for customers like yours — regulated enterprise, working through the architecture and evals end-to-end. I've spent the last few years embedded with BFSI and healthcare customers building the same kinds of platforms you're considering, including HIPAA-aligned multi-agent work."*

### Job 2 — Earn the right to ask (45 sec)
*"My orientation is depth-first. I'd rather we ship six use cases at a CISO-trusted bar than twelve at a demo bar. The reason is that the customers I've worked with who shipped fast and cleaned up later spent more time cleaning up than building. So today my goal is to listen — not pitch."*

### Job 3 — Signal safety posture (30 sec)
*"On Claude specifically — the safety properties Apic builds in (Constitutional AI as the training method, the Responsible Scaling Policy as a deployment posture) are a big part of why I'm here. They map directly to the kind of regulatory questions your team will have."*

### Job 4 — Hand back to discovery (30 sec)
*"I'd love to spend most of this hour understanding what you're trying to do — the business problem, what's in production today, what you've tried, what didn't work. If it's helpful, I can share customer architecture patterns later in the call once I know what's relevant."*

*(Strong-Hire body draft, 2:15, leaves 45 sec of buffer — to land in Week 1.)*

---

## What's deliberately absent

### No vendor superiority claims
"Apic is the safest" / "Claude is the best" — both Strong No. Customer hasn't asked you to compare.

### No shame around the failed previous POC
You don't reference the burned POC. The CISO will. When they do, you respond with empathy, not "the previous vendor was wrong."

### No solution mentions before discovery
No mention of architectures, model selection, or specific use cases. That's earning-the-right reversed.

### No "passionate about AI safety"
Reads as cosplay even more loudly in front of a CISO than in an interview.

### No flattery
"It's an honor to work with such a respected institution" — Strong No. Adults don't open with this.

---

## The CISO-specific calibration

The CISO is the most likely person in the room to *already* be skeptical of you in minute 1. Three calibrations:

### Calibration 1 — Don't out-CISO the CISO
You're not here to demonstrate that you know more about RBI guidelines than they do. You're here to demonstrate that *you understand why those guidelines exist* and won't ask them to bend any.

### Calibration 2 — Use one regulatory term correctly
DPDPA, RBI Cyber Security Framework, or RBI's Master Direction on IT Governance. *One*, used in passing, in correct context. More than one looks like you crammed.

### Calibration 3 — Name the trade-off, not the solution
"There are well-understood trade-offs between long-context retrieval and document-level RBAC; we'll work through those together when we get to architecture." Names the trade-off; doesn't pre-resolve it.

---

## Anti-pattern introductions to refuse

### Anti-pattern 1 — The credentials parade
"I have 12 years in the industry, certifications in X, Y, Z, and worked at A, B, C…" — customer doesn't care.

### Anti-pattern 2 — The case study dump
"Let me tell you about a similar deployment we did at a BFSI customer in Singapore…" — too early. Save for after discovery.

### Anti-pattern 3 — The Apic mission monologue
"Apic's mission is…" — Strong No.

### Anti-pattern 4 — The over-promise
"We can definitely solve all six of your use cases by Q3." — Strong No. You haven't done discovery yet.

---

## How this case study advances each subsequent module

This is the first case-study chapter; subsequent modules pick up the thread:

- **Module 03** — the actual 25-min discovery that follows this 3-min intro.
- **Module 04** — the Claude platform decisions that emerge from discovery.
- **Module 05** — the six SystemDesigns that the architect proposes.
- **Module 09** — the CISO architecture review with the same CISO from this room.
- **Module 11** — the executive pitch back to the CIO + CDO + CISO loop.
- **Module 12** — the entire BFSI portfolio, dramatized as the capstone interview round.

---

## Cross-references
- Module 01 case study: [BFSI Customer Intro](../../01-Apic-Calibration/Case-Study/01-BFSI-Customer-Intro.md).
- Sibling notes: [01 Career Arc Thesis](../Notes/01-Career-Arc-Thesis.md), [03 Mission Alignment as Evidence](../Notes/03-Mission-Alignment-as-Evidence.md).
- Successor: [Module 03 Discovery Script](../../03-Customer-Discovery-and-Pre-Sales/Case-Study/01-BFSI-Discovery-Script.md).

## Strong-Hire bar for this file
- Introduction lands all 4 jobs in ≤3 min.
- No vendor-superiority claims, no flattery, no premature solutioning.
- One regulatory term used correctly, in passing.
- CISO-room calibration audible (you don't out-CISO the CISO).
- Hand-back to discovery is explicit.
