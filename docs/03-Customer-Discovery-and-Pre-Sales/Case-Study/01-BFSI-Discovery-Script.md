# 03 · Case Study 01 — BFSI Discovery Script

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The worked-out 30-minute discovery script for the running BFSI customer's call 1 — what the architect would actually say, in order, with the partner's likely responses and the architect's reactions.

> **What this file is NOT.** A pitch script. Not a generic discovery template — calibrated specifically for the running BFSI customer's profile.

---

## Customer recap

Indian BFSI institution, multiple regulated business units, ~50K employees, RBI scrutiny, DPDPA-2023, fresh CISO scar from a previous vendor's hallucination during a demo. Six candidate use cases (per [Module 05 INDEX](../../05-Solution-Architecture-Patterns/INDEX.md)) but the customer hasn't decided which to pilot.

Room: CIO (host), CDO (champion candidate), CISO (skeptic). AE has already done warmup and hand-off.

→ Full customer profile: [Module 01 BFSI Customer Intro](../../01-Apic-Calibration/Case-Study/01-BFSI-Customer-Intro.md).

---

## Script anatomy (30 minutes)

| Time | Phase | Architect | Customer |
|---|---|---|---|
| 0:00–0:03 | Architect intro | 3-min intro per [Module 02 case study](../../02-Personal-Narrative-and-Role-Fit/Case-Study/01-Introducing-Yourself-to-the-BFSI-Customer.md) | Listening |
| 0:03–0:08 | Customer context | Open questions | CIO does most of talking |
| 0:08–0:18 | Problem surfacing | Probing, careful | CDO leads; CISO interjects |
| 0:18–0:24 | Constraint enumeration | Specific, technical | All three speak |
| 0:24–0:28 | Pattern recognition | 1–2 customer parallels, no proposals | Listening |
| 0:28–0:30 | Hand-off | "What should I have asked?" | Surfaces the constraint they were waiting to volunteer |

---

## The script (placeholder)

> *Body to be filled in Week 2 with the actual question wording, anticipated customer responses, and architect's reactions. The skeleton below holds the slots.*

### 0:03 — Customer context
- *"Walk me through what your team does today — what's in production with AI, what's in pilot, what's only an idea."*
- Likely response: CIO talks; mentions a previous internal-search POC (not the burned one).

### 0:05 — Use case discovery
- *"Of the six use cases on your radar — support agent assist, internal SOP search, RM copilot, compliance doc review, developer productivity, exec analytics — which one keeps coming back in conversations? Which one would your CFO notice if it shipped?"*
- Likely response: customer either picks one (clear champion) or says "all six matter."
- If "all six matter": *"Which one carries the lowest blast radius if it failed publicly? That's the one to start with."*

### 0:10 — Surface the previous-vendor scar (if not volunteered)
- *"You mentioned earlier you'd worked with another vendor on something similar. What did the previous attempt teach you?"*
- Expected: CISO surfaces the hallucination story.
- Architect's response: empathy, no competitor disparagement. *"That's a class of failure we'd want to design against from the start — eval-gated rollout, citation-grounded retrieval, never autonomous on first deployment."*

### 0:18 — Constraints
- Residency: *"Where does the underlying data live? Mumbai-only, or open to Singapore?"*
- Integration: *"What systems would we be reaching into — ServiceNow, Salesforce, SharePoint, Confluence, Snowflake?"*
- Sign-off: *"Beyond the room here, who else has to bless this before production?"*
- Timeline: *"What's the time horizon for first-value? And what's driving that timeline?"*
- Budget shape: *"Per-use-case budget, or pooled? Is this OPEX or CAPEX-shaped?"*

### 0:24 — Pattern recognition
- *"Two parallels worth flagging, not as proposals but as patterns we've seen. First — the previous-vendor scar is recoverable; the architecture pattern that closes it is eval-gated rollout with citation-grounded retrieval. Second — the six use cases share more than they diverge; we'd suggest scoping them as a portfolio with shared eval infrastructure, not as six separate POCs."*

### 0:28 — Close
- *"Two questions before we wrap. One: what should I have asked you that I didn't? And what's a good answer? Two: who else needs to be in the next conversation?"*

---

## What the architect would *not* do

- Propose model selection in this call.
- Compare Claude to OpenAI/Gemini.
- Reach for slides.
- Ask the CISO for a "list of compliance requirements." (You research RBI/DPDPA before the call; the CISO tests whether you know.)
- Try to close on a POC scope by minute 30.

---

## What this discovery output feeds

- A 1-page problem-and-constraint definition (the gate to Stage 2 — Eval Design).
- Module 04: Claude platform decisions calibrated to *this customer's* residency + integration shape.
- Module 05: prioritized use-case sequence (probably internal SOP search first; lowest blast radius, clearest eval).
- Module 06: eval framework kickoff for the chosen first use case.
- Module 09: CISO architecture review prep for call 3.

---

## Cross-references
- Predecessor: [Module 02 Case Study](../../02-Personal-Narrative-and-Role-Fit/Case-Study/01-Introducing-Yourself-to-the-BFSI-Customer.md).
- Successor: [Module 04 Case Study](../../04-Claude-Platform-Architecture/Case-Study/01-Claude-Platform-Decisions-for-BFSI.md).
- Notes: [01](../Notes/01-Discovery-Call-Structure.md), [02](../Notes/02-Discovery-to-Production-Motion.md), [03](../Notes/03-Stakeholder-Mapping.md).
- Drill: [01](../Drills/01-Discovery-Call-Simulation.md).

## Strong-Hire bar for this file
- Script lands all 6 phases in 30 min, with the architect ≤30% speak-time.
- Previous-vendor scar surfaced or addressed without disparagement.
- Constraint enumeration covers residency + integration + sign-off + timeline + budget.
- "Use cases as portfolio with shared eval infra" framing audible by minute 26.
- Hand-off output is a 1-page problem-and-constraint definition, not a POC commit.
