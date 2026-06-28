# 15 — Result Processing Use Cases

Transforms the Result Processing business rules (`RES-001`…`RES-009`) into use cases. Turns locked marks into published results: deterministic and provenance-stamped, layered pass/fail, single-point rounding, classification/GPA, completeness-gated, governed revision, fair withholding, deterministic ranking, and async publishing.

## 1. Primary Actors
Examination Controller / Result Administrator (computes, reviews, publishes, revises).

## 2. Secondary Actors
System (deterministic computation, provenance stamping, ranking, publishing), Grading module (scale resolution), Examination module (locked marks), Finance (withholding), Notification & File modules (publish/marksheets), Audit service.

## 3. Goals
Compute results deterministically and reproducibly (stamping rule/config/grade versions); apply layered pass/fail and single-point rounding; derive division/classification/GPA; block on incompleteness (no silent zeros); revise post-publish only under governance; withhold fairly and time-bound; rank deterministically with defined tie-breaks; publish at scale without blocking operations.

## 4. User Journeys
- **Compute:** controller runs result computation over locked, moderated marks → deterministic, provenance-stamped results (rule + config + grade versions).
- **Review & publish:** controller reviews → publishes asynchronously at scale → marksheets generated (immutable, reproducible) → guardians notified.
- **Revise:** a discovered error triggers a governed post-publish revision → new versioned marksheet → affected parties notified.
- **Withhold:** dues/other defined reasons withhold a result fairly and time-bound, with a clear remedy path.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-RES-001 | Compute Result (deterministic, provenance-stamped) | Critical |
| Core | UC-RES-002 | Apply Layered Pass/Fail & Classification/GPA | High |
| Core | UC-RES-003 | Compute Rank/Position (tie-breaking) | High |
| Core | UC-RES-004 | Publish Results (async, at scale) | Critical |
| Core | UC-RES-005 | Governed Post-Publish Revision | Critical |
| Core | UC-RES-006 | Withhold / Release Result (time-bound) | High |
| CRUD | UC-RES-007 | View Result / Marksheet (scoped) | High |
| Admin | UC-RES-008 | Configure Computation (rounding, division, tie-break) | High |
| Core | UC-RES-009 | Completeness Gate (no silent zeros) | High |
| Approval | UC-RES-010 | Approve Result Revision (SoD) | High |
| Search | UC-RES-011 | Search Results | Low |
| Reporting | UC-RES-012 | Result Analysis & Tabulation Report | Medium |
| Bulk | UC-RES-013 | Bulk Compute / Recompute | High |
| Export | UC-RES-014 | Export Results / Tabulation (governed) | Medium |
| Workflow | UC-RES-015 | Revision Workflow (version-pinned) | High |
| Exception | UC-RES-016 | Compute With Incomplete Marks Blocked | High |
| Exception | UC-RES-017 | Re-evaluation Direction Policy (C-02) | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-RES-001 — Compute Result (deterministic, provenance-stamped)
- **Module:** Result Processing · **Priority:** Critical
- **Actors:** Result Administrator (primary), System
- **Goal:** Compute each student's result deterministically and reproducibly, stamping the versions used.
- **Description:** Aggregates locked/moderated marks via the configured rules; applies single-point rounding; resolves grades via the effective grade scale; stamps rule + config + grade-scale versions so the result is reproducible.
- **Business Rules Applied:** RES-001, RES-002, RES-003, RES-005, GRD-007, CFG-004.
- **Preconditions:** Marks locked (UC-EXM-004); grade scheme effective; completeness satisfied.
- **Trigger:** Administrator runs computation.
- **Main Success Scenario:**
  1. System verifies completeness (no missing marks; outcomes recorded, RES-005).
  2. System aggregates component → subject → aggregate with layered pass/fail (RES-002).
  3. System applies single-point rounding (RES-003) and resolves grades via the effective scale (GRD-007).
  4. System stamps provenance (rule/config/grade versions) on each result (RES-001).
- **Alternative Flows:** A1) Recompute after a moderation/correction (idempotent, re-stamped).
- **Exception Flows:** E1) Incomplete marks → block (UC-RES-016). E2) Special outcomes (absent/malpractice) → handled per EXM-006 rules, not as zero unless defined.
- **Validation Rules:** Completeness satisfied; deterministic computation; single rounding point; provenance stamped (RES-001/002/003/005).
- **Permissions Required:** `result.compute`.
- **Notifications Triggered:** None until publish.
- **Audit Events Generated:** `RESULT_COMPUTED` (provenance versions).
- **Data Created:** Computed results (unpublished).
- **Data Updated:** Result store.
- **Data Deleted:** None.
- **Post Conditions:** Deterministic, reproducible results ready for review/publish.
- **Related Use Cases:** UC-RES-002, UC-RES-004, UC-GRD-001.
- **Acceptance Criteria:**
  - Given complete locked marks, When computed, Then results are deterministic and stamped with rule/config/grade versions.
  - Given the same inputs recomputed, When run again, Then identical results are produced (reproducibility).
  - Given rounding, When applied, Then it happens once at the defined point (no compounding).
- **Edge Case Analysis:**
  - *Invalid Input:* missing marks → blocked (RES-005).
  - *Permission Failure:* lacks `result.compute` → 403.
  - *Concurrent Update:* recompute during moderation → serialized; final result re-stamped.
  - *Duplicate Data:* idempotent computation (no duplicate results).
  - *System Failure:* deterministic re-run; partial compute recoverable.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* compute; recompute reproducibility; grade resolution.
  - *Negative:* incomplete marks; double rounding (must not occur).
  - *Boundary:* mark exactly at a grade-band boundary (rounding rule decides); pass/fail boundary.

### UC-RES-005 — Governed Post-Publish Revision
- **Module:** Result Processing · **Priority:** Critical
- **Actors:** Result Administrator (primary), Approver, System
- **Goal:** Correct a published result transparently under governance, producing a new versioned marksheet.
- **Description:** Applies the Governed Correction Pattern (P5): the published result is never silently edited; a revision records old→new with reason and actor, requires approval (SoD), regenerates the marksheet (new version), and notifies affected parties — honoring the institution's re-evaluation direction policy (C-02).
- **Business Rules Applied:** RES-006, RES-001, GRD-008, AUTHZ-009, FILE-007 (Cross-Cutting P5).
- **Preconditions:** Result published; revision justified; approval obtained.
- **Trigger:** A discovered error or a re-evaluation outcome.
- **Main Success Scenario:**
  1. Administrator initiates a revision with a reason (and re-evaluation context if applicable).
  2. Approval is obtained via the Workflow Engine (SoD, requester ≠ approver).
  3. System computes the revised result (re-stamped), preserving the prior version.
  4. System regenerates the marksheet as a new immutable version (FILE-007) and notifies the student/guardian.
- **Alternative Flows:** A1) Re-evaluation may move the result up or down per the disclosed policy (C-02).
- **Exception Flows:** E1) Closed period → governed reopen first (SESS-006). E2) Ungoverned silent edit → blocked.
- **Validation Rules:** Original preserved; reason recorded; approved; re-stamped; new marksheet version; direction policy honored (RES-006, P5, C-02).
- **Permissions Required:** `result.revise` (elevated) + `result.revision.approve` (SoD).
- **Notifications Triggered:** `RESULT_REVISED` to student/guardian; revised marksheet ready.
- **Audit Events Generated:** `RESULT_REVISED` (old→new, reason, approver, versions).
- **Data Created:** Revised result version; new marksheet.
- **Data Updated:** Current result pointer (history retained).
- **Data Deleted:** None.
- **Post Conditions:** Corrected result published transparently; prior version retained; affected parties notified.
- **Related Use Cases:** UC-RES-001, UC-RES-010, UC-EXM-010, UC-SESS-006.
- **Acceptance Criteria:**
  - Given a published result error, When revised under approval, Then a new versioned result and marksheet are produced and the original is preserved.
  - Given a re-evaluation, When the disclosed policy allows either direction, Then the result may go up or down and the student is notified.
  - Given a closed period, When revision is needed, Then a governed reopen is required first.
- **Edge Case Analysis:**
  - *Invalid Input:* revision without reason rejected.
  - *Permission Failure:* self-approval blocked (SoD).
  - *Concurrent Update:* two revisions → serialized; latest version current, all retained.
  - *Duplicate Data:* idempotent identical revision.
  - *System Failure:* atomic with new marksheet; no partial revision.
  - *Workflow Failure:* approval unresolved → revision pending (no silent change).
- **QA Coverage:**
  - *Positive:* approved revision up; approved revision down (if policy); re-evaluation.
  - *Negative:* self-approval; silent edit; closed-period without reopen.
  - *Boundary:* revision crossing a pass/fail or division boundary.

### UC-RES-004 — Publish Results (async, at scale)
- **Module:** Result Processing · **Priority:** Critical
- **Actors:** Result Administrator (primary), System
- **Goal:** Publish results to students/guardians at scale without blocking operations.
- **Description:** Publishing runs asynchronously (e.g., 30k students), generates immutable marksheets, and notifies guardians via batched, deduplicated notifications; withheld results are excluded with a defined reason.
- **Business Rules Applied:** RES-009, RES-007, FILE-007, NOT-006.
- **Preconditions:** Results computed and reviewed; withholding rules applied.
- **Trigger:** Administrator publishes.
- **Main Success Scenario:**
  1. Administrator triggers publication.
  2. System runs publication asynchronously; generates immutable, reproducible marksheets (FILE-007).
  3. System notifies guardians via batched/deduplicated notifications (NOT-006), excluding withheld results (RES-007).
  4. Students/guardians can view published results (scoped).
- **Alternative Flows:** A1) Staged publication by class/section.
- **Exception Flows:** E1) Withheld result → not published; remedy path shown (UC-RES-006). E2) Publish failure mid-batch → resumable; no partial-inconsistent publish.
- **Validation Rules:** Only reviewed results published; withholding honored; async/batched; marksheets immutable (RES-009/007, FILE-007).
- **Permissions Required:** `result.publish`.
- **Notifications Triggered:** `RESULT_PUBLISHED` (batched) to guardians; secure-view link.
- **Audit Events Generated:** `RESULTS_PUBLISHED` (scope, count).
- **Data Created:** Marksheets; publication record.
- **Data Updated:** Result visibility → published.
- **Data Deleted:** None.
- **Post Conditions:** Results visible to authorized viewers; marksheets reproducible; operations unaffected.
- **Related Use Cases:** UC-RES-001, UC-RES-006, UC-FILE-003.
- **Acceptance Criteria:**
  - Given reviewed results, When published, Then marksheets are generated and guardians notified without blocking interactive use.
  - Given a withheld result, When publishing, Then it is excluded with a defined reason.
  - Given a large cohort, When published, Then notifications are batched/deduplicated.
- **Edge Case Analysis:**
  - *Invalid Input:* publishing uncomputed results blocked.
  - *Permission Failure:* lacks `result.publish` → 403.
  - *Concurrent Update:* re-publish idempotent; no duplicate marksheets.
  - *Duplicate Data:* one notification per guardian per publish (NOT-006).
  - *System Failure:* resumable batch; no partial publish.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* full publish; staged publish; withheld exclusion.
  - *Negative:* uncomputed publish; duplicate notification.
  - *Boundary:* 30k-cohort throughput; resume after mid-batch failure.

---

## 7. Compact Specifications (routine use cases)

- **UC-RES-002 — Apply Layered Pass/Fail & Classification/GPA** · *High* · Rules: RES-002, RES-004, GRD-004. Component→subject→aggregate pass/fail; derive division/classification/GPA. *Edge:* component-level pass requirement; layered failure. *QA:* layered pass/fail; GPA correctness.
- **UC-RES-003 — Compute Rank/Position (tie-breaking)** · *High* · Rules: RES-008. Deterministic rank with defined tie-breaks. *Edge:* ties resolved deterministically; scope of ranking. *QA:* ranking; tie-break determinism.
- **UC-RES-006 — Withhold / Release Result (time-bound)** · *High* · Rules: RES-007, FEE. Withhold for defined reasons (dues), fair and time-bound, with remedy. *Audit:* `RESULT_WITHHELD/RELEASED`. *Edge:* time-bound; clear remedy; not punitive-indefinite. *QA:* withhold/release; time bound; remedy path.
- **UC-RES-007 — View Result / Marksheet (scoped)** · *High* · Rules: AUTHZ-003, STU-005, FILE-005. Scoped/owned view; student/guardian see own published result; marksheet via signed URL. *Edge:* pre-publish hidden; withheld shows reason. *QA:* scope; pre-publish hidden; withheld reason.
- **UC-RES-008 — Configure Computation (rounding, division, tie-break)** · *High* · Rules: RES-003/004/008, CFG-004. Configure rounding point, division bands, tie-break (versioned). *Permissions:* `result.config.manage`. *Edge:* single rounding point; versioned. *QA:* config applies; reproducibility.
- **UC-RES-009 — Completeness Gate (no silent zeros)** · *High* · Rules: RES-005. Block computation/publish on missing marks; surface gaps. *QA:* gap blocks; no auto-zero.
- **UC-RES-010 — Approve Result Revision (SoD)** · *High* · Rules: RES-006, AUTHZ-009, WFL-004. Approver (≠ requester) approves revisions. *Edge:* SoD; escalation. *QA:* approval; self-approval blocked.
- **UC-RES-011 — Search Results** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-RES-012 — Result Analysis & Tabulation Report** · *Medium* · Rules: REP-002. Tabulation, pass %, subject analysis. *Edge:* scoped; consistent with computation rules. *QA:* tabulation accuracy.
- **UC-RES-013 — Bulk Compute / Recompute** · *High* · Rules: RES-001, RES-009. Batch compute/recompute (deterministic, re-stamped, resumable). *Edge:* idempotent; partial-failure recovery. *QA:* large compute; recompute reproducibility.
- **UC-RES-014 — Export Results / Tabulation (governed)** · *Medium* · Rules: REP-005, STU-005. Governed, scoped export. *Permissions:* `report.export`. *Edge:* bulk PII gated. *QA:* scoped; gated.
- **UC-RES-015 — Revision Workflow (version-pinned)** · *High* · Rules: WFL-002/004, RES-006. Version-pinned governed revision. *QA:* pinning; SoD; new marksheet version.
- **UC-RES-016 — Compute With Incomplete Marks Blocked (Exception)** · *High* · Rules: RES-005. Computation blocked on incompleteness. *QA:* blocked; complete allowed.
- **UC-RES-017 — Re-evaluation Direction Policy (Exception/Decision, C-02)** · *High* · Rules: RES-006. The disclosed institutional policy governs whether re-evaluation can lower a result; applied uniformly, surfaced at request. *QA:* policy honored; disclosure present; either-direction (if configured).

## 8. Module-level QA & Edge Themes
- **Determinism & reproducibility (RES-001 / P6):** identical inputs → identical results; provenance triple stamped — the headline correctness suite.
- **Single rounding point (RES-003):** no compounding rounding; boundary cases at grade/division edges.
- **Governed revision (RES-006 / P5 / C-02):** no silent edits; SoD; new marksheet version; disclosed re-evaluation direction.
- **No silent zeros (RES-005):** incompleteness blocks, never defaults.
- **Async at scale (RES-009):** 30k-cohort publish is resumable, batched, non-blocking.
