# 08 · System Design — BFSI RM Copilot

> **Status: Outline.** Body fills in the scheduled course week. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** A structured system design scaffold for the Apic Applied AI Architect prep course.

## Purpose

- Design an agentic RM copilot with customer 360, portfolio context, SOP search, proposal drafting, and CRM logging.
- Include tool catalog, state model, approval gates, evals, security boundaries, and latency/cost controls.
- Separate drafting from execution so the RM remains accountable.

## Fill-in structure

- **Context:** customer, stakeholder, risk, and business goal.
- **Architecture/eval decision:** the concrete design, rubric, or conversation pattern this file teaches.
- **Trade-offs:** the rejected alternatives and the condition under which they become reasonable.
- **BFSI constraints:** data residency, RBAC, PII handling, auditability, latency, cost, and human approval where relevant.
- **Strong-Hire answer:** the crisp version you can say in an interview without reading notes.

## Strong-Hire bar

- Explains the decision in customer language before implementation language.
- Names measurable success criteria and failure modes.
- Shows safety, governance, and eval thinking as part of the architecture, not as afterthoughts.
- Can be defended to an engineering lead, CISO, and executive sponsor.

