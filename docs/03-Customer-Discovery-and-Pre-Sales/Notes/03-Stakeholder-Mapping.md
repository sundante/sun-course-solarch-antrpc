# 03 · Note 03 — Stakeholder Mapping

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The four-role map every regulated-enterprise deal has, who plays each role, and how the architect engages each one differently.

> **What this file is NOT.** A persona spec. Not a CRM data model. Not org-chart navigation.

---

## The four roles

Every deal has four functional roles. One person can play multiple roles; multiple people can split one role. Map them in writing before call 2.

| Role | What they want to know | Failure mode if you ignore them |
|---|---|---|
| **Economic buyer** | "Will this pay back, and when?" | Deal stalls at procurement. |
| **Technical champion** | "Is this real, and can I defend it internally?" | No internal momentum between calls. |
| **Security blocker** | "Will this break what I've spent years securing?" | Late-stage CISO veto kills the deal. |
| **Executive sponsor** | "Is this strategic, and is the team ready?" | Re-prio'd out of next quarter's investments. |

---

## Role-by-role engagement pattern

### Economic buyer (CIO, CFO-adjacent, sometimes CDO)
- They want a credible cost-and-payback story, not a technology pitch.
- The architect's deliverable to them: per-business-unit cost projection + value hypothesis. → [Module 10 Case Study](../../10-Cost-Latency-Reliability/Case-Study/01-BFSI-Cost-and-Latency-Model.md).
- Don't pitch architecture. Pitch *predictability*.

### Technical champion (CDO, head of data/AI, principal architect)
- They want to internalize the architecture well enough to defend it without you in the room.
- The architect's deliverable to them: HLD + LLD + SystemDesigns. → [Module 05](../../05-Solution-Architecture-Patterns/INDEX.md).
- Treat them as a peer, not a pre-sales target.

### Security blocker (CISO, sometimes head of compliance)
- Recently burned by a generic GenAI POC. Skeptical by default.
- The architect's deliverable to them: CISO-ready architecture review. → [Module 09 Case Study](../../09-Security-Privacy-Governance/Case-Study/01-BFSI-CISO-Architecture-Review.md).
- Earn the right to ask before proposing. Never out-CISO the CISO.

### Executive sponsor (CEO-1, sometimes the CIO if no separate sponsor)
- They want to know this is strategic, not science-fair.
- The architect's deliverable to them: 1-page exec pitch + 3-version messaging. → [Module 11 Case Study](../../11-Storytelling-Workshops-Demos/Case-Study/01-BFSI-Executive-Pitch-and-Workshop.md).
- Speak in business outcomes, not architecture.

---

## The mapping exercise

After call 1, write the map:

```
Economic buyer:    ____________________
Technical champion:____________________
Security blocker:  ____________________
Executive sponsor: ____________________

Who's missing from the room?
- ____________________
- ____________________

Who do we need to see in call 2?
- ____________________

Who's the most under-engaged?
- ____________________
```

If any role is empty after call 1, that's a discovery follow-up question for call 2 — not a presumption to make.

---

## Reading the room — power dynamics

### Pattern 1 — CIO is also the economic buyer + executive sponsor
Common in mid-market. Means one person carries 3 roles, which means *one* mistake closes the deal.

### Pattern 2 — CISO is louder than the CIO
Means the security blocker has effective veto. Don't be surprised; engage CISO directly. Module 09 work matters more than usual.

### Pattern 3 — CDO is the champion, but champion ≠ buyer
The CDO will do the internal selling, but doesn't sign. Make sure the buyer sees the same architecture in their language.

### Pattern 4 — No identified executive sponsor
Caution sign. Without exec air-cover, the deal lives quarter-to-quarter. Surface the missing role to the AE.

---

## Anti-patterns

### Anti-pattern 1 — Talking only to the technical champion
Most architects' default. Deal stalls at procurement.

### Anti-pattern 2 — Treating CISO as a roadblock
Strong No mindset. CISO is a stakeholder whose constraints make the architecture *better*. Engage them as a peer.

### Anti-pattern 3 — Trying to replace the AE
The architect-AE partnership is split for a reason. → [Note 05 — Working with AEs](05-Working-With-Account-Executives.md).

### Anti-pattern 4 — Blanket-deck-to-everyone
Same architecture, four registers. → [Module 11 Note 01 — Five Registers Doctrine](../../11-Storytelling-Workshops-Demos/Notes/01-Five-Registers-Doctrine.md).

---

## Cross-references
- Sibling: [Note 01 — Discovery Call Structure](01-Discovery-Call-Structure.md).
- Sibling: [Note 05 — Working with AEs](05-Working-With-Account-Executives.md).
- Module 11: [Five Registers Doctrine](../../11-Storytelling-Workshops-Demos/Notes/01-Five-Registers-Doctrine.md).
- BFSI customer: [Customer Intro](../../01-Apic-Calibration/Case-Study/01-BFSI-Customer-Intro.md).

## Strong-Hire bar for this file
- All four roles named, with the engagement pattern memorized for each.
- Mapping exercise run after every call 1.
- Power-dynamic patterns recognized in real time during call 1.
- "Treat CISO as peer, not roadblock" is reflex.
