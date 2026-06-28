# 16 — Grading Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`GRD-001…008`), and Use Case Repository (`UC-GRD-001…015`). No new architecture or business rules introduced.
>
> **ID convention:** Grading uses the `GRD` prefix (Guardian uses `GRD-N`) — they never collide.

---

## 1. Module Overview

**Purpose.** Own the grade-scale authority: configurable versioned grade scales with complete non-overlapping band coverage, deterministic boundaries, GPA/CGPA rules, absolute vs relative schemes, special grades excluded correctly, effective-dated resolution with result stamping, and governed transparent overrides.

**Business Goal.** Guarantee that marks map to grades correctly, reproducibly, and fairly — every mark falls in exactly one band, historical results reproduce against their stamped scale version, and overrides are transparent.

**Scope.** Grade-scale definition/versioning (bands, points, labels); complete non-overlapping coverage validation; deterministic boundary rules; GPA/CGPA computation; absolute vs relative (scope-specific) schemes; special grades (defined, excluded); effective-dated resolution + result stamping; governed transparent grade overrides.

**Out of Scope.** Result computation/aggregation (Result module — consumes resolved grades). Mark entry (Examination module). Subject credit/weightage definition (Subject module — Grading consumes for GPA). Config versioning mechanics (Configuration Engine).

---

## 2. Actors

**Primary Actors.** Grading / Assessment Administrator (defines/versions scales, overrides), Approver (scale changes/overrides — SoD), System (coverage validation, deterministic boundary resolution, effective-dating, stamping).

**Secondary Actors.** Result module (consumer/stamper), Subject module (credit/weightage), Configuration Engine (versioning), Workflow Engine (scale-change/override approval), Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Define/version grade scale | Define bands (range, label, points) as immutable versions. | Critical | Configuration Engine |
| FR-002 | Complete non-overlapping coverage | Validate full coverage with no gaps or overlaps. | Critical | FR-001 |
| FR-003 | Deterministic boundaries | Exactly one band owns each boundary mark. | High | FR-002 |
| FR-004 | GPA/CGPA computation | Configure grade-point mapping and GPA/CGPA aggregation. | High | Subject (credit) |
| FR-005 | Scheme scope (absolute/relative) | Support absolute and relative (curved) schemes per scope. | High | Result |
| FR-006 | Special grades | Define special grades excluded correctly from GPA/aggregates. | Medium | Result |
| FR-007 | Effective-dated resolution & stamping | Resolve the effective scale and stamp its version on results. | High | Result (RES-001) |
| FR-008 | Governed transparent overrides | Override a grade transparently under SoD approval. | High | Workflow, AUTHZ-009 |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Versioned scales | Immutable, effective-dated. | Reproducible grading. |
| Coverage validation | No gaps/overlaps. | Every mark graded once. |
| Deterministic boundaries | Unambiguous edges. | Fairness; predictability. |
| GPA/CGPA | Configurable aggregation. | Correct academic standing. |
| Absolute/relative schemes | Scope-specific. | Fits varied grading models. |
| Special grades | Defined exclusions. | Correct GPA handling. |
| Effective-dated stamping | History reproduces. | Defensible records. |
| Governed overrides | Transparent, approved. | Integrity with flexibility. |

---

## 5. Screens

Grade Scale List; Grade Scale Detail; Define/Version Scale (band editor with coverage validation); GPA/CGPA Rule; Scheme Scope (absolute/relative); Special Grades; Governed Override; Re-version Scale (effective-dated); Grade Distribution Report; Import/Export Grade Scale; Scale-Change/Override Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Grade Scale List | Define, Open, Search, Export | — |
| Scale Detail/Editor | Add/Edit Band, Validate Coverage, Publish Version | — |
| GPA/CGPA Rule | Set Grade Points, Aggregation, Save | — |
| Scheme Scope | Set Absolute/Relative, Define Scope, Save | — |
| Special Grades | Add/Edit (Absent/Exempt/Incomplete/Withheld), Set Exclusion | — |
| Governed Override | Propose Override (reason), Submit (→ approval) | — |
| Re-version Scale | New Version, Set Effective Date, Publish (→ approval) | — |
| Distribution Report | Run, Filter, Export | Export |

---

## 7. Forms

**Define/Version Scale** — bands list (`rangeFrom`, `rangeTo`, `gradeLabel`, `points`), `effectiveDate`. Validation: complete coverage, no gaps/overlaps (GRD-002 / UC-GRD-014); deterministic boundary ownership (GRD-003 / UC-GRD-015); immutable version on publish (GRD-001/CFG-004).

**GPA/CGPA Rule** — `gradePoints` (per band), `aggregation` (credit-weighted), special-grade exclusions. Validation: special grades excluded (GRD-006); credit weighting from Subject (GRD-004).

**Scheme Scope** — `scheme` (absolute/relative), `scope` (institute/class/exam), relative cohort/scope. Validation: absolute default; relative scope/cohort defined (GRD-005).

**Governed Override** — `result/grade` (target), `overrideGrade`, `reason` (text, required). Validation: computed grade preserved; SoD approval (GRD-008/AUTHZ-009); transparent (P5).

---

## 8. Search & Filter Requirements

**Grade scales:** by name, version, scheme (absolute/relative), effective date, status (draft/published/superseded). Sorting: effective date/version. Pagination: server-side, 25 default. Scope-bound.

---

## 9. Table Requirements

**Scale table:** Name, Version, Scheme, Effective Date, Status, #Bands. **Band sub-table:** Range, Grade, Points. Sorting on Effective Date/Version. Filtering as above. Export (governed). No destructive bulk (versioned).

---

## 10. Workflow Requirements

**Trigger events:** define/version scale, GPA rule change, scheme change, special-grade change, override, re-version. **Status changes:** Scale `DRAFT → PUBLISHED → SUPERSEDED`; override `PROPOSED → APPROVED`. **Approvals:** scale changes and overrides via Workflow Engine (SoD). **Notifications:** scale change to result admins; override outcome. **Audit:** scale versions, GPA/scheme/special-grade changes, overrides (before/after, immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| View grade scale(s) | `grading.view` |
| Define/version scale | `grading.config.manage` |
| Configure GPA/scheme/special grades | `grading.config.manage` |
| Govern override | `grading.override` (elevated) |
| Approve scale change/override | `grading.change.approve` (SoD) |
| Import/export scale | `grading.import`, `grading.export` |

Scale changes and overrides are governed (SoD); resolution/stamping is system-driven for Result.

---

## 12. Business Rule References

GRD-001 (configurable versioned grade scale), GRD-002 (complete non-overlapping coverage), GRD-003 (deterministic rounding & boundary rules), GRD-004 (GPA/CGPA computation), GRD-005 (scope-specific absolute vs relative), GRD-006 (special grades defined & excluded), GRD-007 (effective-dated resolution & result stamping), GRD-008 (governed transparent overrides). Cross-cutting: RES-001 (provenance stamping), RES-002/004 (consumes grades/GPA), SUB-005 (credit/weightage), CFG-004 (versioning), WFL-002/004, AUTHZ-009, AUD-001.

## 13. Use Case References

UC-GRD-001 (Define/Version Scale — validated coverage), UC-GRD-002 (Resolve Effective Scale & Stamp Result), UC-GRD-003 (Configure GPA/CGPA), UC-GRD-004 (Configure Scheme Scope), UC-GRD-005 (Manage Special Grades), UC-GRD-006 (Governed Grade Override), UC-GRD-007 (View Grade Scale), UC-GRD-008 (Update/Re-version Scale), UC-GRD-009 (Approve Scale Change/Override — SoD), UC-GRD-010 (Search), UC-GRD-011 (Grade Distribution Report), UC-GRD-012 (Import/Export Grade Scale), UC-GRD-013 (Scale-Change/Override Workflow), UC-GRD-014 (Band Gap/Overlap Blocked), UC-GRD-015 (Boundary Ambiguity Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Define / version grade scale | POST | Grading Admin |
| Get / list grade scales | GET | Authorized roles |
| Resolve effective scale (for Result) | GET (internal) | System / Result |
| Configure GPA / scheme / special grades | PUT | Grading Admin |
| Re-version scale (effective-dated) | POST | Grading Admin (→ approval) |
| Propose / approve grade override | POST | Grading Admin / Approver |
| Grade distribution report | GET | Admin |
| Import / export grade scale | POST/GET | Admin |

Coverage and boundary validation run at write (GRD-002/003); effective-dated resolution returns the scale + version for stamping (GRD-007).

---

## 15. Database Requirements

**Entities:** `GradeScale` (name, version, scheme, effectiveDate, status), `GradeBand` (scaleId, rangeFrom, rangeTo, label, points), `GpaRule` (grade points, aggregation), `SpecialGrade` (label, exclusion), `GradeOverride` (result, computed, override, reason, approver). **Relationships:** GradeScale 1—* GradeBand; GradeScale 1—1 GpaRule; GradeScale 1—* SpecialGrade. **Indexes:** index(GradeScale.effectiveDate, status), index(GradeBand.scaleId), unique constraint on non-overlapping bands per scale (validated). Published versions immutable (GRD-001); effective-dated resolution (GRD-007).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| In-App | Scale change/version notices to result admins; override outcomes; pending approvals. |
| Email | Major scale changes (high-impact). |
| SMS/Push | Not used. |

---

## 17. Audit Requirements

Log: scale define/version (bands, effective date), GPA/scheme/special-grade changes, overrides (computed→override, reason, approver). Record who/when/before/after. Scale versions and overrides are first-class audit events. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Grade distribution/curve, Scale version history, Override log, GPA distribution. **Exports:** governed grade-scale export. **Dashboards:** grading config health (active scale, pending changes, override frequency).

---

## 19. Error Handling

**Validation:** band gap/overlap, boundary ambiguity, inverted range → specific errors (UC-GRD-014/015). **Permission:** override without elevation/self-approval → blocked. **Workflow:** scale change/override pending → not effective until approved. **System:** missing effective scale at computation → Result compute blocked (GRD-007); mid-run scale change → run uses pinned version.

---

## 20. Edge Cases

**Concurrent updates:** two scale edits → versioned, last coherent wins. **Duplicate data:** duplicate band → rejected. **Partial failures:** import partial → per-row report; coverage validated on import. **Rollback:** override reversed → computed grade restored (preserved). **Effective-date race:** scale change mid-computation → run pins the version at start; earlier results keep theirs.

---

## 21. Acceptance Criteria

**Functional.** Grade-scale bands completely and non-overlappingly cover the range with deterministic boundaries; scales are immutable versions effective-dated forward; results stamp the resolved scale version so historical results reproduce identically; GPA/CGPA computes from configured points/credits with special grades excluded; absolute and relative schemes resolve per scope; grade overrides are transparent, SoD-approved, and preserve the computed grade.

**Business.** Every mark maps to exactly one grade, reproducibly and fairly; historical grading is defensible against its stamped version; grade corrections are transparent and governed, never silent.

---

## 22. Future Enhancements

Visual band-coverage editor with live gap/overlap detection; curve/relative-grading simulators; multi-board grade-scale templates; cross-session CGPA transcripts; grade-appeal workflow; configurable boundary-rounding strategies; comparative grade-distribution analytics.
