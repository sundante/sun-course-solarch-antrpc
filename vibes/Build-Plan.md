# Build Plan — 6 Weeks to Strong Hire

> **Planning document — lives in `vibes/`, not in the MkDocs site.** File links below are relative to `docs/`.

> **The single page that tells you what you're building this week.** A hierarchical ToC of every file in the course, mapped to the 6-week sequence, with status flags so you can see the whole shape at a glance.

> **How to read this page.**
> - Open it on Monday morning. Find the current week's section.
> - The files listed under that week are the ones you fill in this week.
> - Status flags (Done / Outline / TBD) tell you where each file is in its lifecycle.
> - Every leaf is a link to a real file under `docs/`.

---

## Status legend

| Flag | Meaning |
|---|---|
| **Done** | File body is written at Strong-Hire grade. No further work required this cycle. |
| **Outline** | File exists with a detailed H2/H3 outline + briefs. Body still to be filled in. |
| **TBD** | File is named but does not yet exist. Will be created during the relevant week. |

Module 01 is **Done**. Modules 02–12 are **Outline** as of this scaffolding pass — every Notes, Drill, Case-Study, CodeLabs, and SystemDesigns file listed below exists with a structured outline you can fill in directly.

---

## 6-week build sequence

> The week each module's body is written is fixed by the dependency graph in `docs/index.md`. You can pull a later module forward only if its predecessors are at **Done**.

| Week | Module(s) | Why this week | Hard deliverable |
|---|---|---|---|
| **Week 1** | 02 — Personal Narrative & Role-Fit | The narrative gates the application. The artifact build (Week 2) is wasted effort if the lateral-move answer isn't already Strong Hire. | Apic-tailored `.docx` resume; clean lateral-move answer |
| **Week 2** | 03 — Customer Discovery & Pre-Sales · 04 — Claude Platform Architecture | Discovery framing makes the platform decisions defensible. CodeLabs land here because they're the largest credibility gap and need maximum soak time before applying. | Discovery checklist; **Claude API artifact published on GitHub** |
| **Week 3** | 05 — Solution Architecture Patterns · 06 — Evaluation Architecture · 07 — RAG in Regulated Enterprises | The core technical depth. Six SystemDesigns + the eval framework + two RAG architectures. This is the heaviest week. | 6 SystemDesigns + 1 BFSI eval framework + 2 RAG SystemDesigns |
| **Week 4** | 08 — Agentic & Tool-Use Architecture · 09 — Security, Privacy, Governance · 10 — Cost, Latency, Reliability | The "production-and-CISO-ready" layer. These three modules stress-test every architecture from Week 3. | 2 agentic SystemDesigns; CISO architecture review; cost/latency model for the BFSI customer |
| **Week 5** | 11 — Storytelling, Workshops, Demos | Compress the architecture depth into 5-register storytelling. You can only do this once the depth exists. | Demo script; 15-objection library; 1-page exec pitch (3 versions) |
| **Week 6** | 12 — Full Loop Simulation | Capstone. Mock the loop end-to-end with grading. | 5 graded mock-round transcripts; pre-interview-day checklist; final go/no-go |

---

## Application trigger checklist

You do **not** apply until all four are true. Each is owned by a specific module deliverable:

- [ ] **Apic-tailored `.docx` resume** — Module 02, Week 1
- [ ] **Claude API artifact on GitHub** (Python SDK + tool use + structured output + prompt caching + custom eval harness) — Module 04 CodeLabs, Week 2
- [ ] **TMAY + "Why Apic"** both at Strong Hire — Module 01 Drills (already Done) re-validated against fresh material from Module 02
- [ ] **Clean lateral-move answer** — Module 02, Week 1

The artifact alone is 4–6 hours of focused work. Plan the calendar around it.

---

## One-glance hierarchy

> Every file. Every module. Status flag. Links are relative to `docs/`.

### Module 01 — Apic Calibration · **Done**

- Overview · [INDEX](../docs/01-Apic-Calibration/INDEX.md) · **Done**
- Notes
    - [01 Mission, Values, Constitutional AI](../docs/01-Apic-Calibration/Notes/01-Mission-Values-Constitutional-AI.md) · **Done**
    - [02 Role Decoded — Applied AI Architect](../docs/01-Apic-Calibration/Notes/02-Role-Decoded-Applied-AI-Architect.md) · **Done**
    - [03 Interview Loop Map](../docs/01-Apic-Calibration/Notes/03-Interview-Loop-Map.md) · **Done**
    - [04 What Good vs Weak Looks Like](../docs/01-Apic-Calibration/Notes/04-What-Good-vs-Weak-Looks-Like.md) · **Done**
    - [05 Lateral Move and Mission Alignment](../docs/01-Apic-Calibration/Notes/05-Lateral-Move-and-Mission-Alignment.md) · **Done**
- Drills
    - [01 Tell Me About Yourself](../docs/01-Apic-Calibration/Drills/01-Tell-Me-About-Yourself.md) · **Done**
    - [02 Why Apic](../docs/01-Apic-Calibration/Drills/02-Why-Apic.md) · **Done**
    - [03 Customer Architecture Story](../docs/01-Apic-Calibration/Drills/03-Customer-Architecture-Story.md) · **Done**
- Case Study
    - [01 BFSI Customer Intro](../docs/01-Apic-Calibration/Case-Study/01-BFSI-Customer-Intro.md) · **Done**
- [Q&A Bank — 26 graded questions](../docs/01-Apic-Calibration/Interview-QA-Bank.md) · **Done**

### Module 02 — Personal Narrative & Role-Fit · **Outline** (Week 1)

- Overview · [INDEX](../docs/02-Personal-Narrative-and-Role-Fit/INDEX.md) · **Outline**
- Notes
    - [01 Career Arc Thesis](../docs/02-Personal-Narrative-and-Role-Fit/Notes/01-Career-Arc-Thesis.md) · **Outline**
    - [02 Lateral Move Answer](../docs/02-Personal-Narrative-and-Role-Fit/Notes/02-Lateral-Move-Answer.md) · **Outline**
    - [03 Mission Alignment as Evidence](../docs/02-Personal-Narrative-and-Role-Fit/Notes/03-Mission-Alignment-as-Evidence.md) · **Outline**
    - [04 Resume Rewrite Plan](../docs/02-Personal-Narrative-and-Role-Fit/Notes/04-Resume-Rewrite-Plan.md) · **Outline**
    - [05 Three-Register Intros](../docs/02-Personal-Narrative-and-Role-Fit/Notes/05-Three-Register-Intros.md) · **Outline**
- Drills
    - [01 Two-Minute Intro](../docs/02-Personal-Narrative-and-Role-Fit/Drills/01-Two-Minute-Intro.md) · **Outline**
    - [02 Why This Role Now](../docs/02-Personal-Narrative-and-Role-Fit/Drills/02-Why-This-Role-Now.md) · **Outline**
    - [03 Lateral Move Defense](../docs/02-Personal-Narrative-and-Role-Fit/Drills/03-Lateral-Move-Defense.md) · **Outline**
- Case Study
    - [01 Introducing Yourself to the BFSI Customer](../docs/02-Personal-Narrative-and-Role-Fit/Case-Study/01-Introducing-Yourself-to-the-BFSI-Customer.md) · **Outline**
- Q&A Bank · **TBD** (deferred)

### Module 03 — Customer Discovery & Pre-Sales · **Outline** (Week 2)

- Overview · [INDEX](../docs/03-Customer-Discovery-and-Pre-Sales/INDEX.md) · **Outline**
- Notes
    - [01 Discovery Call Structure](../docs/03-Customer-Discovery-and-Pre-Sales/Notes/01-Discovery-Call-Structure.md) · **Outline**
    - [02 Discovery to Production Motion](../docs/03-Customer-Discovery-and-Pre-Sales/Notes/02-Discovery-to-Production-Motion.md) · **Outline**
    - [03 Stakeholder Mapping](../docs/03-Customer-Discovery-and-Pre-Sales/Notes/03-Stakeholder-Mapping.md) · **Outline**
    - [04 When to Say Claude Is Not the Tool](../docs/03-Customer-Discovery-and-Pre-Sales/Notes/04-When-to-Say-Claude-Is-Not-The-Tool.md) · **Outline**
    - [05 Working with Account Executives](../docs/03-Customer-Discovery-and-Pre-Sales/Notes/05-Working-With-Account-Executives.md) · **Outline**
- Drills
    - [01 Discovery Call Simulation](../docs/03-Customer-Discovery-and-Pre-Sales/Drills/01-Discovery-Call-Simulation.md) · **Outline**
    - [02 Surface the Real Problem](../docs/03-Customer-Discovery-and-Pre-Sales/Drills/02-Surface-the-Real-Problem.md) · **Outline**
    - [03 Three Discovery Objections](../docs/03-Customer-Discovery-and-Pre-Sales/Drills/03-Three-Discovery-Objections.md) · **Outline**
- Case Study
    - [01 BFSI Discovery Script](../docs/03-Customer-Discovery-and-Pre-Sales/Case-Study/01-BFSI-Discovery-Script.md) · **Outline**
- Q&A Bank · **TBD** (deferred)

### Module 04 — Claude Platform Architecture · **Outline** (Week 2) · *contains the artifact build*

- Overview · [INDEX](../docs/04-Claude-Platform-Architecture/INDEX.md) · **Outline**
- Notes
    - [01 Claude API Surface](../docs/04-Claude-Platform-Architecture/Notes/01-Claude-API-Surface.md) · **Outline**
    - [02 Model Selection Matrix](../docs/04-Claude-Platform-Architecture/Notes/02-Model-Selection-Matrix.md) · **Outline**
    - [03 Prompt Caching Architecture](../docs/04-Claude-Platform-Architecture/Notes/03-Prompt-Caching-Architecture.md) · **Outline**
    - [04 Tool Use and Structured Outputs](../docs/04-Claude-Platform-Architecture/Notes/04-Tool-Use-and-Structured-Outputs.md) · **Outline**
    - [05 Deployment Surfaces](../docs/04-Claude-Platform-Architecture/Notes/05-Deployment-Surfaces.md) · **Outline**
- Drills
    - [01 30-Second Model Selection](../docs/04-Claude-Platform-Architecture/Drills/01-30-Second-Model-Selection.md) · **Outline**
    - [02 Whiteboard Prompt Cache ROI](../docs/04-Claude-Platform-Architecture/Drills/02-Whiteboard-Prompt-Cache-ROI.md) · **Outline**
    - [03 Bedrock vs Vertex vs First-Party](../docs/04-Claude-Platform-Architecture/Drills/03-Bedrock-vs-Vertex-vs-First-Party.md) · **Outline**
- Case Study
    - [01 Claude Platform Decisions for BFSI](../docs/04-Claude-Platform-Architecture/Case-Study/01-Claude-Platform-Decisions-for-BFSI.md) · **Outline**
- CodeLabs *(the GitHub artifact)*
    - [01 First Claude Call](../docs/04-Claude-Platform-Architecture/CodeLabs/01-First-Claude-Call.md) · **Outline**
    - [02 Tool Use](../docs/04-Claude-Platform-Architecture/CodeLabs/02-Tool-Use.md) · **Outline**
    - [03 Structured Output](../docs/04-Claude-Platform-Architecture/CodeLabs/03-Structured-Output.md) · **Outline**
    - [04 Prompt Caching](../docs/04-Claude-Platform-Architecture/CodeLabs/04-Prompt-Caching.md) · **Outline**
    - [05 Custom Eval Harness](../docs/04-Claude-Platform-Architecture/CodeLabs/05-Custom-Eval-Harness.md) · **Outline**
- Q&A Bank · **TBD** (deferred)

### Module 05 — Solution Architecture Patterns · **Outline** (Week 3)

- Overview · [INDEX](../docs/05-Solution-Architecture-Patterns/INDEX.md) · **Outline**
- Notes
    - [01 Reference Architecture Anatomy](../docs/05-Solution-Architecture-Patterns/Notes/01-Reference-Architecture-Anatomy.md) · **Outline**
    - [02 Integration Boundary Doctrine](../docs/05-Solution-Architecture-Patterns/Notes/02-Integration-Boundary-Doctrine.md) · **Outline**
    - [03 Failure Modes Catalog](../docs/05-Solution-Architecture-Patterns/Notes/03-Failure-Modes-Catalog.md) · **Outline**
    - [04 Cost & Latency Modeling Primer](../docs/05-Solution-Architecture-Patterns/Notes/04-Cost-and-Latency-Modeling-Primer.md) · **Outline**
    - [05 Rollout Patterns](../docs/05-Solution-Architecture-Patterns/Notes/05-Rollout-Patterns.md) · **Outline**
- Drills
    - [01 Whiteboard Support Agent Assist](../docs/05-Solution-Architecture-Patterns/Drills/01-Whiteboard-Support-Agent-Assist.md) · **Outline**
    - [02 Cross-Architecture Pattern Recognition](../docs/05-Solution-Architecture-Patterns/Drills/02-Cross-Architecture-Pattern-Recognition.md) · **Outline**
    - [03 Trade-off Defense](../docs/05-Solution-Architecture-Patterns/Drills/03-Trade-off-Defense.md) · **Outline**
- Case Study
    - [01 Six Use-Case Meta-Pattern](../docs/05-Solution-Architecture-Patterns/Case-Study/01-Six-Use-Case-Meta-Pattern.md) · **Outline**
- SystemDesigns *(six BFSI use cases)*
    - [01 Customer Support Agent Assist](../docs/05-Solution-Architecture-Patterns/SystemDesigns/01-Customer-Support-Agent-Assist.md) · **Outline**
    - [02 Internal SOP & Policy Search](../docs/05-Solution-Architecture-Patterns/SystemDesigns/02-Internal-SOP-Policy-Search.md) · **Outline**
    - [03 Relationship Manager Copilot](../docs/05-Solution-Architecture-Patterns/SystemDesigns/03-Relationship-Manager-Copilot.md) · **Outline**
    - [04 Compliance Document Review](../docs/05-Solution-Architecture-Patterns/SystemDesigns/04-Compliance-Document-Review.md) · **Outline**
    - [05 Developer Productivity with Claude Code](../docs/05-Solution-Architecture-Patterns/SystemDesigns/05-Developer-Productivity-with-Claude-Code.md) · **Outline**
    - [06 Executive Analytics Assistant](../docs/05-Solution-Architecture-Patterns/SystemDesigns/06-Executive-Analytics-Assistant.md) · **Outline**
- Q&A Bank · **TBD** (deferred)

### Module 06 — Evaluation Architecture · **Outline** (Week 3)

- Overview · [INDEX](../docs/06-Evaluation-Architecture/INDEX.md) · **Outline**
- Notes
    - [01 Evals as Architecture Concern](../docs/06-Evaluation-Architecture/Notes/01-Evals-as-Architecture-Concern.md) · **Outline**
    - [02 Eval Taxonomy](../docs/06-Evaluation-Architecture/Notes/02-Eval-Taxonomy.md) · **Outline**
    - [03 LLM-as-Judge Design](../docs/06-Evaluation-Architecture/Notes/03-LLM-as-Judge-Design.md) · **Outline**
    - [04 Online Monitoring & Regression Gates](../docs/06-Evaluation-Architecture/Notes/04-Online-Monitoring-and-Regression-Gates.md) · **Outline**
    - [05 Eval Kickoff Script](../docs/06-Evaluation-Architecture/Notes/05-Eval-Kickoff-Script.md) · **Outline**
- Drills
    - [01 Design an Eval on the Whiteboard](../docs/06-Evaluation-Architecture/Drills/01-Design-an-Eval-on-the-Whiteboard.md) · **Outline**
    - [02 Defend an Acceptance Threshold](../docs/06-Evaluation-Architecture/Drills/02-Defend-an-Acceptance-Threshold.md) · **Outline**
    - [03 Translate a Customer KPI into an Eval](../docs/06-Evaluation-Architecture/Drills/03-Translate-a-Customer-KPI-into-an-Eval.md) · **Outline**
- Case Study
    - [01 Eval Framework for BFSI Compliance Review](../docs/06-Evaluation-Architecture/Case-Study/01-Eval-Framework-for-BFSI-Compliance-Review.md) · **Outline**
- Q&A Bank · **TBD** (deferred)

### Module 07 — RAG in Regulated Enterprises · **Outline** (Week 3)

- Overview · [INDEX](../docs/07-RAG-in-Regulated-Enterprises/INDEX.md) · **Outline**
- Notes
    - [01 Hybrid Retrieval Patterns](../docs/07-RAG-in-Regulated-Enterprises/Notes/01-Hybrid-Retrieval-Patterns.md) · **Outline**
    - [02 Embedding Strategies & Residency](../docs/07-RAG-in-Regulated-Enterprises/Notes/02-Embedding-Strategies-and-Residency.md) · **Outline**
    - [03 Chunking & Citation Fidelity](../docs/07-RAG-in-Regulated-Enterprises/Notes/03-Chunking-and-Citation-Fidelity.md) · **Outline**
    - [04 RAG Security Boundaries](../docs/07-RAG-in-Regulated-Enterprises/Notes/04-RAG-Security-Boundaries.md) · **Outline**
    - [05 Hallucination Control via Retrieval Design](../docs/07-RAG-in-Regulated-Enterprises/Notes/05-Hallucination-Control-via-Retrieval-Design.md) · **Outline**
- Drills
    - [01 RAG vs Fine-Tune vs Long-Context](../docs/07-RAG-in-Regulated-Enterprises/Drills/01-RAG-vs-Fine-Tune-vs-Long-Context.md) · **Outline**
    - [02 Whiteboard RAG with RBAC](../docs/07-RAG-in-Regulated-Enterprises/Drills/02-Whiteboard-RAG-with-RBAC.md) · **Outline**
    - [03 Faithfulness Eval Design](../docs/07-RAG-in-Regulated-Enterprises/Drills/03-Faithfulness-Eval-Design.md) · **Outline**
- Case Study
    - [01 BFSI RAG Meta-Pattern](../docs/07-RAG-in-Regulated-Enterprises/Case-Study/01-BFSI-RAG-Meta-Pattern.md) · **Outline**
- SystemDesigns
    - [01 BFSI Internal SOP Search](../docs/07-RAG-in-Regulated-Enterprises/SystemDesigns/01-BFSI-Internal-SOP-Search.md) · **Outline**
    - [02 BFSI Compliance Document Review RAG](../docs/07-RAG-in-Regulated-Enterprises/SystemDesigns/02-BFSI-Compliance-Document-Review-RAG.md) · **Outline**
- Q&A Bank · **TBD** (deferred)

### Module 08 — Agentic & Tool-Use Architecture · **Outline** (Week 4)

- Overview · [INDEX](../docs/08-Agentic-and-Tool-Use-Architecture/INDEX.md) · **Outline**
- Notes
    - [01 Tool Schema Design Doctrine](../docs/08-Agentic-and-Tool-Use-Architecture/Notes/01-Tool-Schema-Design-Doctrine.md) · **Outline**
    - [02 Agentic Loop Patterns](../docs/08-Agentic-and-Tool-Use-Architecture/Notes/02-Agentic-Loop-Patterns.md) · **Outline**
    - [03 State and Memory](../docs/08-Agentic-and-Tool-Use-Architecture/Notes/03-State-and-Memory.md) · **Outline**
    - [04 MCP Integration](../docs/08-Agentic-and-Tool-Use-Architecture/Notes/04-MCP-Integration.md) · **Outline**
    - [05 Human-in-the-Loop & Decision Rights](../docs/08-Agentic-and-Tool-Use-Architecture/Notes/05-Human-in-the-Loop-and-Decision-Rights.md) · **Outline**
- Drills
    - [01 Should This Be an Agent](../docs/08-Agentic-and-Tool-Use-Architecture/Drills/01-Should-This-Be-an-Agent.md) · **Outline**
    - [02 Design a Safe Tool Schema](../docs/08-Agentic-and-Tool-Use-Architecture/Drills/02-Design-a-Safe-Tool-Schema.md) · **Outline**
    - [03 Multi-Agent Coordination Defense](../docs/08-Agentic-and-Tool-Use-Architecture/Drills/03-Multi-Agent-Coordination-Defense.md) · **Outline**
- Case Study
    - [01 BFSI Agentic Workflow Walkthrough](../docs/08-Agentic-and-Tool-Use-Architecture/Case-Study/01-BFSI-Agentic-Workflow-Walkthrough.md) · **Outline**
- SystemDesigns
    - [01 BFSI RM Copilot](../docs/08-Agentic-and-Tool-Use-Architecture/SystemDesigns/01-BFSI-RM-Copilot.md) · **Outline**
    - [02 BFSI Compliance Review Agentic with HITL](../docs/08-Agentic-and-Tool-Use-Architecture/SystemDesigns/02-BFSI-Compliance-Review-Agentic-with-HITL.md) · **Outline**
- Q&A Bank · **TBD** (deferred)

### Module 09 — Security, Privacy, Governance · **Outline** (Week 4)

- Overview · [INDEX](../docs/09-Security-Privacy-Governance/INDEX.md) · **Outline**
- Notes
    - [01 Prompt Injection Classes & Defenses](../docs/09-Security-Privacy-Governance/Notes/01-Prompt-Injection-Classes-and-Defenses.md) · **Outline**
    - [02 Data Leakage Surfaces](../docs/09-Security-Privacy-Governance/Notes/02-Data-Leakage-Surfaces.md) · **Outline**
    - [03 PII / DPDPA / RBI Posture](../docs/09-Security-Privacy-Governance/Notes/03-PII-DPDPA-RBI-Posture.md) · **Outline**
    - [04 Audit Logging & Decision Rights](../docs/09-Security-Privacy-Governance/Notes/04-Audit-Logging-and-Decision-Rights.md) · **Outline**
    - [05 CISO Trust-Building & Honest Hallucination](../docs/09-Security-Privacy-Governance/Notes/05-CISO-Trust-Building-and-Honest-Hallucination.md) · **Outline**
- Drills
    - [01 CISO Architecture Review Simulation](../docs/09-Security-Privacy-Governance/Drills/01-CISO-Architecture-Review-Simulation.md) · **Outline**
    - [02 Prompt Injection Tabletop](../docs/09-Security-Privacy-Governance/Drills/02-Prompt-Injection-Tabletop.md) · **Outline**
    - [03 Should This Be Claude-Driven](../docs/09-Security-Privacy-Governance/Drills/03-Should-This-Be-Claude-Driven.md) · **Outline**
- Case Study
    - [01 BFSI CISO Architecture Review](../docs/09-Security-Privacy-Governance/Case-Study/01-BFSI-CISO-Architecture-Review.md) · **Outline**
- Q&A Bank · **TBD** (deferred)

### Module 10 — Cost, Latency, Reliability · **Outline** (Week 4)

- Overview · [INDEX](../docs/10-Cost-Latency-Reliability/INDEX.md) · **Outline**
- Notes
    - [01 Cost Architecture](../docs/10-Cost-Latency-Reliability/Notes/01-Cost-Architecture.md) · **Outline**
    - [02 Latency Architecture](../docs/10-Cost-Latency-Reliability/Notes/02-Latency-Architecture.md) · **Outline**
    - [03 Reliability Patterns](../docs/10-Cost-Latency-Reliability/Notes/03-Reliability-Patterns.md) · **Outline**
    - [04 Observability](../docs/10-Cost-Latency-Reliability/Notes/04-Observability.md) · **Outline**
    - [05 Production Readiness Review](../docs/10-Cost-Latency-Reliability/Notes/05-Production-Readiness-Review.md) · **Outline**
- Drills
    - [01 Estimate Tokens from Business Volume](../docs/10-Cost-Latency-Reliability/Drills/01-Estimate-Tokens-from-Business-Volume.md) · **Outline**
    - [02 Defend a Latency Budget](../docs/10-Cost-Latency-Reliability/Drills/02-Defend-a-Latency-Budget.md) · **Outline**
    - [03 Blast Radius Conversation](../docs/10-Cost-Latency-Reliability/Drills/03-Blast-Radius-Conversation.md) · **Outline**
- Case Study
    - [01 BFSI Cost & Latency Model](../docs/10-Cost-Latency-Reliability/Case-Study/01-BFSI-Cost-and-Latency-Model.md) · **Outline**
- Q&A Bank · **TBD** (deferred)

### Module 11 — Storytelling, Workshops, Demos · **Outline** (Week 5)

- Overview · [INDEX](../docs/11-Storytelling-Workshops-Demos/INDEX.md) · **Outline**
- Notes
    - [01 Five Registers Doctrine](../docs/11-Storytelling-Workshops-Demos/Notes/01-Five-Registers-Doctrine.md) · **Outline**
    - [02 Audience-Tuned Versions](../docs/11-Storytelling-Workshops-Demos/Notes/02-Audience-Tuned-Versions.md) · **Outline**
    - [03 Workshop Motion](../docs/11-Storytelling-Workshops-Demos/Notes/03-Workshop-Motion.md) · **Outline**
    - [04 Demo Discipline](../docs/11-Storytelling-Workshops-Demos/Notes/04-Demo-Discipline.md) · **Outline**
    - [05 Objection Handling Library](../docs/11-Storytelling-Workshops-Demos/Notes/05-Objection-Handling-Library.md) · **Outline**
- Drills
    - [01 Same Story, Five Registers](../docs/11-Storytelling-Workshops-Demos/Drills/01-Same-Story-Five-Registers.md) · **Outline**
    - [02 Live Demo Recovery](../docs/11-Storytelling-Workshops-Demos/Drills/02-Live-Demo-Recovery.md) · **Outline**
    - [03 Objection Volley](../docs/11-Storytelling-Workshops-Demos/Drills/03-Objection-Volley.md) · **Outline**
- Case Study
    - [01 BFSI Executive Pitch & Workshop](../docs/11-Storytelling-Workshops-Demos/Case-Study/01-BFSI-Executive-Pitch-and-Workshop.md) · **Outline**
- Q&A Bank · **TBD** (deferred)

### Module 12 — Full Loop Simulation · **Outline** (Week 6)

- Overview · [INDEX](../docs/12-Full-Loop-Simulation/INDEX.md) · **Outline**
- Notes
    - [01 Apic Loop Anatomy](../docs/12-Full-Loop-Simulation/Notes/01-Apic-Loop-Anatomy.md) · **Outline**
    - [02 The Project Deep-Dive Round](../docs/12-Full-Loop-Simulation/Notes/02-The-Project-Deep-Dive-Round.md) · **Outline**
    - [03 Behavioral & Values Probing](../docs/12-Full-Loop-Simulation/Notes/03-Behavioral-and-Values-Probing.md) · **Outline**
    - [04 Recovery Patterns](../docs/12-Full-Loop-Simulation/Notes/04-Recovery-Patterns.md) · **Outline**
    - [05 Pre-Interview-Day Checklist](../docs/12-Full-Loop-Simulation/Notes/05-Pre-Interview-Day-Checklist.md) · **Outline**
- Drills
    - [01 Mock Recruiter Screen](../docs/12-Full-Loop-Simulation/Drills/01-Mock-Recruiter-Screen.md) · **Outline**
    - [02 Mock HM Screen](../docs/12-Full-Loop-Simulation/Drills/02-Mock-HM-Screen.md) · **Outline**
    - [03 Mock Onsite Project Deep-Dive](../docs/12-Full-Loop-Simulation/Drills/03-Mock-Onsite-Project-Deep-Dive.md) · **Outline**
- Case Study
    - [01 Capstone BFSI Mock Loop](../docs/12-Full-Loop-Simulation/Case-Study/01-Capstone-BFSI-Mock-Loop.md) · **Outline**
- Q&A Bank · **TBD** (deferred)

---

## Effort estimates (hours, with calibration band)

> Tier S = ≤1 hr · Tier M = 1–3 hr · Tier L = 3–6 hr · Tier XL = 6+ hr (only the artifact and the eval framework hit XL).
> Numbers assume **filling in an outlined file**, not designing one cold.

| Week | Hours (low) | Hours (high) | Notes |
|---|---|---|---|
| 1 | 8 | 14 | Lighter — narrative is mostly in your head; resume rewrite is the heaviest item. |
| 2 | 18 | 26 | **Artifact dominates.** 4–6 hr just for the GitHub artifact + repo polish. |
| 3 | 22 | 32 | Heaviest week. 6 SystemDesigns alone are 18+ hr. |
| 4 | 16 | 24 | CISO + cost/latency model are the spikes. |
| 5 | 8 | 14 | Storytelling compresses things you've already designed. |
| 6 | 10 | 16 | Mock rounds + post-mortems. |
| **Total** | **82** | **126** | Roughly **2–3 evening hours per day, 6 days/week**. |

If you fall behind by a week, the natural cut is **Module 11** (storytelling can be shortened) or **Module 06's Eval Kickoff Script** — never the artifact (Module 04) or CISO posture (Module 09).

---

## How to use this page through the week

1. **Monday morning:** open this page. Find the current week's section. Scroll to the file-by-file list under that week.
2. **For each file:** click in, read the outline, fill in the body. Use the smaller models for body text; come back to Opus only for the rubric calibrations and the BFSI case-study chapters that thread the running customer.
3. **End of week:** flip the status flags from **Outline** to **Done** (edit this page) and re-read the week's deliverable column to confirm you actually shipped it.
4. **End of week 6:** every leaf is **Done**. The four-item Application Trigger Checklist at the top should be ticked. Apply.

---

## See also

- [Home](../docs/index.md) — course intro + 12-module table
- [Drill Tracker](../docs/Drill-Tracker.md) — graded log of every drill run, newest first
- [Knowledge Check Index](../docs/Interview-Questions/INDEX.md) — cross-module Q&A aggregator (per-module banks land Week 6)
