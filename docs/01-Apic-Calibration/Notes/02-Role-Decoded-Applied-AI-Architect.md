# Role Decoded — Applied AI Architect

> Most candidates misread this role as "generic Solutions Architect at an AI company." It is not. This note decodes the JD line by line so you can speak to what the role actually is.

---

## The headline

**"Applied AI Architect"** — not "Solutions Architect," not "Sales Engineer," not "Forward Deployed Engineer." Apic chose this title deliberately. The signal:

- **Applied** — production work, not research; customer-deployed, not internal R&D
- **AI** — the platform you're applying is Claude specifically; LLM literacy is assumed
- **Architect** — you design systems; you do not (primarily) build them; you guide customer engineers who do

The combination places you in the **technical pre-sales spine** of the company — the role that translates Apic's product capabilities into customer-deployed enterprise architecture, with India/APAC as your scope.

---

## The location signal

> **Location — Mumbai. Hybrid 25% in-office.**

Mumbai is meaningful. Mumbai is the BFSI capital of India. The largest banks (HDFC, ICICI, SBI, Axis), the largest insurers (LIC, Bajaj, ICICI Lombard), and the largest financial services groups (Tata, Aditya Birla, Reliance) are headquartered or have major presence here. The implication: a meaningful share of your customer pipeline will be regulated financial services. The running BFSI case study in this course is not arbitrary — it's the role's center of gravity.

The 25% hybrid policy is real but light. Travel within India and APAC is in the JD ("Travel occasionally within India and the APAC region to customer sites for workshops, technical deep dives, and relationship building"). Bangalore (tech), Delhi-NCR (BFSI + government + GCCs), Hyderabad (pharma + tech), Singapore (regional HQ for many APAC enterprises), Jakarta, Sydney, Tokyo are all on the realistic travel map.

---

## The responsibility list — decoded

The JD's responsibility list, with what each phrase actually signals:

### "Partner with account executives to deeply understand customer requirements and translate them into technical solutions, ensuring alignment between business objectives and technical implementation"

- **Signal:** the role is not solo. You operate in an account-team motion with an AE owning the commercial side. You own the technical side.
- **What this tests in interviews:** "tell me about a time you worked with a non-technical sales counterpart in a complex deal." If you've never operated in this motion, you'll need to map your closest analog (consulting partnership, pre-sales tech advisor, founder GTM partnership).

### "Serve as the primary technical advisor to enterprise customers throughout their Claude adoption journey, from discovery to initial evaluation through deployment"

- **Signal:** you own the customer's technical relationship across the entire pre-sales lifecycle: discovery → eval → deployment. Not just the first call. Not just the contract.
- **What this tests:** "walk me through how you'd guide a Tier 1 BFSI customer from first discovery call to first production use case." This is essentially the entire course in one question.

### "Support customers building with both the Claude API and Claude for Work"

- **Signal:** the JD names two distinct products. The Claude API is the developer-facing surface (Apic SDK, custom integration, evals). Claude for Work is the enterprise product (admin controls, deployment, governance, end-user productivity). You need fluency in both.
- **What this tests:** can you architect a customer rollout that uses Claude for Work for end-user productivity *and* the Claude API for custom workflows in the same enterprise — and explain when to choose which?

### "Create and deliver compelling technical content tailored to different audiences. You will need to be able to spread the gamut from technical deep dives for engineering & development teams up to business value focused conversations with executives"

- **Signal:** five-register communication is not optional. Module 11 is built around this skill.
- **What this tests:** you may be asked to "explain Claude's value to a CIO in 30 seconds," then "now explain it to the CIO's lead architect in 5 minutes," in the same interview round.

### "Guide technical architecture decisions and help customers integrate Claude effectively into their existing technology stack"

- **Signal:** integration is a first-class concern. Apic knows enterprise tech stacks are messy. You need to know how Claude fits with Salesforce / ServiceNow / Snowflake / Databricks / SAP / etc., not just standalone API patterns.
- **What this tests:** "we have an existing Salesforce + Snowflake stack. Where does Claude fit?" The answer requires fluency in both Claude *and* the surrounding ecosystem.

### "Help customers develop evaluation frameworks to measure Claude's performance for their specific use cases"

- **This phrase is load-bearing.** Notice: not "show customers our benchmarks," not "demo Claude's quality." **Develop evaluation frameworks.** Plural. Custom. Per-use-case.
- **Signal:** evals are central. Module 06 is built around this. The fit analysis flagged this as a place where few SA candidates can speak credibly — but you can (RAGAs, custom safety/hallucination evals, A/B evaluation pipelines).
- **What this tests:** "design an eval framework for the customer support agent assist use case for an Indian BFSI." A Strong Hire candidate has a structured answer in 60 seconds.

### "Identify common integration patterns and contribute insights back to our Product and Engineering teams"

- **Signal:** voice-of-customer to Product loop. You are explicitly tasked with making the product better. This is the part of the role that is *not* about today's deal.
- **What this tests:** "give me an example of a customer pattern you noticed that should change the product." You'll need to demonstrate the discipline of observing-and-aggregating across customers.

### "Travel occasionally within India and the APAC region to customer sites for workshops, technical deep dives, and relationship building"

- **Signal:** field work is real. Workshops are a deliverable, not an aside. Module 11 covers workshop design.
- **What this tests:** can you design and deliver a Claude API workshop for a customer's engineering team? An evals workshop? An executive vision workshop?

### "Maintain strong knowledge of the latest developments in LLM capabilities and implementation patterns"

- **Signal:** continuous learning is a job requirement, not a hobby. The model lineup will change. You will be expected to track and reason about it.

### "Collaborate across time zones with global teams while serving as the technical expert for the India market"

- **Signal:** you are *the* India technical voice. Apic's product/engineering teams are mostly US-based. You translate India market reality (BFSI/RBI/DPDPA/data residency/INR pricing sensitivity) to those teams.

---

## The "good fit" criteria — decoded

> **10+ years of experience in technical customer-facing roles such as Solutions Architect, Sales Engineer, or Technical Account Manager**

- The exact-match credential is "10 years as SA/SE/TAM." You don't have that exact title. You do have **14+ years in customer-embedded technical leadership** with FDE-equivalent customer engagement, multi-million-dollar engagement scoping, and 15+ executive discovery engagements. Your job in Module 02 is to make the equivalence undeniable, not deny it.

> **Experience working with enterprise customers in India or the APAC region, navigating complex buying cycles involving multiple stakeholders**

- Strong fit. Mumbai-based, India/APAC career, BFSI exposure (the global investment bank, the US financial services firm), regulated industry depth. This is one of your strongest credentials.

> **Exceptional ability to build relationships with and communicate technical concepts to diverse stakeholders to include C-suite executives, engineering & IT teams, and more**

- Strong fit. The the venture builder EIR experience (founder-side advisory) plus your 15+ executive discovery engagements at the GenAI services firm are direct evidence.

> **Strong technical communication skills with the ability to translate customer requirements between technical and business stakeholders**

- This is the core skill the role is built around. Module 11 polishes it; the rest of the course gives it depth to draw on.

> **Experience designing scalable cloud architectures and integrating with enterprise systems**

- Strong fit. Multi-cloud (GCP, AWS, Azure), Vertex AI, Bedrock, enterprise integration patterns.

> **Comfortable with Python**

- This is the bullet that matters most for credibility. "Comfortable" is doing real work. The Claude API artifact (Module 04 CodeLabs) is the proof.

> **Familiarity with common LLM frameworks and tools or a background in machine learning or data science**

- Strong fit. LangChain, LangGraph, CrewAI, Google ADK; ML/DS background going back a decade.

> **A love of teaching, mentoring, and helping others succeed**

- Strong fit, with proof: built and led a 15+ member GenAI FDE squad from zero, delivered tech talks at the US financial services firm, mentored at the venture builder.

> **Passion for thinking creatively about how to use technology in a way that is safe and beneficial, and ultimately furthers the goal of advancing safe AI systems**

- This is the mission-alignment line. Operationalized via your HIPAA / SHAP / governance-first work. Don't perform passion; demonstrate it via past architectural choices.

---

## The two unstated risks (that the fit analysis named)

The JD doesn't say it, but the fit analysis flagged two risks the interview loop will probe:

1. **The IC lateral move.** You've been leading 15-person teams. This role is IC. The risk: you sound like someone making a step back. The Strong Hire framing is that this is a step *toward* depth — going deep with the strongest customers on the most important AI being deployed today, not breadth across more team members. Module 02 builds this answer in full; Note 05 of this module previews it.
2. **Performative vs. operationalized mission alignment.** Covered in [Note 01](01-Mission-Values-Constitutional-AI.md). The cure is evidence over vocabulary.

If you don't confront these two risks directly in your prep, the interviewer will surface them at the worst possible moment.

---

## Working memory for the interview

Three things to have at instant recall about the role itself:

1. **What the role is not** — it's not delivery, not research, not generic SA. It's pre-sales architecture for Claude adoption, India/APAC scope, with eval-framework building and voice-of-customer-to-Product as core (not optional) responsibilities.
2. **Two products in scope** — Claude API and Claude for Work. Not one. Both.
3. **The Mumbai = BFSI implication** — a meaningful share of your work will be regulated financial services. Lean into it; it's where you're strongest.
