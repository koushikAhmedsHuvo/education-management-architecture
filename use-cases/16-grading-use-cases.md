# 16 — Grading Use Cases

Transforms the Grading business rules (`GRD-001`…`GRD-008`) into use cases. The grade-scale authority: configurable, versioned scales with complete non-overlapping bands, deterministic boundaries, GPA rules, absolute/relative schemes, special grades, effective-dated resolution with result stamping, and governed overrides.

## 1. Primary Actors
Grading / Assessment Administrator (defines and versions grade scales).

## 2. Secondary Actors
System (band-coverage validation, deterministic boundary resolution, effective-dating, stamping), Result module (consumer), Configuration Engine (versioning), Audit service.

## 3. Goals
Define versioned grade scales whose bands completely and non-overlappingly cover the range; resolve boundaries deterministically; compute GPA/CGPA; support absolute vs relative (scope-specific) schemes; define special grades excluded correctly; resolve the effective-dated scale and stamp it on results; govern grade overrides transparently.

## 4. User Journeys
- **Define a scale:** admin creates a grade scale (bands with ranges, points, labels) → system validates complete, non-overlapping coverage → publishes a version with an effective date.
- **Resolve & stamp:** result computation resolves the effective scale for the period and stamps the version on each result (reproducibility).
- **Evolve:** a new scale version is effective-dated forward; historical results keep their stamped version.
- **Override:** a specific grade override is applied transparently under governance.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-GRD-001 | Define / Version Grade Scale (validated coverage) | Critical |
| Core | UC-GRD-002 | Resolve Effective Scale & Stamp Result | High |
| Core | UC-GRD-003 | Configure GPA / CGPA Rule | High |
| Core | UC-GRD-004 | Configure Scheme Scope (absolute vs relative) | High |
| Core | UC-GRD-005 | Manage Special Grades (exclusions) | Medium |
| Core | UC-GRD-006 | Governed Grade Override (transparent) | High |
| CRUD | UC-GRD-007 | View Grade Scale(s) | Medium |
| CRUD | UC-GRD-008 | Update / Re-version Scale (effective-dated) | High |
| Approval | UC-GRD-009 | Approve Scale Change / Override (SoD) | High |
| Search | UC-GRD-010 | Search Grade Scales | Low |
| Reporting | UC-GRD-011 | Grade Distribution Report | Medium |
| Import/Export | UC-GRD-012 | Import / Export Grade Scale | Low |
| Workflow | UC-GRD-013 | Scale-Change / Override Workflow | High |
| Exception | UC-GRD-014 | Band Gap / Overlap Blocked | Critical |
| Exception | UC-GRD-015 | Boundary Ambiguity Blocked | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-GRD-001 — Define / Version Grade Scale (validated coverage)
- **Module:** Grading · **Priority:** Critical
- **Actors:** Grading Administrator (primary), System
- **Goal:** Create a versioned grade scale whose bands completely and non-overlappingly cover the mark range, with deterministic boundaries.
- **Description:** Defines bands (range, grade label, points); the system validates full coverage with no gaps or overlaps and deterministic boundary handling; publishes an immutable version with an effective date.
- **Business Rules Applied:** GRD-001, GRD-002, GRD-003, CFG-004.
- **Preconditions:** Actor holds `grading.config.manage`.
- **Trigger:** Administrator defines/edits a grade scale.
- **Main Success Scenario:**
  1. Administrator enters bands (e.g., 80–100 A+, 70–79 A, …) with points/labels.
  2. System validates complete coverage of the range with no gaps or overlaps (GRD-002).
  3. System validates deterministic boundary rules (which band owns an exact boundary, GRD-003).
  4. System publishes an immutable scale version with an effective date (GRD-001/CFG-004).
- **Alternative Flows:** A1) Scope-specific scheme (absolute/relative) attached (UC-GRD-004).
- **Exception Flows:** E1) Gap or overlap in bands → blocked (UC-GRD-014). E2) Ambiguous boundary → blocked (UC-GRD-015).
- **Validation Rules:** Bands cover the full range; non-overlapping; deterministic boundaries; versioned/effective-dated (GRD-001/002/003).
- **Permissions Required:** `grading.config.manage` (+ approval for changes).
- **Notifications Triggered:** Scale change notice to result admins.
- **Audit Events Generated:** `GRADE_SCALE_PUBLISHED` (version, effective date).
- **Data Created:** Grade scale version.
- **Data Updated:** None (new version).
- **Data Deleted:** None.
- **Post Conditions:** Valid, versioned scale available for effective-dated resolution.
- **Related Use Cases:** UC-GRD-002, UC-GRD-008, UC-RES-001.
- **Acceptance Criteria:**
  - Given bands covering the full range without gaps or overlaps, When published, Then the scale version is accepted with an effective date.
  - Given a gap or overlap, When saved, Then it is blocked.
  - Given an exact boundary mark, When resolved, Then exactly one band owns it deterministically.
- **Edge Case Analysis:**
  - *Invalid Input:* band range inverted/negative rejected.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* two edits → versioned; last coherent version wins.
  - *Duplicate Data:* duplicate band rejected.
  - *System Failure:* atomic publish; partial scale not saved.
  - *Workflow Failure:* approval pends for changes.
- **QA Coverage:**
  - *Positive:* valid scale; absolute and relative schemes.
  - *Negative:* gap; overlap; ambiguous boundary; inverted band.
  - *Boundary:* exact boundary mark ownership; full-range edges (0 and max).

### UC-GRD-002 — Resolve Effective Scale & Stamp Result
- **Module:** Grading · **Priority:** High
- **Actors:** System (primary, via result computation)
- **Goal:** Resolve the grade scale in force for the period and stamp its version on each result for reproducibility.
- **Description:** During computation, the system resolves the effective-dated scale (and scope-specific scheme) and records the resolved version on the result, so historical results reproduce exactly.
- **Business Rules Applied:** GRD-007, GRD-005, RES-001, CFG-004.
- **Preconditions:** A published scale effective for the period; result computation running.
- **Trigger:** Result computation requests grade resolution.
- **Main Success Scenario:**
  1. System resolves the effective-dated scale for the result's period/scope (GRD-007).
  2. System maps each subject/aggregate mark to a grade deterministically.
  3. System stamps the resolved scale version on the result (RES-001/P6).
- **Alternative Flows:** A1) Relative scheme → grades depend on the cohort distribution (GRD-005) within the defined scope.
- **Exception Flows:** E1) No effective scale → block computation (configuration error). E2) Special grades excluded correctly from GPA/aggregates (UC-GRD-005).
- **Validation Rules:** Effective scale resolved; deterministic mapping; version stamped (GRD-007).
- **Permissions Required:** System (consumed by result compute).
- **Notifications Triggered:** None.
- **Audit Events Generated:** Captured within `RESULT_COMPUTED` (grade version).
- **Data Created/Updated:** Grade stamps on results.
- **Data Deleted:** None.
- **Post Conditions:** Results carry the exact grade-scale version used; reproducible.
- **Related Use Cases:** UC-RES-001, UC-GRD-001.
- **Acceptance Criteria:**
  - Given a result for a past period, When reproduced, Then the stamped scale version yields identical grades.
  - Given a new effective scale, When results are computed after its date, Then the new version applies; earlier results keep theirs.
  - Given a relative scheme, When resolved, Then grades reflect the cohort within the defined scope.
- **Edge Case Analysis:**
  - *Invalid Input:* missing effective scale → block.
  - *Permission Failure:* N/A (system).
  - *Concurrent Update:* scale change mid-computation → the run uses the version pinned at start.
  - *Duplicate Data:* idempotent stamping.
  - *System Failure:* deterministic resolution; recoverable.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* absolute resolution; relative resolution; historical reproduction.
  - *Negative:* no effective scale; mid-run change isolation.
  - *Boundary:* effective-date edge (before/after); boundary-mark grade.

### UC-GRD-006 — Governed Grade Override (transparent)
- **Module:** Grading · **Priority:** High
- **Actors:** Grading Administrator (primary), Approver, System
- **Goal:** Override a computed grade in a specific, justified case transparently and under governance.
- **Description:** A grade override is recorded (computed vs overridden), justified, approved (SoD), and visible — never a silent change to the scale or the result.
- **Business Rules Applied:** GRD-008, RES-006, AUTHZ-009 (Cross-Cutting P5).
- **Preconditions:** A specific result/grade case; override justified; approval obtained.
- **Trigger:** Administrator applies an override.
- **Main Success Scenario:**
  1. Administrator specifies the override with a reason for a specific result.
  2. Approval is obtained (SoD, requester ≠ approver).
  3. System records the override (computed grade preserved, overridden grade applied, reason/actor) and re-stamps.
  4. The override is transparent on the result/marksheet per policy.
- **Alternative Flows:** A1) Override as part of a result revision (UC-RES-005).
- **Exception Flows:** E1) Silent/ungoverned override → blocked. E2) Override beyond policy bounds → blocked.
- **Validation Rules:** Computed grade preserved; reason recorded; approved; transparent (GRD-008, P5).
- **Permissions Required:** `grading.override` (elevated) + approval.
- **Notifications Triggered:** Override notice per policy; result-revision notice if published.
- **Audit Events Generated:** `GRADE_OVERRIDDEN` (computed→overridden, reason, approver).
- **Data Created:** Override record.
- **Data Updated:** Effective grade (computed retained).
- **Data Deleted:** None.
- **Post Conditions:** Transparent, governed override applied; computed grade retained.
- **Related Use Cases:** UC-RES-005, UC-GRD-009.
- **Acceptance Criteria:**
  - Given a justified override under approval, When applied, Then the computed grade is preserved and the override is transparent.
  - Given a silent override attempt, When made, Then it is blocked.
  - Given a published result, When overridden, Then it follows the governed revision path with notification.
- **Edge Case Analysis:**
  - *Invalid Input:* override without reason rejected.
  - *Permission Failure:* self-approval blocked (SoD).
  - *Concurrent Update:* override + recompute → serialized; override retained over recompute per policy.
  - *Duplicate Data:* idempotent identical override.
  - *System Failure:* atomic; computed grade never lost.
  - *Workflow Failure:* approval pends.
- **QA Coverage:**
  - *Positive:* approved transparent override.
  - *Negative:* silent override; self-approval; out-of-bounds.
  - *Boundary:* override crossing pass/fail; override on a published result.

---

## 7. Compact Specifications (routine use cases)

- **UC-GRD-003 — Configure GPA / CGPA Rule** · *High* · Rules: GRD-004. Define grade-point mapping and GPA/CGPA aggregation. *Edge:* special grades excluded; credit weighting. *QA:* GPA correctness; exclusions.
- **UC-GRD-004 — Configure Scheme Scope (absolute vs relative)** · *High* · Rules: GRD-005. Set absolute or relative (curved) scheme per scope. *Edge:* relative scope/cohort defined; absolute default. *QA:* absolute vs relative resolution.
- **UC-GRD-005 — Manage Special Grades (exclusions)** · *Medium* · Rules: GRD-006. Define special grades (Absent/Exempt/Incomplete/Withheld) excluded correctly from GPA/aggregates. *Edge:* exclusion vs zero distinction. *QA:* special-grade handling; correct exclusion.
- **UC-GRD-007 — View Grade Scale(s)** · *Medium* · Rules: AUTHZ-002. Scoped read of scales/versions. *QA:* scope respected.
- **UC-GRD-008 — Update / Re-version Scale (effective-dated)** · *High* · Rules: GRD-001/007, CFG-004. New scale version effective forward; historical results keep theirs. *Audit:* `GRADE_SCALE_VERSIONED`. *Edge:* immutable published versions; effective-dating. *QA:* re-version forward; history fidelity.
- **UC-GRD-009 — Approve Scale Change / Override (SoD)** · *High* · Rules: GRD-008, AUTHZ-009, WFL-004. Approver (≠ requester) approves scale changes/overrides. *Edge:* SoD; high-impact. *QA:* approval gates; self-approval blocked.
- **UC-GRD-010 — Search Grade Scales** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-GRD-011 — Grade Distribution Report** · *Medium* · Rules: REP-002. Grade distribution/curve analysis. *Edge:* scoped; consistent with stamped scale. *QA:* distribution accuracy.
- **UC-GRD-012 — Import / Export Grade Scale** · *Low* · Rules: CFG-002, REP-005. Import/export validated scale definitions. *Edge:* coverage validated on import. *QA:* valid import; gap/overlap rejected.
- **UC-GRD-013 — Scale-Change / Override Workflow** · *High* · Rules: WFL-002/004, GRD-008. Version-pinned approval for scale changes/overrides. *QA:* pinning; SoD; escalation.
- **UC-GRD-014 — Band Gap / Overlap Blocked (Exception)** · *Critical* · Rules: GRD-002. Incomplete or overlapping coverage blocked. *QA:* gap blocked; overlap blocked; full coverage allowed.
- **UC-GRD-015 — Boundary Ambiguity Blocked (Exception)** · *High* · Rules: GRD-003. Ambiguous boundary ownership blocked; deterministic rule required. *QA:* ambiguity blocked; deterministic boundary allowed.

## 8. Module-level QA & Edge Themes
- **Complete non-overlapping coverage (GRD-002):** the headline integrity suite — no mark falls into a gap or two bands.
- **Deterministic boundaries (GRD-003):** exact-boundary ownership is unambiguous and tested at every band edge.
- **Effective-dated stamping (GRD-007 / P6):** results reproduce against their stamped scale version; new versions apply forward only.
- **Governed override (GRD-008 / P5):** overrides are transparent, approved, and preserve the computed grade.
- **Special-grade exclusion (GRD-006):** special grades never silently become zeros in GPA/aggregates.
