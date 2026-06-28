# 14 — Examination Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`EXM-001…009`), and Use Case Repository (`UC-EXM-001…018`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Define examinations and capture marks with integrity: configurable exam definitions with per-subject maxima/components, eligibility + admit-card gating, ownership-scoped mark entry within a window, distinct special outcomes, submit-and-lock immutability, and bounded transparent moderation.

**Business Goal.** Produce trustworthy, tamper-resistant marks — entered only by the owning teacher, validated against maxima, locked on submission — that feed result processing without any silent edits.

**Scope.** Exam definition (type, schedule, per-subject maxima/components/weightage from Subject); eligibility & admit card; mark entry (ownership-scoped, validated, windowed); special outcomes (absent/exempt/malpractice); submit-and-lock immutability; post-lock governed correction; bounded transparent moderation/grace; mark-entry progress.

**Out of Scope.** Result computation/aggregation (Result module — consumes locked marks). Grade resolution (Grading module). Subject component definitions (Subject module — Examination consumes them via EXM-002). Eligibility threshold computation (Attendance module — Examination consumes ATT-009).

---

## 2. Actors

**Primary Actors.** Examination Controller / Administrator (defines exams, moderation), Subject-Teacher (enters marks for owned subjects), Examiner/Invigilator (records outcomes), Approver (post-lock corrections — SoD).

**Secondary Actors.** Subject module (maxima/components/ownership), Attendance module (eligibility), Result module (consumer), File module (admit cards), Workflow Engine (correction/moderation approval), Configuration Engine (windows/eligibility), Audit, Reporting, Notification.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Configurable exam definition | Define exam type/schedule with per-subject maxima/components/weightage. | High | Subject (EXM-002) |
| FR-002 | Eligibility & admit card | Determine eligibility (attendance/dues) and issue admit cards to eligible only. | High | Attendance, Fee, File |
| FR-003 | Ownership-scoped mark entry | Only the owning subject-teacher enters marks for that subject/section. | Critical | Subject (SUB-004) |
| FR-004 | Mark validation against maxima | Reject marks above the configured maximum. | Critical | FR-001 |
| FR-005 | Mark-entry window & completeness | Enforce a window; surface incompleteness (no silent zeros). | High | Config, Result (RES-005) |
| FR-006 | Special outcomes | Record absent/exempt/malpractice as distinct, non-numeric outcomes. | High | Result |
| FR-007 | Submit-and-lock immutability | Lock marks on submit; post-lock changes only via governed correction. | Critical | Workflow (P5) |
| FR-008 | Bounded transparent moderation | Apply moderation/grace within bounds, recorded transparently. | High | Workflow, AUTHZ-009 |
| FR-009 | Post-lock correction (governed) | Correct locked marks only with SoD approval, preserving original. | High | Workflow Engine |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Configurable exams | Type/schedule/maxima/components. | Fits any assessment model. |
| Eligibility & admit cards | Gate sitting; issue cards. | Policy-driven, fair access. |
| Ownership-scoped entry | Only owner marks. | Accountability; access control. |
| Maxima validation | No mark above max. | Data integrity. |
| Submit-and-lock | Immutable on submit. | Tamper-resistance. |
| Special outcomes | Absent/exempt/malpractice distinct. | Correct downstream handling. |
| Bounded moderation | Transparent, recorded grace. | Fairness with auditability. |
| Governed corrections | Approved, original-preserving. | Integrity; no silent edits. |

---

## 5. Screens

Exam List; Exam Definition (maxima/components/schedule); Eligibility & Admit Card; Mark Entry (roster, windowed, owned); Special Outcome Entry; Submit & Lock; Moderation/Grace; Post-Lock Correction; Mark-Entry Progress; Exam/Mark Sheet Search; Mark Report; Bulk Mark Entry/Import; Mark Export; Correction Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Exam List | Define, Open, Search/Filter, Export | — |
| Exam Definition | Set Maxima/Components/Schedule, Save (versioned), Edit (pre-marks) | — |
| Eligibility & Admit Card | Determine Eligibility, Issue Admit Cards, Override (governed) | Bulk Issue |
| Mark Entry | Enter Marks, Mark Special Outcome, Save Draft, Submit & Lock | Bulk Entry/Import |
| Moderation/Grace | Apply (bounded), Preview, Submit (→ approval) | — |
| Post-Lock Correction | Propose Correction (reason), Submit (→ approval) | — |
| Mark-Entry Progress | View Pending Sheets, Remind | — |
| Mark Report | Run, Filter, Export | Export |

---

## 7. Forms

**Exam Definition** — `name`/`type` (required), `schedule` (dates), per-subject `maxMarks`, `passMark`, `components` (from Subject), `weightage`. Validation: maxima/components consistent with Subject (EXM-002/SUB-003); versioned (CFG-004).

**Mark Entry** — per-student `mark` (number) or `specialOutcome` (absent/exempt/malpractice). Validation: within [0, max] (EXM-004); marker owns subject/section (EXM-005/SUB-004); within window (EXM-009); component structure honored.

**Submit & Lock** — confirm. Validation: completeness (all students have mark or outcome — EXM-009/RES-005); lock irreversible except governed correction (EXM-007).

**Moderation/Grace** — `rule` (grace-to-pass / scaling), `bound` (within config). Validation: within configured bounds (EXM-008); transparent/recorded; approved where required.

**Post-Lock Correction** — `student`, `newMark`, `reason` (text, required). Validation: locked marks (EXM-007); SoD approval (AUTHZ-009); original preserved (P5).

**Eligibility/Window Config** — `eligibilityCriteria` (attendance %/dues), `markEntryWindow` (structured). Validation: typed/versioned (CFG-002/004); threshold consumes ATT-009.

---

## 8. Search & Filter Requirements

**Exams/Marks:** by exam, subject, section, marker, status (draft/locked/moderated), entry-progress. Sorting: exam/subject/status. Pagination: server-side, 25 default. Scope + ownership (teachers see own subjects; students see own post-publish).

---

## 9. Table Requirements

**Mark sheet table:** Roll No., Student, Component marks, Total, Status (draft/locked), Special Outcome. **Progress table:** Subject/Section, Marker, Entered/Total, Locked?. Sorting on Roll/Status. Filtering as above. Export (governed). Bulk: entry/import.

---

## 10. Workflow Requirements

**Trigger events:** define exam, issue admit card, enter/submit marks, lock, special outcome, moderation, post-lock correction. **Status changes:** marks `DRAFT → LOCKED → CORRECTED(governed)`; exam `DEFINED → IN_PROGRESS → COMPLETED`. **Approvals:** post-lock corrections and moderation via Workflow Engine (SoD). **Notifications:** admit card issued, mark-entry reminders, correction outcomes. **Audit:** marking, locking, moderation, corrections (before/after, immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Define/configure exam | `exam.config.manage` |
| Determine eligibility / issue admit card | `exam.eligibility.manage` |
| Enter marks (owned) | `exam.marks.enter` |
| Submit & lock marks | `exam.marks.submit` |
| Apply moderation | `exam.moderation.manage` |
| Approve post-lock correction | `exam.correction.approve` |
| View/search/report | `exam.view`, `exam.report.view` |
| Export marks | `report.export` |

Mark entry/submit are ownership-scoped (EXM-005); corrections/moderation enforce SoD.

---

## 12. Business Rule References

EXM-001 (configurable exam definition), EXM-002 (per-subject maxima/pass/components), EXM-003 (eligibility & admit-card gating), EXM-004 (mark validation against maxima), EXM-005 (ownership-scoped mark entry), EXM-006 (absent/exempt/malpractice distinct), EXM-007 (submit-and-lock immutability), EXM-008 (bounded transparent moderation), EXM-009 (mark-entry window & completeness). Cross-cutting: SUB-003/004 (components/ownership), ATT-009 (eligibility threshold), RES-005 (completeness/no silent zeros), FILE-007 (admit card immutable), WFL-002/004 (correction approval), AUTHZ-009 (SoD), AUD-001.

## 13. Use Case References

UC-EXM-001 (Define Exam), UC-EXM-002 (Eligibility & Admit Card), UC-EXM-003 (Enter Marks — ownership-scoped), UC-EXM-004 (Submit & Lock), UC-EXM-005 (Special Outcome), UC-EXM-006 (Moderation/Grace), UC-EXM-007 (View Marks), UC-EXM-008 (Update Definition pre-marks), UC-EXM-009 (Configure Eligibility & Window), UC-EXM-010 (Approve Post-Lock Correction), UC-EXM-011 (Search), UC-EXM-012 (Mark-Entry Progress), UC-EXM-013 (Bulk Mark Entry/Import), UC-EXM-014 (Export Marks), UC-EXM-015 (Mark-Correction Workflow), UC-EXM-016 (Mark Exceeds Maximum Blocked), UC-EXM-017 (Edit After Lock Blocked), UC-EXM-018 (Incomplete Mark Entry at Window Close).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Define / update exam (pre-marks) | POST/PUT | Controller |
| Determine eligibility / issue admit cards | POST | Controller |
| Get markable roster (owned, windowed) | GET | Subject-Teacher |
| Enter / save draft marks | POST | Subject-Teacher |
| Submit & lock marks | POST | Subject-Teacher |
| Record special outcome | POST | Subject-Teacher |
| Apply moderation/grace | POST | Controller (→ approval) |
| Propose / approve post-lock correction | POST | Teacher / Approver |
| Mark-entry progress / exam report | GET | Controller |
| Bulk mark entry / import / export | POST/GET | Controller |

Mark entry validates ownership + maxima + window server-side (EXM-004/005/009); lock is irreversible except via governed correction (EXM-007).

---

## 15. Database Requirements

**Entities:** `Exam` (type, schedule, status), `ExamSubjectConfig` (maxima, passMark, components, weightage, version), `Mark` (studentId, examId, subjectId, component marks, total, status, specialOutcome, lockedAt), `MarkCorrection` (old/new, reason, approver), `Moderation` (rule, bound, actor), `AdmitCard` (eligibility, file ref). **Relationships:** Exam 1—* ExamSubjectConfig; Exam 1—* Mark; Mark 1—* Correction. **Indexes:** unique(Mark.examId, studentId, subjectId), index(Mark.status), index(Mark.subjectId, sectionId). Lock state immutable at data layer (EXM-007); marks feed Result.

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Admit card issued; mark-entry reminders to teachers; correction outcomes. |
| In-App | Mark-entry progress; pending corrections/moderation approvals. |
| SMS/Push | Admit card / exam schedule alerts (minimized). |

Admit cards delivered via signed URL (FILE-005); recipient routing custody-aware (NOT-002).

---

## 17. Audit Requirements

Log: exam definition, eligibility determination, admit-card issuance, mark entry (draft), submit-and-lock, special outcomes, moderation (rule/bound/actor), post-lock corrections (before/after, reason, approver). Record who/when/before/after. Lock and correction are first-class audit events. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Mark-entry progress, Exam mark sheets, Moderation log, Eligibility/admit-card status, Subject performance. **Exports:** governed mark export (scoped). **Dashboards:** exam control (entry completion, pending locks, ineligible students).

---

## 19. Error Handling

**Validation:** mark > maximum, post-lock edit, marking outside window, incomplete at close → specific errors (UC-EXM-016/017/018). **Permission:** non-owner entry → 403. **Workflow:** correction/moderation pending → clear state; self-approval → SoD block. **System:** Attendance eligibility service unavailable → eligibility deferred (no silent pass/fail); lock atomic with audit (outbox).

---

## 20. Edge Cases

**Concurrent updates:** co-teacher concurrent entry → ownership-scoped; last draft save wins, audited. **Duplicate data:** re-entry overwrites draft (not duplicate). **Partial failures:** bulk entry partial → per-row report (over-max/non-owner/out-of-window rejected). **Rollback:** correction approval reversed → prior locked mark restored, original preserved. **Lock race:** double submit → idempotent single lock event.

---

## 21. Acceptance Criteria

**Functional.** Only the owning subject-teacher enters marks, validated against maxima, within the window; special outcomes are distinct from zero; submit locks marks immutably and post-lock changes require SoD-approved governed correction preserving the original; moderation/grace is bounded, transparent, and recorded; incompleteness is surfaced, never silently zeroed.

**Business.** Marks are trustworthy and tamper-resistant; no teacher can quietly change a grade after submission; the locked, complete, validated marks feed result processing with full provenance and audit.

---

## 22. Future Enhancements

OMR/scanner mark import; on-screen marking for digital scripts; question-wise analytics; configurable moderation algorithms; re-evaluation request intake (feeding RES-006); invigilator/seating plans; offline mark-entry sync.
