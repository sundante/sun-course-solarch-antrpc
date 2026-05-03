# 03 · Note 02 — Discovery → Eval → POC → Production

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The four-stage motion that moves a regulated customer from first-call to production-deployment, with the architect's role at each stage and the gates between them.

> **What this file is NOT.** A sales pipeline. Not a project-management Gantt. The architect's view of the path.

---

## The four stages

```
Discovery ──► Eval Design ──► POC (eval-gated) ──► Production (rollout-gated)
```

The gates between stages are the load-bearing part. Each gate is *evidence*, not approval.

---

## Stage 1 — Discovery (covered in Note 01)

- **Output:** a 1-page problem definition + constraint list, signed by both sides.
- **Gate to next stage:** the architect can write the eval kickoff brief without further customer input.

## Stage 2 — Eval Design

- **Output:** the eval framework — task definitions, golden answers, rubrics, acceptance thresholds, regression gates.
- **Architect's role:** lead the eval design with the customer's domain experts. The customer brings the answer key.
- **Gate to next stage:** the eval framework is *runnable* against any candidate model with a deterministic score.
- **Why eval before POC:** without evals, the POC has no go/no-go criterion. POCs without evals turn into demos that "feel good" then stall in production review. → See [Module 06](../../06-Evaluation-Architecture/INDEX.md).

## Stage 3 — POC

- **Output:** a runnable system that hits the eval framework on the agreed acceptance threshold.
- **Architect's role:** scope tight, scope reversible, scope eval-gated.
- **Gate to next stage:** acceptance threshold met *and* CISO-readable risk posture documented.
- **POC anti-pattern:** demo-driven scope. If the POC is shaped by what looks impressive, not by what passes evals, it will fail production review.

## Stage 4 — Production rollout

- **Output:** a phased rollout (shadow → assist → autonomous) with reversibility at each stage.
- **Architect's role:** co-design the rollout with the customer's risk owner; design the regression gates that catch drift in production.
- **Gate to "done":** SLOs met, regression gates green for N consecutive weeks, CISO post-deployment review signed.

---

## When to push for smaller scope

The single highest-leverage move in this motion is *getting the customer to scope down*. Indicators:

- The customer's eval framework would take 4+ weeks to design.
- The first-stage use case touches more than 2 source systems.
- The customer's "must-have" includes a capability Claude can't yet do reliably.

Strong-Hire phrasing: *"Six use cases at a CISO-trusted bar > twelve at a demo bar. Let's pick the one with the cleanest eval and the lowest blast radius. We'll re-scope after."*

## When to suggest Claude is not the right tool

Apic interviewers grade *highly* on this judgment. → See [Note 04](04-When-to-Say-Claude-Is-Not-The-Tool.md).

## When to bring engineering in

Earlier than call 3 = signal of dependency. Later than POC kickoff = signal of avoidance. The right zone is *late discovery, early eval design* — when the residency / MCP / capability unknowns are about to bind.

---

## The architect's deliverables across the motion

| Stage | Deliverable | Owner |
|---|---|---|
| Discovery | 1-page problem + constraint definition | Architect drafts, customer signs |
| Eval design | Eval framework + acceptance thresholds | Architect leads, customer's domain SMEs co-design |
| POC | Runnable system + eval scorecard | Engineering builds, architect designs |
| Production | Phased rollout plan + regression gates | Architect co-designs with risk owner |

---

## Cross-references
- Predecessor: [Note 01 — Discovery Call Structure](01-Discovery-Call-Structure.md).
- Successor: [Note 03 — Stakeholder Mapping](03-Stakeholder-Mapping.md).
- Module 06: [Evaluation Architecture](../../06-Evaluation-Architecture/INDEX.md) — eval design depth.
- Module 09: [Security, Privacy, Governance](../../09-Security-Privacy-Governance/INDEX.md) — CISO-readable posture.

## Strong-Hire bar for this file
- Each of the 4 gates is *evidence*, not *approval*.
- Eval-before-POC sequencing is non-negotiable.
- "Push to smaller scope" is reflex, not conditional.
- Architect's deliverables at each stage are concrete, not narrative.
