# 06 · Note 05 — Eval Kickoff Script

> **Status: Outline.** Body fills in Week 3. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The facilitation script for the first customer workshop session on eval design — five structured questions that turn a vague business goal into a measurable success criterion the team can actually build against.

> **What this file is NOT.** A project plan template. Not a requirements gathering checklist. Not a technical specification framework.

---

## Why you run this in the room, not as homework

The instinct is to send the customer a survey: "please fill in your success criteria before our next meeting." This does not work for three reasons:

**Reason 1 — Business owners don't know what "acceptable error rate" means.** The question is not intuitive. "What percentage of the time is it OK for the system to be wrong?" has no obvious answer until you've walked through a failure scenario together. The answer only emerges when you make the failure concrete: "If this system is wrong 2% of the time, and you review 500 documents per week, that's 10 wrong assessments per week reaching your HITL reviewer. Is that acceptable?" That conversation requires real-time interaction.

**Reason 2 — The answer changes with context.** The same stakeholder will give different acceptable error rates depending on what consequences you describe. Walking through consequences in the room, in real time, is the only way to get the number that will actually hold up when you present the eval framework to the CISO.

**Reason 3 — Misaligned answers create project failure.** If the ML team builds to 90% accuracy and the compliance officer expected 99.5%, the project fails at acceptance testing — not at delivery. Running the kickoff in the room aligns expectations before any engineering work starts.

**Format:** 60–90 minutes, whiteboard session, product owner + compliance SME + IT/ML lead present. You facilitate. You write. You end with a signed-off success criteria card.

---

## The 5-question eval kickoff script

### Question 1 — What is the business goal in measurable terms?

**Why this question first:** the business goal defines the unit of measurement. "Improve compliance review efficiency" is not a business goal — it is a direction. "Reduce the time from document receipt to compliance decision from 4 hours to 90 minutes" is a business goal. The eval can be built against the latter; not the former.

**How to ask it:**
> "Let's start with what success looks like in six months. Not the AI capability — the business outcome. If this project is working, what number is better? What does that number look like today?"

**What you're listening for:** a measurable outcome with a baseline and a target. If the answer is qualitative ("the compliance team feels less overwhelmed"), probe: "If we could measure that feeling, what metric would change?" Push until you have a number.

**Common answers:**
- Time-per-document-review from 4 hours to 90 minutes
- Compliance officer capacity from 20 reviews/day to 60 reviews/day
- Escalation rate from HITL to senior reviewer from 30% to 10%

**What to do with the answer:** write it on the whiteboard as a box. This is the north star metric. Every subsequent eval design decision must trace back to this box.

---

### Question 2 — What is the failure mode that would make this project fail?

**Why this question second:** the failure mode defines what the eval must catch. Most customers have not explicitly thought about this. They think about success — the positive case. You need them to think about the negative case, because that is what determines eval design.

**How to ask it:**
> "Let's think about the scenario where this project creates a real problem — not just 'didn't improve efficiency,' but caused harm. What would that look like? Walk me through the worst version."

**What you're listening for:** a specific failure mode with identifiable characteristics. Not "the AI makes mistakes" — that is abstract. "The AI cites an RBI circular that doesn't exist and the compliance officer doesn't catch it before submitting to the regulator" — that is specific. You can build an eval for that.

**Common BFSI failure modes:**
- Fabricated regulatory citation reaches regulator submission
- Wrong customer data surfaces in RM copilot recommendation (DPDPA breach risk)
- Escalation route miscalculated → high-priority complaint handled as routine
- SQL query aggregates across wrong tenant boundary → executive sees another business unit's data

**What to do with the answer:** write it next to the north star metric. Label it "failure we cannot tolerate." This becomes the criterion for the highest-priority eval dimension.

---

### Question 3 — What is the acceptable error rate for the critical failure mode?

**Why this question third:** this is where most eval kickoffs fail. The instinct is to say "zero errors" or "100% accuracy." Push back on this — not to lower the standard, but to make it realistic and buildable.

**How to ask it:**
> "If the system makes one error of this type per [week / 100 documents / 1000 queries], is that acceptable? What about one per month? Where's the line?"

**What you're listening for:** a concrete number the business owner can defend. The number will be anchored to consequences: "one per month is acceptable because we have HITL and one per month would still be caught before submission." That reasoning is exactly what you need — it tells you what the HITL layer is actually expected to do.

**When you get "zero errors":** this is the most common answer, and it is almost never operationally defensible. Probe:
> "If we implement HITL review before every submission, what error rate on the pre-HITL output is acceptable? The human reviewer exists precisely to catch errors — what rate would make the reviewer's job feasible?"

This reframe usually unlocks a concrete number. A compliance officer reviewing 50 documents per day can catch 2–3 errors per day without fatigue. That's a 4–6% error rate on the AI pre-HITL output. That is the number you're targeting.

**What to do with the answer:** write it on the whiteboard: "acceptable error rate = X%." This is the threshold for your LLM-as-judge eval dimension. Anything above X% blocks deploy.

---

### Question 4 — Which eval class fits this failure mode?

**This question is for you, not the customer.** You are doing the translation. The customer doesn't need to know the taxonomy — they need to see you picking the right tool.

**How to facilitate:**
> "OK, so we need to catch [failure mode] at a rate below [X%]. Let me sketch what the eval looks like..."

Then walk through: what is the ground-truth structure of the correct answer for this failure mode? Is it deterministic (exact-match)? Does it require semantic judgment (LLM-as-judge)? Does it require regulatory expertise (human eval for calibration)?

**BFSI compliance review:**
- Fabricated citation: exact-match + LLM-as-judge (check that every cited RBI circular exists in the retrieved corpus → exact-match; check that the characterization of the circular is accurate → LLM-as-judge)
- Completeness gap: LLM-as-judge (did the assessment cover all required RBI checklist items?)
- Format compliance: exact-match (does the output contain the required structured fields?)

**What to do with the answer:** write the eval class next to each failure mode. You now have the skeleton of the eval framework.

---

### Question 5 — What is the green threshold for deployment?

**Why this question last:** the threshold cannot be set until you know the acceptable error rate (Question 3) and the eval class (Question 4). Setting the threshold first inverts the logic — you end up adjusting the threshold to match whatever the system achieves rather than designing the system to match the threshold.

**How to ask it:**
> "Given [acceptable error rate = X%] for this failure mode, the eval must have a pass threshold of at least [derived threshold]. Is there a reason that threshold can't be the gate for deployment? What would make you comfortable shipping to production?"

**The translation from error rate to eval threshold:**
- Acceptable error rate 0.5% → judge faithfulness threshold ≥ 4.5/5.0 (calibrated to catch ~99.5% of failures in the calibration set)
- Acceptable error rate 2% → judge faithfulness threshold ≥ 4.2/5.0
- Note: this translation requires calibration data. The first version of the threshold is an informed estimate; it is refined after the first month of production data.

**What to do with the answer:** write the threshold on the whiteboard. Sign off with the product owner: "this is the number that determines whether we ship." Take a photo of the whiteboard. Send it as a follow-up email. That email is the eval framework contract.

---

## Worked BFSI example: compliance document review eval kickoff

**Session setup:** 75 minutes. Attendees: Compliance Head (business owner), Chief Compliance Officer's deputy, IT Lead for AI Platform, the candidate facilitating.

**Question 1 — Business goal:**
> Attendees: "We want to reduce the time our compliance team spends reviewing loan sanction letters for RBI Fair Practices Code compliance."
>
> Probing: "What does that mean in numbers today?"
>
> Attendees: "Each reviewer handles about 12 letters per day, taking ~25 minutes each. We have 8 reviewers. We want to get to 30 per day per reviewer."
>
> North star metric: **Review capacity from 12/day/reviewer to 30/day/reviewer (2.5× uplift).**

**Question 2 — Failure mode:**
> Attendees: "The worst case is the system misses a compliance gap and we submit to the regulator with an uncaught violation. That's a regulatory finding — potential fine, audit trigger."
>
> Failure mode: **Undetected compliance gap in RBI Fair Practices Code check before regulator submission.**

**Question 3 — Acceptable error rate:**
> Initial answer: "Zero."
>
> Probe: "Currently, your human reviewers catch 100% of compliance gaps? Do they ever miss something?"
>
> Attendees: "...There are probably cases that slip through. Maybe 1 per quarter."
>
> Reframe: "So with 8 reviewers × 12 docs/day × 250 working days, you're reviewing ~24,000 documents per year, and you estimate ~4 slip through. That's a ~0.017% miss rate today. If the AI-assisted system has a pre-HITL error rate of 0.5%, and HITL catches 95% of those, your end-to-end miss rate is 0.025% — slightly worse than today but with 2.5× the throughput."
>
> Attendees: "That's actually acceptable. The 0.5% pre-HITL rate is fine if we have good HITL coverage."
>
> **Acceptable error rate: 0.5% pre-HITL fabrication or missed gap rate.**

**Question 4 — Eval class:**
> Facilitator sketch: "We need two eval dimensions. First: did the AI cite any regulatory reference that doesn't exist in the RBI corpus? That's exact-match — we check every cited clause against the corpus. Second: did the AI correctly characterize the clause it cited? That's LLM-as-judge with an anchored faithfulness rubric, calibrated by your compliance SMEs on a 300-case set."

**Question 5 — Green threshold:**
> "0.5% error rate translates to a judge faithfulness threshold of ≥4.4/5 on the calibration set. Exact-match citation validity must be 100% — zero fabricated references pass the gate."
>
> Sign-off: Compliance Head agrees. IT Lead notes it in the project charter.
>
> **Whiteboard output:**
> - North star: 30 reviews/day/reviewer
> - Failure mode: undetected compliance gap → regulator submission
> - Acceptable error rate: 0.5% pre-HITL
> - Eval: exact-match (citation validity 100%) + LLM-as-judge faithfulness (≥4.4/5)
> - Gate: all pass → deploy; any fail → block

---

## What to do when the customer can't name an acceptable error rate

This happens. The answer is to anchor to a comparison — not to accept "zero" as the specification.

**Comparison anchors that work:**

1. **Current human performance:** "Your current team makes approximately X errors per Y documents. What multiple of that would still be acceptable with the efficiency gain?"

2. **Downstream consequence:** "A 0.5% error rate on 100 documents per day means 0.5 errors per day reach your HITL reviewer. If the HITL reviewer can catch that in 5 minutes of additional review, is that worth a 2× efficiency gain?"

3. **Regulatory tolerance:** "For a standard regulatory submission, would you accept an AI review with a 99.5% accuracy rate if a human signs off? For an internal risk report with no external submission, would you accept 97%?"

4. **Reverse engineering from HITL capacity:** "If HITL can review X borderline cases per day without fatigue, and you have Y HITL reviewers, then the error rate the eval must catch before HITL is [X×Y / total volume]. Let's start there."

If none of these work and the customer insists on "zero errors," respond: "We can design for zero errors in the pre-HITL system with 100% HITL coverage of every output — but that eliminates the efficiency gain. The eval design depends on the acceptable non-HITL path. If every output goes to HITL, the eval is different from if 80% of outputs bypass HITL." This usually forces a concrete decision.

---

## Cross-references

- [Note 01 — Evals as Architecture Concern](./01-Evals-as-Architecture-Concern.md)
- [Note 02 — Eval Taxonomy](./02-Eval-Taxonomy.md) — the four classes you're selecting in Question 4
- [Drill 03 — Translate a Customer KPI into an Eval](../Drills/03-Translate-a-Customer-KPI-into-an-Eval.md) — practice translating the output of this script
- [Case Study 01 — Eval Framework for BFSI Compliance Review](../Case-Study/01-Eval-Framework-for-BFSI-Compliance-Review.md) — full implementation

---

## Strong-Hire bar for this file

- Can run the 5-question script from memory in a live interview whiteboard without consulting notes
- Knows why Question 3 (acceptable error rate) is the hardest question and has at least two reframing moves ready
- Can distinguish the "acceptable error rate → eval threshold" translation step and explain why it requires calibration data
- Knows the "zero errors" counter-move: anchor to comparison, not capitulation
- Delivers the compliance review worked example with specific numbers (12→30 reviews/day, 0.5% threshold, 4.4/5 judge score)

---

## The interview-room answer (60 sec)

> "The eval kickoff is the first customer workshop session, and I run it in the room — not as a homework assignment — because the acceptable error rate answer only emerges when you make the failure scenario concrete. The script has five questions: what's the business goal in measurable terms, what failure mode would make the project fail, what's the acceptable error rate for that failure mode, which eval class fits it, and what's the green threshold for deployment. The hardest one is always the acceptable error rate. 'Zero' is the instinctive answer. The reframe is: anchor to current human performance, or to HITL capacity, or to downstream consequence. In the BFSI compliance review kickoff, 'zero' becomes 0.5% pre-HITL once you walk through the math: 100 documents per day, 0.5% is 0.5 errors per day reaching HITL, the HITL reviewer catches them in 5 extra minutes, and the team still gets the 2.5× efficiency gain. That 0.5% is the number the eval threshold is designed around. You end the session with a signed-off success criteria card, not a vague alignment statement."
