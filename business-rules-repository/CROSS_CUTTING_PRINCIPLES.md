# Cross-Cutting Principles

Patterns that recur across many modules, consolidated here as single canonical statements so they are implemented and tested **once**. Several were established as rulings in the Conflict Register; this document is their permanent home. Every module conforms to these; where a module appears to differ, the cross-cutting statement wins.

## P1 — Per-deployment isolation
All data and configuration live within one client deployment; cross-client access is structurally impossible. (Foundation for AUTHZ-002, CFG-011, REP-006.)

## P2 — Three-layer authorization on every access path
Permission + scope + ownership, deny-by-default, applied in the data layer — including reports and exports (REP-002). No access path is exempt. (AUTHZ-001/002/003.)

## P3 — Archive, never hard-delete; anonymize per retention
Operational records bearing history are archived, not destroyed; deletion is retention-driven anonymization with export-before-purge. (INST-008, STU-007, STF-008, and every Data Retention section.)

## P4 — Retention Precedence Order *(Conflict C-08)*
When obligations collide: **Legal hold > Statutory retention (financial/academic/audit) > Erasure request.** Erasure removes only the lawfully-erasable subset; statutory records persist (audit pseudonymized) until their minimums elapse. Implemented centrally; consumed by every anonymization path.

## P5 — Governed Correction Pattern *(Conflict C-07)*
No finalized record is ever edited or deleted. Corrections are new, linked entries carrying original value + new value + reason + actor; require elevated permission (and step-up/approval where specified); cascade to dependents; notify affected parties; and need a governed reopen for closed periods. (SESS-005/008, EXM-007, RES-006, FEE-003, HR-003/006, ATT-007.)

## P6 — The Reproducibility Triple
Every produced record (marksheet, invoice, payslip, decision) is reproducible against the **rule version + configuration version + workflow version** stamped on it. Records are reproduced, never re-evaluated against newer logic. (CFG-004 + WFL-002 + Rule Versioning Strategy.)

## P7 — Version-stamped, effective-dated configuration
Anything "configurable" is a registered, typed, validated definition; values are scoped (most-specific-wins), versioned, and effective-dated; consumers stamp the version used. (Doc 27 in full; consumed everywhere.)

## P8 — Governed processes via the Workflow Engine
Every approval/escalation/delegation/timeout runs on the version-pinned, SoD-enforcing Workflow Engine; sensitive steps never silently auto-approve; no instance is orphaned. (Doc 28; consumed by ADM, LEV, DSC, SCH, RES, HR, CFG.)

## P9 — Separation of Duties on consequential actions
The requester of a consequential action cannot approve it: discounts/waivers (DSC-003), payroll (HR-006), result revision (RES-006), high-impact config (CFG-007), scholarship select-vs-disburse (SCH-003). Enforced by the Workflow Engine (WFL-004) over AUTHZ-009.

## P10 — Idempotency on money and messages
Operations that must not double-apply use idempotency keys: payments (PAY-008), invoice generation (FEE-004), workflow actions (WFL-010), notification delivery (NOT-005). The same event applies exactly once.

## P11 — Minors-first data protection
Students are treated as minors by default: mandatory guardianship (STU-006), sensitive-field masking/encryption (STU-005), custody-aware access and routing (GRD-N-006, NOT-002, FILE-004), consent for optional processing (GRD-N-008), private signed-URL media (FILE-001/005), heightened audit. Never weakened without DPO sign-off.

## P12 — Audit everything consequential, immutably
Every consequential action emits an immutable, tamper-evident audit event via the transactional outbox; secrets are never stored in audit; audit access is itself audited. (Doc 29; every module's Audit Requirements.)

## P13 — Definition/instance separation
Timeless structure definitions (classes, subjects, fee structures, grading schemes) are instantiated per session/period; editing a definition never rewrites historical instances. (SESS-004, CLS-001, SUB-007, FEE-001, GRD-001.)

## P14 — Reliable async for peak events; never block the source action
Bulk/peak operations (result publish RES-009, mass invoicing, bulk notification NOT-006, large reports REP-007) run asynchronously, batched, idempotent; a downstream failure (e.g., notification) never rolls back or blocks the business action that raised it.

---

These fourteen principles, plus the ten Conflict-Register rulings, are the connective tissue that makes 30 modules and 248 rules behave as one coherent system.
