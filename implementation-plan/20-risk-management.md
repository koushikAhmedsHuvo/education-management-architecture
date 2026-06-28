# 20 — Risk Management

**Document type:** Governance (living register) · **Maps to:** Blueprint Part 21

The per-phase and cross-program risk register. A living document reviewed each phase; risks get owners, severity, and status when tracked in the team's tooling.

## Objectives
- Make every phase's technical and delivery risks visible with concrete mitigations.
- Elevate the cross-program risks that no single phase owns to program-level ownership.

## Scope
**In:** technical + delivery risks per phase with mitigation; cross-program risks (team-size, bus-factor, ops capacity, isolation economics).
**Out:** generic risk theory; risks already eliminated by architecture decisions.

## Deliverables (the register)
| Phase | Technical risk | Delivery risk | Mitigation |
|---|---|---|---|
| Project Setup | Over-built CI/monorepo | Time sunk before features | Timebox ~1 sprint; defer fleet orchestration |
| Foundation | Engines over-built speculatively | **Overruns, starves features (top risk)** | Thin walking skeleton; grow feature-driven |
| Core Platform | Scope/ownership leak (minors) | Auth complexity slips | **Isolation suite first**; rotation tested; roles as data |
| Config Engine | Resolution/rollback bugs radiate | Hardest component | Best engineer; ~100% coverage; pressure-release process |
| Workflow Engine | State races / stuck instances | Speculative generality | Minimum for Admission; idempotent transitions; sweep |
| Academic | Structure/instance conflation | Rollover bugs | Enforce separation; test rollover |
| Student Lifecycle | First workflow/file consumer gaps | Admission complexity | Engine-hardening sprint; signed-URL+scan |
| Attendance | Write hotspots / N+1 | Volume underestimated | Batching, partitioning, query-count tests |
| Examination | Mark tampering | Integrity disputes | Submit-lock; audit; step-up MFA |
| Results | Calc errors; publish load | Reputational severity | Golden-dataset; async batched publish; replica |
| Fees | Financial errors | Disputes | Immutable version-stamped invoices; effective-dating |
| HR/Staff | Assignment drives ownership | Payroll scope creep | Test assignment→ownership; defer payroll |
| Reporting | Projection lag; unsafe builder | Stale reports at peak | Monitor lag; scale workers; curated datasets |
| Notification | SMS cost runaway | Provider lock-in | Dedup/rate-limit/opt-out; adapter port |
| Portals | **Cross-child/student access** | Highest access surface | Exhaustive ownership tests; reuse components |
| Production Readiness | Late write-ceiling discovery | "Harden later" skipped | Mandatory gate; load-test publish early |
| Deployment | Manual provisioning errors | Onboarding doesn't scale | Automate early; runbook-as-code |

**Cross-program risks (program-owned):**
- Team-size vs ambition → ruthless MVP scope + ramp the team (pick one of scope/time/people to flex).
- Bus-factor → ADRs + pairing on the config and workflow engines.
- Operational capacity for 200 deployments → buy managed infrastructure.
- Small-client isolation economics → decide price floor / future pooled tier before scaling sales.

## Dependencies
Reviewed against each phase's plan at phase kickoff.

## Risks (meta)
- **The register becomes shelfware.** *Mitigation:* reviewed at every phase gate; risks carried into the team's tracker with owners and status.

## Acceptance Criteria
- Every active phase has its risks logged with an owner and a mitigation in progress.

## Exit Criteria
- No high-severity risk is unowned or unmitigated at any phase gate.
- The register is current at the close of each phase.
