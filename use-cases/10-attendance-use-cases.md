# 10 — Attendance Use Cases

Transforms the Attendance business rules (`ATT-001`…`ATT-009`) into use cases. The daily operational heartbeat: configurable modes/statuses, ownership by mode, active-enrollment-only marking, windowed/backdating control, leave reflection, holiday exclusion, governed corrections, safeguarding alerts, and exam-eligibility summaries.

## 1. Primary Actors
Teacher / Class-Teacher (marks attendance for owned sections/subjects), Attendance Administrator (configures modes, corrects, oversees).

## 2. Secondary Actors
System (window enforcement, holiday/leave reflection, summary computation, safeguarding signals), Leave module (approved leave → excused), Notification module (absence alerts), Exam module (eligibility consumer), Audit service.

## 3. Goals
Capture accurate attendance per the configured mode; restrict marking to active enrollments and the allowed window; reflect approved leave as excused; exclude non-instructional days; correct only via governed flows; alert guardians and surface safeguarding signals on absence; compute summaries that drive exam eligibility.

## 4. User Journeys
- **Daily marking:** teacher opens today's roster for an owned section/subject → marks statuses → submits within the window → guardians of absentees are notified.
- **Leave & holidays:** approved student leave shows as excused (not absent); holidays are excluded from the denominator.
- **Correction:** a mistaken mark is fixed via a governed, audited correction (not a silent edit).
- **Eligibility:** at term end, attendance summaries determine exam eligibility against the configured threshold.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-ATT-001 | Mark Attendance (windowed) | Critical |
| Core | UC-ATT-002 | Approved Leave Reflects as Excused | High |
| Core | UC-ATT-003 | Absence Notification & Safeguarding Signal | High |
| Core | UC-ATT-004 | Compute Attendance Summary & Eligibility | High |
| Core | UC-ATT-005 | Correct Attendance (governed) | High |
| CRUD | UC-ATT-006 | View Attendance (roster / student) | Medium |
| Admin | UC-ATT-007 | Configure Mode, Statuses & Threshold | High |
| Admin | UC-ATT-008 | Backdated / Window-Override Marking | Medium |
| Admin | UC-ATT-009 | Manage Holidays / Non-Instructional Days | Medium |
| Approval | UC-ATT-010 | Approve Attendance Correction | Medium |
| Search | UC-ATT-011 | Search Attendance Records | Low |
| Reporting | UC-ATT-012 | Attendance Report & Defaulter List | High |
| Bulk | UC-ATT-013 | Bulk / Import Attendance | Medium |
| Export | UC-ATT-014 | Export Attendance | Low |
| Workflow | UC-ATT-015 | Correction Workflow | Medium |
| Exception | UC-ATT-016 | Marking Outside Window Blocked | High |
| Exception | UC-ATT-017 | Marking Inactive Enrollment Blocked | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-ATT-001 — Mark Attendance (windowed)
- **Module:** Attendance · **Priority:** Critical
- **Actors:** Teacher / Class-Teacher (primary), System
- **Goal:** Record attendance for an owned roster within the allowed window, for active enrollments only.
- **Description:** The owner (by mode — class-teacher for daily, subject-teacher for period) marks statuses for active-enrolled students within the marking window; holidays/leave are pre-reflected.
- **Business Rules Applied:** ATT-001, ATT-002, ATT-003, ATT-004, ATT-006.
- **Preconditions:** Instructional day; marker owns the section/subject (SEC-004/SUB-004); within the marking window.
- **Trigger:** Teacher opens the roster and submits attendance.
- **Main Success Scenario:**
  1. System presents the roster of active enrollments (ATT-003) for an instructional day (ATT-006).
  2. Teacher marks each student with a configured status (ATT-001); granularity per mode (ATT-002).
  3. System validates the marking window (ATT-004) and ownership.
  4. System records attendance; approved leave pre-fills as excused (ATT-005).
- **Alternative Flows:** A1) Period/subject mode → per-period marking by subject-teacher. A2) Bulk "all present" then adjust exceptions.
- **Exception Flows:** E1) Outside the window → blocked; needs override (UC-ATT-016/008). E2) Inactive enrollment in roster → not markable (UC-ATT-017). E3) Holiday → marking disabled (ATT-006).
- **Validation Rules:** Active enrollment; instructional day; within window; marker owns the roster; valid status (ATT-001/002/003/004/006).
- **Permissions Required:** `attendance.mark` (ownership-scoped).
- **Notifications Triggered:** Triggers absence alerts downstream (UC-ATT-003).
- **Audit Events Generated:** `ATTENDANCE_MARKED` (section, date, marker).
- **Data Created:** Attendance records.
- **Data Updated:** Daily/period attendance state.
- **Data Deleted:** None.
- **Post Conditions:** Attendance recorded for the day; summaries and alerts update.
- **Related Use Cases:** UC-ATT-002, UC-ATT-003, UC-ATT-005.
- **Acceptance Criteria:**
  - Given an owned roster on an instructional day within the window, When marked, Then attendance is recorded for active enrollments.
  - Given an attempt outside the window, When submitted, Then it is blocked pending override.
  - Given a holiday, When marking is attempted, Then it is disabled.
  - Given a student on approved leave, When the roster loads, Then they are pre-marked excused.
- **Edge Case Analysis:**
  - *Invalid Input:* invalid/unsupported status rejected.
  - *Permission Failure:* non-owner marking another's section → 403.
  - *Concurrent Update:* two markers (e.g., class vs period mode) → mode determines the authoritative owner (ATT-002); no conflict.
  - *Duplicate Data:* re-submitting the same day → updates within window, not duplicate records.
  - *System Failure:* atomic submit; partial roster not half-saved.
  - *Workflow Failure:* N/A (correction is separate).
- **QA Coverage:**
  - *Positive:* daily mark; period mark; leave pre-fill.
  - *Negative:* outside window; inactive enrollment; holiday; non-owner.
  - *Boundary:* marking at the exact window open/close; full-present vs all-absent.

### UC-ATT-003 — Absence Notification & Safeguarding Signal
- **Module:** Attendance · **Priority:** High
- **Actors:** System (primary), Class-Teacher / Safeguarding Lead (recipients)
- **Goal:** Notify guardians of absences and surface safeguarding signals (e.g., unexplained or repeated absence).
- **Description:** On recorded absence, the system notifies the appropriate guardian (custody-aware) and raises safeguarding signals when patterns/thresholds indicate concern.
- **Business Rules Applied:** ATT-008, NOT-002, NOT-003, GRD-N-006.
- **Preconditions:** Attendance marked with absence(s).
- **Trigger:** Absence recorded; or a safeguarding pattern detected.
- **Main Success Scenario:**
  1. System detects an absence (especially unexplained/consecutive).
  2. System resolves the custody-appropriate guardian recipient (NOT-002).
  3. System sends a mandatory absence notification (safeguarding category overrides opt-out, NOT-003).
  4. On configured patterns (e.g., N consecutive unexplained), System raises a safeguarding signal to the lead.
- **Alternative Flows:** A1) Pre-approved leave → no absence alert (excused).
- **Exception Flows:** E1) Custody-restricted party → excluded; alert goes to the permitted guardian (GRD-N-006).
- **Validation Rules:** Absence genuine (not excused); recipient custody-appropriate; safeguarding thresholds applied (ATT-008).
- **Permissions Required:** System-generated; safeguarding view needs `safeguarding.view`.
- **Notifications Triggered:** `STUDENT_ABSENT` to guardian; `SAFEGUARDING_SIGNAL` to the lead.
- **Audit Events Generated:** `ABSENCE_NOTIFIED`, `SAFEGUARDING_SIGNAL_RAISED`.
- **Data Created:** Notification/signal records.
- **Data Updated:** Absence-pattern counters.
- **Data Deleted:** None.
- **Post Conditions:** Guardian informed; safeguarding concerns surfaced to the right person.
- **Related Use Cases:** UC-ATT-001, UC-NOT-01, UC-GRD-N-004.
- **Acceptance Criteria:**
  - Given a recorded unexplained absence, When processed, Then a custody-appropriate guardian receives a mandatory alert.
  - Given a custody restriction, When alerting, Then the restricted party is excluded.
  - Given N consecutive unexplained absences, When detected, Then a safeguarding signal is raised.
- **Edge Case Analysis:**
  - *Invalid Input:* excused/leave day → no false alert.
  - *Permission Failure:* safeguarding view without permission → denied.
  - *Concurrent Update:* late correction to "present" → suppress/retract pending alert appropriately.
  - *Duplicate Data:* one alert per absence event (deduped, NOT-006).
  - *System Failure:* delivery failure retried; signal not lost (outbox).
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* absence alert; safeguarding threshold trigger.
  - *Negative:* leave (no alert); custody-restricted exclusion.
  - *Boundary:* exactly the consecutive-absence threshold; correction just before/after alert send.

### UC-ATT-004 — Compute Attendance Summary & Eligibility
- **Module:** Attendance · **Priority:** High
- **Actors:** System (primary), Exam module (consumer), Teacher/Admin (viewer)
- **Goal:** Compute per-student attendance percentages on the correct denominator and determine exam eligibility.
- **Description:** Aggregates attendance excluding non-instructional days, counting excused leave per policy, and compares against the configured eligibility threshold used by exams.
- **Business Rules Applied:** ATT-009, ATT-006, ATT-005, EXM (eligibility).
- **Preconditions:** Attendance recorded over a period; threshold configured.
- **Trigger:** Summary requested / exam-eligibility check.
- **Main Success Scenario:**
  1. System computes attendance % = present (+ excused per policy) / instructional days (ATT-006/005).
  2. System compares against the configured eligibility threshold (ATT-009).
  3. System flags eligible/ineligible for exams; results feed the Exam module.
- **Alternative Flows:** A1) Policy variations (excused counts toward or neutral to the denominator) per configuration.
- **Exception Flows:** E1) Ineligible student → flagged; exam registration blocked/conditional per policy.
- **Validation Rules:** Denominator excludes holidays; leave handled per policy; threshold versioned (ATT-009, CFG-004).
- **Permissions Required:** `attendance.report.view`; eligibility consumed by exam.
- **Notifications Triggered:** Eligibility-risk warnings to guardians (configurable).
- **Audit Events Generated:** `ELIGIBILITY_COMPUTED`.
- **Data Created:** Summary snapshots.
- **Data Updated:** Eligibility flags.
- **Data Deleted:** None.
- **Post Conditions:** Accurate summaries; eligibility determined consistently with exam rules.
- **Related Use Cases:** UC-EXM (eligibility), UC-ATT-012.
- **Acceptance Criteria:**
  - Given attendance over a term, When computed, Then the percentage excludes holidays and handles excused leave per policy.
  - Given a student below threshold, When eligibility is computed, Then they are flagged ineligible consistently with exam rules.
  - Given a config change to the threshold, When recomputed, Then the versioned threshold is applied transparently.
- **Edge Case Analysis:**
  - *Invalid Input:* period with zero instructional days → defined handling (no divide-by-zero).
  - *Permission Failure:* report view without permission → denied.
  - *Concurrent Update:* late correction → recompute reflects corrected data.
  - *Duplicate Data:* idempotent computation.
  - *System Failure:* recompute deterministic; snapshot versioned.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* normal %; excused handling; threshold pass.
  - *Negative:* below threshold; zero-instructional-day period.
  - *Boundary:* exactly at threshold (eligible vs not); 100%/0% cases.

### UC-ATT-005 — Correct Attendance (governed)
- **Module:** Attendance · **Priority:** High
- **Actors:** Teacher / Attendance Administrator (primary), System, Approver
- **Goal:** Fix an attendance error without silently overwriting the record.
- **Description:** Applies the Governed Correction Pattern (P5): the original mark is preserved; a correcting entry records old→new value, reason, and actor; post-window corrections require elevated permission/approval.
- **Business Rules Applied:** ATT-007, ATT-004, AUTHZ-009 (Cross-Cutting P5).
- **Preconditions:** An attendance record exists; correction justified.
- **Trigger:** A mistaken mark is identified.
- **Main Success Scenario:**
  1. Marker requests a correction with a reason.
  2. Within the window, the owner may correct directly; outside it, elevated permission/approval is required (ATT-007).
  3. System records a correcting entry (original + new + reason + actor); summaries recompute.
- **Alternative Flows:** A1) Correction routed through approval (UC-ATT-010/015).
- **Exception Flows:** E1) Ungoverned silent edit attempt → blocked. E2) Correction in a closed period → requires governed reopen (SESS-008).
- **Validation Rules:** Original preserved; reason recorded; authority sufficient for the timing (ATT-007, P5).
- **Permissions Required:** `attendance.mark` (in-window) / `attendance.correct` (out-of-window, elevated).
- **Notifications Triggered:** Correction notice if a prior alert was sent.
- **Audit Events Generated:** `ATTENDANCE_CORRECTED` (old→new, reason, actor).
- **Data Created:** Correcting entry.
- **Data Updated:** Effective attendance; summaries.
- **Data Deleted:** None (original retained).
- **Post Conditions:** Corrected transparently; full audit; downstream recomputed.
- **Related Use Cases:** UC-ATT-001, UC-ATT-004, UC-SESS-006.
- **Acceptance Criteria:**
  - Given an in-window error, When the owner corrects it, Then a correcting entry is recorded and the original preserved.
  - Given an out-of-window correction, When attempted, Then elevated permission/approval is required.
  - Given a closed-period correction, When needed, Then a governed reopen is required first.
- **Edge Case Analysis:**
  - *Invalid Input:* correction without reason rejected.
  - *Permission Failure:* out-of-window correction without elevation → 403.
  - *Concurrent Update:* two corrections → serialized; latest correcting entry wins, all retained.
  - *Duplicate Data:* idempotent identical correction.
  - *System Failure:* atomic; original never lost.
  - *Workflow Failure:* approval pends.
- **QA Coverage:**
  - *Positive:* in-window correction; approved out-of-window correction.
  - *Negative:* silent edit; out-of-window without elevation; closed-period without reopen.
  - *Boundary:* correction exactly at window close.

---

## 7. Compact Specifications (routine use cases)

- **UC-ATT-002 — Approved Leave Reflects as Excused** · *High* · Rules: ATT-005, LEV-009. Approved student leave auto-marks excused (not absent); Leave owns the workflow, Attendance reflects (C-04). *Audit:* reflected status. *Edge:* leave approval timing vs marking. *QA:* excused reflection; no false absence.
- **UC-ATT-006 — View Attendance (roster / student)** · *Medium* · Rules: AUTHZ-003. Scoped/owned view; guardian sees own child. *QA:* scope/ownership.
- **UC-ATT-007 — Configure Mode, Statuses & Threshold** · *High* · Rules: ATT-001/002/009, CFG-004. Configure daily/period mode, status set, eligibility threshold (versioned). *Permissions:* `attendance.config.manage`. *Edge:* mode change affects ownership/granularity. *QA:* config applies; versioned threshold.
- **UC-ATT-008 — Backdated / Window-Override Marking** · *Medium* · Rules: ATT-004, ATT-007. Governed backdating beyond the window (elevated/audited). *Edge:* limited days; reason required. *QA:* governed backdate; ungoverned blocked.
- **UC-ATT-009 — Manage Holidays / Non-Instructional Days** · *Medium* · Rules: ATT-006. Define holidays excluded from denominators/marking. *Edge:* retroactive holiday recomputes summaries. *QA:* holiday excludes; recompute.
- **UC-ATT-010 — Approve Attendance Correction** · *Medium* · Rules: ATT-007, WFL-004. Approver gates out-of-window corrections. *Edge:* SoD; escalation. *QA:* approval gates; self-approval blocked.
- **UC-ATT-011 — Search Attendance Records** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-ATT-012 — Attendance Report & Defaulter List** · *High* · Rules: REP-002, ATT-009. Reports incl. below-threshold defaulters. *Edge:* scoped; correct denominator. *QA:* counts/percentages accurate; defaulter threshold.
- **UC-ATT-013 — Bulk / Import Attendance** · *Medium* · Rules: ATT-003/004. Bulk mark / import (active-enrollment + window validated). *Edge:* per-row validation; inactive/holiday rows rejected. *QA:* clean bulk; invalid rows.
- **UC-ATT-014 — Export Attendance** · *Low* · Rules: REP-005. Scoped export. *QA:* scoped.
- **UC-ATT-015 — Correction Workflow** · *Medium* · Rules: WFL-002/004, ATT-007. Version-pinned correction approval. *QA:* pinning; SoD; escalation.
- **UC-ATT-016 — Marking Outside Window Blocked (Exception)** · *High* · Rules: ATT-004. Marking past the window blocked pending override. *QA:* blocked; override path.
- **UC-ATT-017 — Marking Inactive Enrollment Blocked (Exception)** · *High* · Rules: ATT-003. Cannot mark non-active enrollments. *QA:* blocked; active-only roster.

## 8. Module-level QA & Edge Themes
- **Correct denominator (ATT-006/009):** holiday exclusion and excused-leave handling are the headline correctness suites feeding exam eligibility.
- **Safeguarding (ATT-008):** mandatory, custody-aware absence alerts and pattern signals — a child-safety-critical suite.
- **Governed correction (ATT-007 / P5):** no silent edits; out-of-window and closed-period paths require escalating governance.
- **Ownership by mode (ATT-002):** daily vs period mode determines the authoritative marker — no double-ownership conflicts.
