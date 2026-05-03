# 02 · Note 04 — Resume Rewrite Plan

> **Status: Outline.** Body fills in Week 1. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The section-by-section plan to transform the current GCP-tailored resume into an Apic Applied AI Architect resume. The plan, not the resume itself (the `.docx` lives outside the repo).

> **What this file is NOT.** A generic resume guide. Not advice on bullet structure. Not LinkedIn optimization.

---

## Why a rewrite, not a tweak

The current resume is calibrated for a GCP Solutions-Architect lens — cloud-first, infrastructure-leaning, breadth-tilted. The Apic Applied AI Architect role is depth-tilted, customer-embedded, eval-and-safety-first, and *anti-generic-AI-passion*. Three differences make a tweak insufficient:

1. **Headline shape:** "Cloud SA" → "Customer-Embedded AI Architect / FDE." Different identity.
2. **Bullet calibration:** outcome-first cloud bullets read as marketing in an Apic context. Bullets need to be problem → constraint → choice → outcome, with the constraint visible.
3. **Mission signal density:** the Apic resume needs operational evidence of safety/governance/eval-driven delivery in *every* role, not just the most recent.

---

## Section-by-section plan

### Section 1 — Headline + summary
- **Old:** "Senior AI Solutions Architect with deep expertise in GCP / Vertex AI…"
- **New (target):** *"Customer-embedded AI architect for regulated enterprise. Eval-framework-led delivery. ~3 years building Forward Deployed Engineering practices for BFSI, healthcare, retail; recent work on HIPAA-aligned multi-agent platforms with Claude."*
- **Why:** The role wants depth + customer + eval + regulated. The summary should land four of those four signals in one sentence.

### Section 2 — Experience bullets
Each bullet rewritten to **problem → constraint → choice → outcome**. The constraint must be visible.

- **Old (outcome-led):** "Improved customer support resolution time 35%."
- **New (problem-led):** "Architected support-agent assist copilot with policy-compliance overlay; the bind was RBI-grade auditability + 95th-percentile <2s latency. Chose Claude tool-use loop with eval-gated rollout (shadow → assist). Resolution time down 35% at zero CISO escalation."

The outcome stays, but it's the *fourth* data point, not the first.

### Section 3 — Apic-specific anchors
A short "Recent Apic-relevant work" subsection, surfacing:

- HIPAA-aligned governance work
- SHAP / responsible-AI artifact
- Eval-framework design for at least one customer
- Regulated-workload rollout patterns (shadow → assist → autonomous)

Each as one bullet, each with the constraint visible.

### Section 4 — Skills / tools
- **Drop:** generic cloud certifications, breadth lists.
- **Keep:** Claude / Apic SDK, eval frameworks (RAGAs-style, LLM-as-judge), MCP, LangGraph (only if used in production), regulatory frameworks (HIPAA, DPDPA, RBI guidelines).
- **Why:** the skill list is a credibility signal, not a search-engine input. Breadth dilutes; specificity compounds.

### Section 5 — Education + credentials
Minimal. One line each. No course-list inflation.

### Section 6 — Public artifacts (NEW section)
A 2-line block at the bottom pointing to:

- The Claude API artifact (Module 04 deliverable, GitHub).
- This course site (`learnapic.sunmintz.com`) — *if* you decide to make it semi-public.

---

## Calibration rules

### Rule 1 — One page or two
Two pages is fine for ~10+ years of experience, but the second page cannot be filler. If the second page is just a long skills list, cut it.

### Rule 2 — No "passionate about AI safety"
Anywhere. Not in summary, not in skills, not in interests. Mission alignment lands through *what you did*, not *what you care about*.

### Rule 3 — Every role gets one safety/governance bullet
Even pre-AI roles can carry this signal — *"introduced production readiness review gates for ML model deployments"* counts.

### Rule 4 — Numbers without units are noise
"35% improvement" without "of what, against what baseline" is filler. Either include the unit or cut the bullet.

### Rule 5 — The first bullet of each role is the highest-stakes choice you made
Not the longest-running responsibility. The choice with the most thinking behind it.

---

## Bullet rewrite worksheet (template)

For each existing bullet, run it through this template:

```
Original bullet:
________________________________________________________________

The customer (1 phrase):
________________________________________________________________

The constraint that bit (1 phrase):
________________________________________________________________

The architectural choice (1 phrase):
________________________________________________________________

The trade-off you accepted (1 phrase):
________________________________________________________________

The outcome (1 phrase, with unit + baseline):
________________________________________________________________

Rewritten bullet (one line):
________________________________________________________________
```

Run every bullet through this. Discard bullets that can't fill 4 of the 5 slots above; they're filler.

---

## Output

A `.docx` resume kept locally (not in this repo). Module 02 closes when the `.docx` exists, has been read aloud once for tone, and a non-Apic friend can read it in 90 seconds and describe the candidate's career arc back to you in 1 sentence.

→ Application trigger checklist item #1 closes here.

---

## Cross-references
- Sibling: [Note 01 — Career Arc Thesis](01-Career-Arc-Thesis.md) — the resume should be a written form of the same arc.
- Sibling: [Note 03 — Mission Alignment as Evidence](03-Mission-Alignment-as-Evidence.md) — anchors that load the bullets.
- Module 04 deliverable: the GitHub artifact referenced in the Public Artifacts section.

## Strong-Hire bar for this file
- Resume passes the "constraint-visible" test: every experience bullet names the constraint that bit.
- Headline lands 4 signals in 1 sentence: depth + customer + eval + regulated.
- No mission-paraphrase phrases anywhere.
- Public Artifacts section points to the Claude API artifact (or notes it as pending).
- Friend-readback test passed: 90-second read, arc summarized in 1 sentence.
