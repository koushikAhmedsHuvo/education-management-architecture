# 15 — Result Processing Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`RES-001…009`), and Use Case Repository (`UC-RES-001…017`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Turn locked marks into published results: deterministic, provenance-stamped computation; layered pass/fail; single-point rounding; division/classification/GPA; completeness gating; governed post-publish revision; fair time-bound withholding; deterministic ranking; and asynchronous performance-safe publishing.

**Business Goal.** Produce correct, reproducible, defensible results at scale — identical inputs always yield identical outputs — with corrections only under governance and publishing that never blocks operations.

**Scope.** Result computation (deterministic, stamped with rule/config/grade versions); layered pass/fail (component→subject→aggregate); single rounding point; division/classification/GPA; completeness gate (no silent zeros); governed post-publish revision (with the C-02 re-evaluation-direction policy); withholding (time-bound, fair); deterministic rank with tie-breaks; async publishing + marksheet generation.

**Out of Scope.** Mark entry/locking (Examination module — provides locked marks). Grade-scale definition/resolution (Grading module — provides the scale; this module stamps it). Marksheet file storage (File module — generates via FILE-007). Notification delivery (Notification module).

---

## 2. Actors

**Primary Actors.** Examination Controller / Result Administrator (computes, reviews, publishes, revises), Approver (revisions — SoD), System (deterministic computation, ranking, async publish).

**Secondary Actors.** Examination module (locked marks), Grading module (scale resolution/stamping), Fee module (withholding), File module (marksheets), Notification (publish/revision notices), Workflow Engine (revision approval), Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Deterministic provenance-stamped computation | Compute results reproducibly, stamping rule/config/grade versions. | Critical | Examination, Grading, Config |
| FR-002 | Layered pass/fail & classification/GPA | Component→subject→aggregate pass/fail; derive division/GPA. | High | Grading (GPA) |
| FR-003 | Single-point rounding | Apply rounding once at the defined point (no compounding). | High | Config |
| FR-004 | Completeness gate | Block computation/publish on missing marks (no silent zeros). | High | Examination (RES-005) |
| FR-005 | Deterministic ranking | Compute rank/position with defined tie-breaking. | High | FR-001 |
| FR-006 | Async publishing | Publish at scale without blocking; generate immutable marksheets. | Critical | File, Notification |
| FR-007 | Governed post-publish revision | Revise published results only with SoD approval + new marksheet version. | Critical | Workflow (P5), File |
| FR-008 | Withholding (time-bound) | Withhold results for defined reasons, fairly and time-bound. | High | Fee |
| FR-009 | Re-evaluation direction policy | Apply the disclosed C-02 policy (up/down) uniformly at revision. | High | Config, Governance |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Reproducible computation | Stamped provenance triple. | Defensible historical results. |
| Layered pass/fail + GPA | Correct multi-level outcomes. | Accurate results. |
| Single rounding point | No compounding errors. | Fairness; precision. |
| Completeness gate | No silent zeros. | Integrity. |
| Deterministic ranking | Defined tie-breaks. | Fair, reproducible positions. |
| Async publish at scale | Non-blocking, marksheets. | 30k-cohort performance. |
| Governed revision | SoD, versioned marksheet. | Transparent corrections. |
| Time-bound withholding | Fair, defined, remediable. | Policy with fairness. |

---

## 5. Screens

Compute Result; Result Review; Publish Results (async); Result/Marksheet View (scoped); Governed Revision; Withhold/Release Result; Computation Configuration (rounding/division/tie-break); Completeness Gate; Result Analysis & Tabulation Report; Bulk Compute/Recompute; Result Export; Revision Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Compute Result | Run Compute, Recompute, View Provenance | Bulk Compute |
| Result Review | Review, Flag, Approve for Publish | — |
| Publish Results | Publish (async), Staged Publish, View Status | Bulk/Staged Publish |
| Marksheet View | View, Download (signed URL), Reprint | — |
| Governed Revision | Propose Revision (reason), Submit (→ approval) | — |
| Withhold/Release | Withhold (reason, time-bound), Release | Bulk Withhold |
| Computation Config | Set Rounding/Division/Tie-break, Save (versioned) | — |
| Tabulation Report | Run, Filter, Export | Export |

---

## 7. Forms

**Compute** — `exam`/`scope` (required). Validation: marks locked (EXM-004); completeness satisfied (RES-005); resolves effective grade scale (GRD-007); stamps rule/config/grade versions (RES-001).

**Computation Config** — `roundingPoint` (single), `divisionBands` (structured), `tieBreak` (ordered rule), `gpaRule`. Validation: single rounding point (RES-003); deterministic tie-break (RES-008); versioned (CFG-004).

**Governed Revision** — `student`/`scope`, `reason` (text, required), re-evaluation context. Validation: published result (RES-006); SoD approval (AUTHZ-009); new marksheet version (FILE-007); direction per disclosed policy (C-02).

**Withhold/Release** — `reason` (select: dues/other), `until` (date — time-bound). Validation: defined reason; time-bound and remediable (RES-007).

---

## 8. Search & Filter Requirements

**Results:** by student, exam, class/section, status (computed/published/withheld/revised), division/grade, rank range. Sorting: rank/total/status. Pagination: server-side, 50 default (scale). Scope + ownership (students/guardians see own published).

---

## 9. Table Requirements

**Result table:** Roll No., Student, Subject grades, Total, GPA, Division, Rank, Status. **Tabulation:** subject-wise pass %, toppers. Sorting on Rank/Total. Filtering as above. Export (governed). Bulk: compute/recompute, publish, withhold.

---

## 10. Workflow Requirements

**Trigger events:** compute, review, publish, revise, withhold/release. **Status changes:** result `COMPUTED → PUBLISHED → REVISED(governed)`; `WITHHELD ⇄ RELEASED`. **Approvals:** revisions via version-pinned workflow (SoD). **Notifications:** publish (batched), revision, withholding/release (custody-aware). **Audit:** computation (provenance), publish, revision (old→new, versions), withholding (immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Compute / recompute | `result.compute` |
| Publish results | `result.publish` |
| Revise (governed) | `result.revise` (elevated) |
| Approve revision | `result.revision.approve` (SoD) |
| Withhold / release | `result.withhold.manage` |
| Configure computation | `result.config.manage` |
| View result/marksheet | `result.view` (scoped) |
| Export results | `report.export` |

Revision is elevated + SoD; computation/publish are controller-scoped.

---

## 12. Business Rule References

RES-001 (deterministic, provenance-stamped), RES-002 (layered pass/fail), RES-003 (single-point rounding), RES-004 (division/classification & GPA), RES-005 (completeness gate / no silent zeros), RES-006 (governed post-publish revision), RES-007 (withholding defined/fair/time-bound), RES-008 (deterministic rank/tie-break), RES-009 (async publishing). Cross-cutting: EXM-004/007 (locked marks), GRD-007 (effective-dated scale stamping), FILE-007 (immutable marksheets), NOT-006 (batched publish), C-02 (re-evaluation direction), WFL-002/004, AUTHZ-009, AUD-001.

## 13. Use Case References

UC-RES-001 (Compute — provenance-stamped), UC-RES-002 (Layered Pass/Fail & GPA), UC-RES-003 (Rank/Position), UC-RES-004 (Publish — async), UC-RES-005 (Governed Revision), UC-RES-006 (Withhold/Release), UC-RES-007 (View Result/Marksheet), UC-RES-008 (Configure Computation), UC-RES-009 (Completeness Gate), UC-RES-010 (Approve Revision — SoD), UC-RES-011 (Search), UC-RES-012 (Analysis & Tabulation), UC-RES-013 (Bulk Compute/Recompute), UC-RES-014 (Export), UC-RES-015 (Revision Workflow), UC-RES-016 (Incomplete Marks Blocked), UC-RES-017 (Re-evaluation Direction Policy — C-02).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Compute / recompute results | POST | Controller |
| Get result / marksheet (scoped) | GET | Authorized roles / Student-Guardian |
| Publish results (async) | POST | Controller |
| Propose / approve revision | POST | Controller / Approver |
| Withhold / release result | POST | Controller |
| Configure computation (rounding/division/tie-break) | GET/PUT | Controller |
| Result analysis & tabulation report | GET | Controller |
| Bulk compute / export | POST/GET | Controller |

Computation is idempotent and re-stamped; publish runs asynchronously and is resumable (RES-009).

---

## 15. Database Requirements

**Entities:** `Result` (studentId, examId, totals, GPA, division, rank, status, provenance{ruleVer, configVer, gradeVer}), `ResultRevision` (old/new, reason, approver, version), `Withholding` (reason, until), `ComputationConfig` (rounding/division/tie-break via Config), `Marksheet` (file ref, version). **Relationships:** Result 1—* Revision; Result 1—1 Marksheet (current) + versions; Result *—1 Exam. **Indexes:** unique(Result.examId, studentId), index(Result.status), index(Result.classId, rank). Provenance stamped per result (RES-001); revisions retain history (RES-006).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Result published (batched), revised, withheld/released — to guardians (custody-aware). |
| SMS | Result-ready alert with secure-view link (minimized — no full marks). |
| Push | Result/revision notices. |
| In-App | Revision approvals; withholding status. |

Publish notifications batched/deduped at scale (NOT-006); content minimized (NOT-004).

---

## 17. Audit Requirements

Log: computation (provenance versions), publish (scope, count), revision (old→new, reason, approver, versions), withholding/release. Record who/when/before/after. Computation and revision are first-class audit events with full lineage. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Tabulation/result analysis, Pass %, Subject analysis, Rank lists, Division distribution, Withholding register. **Exports:** governed result/tabulation export (PII-gated). **Dashboards:** result-processing control (compute status, publish progress, revisions pending).

---

## 19. Error Handling

**Validation:** incomplete marks, double-rounding (prevented), invalid config → specific errors (UC-RES-016). **Permission:** revision without elevation/self-approval → blocked. **Workflow:** revision pending → result unchanged; closed period → governed reopen first (SESS-006). **System:** Grading scale unavailable → compute blocked; publish mid-batch failure → resumable, no partial-inconsistent publish.

---

## 20. Edge Cases

**Concurrent updates:** recompute during moderation/correction → serialized; final re-stamped. **Duplicate data:** idempotent computation (no duplicate results); one notification per guardian per publish. **Partial failures:** publish mid-batch failure → resumable; no partial publish. **Rollback:** revision reversed → prior version current, all retained. **Boundary race:** revision crossing pass/fail or division boundary → recomputed deterministically.

---

## 21. Acceptance Criteria

**Functional.** Identical inputs produce identical results stamped with rule/config/grade versions; rounding happens once; incompleteness blocks (no silent zeros); ranking is deterministic with defined tie-breaks; publishing runs asynchronously generating immutable marksheets without blocking operations; post-publish revision requires SoD approval, preserves the original, and produces a new versioned marksheet honoring the disclosed re-evaluation-direction policy; withholding is defined, fair, and time-bound.

**Business.** Results are correct, reproducible, and defensible at scale; corrections are transparent and governed; a reprinted historical marksheet reproduces exactly from its stamped versions.

---

## 22. Future Enhancements

Configurable result formats/templates per board; predictive/early result analytics; re-evaluation request portal with fee; comparative cohort analytics; transcript/CGPA aggregation across sessions; digital signing/verification (QR) of marksheets; result-publish scheduling.
