# Development Readiness Review

> Final assessment of the Module Functional Specification Repository for enterprise ERP development. Covers traceability, gaps, overlaps, missing requirements, and implementation risks. Findings are derived from automated extraction across all 30 module specifications plus the approved Architecture Blueprint, Business Rules Catalog (248 rules), and Use Case Repository (512 use cases).

---

## 1. Readiness Summary

| Dimension | Result | Status |
|---|---|---|
| Modules specified | 30 / 30 | ✅ Complete |
| Section completeness | 22 / 22 sections in every module | ✅ Complete |
| Functional requirements | 270 (local FR-IDs per module) | ✅ |
| Business-rule coverage | 248 / 248 cited (100%) | ✅ No orphan rules |
| Use-case coverage | 512 / 512 cited (100%) | ✅ No orphan use cases |
| Home-citation completeness | Every module cites all its own home rules + use cases | ✅ |
| Permissions declared | 265 (namespaced `module.entity.action`) | ✅ |
| API endpoint purposes | 317 (method + actor) | ✅ Purpose-level |
| Screens inventoried | 416 across 30 modules | ✅ |
| Approval authorities (SoD) | 24 distinct `.approve` permissions | ✅ |
| Reference integrity | 0 broken rule/UC references | ✅ |

**Overall: the repository is implementation-ready at the functional-specification layer.** Every approved rule and use case maps to at least one module, every module is complete to the 22-section template, and all cross-references resolve. The items below are the honest residue — they are not blockers to starting development, but several must be resolved before specific modules reach code-complete.

---

## 2. Gaps

### 2.1 Resolved during this review
- **7 use-case citation gaps** in the two earliest-built modules (Authentication, Authorization) — self-service session/password, user/role search, security/login reporting, role-set import, and role-change-approval-workflow use cases were implemented in the spec bodies (screens, permissions, workflow sections) but not enumerated in §13. **Patched** to explicit citations; traceability is now 100%. *Lesson: the first modules authored predate the citation-discipline conventions refined later; this is the expected place for such drift.*

### 2.2 Deliberate scope boundaries (not defects — flagged for product-owner decision)
- **Re-evaluation request intake (RES / C-02).** The specs cover the result *revision mechanism* (RES-006, version-pinned, SoD) and the disclosed re-evaluation *direction policy* (C-02), but **no applicant-facing "request a recheck" intake flow** exists, because no approved business rule defines one. Kept as a Future Enhancement in Result Processing (15). **Decision needed:** if re-evaluation requests are first-class for launch, a new use case + rule must be authorized before module 15 is code-complete.
- **API request/response contracts.** §14 of every module specifies endpoint *purpose, method, and actor only* — by design at the functional layer. The field-level request/response schemas, status codes, and error envelopes are **not yet specified**. This is the single largest "known unspecified" surface (317 endpoints). See §4 (Missing Requirements) and the recommended next artifact.

### 2.3 No coverage gaps in rules or use cases
Automated sweep confirms **zero** approved rules or use cases without an implementing module. There are no "phantom" requirements and no silently dropped catalog items.

---

## 3. Overlaps

All identified overlaps are **intentional cross-cutting boundaries** that were explicitly resolved in the Business Rules conflict register (C-01…C-10) and are documented in the relevant specs' §1 Out-of-Scope. They are listed here so the build teams treat them as *coordination points*, not duplication to be eliminated.

| Overlap | Modules | Resolution | Risk if mishandled |
|---|---|---|---|
| Section capacity vs class aggregate capacity | Section (12), Class (11), Enrollment (08) | **C-01**: section hard cap is authoritative at enrollment; class aggregate is advisory | Over-enrollment if aggregate treated as the gate |
| Waiver semantics vs non-waivable classification | Discount (19), Fee (17) | **C-03**: Discount owns waiver semantics; Fee classifies non-waivable heads | Statutory charges waived in error |
| Student-leave workflow vs attendance reflection | Leave (24), Attendance (10) | **C-04**: Leave owns the workflow; Attendance only reflects as excused | Duplicate/contradictory leave logic |
| Ownership re-resolution on reassignment | Teacher (21), Subject (13), Section (12) | **C-05**: live ownership ends immediately; history attributed | Marks/attendance mis-attributed across changes |
| Config immutable-after-use lock | Configuration (27), Institute (03) | **C-06**: single lock mechanism in Config | Two divergent lock implementations |
| Retention precedence | Audit (29), Student (07), File (30) | **C-08**: legal-hold > statutory > erasure | Unlawful erasure or failure to erase |
| **Refund decision vs execution** | Fee (17), Payment (18) | Fee governs the refund decision; Payment executes the reversal (PAY-007) | **Refund logic built twice or split incorrectly — confirm with finance lead** |

The refund split is the one overlap worth an explicit sign-off before the Finance squad starts (it spans two modules and two teams).

---

## 4. Missing Requirements (to specify before/at the relevant build stage)

These are deliberately outside the functional-spec layer but **must** be produced before the named modules reach code-complete. None blocks project kickoff.

1. **API contract specification (OpenAPI).** Request/response schemas, validation error envelopes, pagination/filtering conventions, idempotency-key headers, status codes. *Highest priority — gates every backend module.*
2. **Data dictionary / physical schema.** §15 specifies entities, key relationships, and critical indexes per module, but not column-level types, nullability, FK actions, or partition keys (flagged for Attendance §15 and Result §15 where volume demands it). *Gates database design.*
3. **Non-functional requirements baseline.** Concrete SLOs for the hot paths called out in specs (Attendance write throughput, Result async-publish at 30k-cohort scale, Notification peak batching, signed-URL issuance latency). Specs name these as constraints; numbers are not yet set.
4. **Notification template catalog.** §16 enumerates *what* each module sends; the actual per-event templates (copy, placeholders, channel, mandatory-category flag) are configuration to be authored against NOT-001/003/004.
5. **Report definition catalog.** §18 names reports per module; the parameterized definitions (D42 config) with masking rules and entitlement bindings (REP-002/003) are to be authored.
6. **Migration / seeding spec.** Type-template seeds per deployment (CFG-011), opening balances (Fee, Leave), and historical data import mappings (the many `Import` use cases) need a consolidated migration plan.

---

## 5. Implementation Risks

Ranked by combined likelihood × impact. Each cites the spec sections that contain the controlling requirement.

### 5.1 Critical

- **R1 — Money integrity (Fee 17 / Payment 18).** Idempotent payment processing (PAY-008), gapless immutable receipts (PAY-001), invoice immutability + idempotent generation (FEE-003/004), and reversal-not-delete (PAY-007) must be enforced as **database constraints**, not application checks. *Risk:* under concurrency, double-credit or duplicate invoices. *Mitigation:* unique idempotency keys + unique receipt/invoice numbers at the DB layer (specified in §15 of both); contract tests on the replay/double-submit edge cases (§20).
- **R2 — Result reproducibility & determinism (Result 15 / Grading 16).** Provenance triple `{ruleVer, configVer, gradeVer}` stamping (RES-001), single-point rounding (RES-003), deterministic tie-breaking (RES-008), and version-pinned computation. *Risk:* a historical marksheet that cannot be reproduced, or non-deterministic ranks. *Mitigation:* pin all three versions at compute start (§7/§15); golden-master tests that recompute archived results and diff to zero.
- **R3 — Payroll separation of duties (HR 23).** `payroll.compute` ≠ `payroll.approve` ≠ `payroll.disburse` as three distinct grants held by three roles, finalized payslips immutable (HR-006). *Risk:* self-approval or post-finalization edit = fraud exposure. *Mitigation:* enforce SoD in the Workflow Engine (not in HR code); immutability as a DB write-lock (§15).
- **R4 — Audit fail-safe & immutability (Audit 29).** No update/delete path (AUD-001), fail-closed on audit unavailability for sensitive actions (AUD-007), capture via transactional outbox (AUD-002). *Risk:* an unaudited sensitive action, or a tamperable log. *Mitigation:* append-only storage with no UPDATE/DELETE grants; the outbox (D13) is on the critical path and needs its own reliability testing.
- **R5 — Minors'-data protection chain (Guardian 09 / Notification 25 / File 30 / Reporting 26).** Custody-aware recipient resolution (NOT-002/GRD-N-006), signed-URL-only file access with EXIF/GPS stripping (FILE-004/005/006), reports-never-bypass-authorization (REP-002). *Risk:* a restricted party reached, a child's location leaked via photo metadata, or a report exposing out-of-scope data. *Mitigation:* enforce custody/scope at the **data layer and shared components** (see Screen Matrix), not per-module; penetration-test the signed-URL and report-parameter surfaces.

### 5.2 High

- **R6 — Outbox reliability (cross-cutting; D13).** Audit (AUD-002), Notification (NOT-005), and domain events all depend on the transactional outbox. *Risk:* a single under-engineered outbox becomes a platform-wide reliability ceiling. *Mitigation:* treat the outbox as a first-class, separately load-tested component before module work scales.
- **R7 — Workflow Engine as universal substrate (28).** 23 of 30 modules route approvals through it; version-pinning (WFL-002), SoD (WFL-004), and safe-timeout (WFL-007) are inherited by all. *Risk:* a defect here propagates to every governed process; "unsafe auto-approve rejected at design time" must actually be enforced. *Mitigation:* the engine is a top-of-critical-path build; its FSM validator and SoD resolver need exhaustive tests independent of any consuming module.
- **R8 — Attendance write volume (10).** Highest write-rate module; composite uniqueness (enrollment, date, period), session partitioning, materialized summaries (§15). *Risk:* table bloat / slow marking at scale. *Mitigation:* the §15 partitioning strategy and the 50/page table default are starting points; validate against real cohort sizes (NFR baseline, §4.3).
- **R9 — Configuration Engine resolution hot path (27).** Every module resolves config on read; cache invalidation by config-version (CFG-008). *Risk:* a slow or incoherent resolver throttles the whole platform; a stale cache yields wrong behavior. *Mitigation:* Redis-backed resolution keyed by config-version with correctness-over-speed fallback (§19).

### 5.3 Moderate

- **R10 — Cross-module ownership transitions (Teacher 21 ↔ Subject/Section).** C-05 immediate re-resolution with history attribution. *Risk:* marks attributed to the wrong teacher across a mid-term reassignment. *Mitigation:* the handover gate (TCH-006) and §20 reassignment-race edge cases are specified; needs careful transactional implementation.
- **R11 — Refund Fee/Payment split (see §3).** Coordination risk across two teams. *Mitigation:* finance-lead sign-off on the decision/execution boundary before build.
- **R12 — Early-module convention drift.** Phase-1 modules predate later conventions (the citation gaps in §2.1 were the symptom). *Mitigation:* a one-pass conformance review of modules 01–05 against the final template before they are handed to engineering.

---

## 6. Recommended Build Sequencing

Derived from the Module Dependency Matrix (fan-in) and risk profile:

1. **Platform core first:** Configuration Engine (27) and Workflow Engine (28) — the two substrates the other 28 modules resolve through. Then Authentication (01), Authorization (02), Audit (29), File (30), Notification (25) as shared services. *These have the highest fan-in and gate everything else.*
2. **Foundational domain:** Institute (03), Campus (04), Session (05), Staff (22) — the scoping and hierarchy backbone (approver resolution depends on Staff).
3. **Student lifecycle:** Guardian (09), Student (07), Admission (06), Enrollment (08).
4. **Academic structure & operations:** Class (11), Section (12), Subject (13), Teacher (21), Attendance (10).
5. **Assessment:** Examination (14), Result (15), Grading (16).
6. **Finance & HR:** Fee (17), Payment (18), Discount (19), Scholarship (20), HR (23), Leave (24), Reporting (26).

Build the **seven shared screen archetypes** (Screen Matrix §"Shared cross-cutting patterns") alongside the platform core so every later module reuses them.

---

## 7. Conclusion

The Module Functional Specification Repository is **complete, internally consistent, and fully traceable** to the approved architecture, 248 business rules, and 512 use cases. It is suitable as the controlling functional baseline for enterprise ERP development. Before code-complete on the affected modules, the team must produce the **API contracts**, **data dictionary**, and **NFR baseline** (§4), obtain **product-owner decisions** on re-evaluation intake and the refund split (§2.2, §3), and treat the **five critical risks** (§5.1) as gated, test-first work items. None of these blocks project kickoff; all are normal next-stage artifacts in an enterprise build.
