# 14 — Examination Use Cases

Transforms the Examination business rules (`EXM-001`…`EXM-009`) into use cases. Defines exams, gates eligibility, controls ownership-scoped mark entry within a window, distinguishes special outcomes, and locks marks immutably on submission.

## 1. Primary Actors
Examination Administrator / Controller (defines exams, moderation), Subject-Teacher (enters marks for owned subjects), Invigilator/Examiner (records outcomes).

## 2. Secondary Actors
System (eligibility gating, mark validation, window enforcement, submit-and-lock, moderation), Subject module (maxima/components), Attendance module (eligibility), Result module (consumer), File module (admit cards), Audit service.

## 3. Goals
Define configurable exams with per-subject maxima/components; gate eligibility and admit cards; validate marks against maxima; restrict entry to subject owners within a window; distinguish absence/exemption/malpractice; lock marks immutably on submit; apply bounded, transparent moderation.

## 4. User Journeys
- **Set up:** controller defines an exam (type, schedule, per-subject maxima/components, weightage) for a session/class.
- **Gate:** eligibility (attendance, dues) determines who can sit; admit cards issue only to eligible students.
- **Mark:** subject-teacher enters marks for owned subjects within the window → marks validated against maxima → submit locks them immutably.
- **Adjust:** bounded moderation/grace applied transparently and audited; special outcomes (absent/exempt/malpractice) recorded distinctly.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-EXM-001 | Define Examination (maxima, components, schedule) | High |
| Core | UC-EXM-002 | Determine Eligibility & Issue Admit Card | High |
| Core | UC-EXM-003 | Enter Marks (ownership-scoped, validated) | Critical |
| Core | UC-EXM-004 | Submit & Lock Marks (immutable) | Critical |
| Core | UC-EXM-005 | Record Special Outcome (absent/exempt/malpractice) | High |
| Core | UC-EXM-006 | Apply Moderation / Grace (bounded, transparent) | High |
| CRUD | UC-EXM-007 | View Exam / Marks (scoped) | Medium |
| CRUD | UC-EXM-008 | Update Exam Definition (pre-marks) | Medium |
| Admin | UC-EXM-009 | Configure Eligibility & Window | High |
| Approval | UC-EXM-010 | Approve Post-Lock Mark Correction | High |
| Search | UC-EXM-011 | Search Exams / Mark Sheets | Low |
| Reporting | UC-EXM-012 | Mark-Entry Progress & Exam Report | Medium |
| Bulk | UC-EXM-013 | Bulk Mark Entry / Import | High |
| Export | UC-EXM-014 | Export Marks (governed) | Medium |
| Workflow | UC-EXM-015 | Mark-Correction Workflow | High |
| Exception | UC-EXM-016 | Mark Exceeds Maximum Blocked | High |
| Exception | UC-EXM-017 | Edit After Lock Blocked | Critical |
| Exception | UC-EXM-018 | Incomplete Mark Entry at Window Close | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-EXM-003 — Enter Marks (ownership-scoped, validated)
- **Module:** Examination · **Priority:** Critical
- **Actors:** Subject-Teacher (primary), System
- **Goal:** Let the owning subject-teacher enter validated marks for their subject within the window.
- **Description:** Only the assigned subject-teacher may enter marks for that subject/section; each mark is validated against the configured maximum, with special outcomes handled distinctly.
- **Business Rules Applied:** EXM-004, EXM-005, EXM-002, EXM-009, SUB-004.
- **Preconditions:** Exam defined with maxima; marker owns the subject (SUB-004); within the mark-entry window (EXM-009).
- **Trigger:** Subject-teacher opens the mark sheet and enters marks.
- **Main Success Scenario:**
  1. System presents the mark sheet for the owned subject/section (EXM-005).
  2. Teacher enters component/subject marks.
  3. System validates each mark ≤ configured maximum (EXM-004) and within the window (EXM-009).
  4. System saves as draft (editable until submit-and-lock).
- **Alternative Flows:** A1) Component-level entry aggregating to the subject (SUB-003/EXM-002). A2) Special outcome instead of a numeric mark (UC-EXM-005).
- **Exception Flows:** E1) Mark > maximum → reject the entry (UC-EXM-016). E2) Non-owner entry → blocked (EXM-005). E3) Outside the window → blocked (UC-EXM-018/EXM-009).
- **Validation Rules:** Mark within [0, max]; marker owns the subject; within window; component structure honored (EXM-002/004/005/009).
- **Permissions Required:** `exam.marks.enter` (ownership-scoped).
- **Notifications Triggered:** None routine; progress visible to the controller.
- **Audit Events Generated:** `MARKS_ENTERED` (draft) (subject, section, marker).
- **Data Created:** Draft marks.
- **Data Updated:** Draft mark sheet.
- **Data Deleted:** None.
- **Post Conditions:** Draft marks saved, editable until lock.
- **Related Use Cases:** UC-EXM-004, UC-EXM-005, UC-SUB-003.
- **Acceptance Criteria:**
  - Given an owned subject within the window, When valid marks are entered, Then they save as draft.
  - Given a mark above the maximum, When entered, Then it is rejected.
  - Given a non-owner, When they attempt entry, Then it is blocked.
- **Edge Case Analysis:**
  - *Invalid Input:* non-numeric/negative mark rejected.
  - *Permission Failure:* non-owner → 403.
  - *Concurrent Update:* co-teacher concurrent entry → ownership-scoped; last draft save wins, audited.
  - *Duplicate Data:* re-entry overwrites draft (not duplicate rows).
  - *System Failure:* draft autosave; no partial corruption.
  - *Workflow Failure:* N/A (correction is post-lock).
- **QA Coverage:**
  - *Positive:* valid entry; component aggregation; special outcome.
  - *Negative:* over-maximum; non-owner; outside window.
  - *Boundary:* mark exactly at maximum and at zero; entry at window edges.

### UC-EXM-004 — Submit & Lock Marks (immutable)
- **Module:** Examination · **Priority:** Critical
- **Actors:** Subject-Teacher (primary), System
- **Goal:** Finalize marks so they become immutable and feed result processing.
- **Description:** On submit, draft marks are locked; further changes require the Governed Correction Pattern (approval), never a silent edit. Completeness is checked.
- **Business Rules Applied:** EXM-007, EXM-009, RES-005 (completeness), Cross-Cutting P5.
- **Preconditions:** Draft marks entered; completeness satisfied or flagged.
- **Trigger:** Teacher submits the mark sheet.
- **Main Success Scenario:**
  1. System verifies completeness (all students have a mark or a special outcome, EXM-009/RES-005).
  2. Teacher submits; System locks the marks immutably (EXM-007).
  3. Locked marks become available to result processing.
- **Alternative Flows:** A1) Partial submit blocked until complete or outcomes recorded.
- **Exception Flows:** E1) Post-lock edit attempt → blocked; requires governed correction (UC-EXM-017/010). E2) Incomplete at window close → escalation/flag (UC-EXM-018).
- **Validation Rules:** Completeness satisfied; lock irreversible except via governed correction (EXM-007/009).
- **Permissions Required:** `exam.marks.submit` (ownership-scoped).
- **Notifications Triggered:** `MARKS_SUBMITTED` to the controller.
- **Audit Events Generated:** `MARKS_LOCKED` (subject, section, marker).
- **Data Created:** None.
- **Data Updated:** Marks status → locked.
- **Data Deleted:** None.
- **Post Conditions:** Marks immutable; result processing can proceed.
- **Related Use Cases:** UC-EXM-003, UC-EXM-010, UC-RES-001.
- **Acceptance Criteria:**
  - Given complete marks, When submitted, Then they lock immutably and feed results.
  - Given a post-lock edit attempt, When made, Then it is blocked and a governed correction is required.
  - Given incomplete marks at window close, When reached, Then it is flagged/escalated (no silent zeros).
- **Edge Case Analysis:**
  - *Invalid Input:* submit with missing marks blocked.
  - *Permission Failure:* non-owner submit → 403.
  - *Concurrent Update:* two submits → idempotent lock; one lock event.
  - *Duplicate Data:* re-submit after lock → no-op/blocked.
  - *System Failure:* lock atomic with audit (outbox); no half-locked sheet.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* complete submit → lock.
  - *Negative:* incomplete submit; post-lock edit; double submit.
  - *Boundary:* submit at window close exactly; last student's mark completing the sheet.

### UC-EXM-006 — Apply Moderation / Grace (bounded, transparent)
- **Module:** Examination · **Priority:** High
- **Actors:** Examination Controller (primary), Approver, System
- **Goal:** Apply moderation or grace marks within bounded, transparent, audited rules.
- **Description:** Moderation/grace adjustments are bounded by configuration, applied transparently (recorded as adjustments, not silent rewrites), and governed; the original marks remain visible.
- **Business Rules Applied:** EXM-008, EXM-007, AUTHZ-009, Cross-Cutting P5.
- **Preconditions:** Marks locked; moderation policy configured; controller authorized.
- **Trigger:** Controller applies moderation/grace.
- **Main Success Scenario:**
  1. Controller defines the moderation/grace (e.g., +N grace to reach pass, subject scaling) within configured bounds.
  2. System validates the adjustment is within policy bounds (EXM-008).
  3. System applies it as a recorded, attributed adjustment over the locked marks; originals preserved.
  4. Approval is obtained where required (SoD).
- **Alternative Flows:** A1) Grace applied only to near-pass cases per policy.
- **Exception Flows:** E1) Adjustment beyond bounds → blocked. E2) Ungoverned/silent moderation → blocked.
- **Validation Rules:** Within configured bounds; transparent/recorded; approved where required (EXM-008, AUTHZ-009).
- **Permissions Required:** `exam.moderation.manage` (+ approval).
- **Notifications Triggered:** Moderation applied notice to admins.
- **Audit Events Generated:** `MODERATION_APPLIED` (rule, bounds, actor).
- **Data Created:** Moderation/adjustment records.
- **Data Updated:** Effective marks (originals retained).
- **Data Deleted:** None.
- **Post Conditions:** Bounded, transparent adjustment applied; results recompute.
- **Related Use Cases:** UC-EXM-004, UC-RES-001.
- **Acceptance Criteria:**
  - Given a within-bounds moderation, When applied, Then it is recorded transparently and originals are preserved.
  - Given an out-of-bounds adjustment, When attempted, Then it is blocked.
  - Given a moderation requiring approval, When applied, Then SoD is enforced.
- **Edge Case Analysis:**
  - *Invalid Input:* malformed rule rejected.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* two moderations → serialized; combined effect bounded and audited.
  - *Duplicate Data:* idempotent identical moderation.
  - *System Failure:* atomic; recompute deterministic.
  - *Workflow Failure:* approval pends.
- **QA Coverage:**
  - *Positive:* grace-to-pass; bounded scaling.
  - *Negative:* over-bound; silent moderation; self-approval.
  - *Boundary:* grace exactly reaching pass; bound limit.

---

## 7. Compact Specifications (routine use cases)

- **UC-EXM-001 — Define Examination (maxima, components, schedule)** · *High* · Rules: EXM-001, EXM-002, SUB-003. Configure exam type/schedule, per-subject maxima/components/weightage (versioned). *Permissions:* `exam.config.manage`. *Audit:* `EXAM_DEFINED`. *Edge:* maxima/components consistent with subject; versioned. *QA:* define; inconsistent maxima rejected.
- **UC-EXM-002 — Determine Eligibility & Issue Admit Card** · *High* · Rules: EXM-003, ATT-009, FEE (dues). Eligibility from attendance/dues; admit card issued to eligible only (immutable doc). *Audit:* `ELIGIBILITY_DETERMINED`, `ADMIT_CARD_ISSUED`. *Edge:* ineligible blocked/conditional; admit card signed-URL. *QA:* eligible issue; ineligible block; threshold boundary.
- **UC-EXM-005 — Record Special Outcome (absent/exempt/malpractice)** · *High* · Rules: EXM-006. Record distinct non-numeric outcomes affecting results differently. *Audit:* `SPECIAL_OUTCOME_RECORDED`. *Edge:* malpractice vs absence vs exemption handled distinctly downstream. *QA:* each outcome distinct; result impact correct.
- **UC-EXM-007 — View Exam / Marks (scoped)** · *Medium* · Rules: AUTHZ-003, STU-005. Scoped/owned view; students see own (post-publish). *Edge:* pre-publish marks not visible to students. *QA:* scope/ownership; pre-publish hidden.
- **UC-EXM-008 — Update Exam Definition (pre-marks)** · *Medium* · Rules: EXM-001, EXM-007. Edit definition before marks exist; locked once marks entered. *Edge:* post-marks definition change blocked. *QA:* pre-marks edit; post-marks blocked.
- **UC-EXM-009 — Configure Eligibility & Window** · *High* · Rules: EXM-003, EXM-009, CFG-004. Set eligibility criteria and mark-entry window (versioned). *Permissions:* `exam.config.manage`. *Edge:* window enforced on entry. *QA:* config applies; window effect.
- **UC-EXM-010 — Approve Post-Lock Mark Correction** · *High* · Rules: EXM-007, AUTHZ-009, WFL-004 (P5). Approver gates post-lock corrections (SoD). *Edge:* self-approval blocked; escalation. *QA:* approval gates; SoD.
- **UC-EXM-011 — Search Exams / Mark Sheets** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-EXM-012 — Mark-Entry Progress & Exam Report** · *Medium* · Rules: REP-002, EXM-009. Entry-completion tracking, pending sheets. *Edge:* scoped; as-of. *QA:* progress accuracy.
- **UC-EXM-013 — Bulk Mark Entry / Import** · *High* · Rules: EXM-004/005/009. Bulk/import marks (validated, ownership/window enforced). *Edge:* over-max/non-owner/out-of-window rows rejected; partial-failure report. *QA:* clean import; invalid rows; ownership.
- **UC-EXM-014 — Export Marks (governed)** · *Medium* · Rules: REP-005, STU-005. Governed, scoped export. *Permissions:* `report.export`. *Edge:* bulk PII gated. *QA:* scoped; gated.
- **UC-EXM-015 — Mark-Correction Workflow** · *High* · Rules: WFL-002/004, EXM-007 (P5). Version-pinned governed correction over locked marks. *QA:* pinning; SoD; original preserved.
- **UC-EXM-016 — Mark Exceeds Maximum Blocked (Exception)** · *High* · Rules: EXM-004. Mark > max rejected. *QA:* blocked; at-max allowed.
- **UC-EXM-017 — Edit After Lock Blocked (Exception)** · *Critical* · Rules: EXM-007. Direct post-lock edit blocked; governed correction only. *QA:* edit blocked; correction path works.
- **UC-EXM-018 — Incomplete Mark Entry at Window Close (Exception)** · *High* · Rules: EXM-009, RES-005. Incomplete entry flagged/escalated; no silent zeros. *QA:* flagged; escalation; no auto-zero.

## 8. Module-level QA & Edge Themes
- **Immutability (EXM-007 / P5):** submit-and-lock and the no-silent-edit rule are the headline integrity suites; post-lock changes only via governed correction.
- **Ownership-scoped entry (EXM-005 / SUB-004):** only the subject owner enters marks; reassignment ends live entry immediately (C-05).
- **Validation against maxima (EXM-004):** boundary suite at 0/max; component aggregation correctness.
- **Bounded transparent moderation (EXM-008):** adjustments recorded, not silent; within configured bounds; originals preserved.
- **No silent zeros (RES-005):** incompleteness is surfaced, never defaulted.
