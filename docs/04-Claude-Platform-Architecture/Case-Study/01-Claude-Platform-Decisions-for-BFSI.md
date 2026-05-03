# 04 · Case Study 01 — Claude Platform Decisions for BFSI

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The platform-decision worksheet for the running BFSI customer — model selection per use case, caching plan, deployment surface, integration shape, MCP yes/no — landed as concrete commitments after Module 03's discovery.

> **What this file is NOT.** A pitch deck. Not the SystemDesigns themselves (those land in Module 05).

---

## Customer recap (post-discovery)

- AWS-shop (~80% workloads on AWS Mumbai), strategic GCP relationship (~15%) on data-platform side.
- Six candidate use cases. Discovery surfaced internal SOP search as the cleanest first-pilot (lowest blast radius, clearest eval).
- Residency: Mumbai-only (RBI-driven).
- CISO scar from previous vendor's hallucination during a demo.
- Timeline: "show value" demo target 4 weeks; production-ready bar set 12 weeks after demo.

→ Discovery output: [Module 03 Case Study](../../03-Customer-Discovery-and-Pre-Sales/Case-Study/01-BFSI-Discovery-Script.md).

---

## Decision 1 — Deployment surface

**Choice:** **Bedrock Mumbai** for API workloads + **Claude for Work** for end-user productivity.

**Rationale:**
- Residency anchored in ap-south-1.
- IAM-native against existing AWS controls (KMS, IAM roles, CloudTrail).
- Procurement clean: rolls into existing AWS spend.
- Claude for Work used by knowledge workers (CDO's analysts, compliance team) for productivity surfaces; doesn't replace the API-driven workflows.

**Trade-off accepted:** feature-lag possible (newest capabilities ship first-party first). Mitigation: eval framework designed for model substitution; new capabilities can be evaluated and rolled in within a sprint when they reach Bedrock.

---

## Decision 2 — Model selection per use case

| Use case | Model (hot path) | Tier components | Why this shape |
|---|---|---|---|
| Internal SOP search (pilot) | Sonnet 4.6 | Haiku 4.5 query rewriter | RAG over 50K docs; Sonnet is fit, Haiku rewriter cuts cost on routing |
| Customer support agent assist | Sonnet 4.6 | Haiku triage classifier | High volume; tier saves ~70% vs naive-Opus |
| RM copilot | Sonnet 4.6 | Opus 4.7 on hard plans, Haiku for query routing | Multi-turn agentic; Opus only when planning depth justifies |
| Compliance doc review | Opus 4.7 (hot path) | Sonnet first-pass draft | Failure mode is regulatory; capability dominates cost |
| Developer productivity (Claude Code) | Sonnet 4.6 default | Opus on hardest tasks | Code reasoning needs Sonnet; hardest tasks promoted |
| Exec analytics | Sonnet 4.6 | Haiku query routing | NL→SQL with safety guardrails; Sonnet sufficient |

→ Full matrix: [Note 02 — Model Selection Matrix](../Notes/02-Model-Selection-Matrix.md).

---

## Decision 3 — Caching plan

| Use case | Cached zone | Expected hit-rate |
|---|---|---|
| SOP search | Tool defs + retrieval rerank prompt | 90%+ |
| Support agent assist | Persona + 30K-token policy bundle | 85–92% |
| RM copilot | Persona + customer-360 schema + tool defs | 80–88% |
| Compliance review | Reg-framework reference + drafting style guide | 70–85% (per regulator) |
| Dev productivity | Defaults from Claude Code | — |
| Exec analytics | Schema + safety guardrails | 90%+ |

**Architectural moves:**
- Sticky-session worker pinning for support agent assist (lifts hit-rate ~10pts).
- Pre-warm at 8:55 AM IST for shift-start traffic.
- Refuse to put dynamic IDs in cached zones.
- Cache state cleared in eval runs (deterministic latency + cost reporting).

→ [Note 03 — Prompt Caching Architecture](../Notes/03-Prompt-Caching-Architecture.md).

---

## Decision 4 — Tool use vs structured outputs vs RAG

| Use case | Pattern |
|---|---|
| SOP search | RAG with citation; structured output for citation list |
| Support agent assist | Tool use (lookup customer state) + RAG (policy retrieval) |
| RM copilot | Tool use (multi-system fetch) + structured output for action proposals |
| Compliance review | Structured output (gap analysis schema) + tool use (regulator-bundle retrieval) |
| Dev productivity | Tool use (file/codebase tools) — handled by Claude Code defaults |
| Exec analytics | Tool use (NL→SQL→Snowflake fetch) + structured output for response shape |

→ [Note 04 — Tool Use and Structured Outputs](../Notes/04-Tool-Use-and-Structured-Outputs.md).

---

## Decision 5 — MCP yes/no

**Choice:** **Yes, MCP** — but staged.

- **Stage 1** (pilot): MCP server for the SOP search corpus + the customer-360 fetch.
- **Stage 2:** add MCP servers for Salesforce, ServiceNow, and Snowflake — reusable across the support agent assist + RM copilot + exec analytics.
- **Stage 3:** all 12 internal tools accessible via MCP.

**Rationale:** the customer's tool count (12+ enterprise systems they want Claude to talk to, eventually) justifies MCP. Without MCP, they'd be building 12 bespoke tool-use integrations. MCP also makes the same tools available across Claude Code, Claude Desktop, and the custom agents.

→ [Module 08 Note 04 — MCP Integration](../../08-Agentic-and-Tool-Use-Architecture/Notes/04-MCP-Integration.md).

---

## What's deliberately deferred

- **Architecture HLDs** — those land in Module 05.
- **Eval framework design** — Module 06.
- **CISO architecture review** — Module 09.
- **Cost model with absolute numbers** — Module 10.

This Case Study only commits to the *platform-level decisions*; the architecture-level work happens in subsequent modules.

---

## How this case study sets up subsequent modules

- Module 05: each use case becomes a SystemDesign, calibrated to the model + caching + deployment decisions above.
- Module 06: eval framework designed to support the chosen tier-architectures.
- Module 07: RAG architecture for SOP search + compliance review.
- Module 08: agentic architecture for RM copilot + compliance review.
- Module 09: CISO architecture review on the chosen deployment surface.
- Module 10: cost-and-latency model with the caching plan baked in.
- Module 11: 3-version exec pitch (CIO / CDO / CISO) using the decisions above as evidence.

---

## Cross-references
- Predecessor: [Module 03 Case Study](../../03-Customer-Discovery-and-Pre-Sales/Case-Study/01-BFSI-Discovery-Script.md).
- Successor: [Module 05 SystemDesigns](../../05-Solution-Architecture-Patterns/SystemDesigns/).
- Notes [01](../Notes/01-Claude-API-Surface.md), [02](../Notes/02-Model-Selection-Matrix.md), [03](../Notes/03-Prompt-Caching-Architecture.md), [04](../Notes/04-Tool-Use-and-Structured-Outputs.md), [05](../Notes/05-Deployment-Surfaces.md).

## Strong-Hire bar for this file
- All 5 decisions made and defended cold, calibrated to the customer's posture.
- Trade-offs surfaced and mitigated (not hidden).
- Caching plan per use case, with hit-rate estimates and architectural moves.
- MCP staged, not pitched as a day-1 commit.
- Subsequent modules' surface area named explicitly (this case study sets the stage).
