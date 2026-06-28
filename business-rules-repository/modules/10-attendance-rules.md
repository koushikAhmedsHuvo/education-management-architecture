# 10 — Attendance Business Rules

## 1. Module Purpose
Govern **attendance** — the daily/period record of student presence, the highest-volume operational data in the system (range-partitioned, D16). Attendance is configurable (daily vs period/subject-wise), owned by the responsible teacher (class teacher for daily, subject teacher for period), constrained by marking windows, integrated with student leave and the academic calendar, and feeds exam-eligibility and guardian/safeguarding notifications. Because unexpected absence of a **minor** can be a safeguarding signal, this module treats absence as more than a statistic.

## 2. Actors
- **Class Teacher** — marks daily attendance for their section (ownership, Doc 12/SEC-004).
- **Subject Teacher** — marks period/subject attendance for assigned sections (Doc 13/SUB-004).
- **Administrator** — configures modes/statuses, corrects within governance, runs summaries.
- **Guardian / Student** — receive/read attendance (own children/self, ownership-scoped).
- **System** — enforces marking windows, eligibility, leave integration, holidays, and absence notifications.

## 3. Use Cases

**Use Case ID:** UC-ATT-01 — Mark Daily Attendance
**Actors:** Class Teacher, System
**Description:** Record presence for a section on a given day.
**Preconditions:** Day is instructional (not a holiday); teacher owns the section; marking window open.
**Main Flow:** 1) System presents the active-enrollment roster defaulted to a configured baseline (e.g., all present). 2) Teacher marks exceptions (absent/late/excused). 3) Teacher submits; attendance for the day is recorded and (per policy) locked.
**Alternative Flow:** A1) Period/subject mode: marked per period by the subject teacher (ATT-002).
**Exception Flow:** E1) Marking on a holiday → blocked (ATT-006). E2) Marking outside the window → blocked or flagged for backdate approval (ATT-004). E3) Student on approved leave → auto-excused, not absent (ATT-005).
**Post Conditions:** Attendance recorded; absences may trigger guardian/safeguarding alerts; audited.
**Business Rules Applied:** ATT-001, ATT-003, ATT-004, ATT-005, ATT-008.

**Use Case ID:** UC-ATT-02 — Correct Attendance
**Actors:** Teacher (within window) / Administrator, System
**Description:** Fix an erroneous attendance entry.
**Preconditions:** Entry exists; corrector authorized; within correction policy.
**Main Flow:** 1) Authorized user opens the entry. 2) Changes the status with a reason. 3) System records the correction with before/after and re-evaluates derived summaries/notifications.
**Exception Flow:** E1) Correction after the window/lock → requires elevated permission + reason (ATT-007). E2) Correction on a closed session → governed reopen only (SESS-005).
**Post Conditions:** Corrected entry; audit trail of the change; summaries updated.
**Business Rules Applied:** ATT-007.

**Use Case ID:** UC-ATT-03 — Attendance Summary & Eligibility
**Actors:** Administrator / Teacher / System
**Description:** Compute attendance percentages and flag exam-eligibility thresholds.
**Preconditions:** Attendance recorded over a period.
**Main Flow:** 1) System computes per-student attendance % over the configured basis. 2) Flags students below the minimum-attendance threshold for exam eligibility (if configured). 3) Surfaces summaries to staff and (scoped) guardians.
**Exception Flow:** E1) Threshold breach → eligibility flag feeds Examination (Doc 14) per policy.
**Post Conditions:** Summaries available; eligibility flags set; audited where consequential.
**Business Rules Applied:** ATT-009.

## 4. Business Rules

**Rule ID:** ATT-001
**Rule Name:** Configurable Attendance Mode & Statuses
**Description:** Attendance mode (daily vs period/subject) and the status set are configurable per institute/class/shift.
**Priority:** High
**Category:** Configuration
**Preconditions:** Attendance configuration.
**Business Rule:** Mode and statuses (e.g., Present, Absent, Late, Excused, Half-day, On-Leave) are configurable; the mode determines who marks and at what granularity. Shift influences the marking window (Doc 12/SEC-005).
**System Action:** Resolve mode/statuses via the Configuration Engine; apply per scope.
**Validation:** Mode/status set valid; consistent with grading/eligibility rules.
**Failure Behavior:** Reject undefined statuses; default to institute config.
**Audit Requirement:** Log attendance configuration changes.
**Example Scenario:** A primary school uses daily mode; a college uses period mode.
**Related Rules:** ATT-002, ATT-003.

**Rule ID:** ATT-002
**Rule Name:** Mode Determines Owner & Granularity
**Description:** Daily mode is owned by the class teacher; period mode by the assigned subject teacher.
**Priority:** High
**Category:** Authorization (ownership)
**Preconditions:** Marking.
**Business Rule:** Daily attendance is one record per student per day, marked by the class teacher; period attendance is per period/subject, marked by the assigned subject teacher. Only the owner (or an admin) may mark.
**System Action:** Route marking rights by mode and assignment.
**Validation:** Marker owns the section/subject for the date.
**Failure Behavior:** Reject marking outside ownership.
**Audit Requirement:** Captured in marking events with marker identity.
**Example Scenario:** The Physics teacher marks Physics-period attendance; the class teacher marks the daily register.
**Related Rules:** SEC-004, SUB-004, AUTHZ-003.

**Rule ID:** ATT-003
**Rule Name:** Attendance Only for Active Enrollments
**Description:** Attendance is recordable only for students with an active enrollment in the section for the date.
**Priority:** High
**Category:** Integrity
**Preconditions:** Marking.
**Business Rule:** The roster derives from active enrollments as-of the date; withdrawn/suspended/not-yet-enrolled students are excluded; transferred students appear on the correct section per their effective transfer date (ENR-005).
**System Action:** Build the date-accurate roster from enrollment state.
**Validation:** Enrollment active for student/section/date.
**Failure Behavior:** Block marking for non-enrolled students.
**Audit Requirement:** N/A beyond marking events.
**Example Scenario:** A student transferred to Section B on April 1 appears on B's roster from April 1, not A's.
**Related Rules:** ENR-005, ENR-008.

**Rule ID:** ATT-004
**Rule Name:** Marking Window & Backdating Control
**Description:** Attendance is marked within a configurable window; backdating beyond it is restricted and audited.
**Priority:** High
**Category:** Integrity / governance
**Preconditions:** Marking attempt for a date.
**Business Rule:** Marking is expected same-day (or within a configurable window). Backdated marking beyond the window requires elevated permission and a reason; far-future marking is forbidden. "Today" resolves in the institute time zone (SESS).
**System Action:** Enforce the window; gate backdating behind permission + reason.
**Validation:** Date within window or backdate authorized; not in the future.
**Failure Behavior:** Block out-of-window marking unless authorized; reject future dates.
**Audit Requirement:** Log `ATTENDANCE_BACKDATED` with reason and actor.
**Example Scenario:** A teacher forgetting Friday's register needs admin-authorized backdating on Monday.
**Related Rules:** ATT-007, SESS (time zone).

**Rule ID:** ATT-005
**Rule Name:** Approved Leave Reflects as Excused, Not Absent
**Description:** A student on approved leave is recorded as on-leave/excused, integrated from the leave workflow.
**Priority:** High
**Category:** Integration / fairness
**Preconditions:** An approved student leave overlaps the date.
**Business Rule:** Where approved student leave (Doc 24-equivalent / configurable) covers the date, attendance auto-reflects on-leave/excused rather than absent, and such days are handled per policy in attendance-% calculations.
**System Action:** Cross-reference approved leave; set the excused status; treat per policy in summaries.
**Validation:** Leave approved and covers the date/period.
**Failure Behavior:** If leave status is ambiguous, default to the teacher's mark and flag for review.
**Audit Requirement:** Log leave-driven status application.
**Example Scenario:** A student with approved medical leave is not marked absent for those days.
**Related Rules:** Leave management, ATT-009.

**Rule ID:** ATT-006
**Rule Name:** Holidays & Non-Instructional Days Excluded
**Description:** Attendance is not marked on holidays/non-instructional days per the academic calendar; retroactive holiday declaration adjusts records.
**Priority:** Medium
**Category:** Calendar integrity
**Preconditions:** Marking; academic calendar defined.
**Business Rule:** Marking is blocked on configured holidays/weekends per the institute/shift calendar. If a day is retroactively declared a holiday, existing attendance for it is neutralized (not counted) with audit.
**System Action:** Consult the calendar; block marking on non-instructional days; neutralize on retroactive holiday.
**Validation:** Date is instructional for the institute/shift.
**Failure Behavior:** Block marking on holidays; handle retroactive declaration via governed adjustment.
**Audit Requirement:** Log `HOLIDAY_ADJUSTMENT` affecting attendance.
**Example Scenario:** An unplanned closure declared after marking neutralizes that day's attendance for percentage purposes.
**Related Rules:** ATT-009, academic calendar (future/SESS).

**Rule ID:** ATT-007
**Rule Name:** Corrections Are Governed & Audited
**Description:** Attendance corrections are permitted within policy, always with reason and full audit; locked/closed records need elevated handling.
**Priority:** High
**Category:** Integrity
**Preconditions:** Correction of an existing entry.
**Business Rule:** Within the correction window the owner may correct with a reason; after lock or in a closed session, correction requires elevated permission (and governed reopen for closed sessions, SESS-005). All corrections keep before/after.
**System Action:** Apply correction with versioned history; re-derive summaries/notifications.
**Validation:** Corrector authorized for the record's state; reason provided.
**Failure Behavior:** Unauthorized/unreasoned correction rejected.
**Audit Requirement:** Log `ATTENDANCE_CORRECTED` with before/after, reason, actor.
**Example Scenario:** Fixing a wrongly-marked absence the next day is an audited correction, not a silent edit.
**Related Rules:** ATT-004, SESS-005.

**Rule ID:** ATT-008
**Rule Name:** Absence Notification & Safeguarding Signal
**Description:** Recorded absence of a student triggers timely guardian notification; unexplained/unexpected absence is a safeguarding signal.
**Priority:** High
**Category:** Child safety / communication
**Preconditions:** A student is marked absent (without prior approved leave).
**Business Rule:** Absence without approved leave triggers a same-day guardian notification per preference/policy; patterns of unexplained absence raise a safeguarding flag for staff review. Notifications respect custody/contact restrictions (GRD-N-006).
**System Action:** Send absence alerts; flag chronic/unexplained absence for review.
**Validation:** Absence confirmed and not leave-covered; recipient not custody-restricted.
**Failure Behavior:** If notification fails, retry via queue; never suppress the safeguarding flag.
**Audit Requirement:** Log `ABSENCE_NOTIFIED` and `SAFEGUARDING_FLAG_RAISED`.
**Example Scenario:** A child absent without notice prompts an SMS to the guardian-of-record the same morning.
**Related Rules:** GRD-N-006, NOT (notifications), ATT-005.

**Rule ID:** ATT-009
**Rule Name:** Attendance Summaries & Exam-Eligibility Threshold
**Description:** Attendance percentages are computed on a configurable basis and can gate exam eligibility.
**Priority:** Medium
**Category:** Assessment integration
**Preconditions:** Attendance recorded; thresholds configured.
**Business Rule:** Attendance % is computed over instructional days (excluding holidays; leave handled per policy); where a minimum-attendance rule exists, students below threshold are flagged ineligible for exams per policy (override possible, audited).
**System Action:** Compute summaries from read models; flag threshold breaches to Examination.
**Validation:** Basis configured; thresholds defined.
**Failure Behavior:** Threshold flags feed eligibility; overrides require authorization + reason.
**Audit Requirement:** Log eligibility flags and overrides.
**Example Scenario:** A student below 75% attendance is flagged ineligible to sit final exams unless an authorized override applies.
**Related Rules:** EXM (eligibility), ATT-006, ATT-005.

## 5. Validation Rules
- Marker owns the section (daily) or subject (period) for the date.
- Roster derives from active enrollments as-of the date.
- Date instructional and within the marking window (or authorized backdate); never future.
- Approved leave overrides to excused.
- Statuses ∈ configured set; summaries exclude holidays per policy.
- Corrections carry a reason and respect record state (window/lock/closed-session).

## 6. State Machine
*(Attendance "marking session" per section/day or section/subject/period)*

**State Name:** UNMARKED
**Description:** Instructional day/period with no attendance yet.
**Allowed Transitions:** → MARKED (submitted); → NEUTRALIZED (retroactive holiday).
**Forbidden Transitions:** marking on a non-instructional day.
**System Actions:** Present roster; await marking within window.

**State Name:** MARKED
**Description:** Attendance submitted for the day/period.
**Allowed Transitions:** → CORRECTED (within policy); → LOCKED (window closes); → NEUTRALIZED (retroactive holiday).
**Forbidden Transitions:** silent overwrite without correction audit.
**System Actions:** Trigger absence notifications; feed summaries.

**State Name:** CORRECTED
**Description:** A marked record adjusted with audit.
**Allowed Transitions:** → LOCKED; → CORRECTED (further governed corrections).
**Forbidden Transitions:** unreasoned correction.
**System Actions:** Version before/after; re-derive summaries/notifications.

**State Name:** LOCKED
**Description:** Window closed; record frozen for routine edits.
**Allowed Transitions:** → CORRECTED only via elevated permission; → ARCHIVED (session close).
**Forbidden Transitions:** routine edits.
**System Actions:** Require elevated handling for changes.

**State Name:** NEUTRALIZED
**Description:** Day retroactively non-instructional; excluded from percentages.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** counting toward attendance %.
**System Actions:** Exclude from summaries; audit the adjustment.

**State Name:** ARCHIVED
**Description:** Attendance for a closed/archived session; immutable.
**Allowed Transitions:** none (governed reopen only).
**Forbidden Transitions:** edits.
**System Actions:** Read-only retention.

## 7. Status Definitions
Marking-session: `UNMARKED` · `MARKED` · `CORRECTED` · `LOCKED` · `NEUTRALIZED` · `ARCHIVED`. Per-student statuses (configurable): `PRESENT` · `ABSENT` · `LATE` · `EXCUSED` · `ON_LEAVE` · `HALF_DAY`.

## 8. Workflow Rules
- Backdating beyond the window and post-lock corrections require elevated permission + reason (ATT-004/ATT-007).
- Retroactive holiday declaration is a governed adjustment that neutralizes affected attendance (ATT-006).
- Exam-eligibility overrides for below-threshold students require authorization + reason (ATT-009).
- Closed-session attendance changes require governed session reopen (SESS-005).

## 9. Permission Rules
- `attendance.mark` — mark attendance for owned sections/subjects.
- `attendance.correct` — correct within window (owner) / post-lock (admin, elevated).
- `attendance.backdate` — mark/adjust beyond the window (elevated).
- `attendance.view` — read attendance (teachers own scope; guardians/students own).
- `attendance.config.manage` — configure modes/statuses/thresholds.

## 10. Notification Rules
- `ABSENCE_NOTIFIED` → same-day guardian alert for unapproved absence (respecting custody restrictions, GRD-N-006).
- `SAFEGUARDING_FLAG_RAISED` → staff alert on chronic/unexplained absence.
- Attendance-threshold breach → notify guardian and academic staff (eligibility risk).
- Corrections affecting a prior notification → corrected notice per policy.

## 11. Audit Requirements
Mandatory: `ATTENDANCE_MARKED` (marker, mode, scope), `ATTENDANCE_CORRECTED` (before/after, reason), `ATTENDANCE_BACKDATED`, `HOLIDAY_ADJUSTMENT/NEUTRALIZED`, `ABSENCE_NOTIFIED`, `SAFEGUARDING_FLAG_RAISED`, eligibility flags/overrides. High-volume — audit captures change-events, not every read.

## 12. Data Retention Rules
- Attendance records retained per academic-record retention (often multi-year; some jurisdictions mandate attendance records).
- Partitioned by date for scale; older partitions archived per the archival strategy.
- Safeguarding flags retained per child-protection policy (potentially longer/with restricted access).
- Anonymization follows the student's erasure, preserving non-identifying aggregate integrity.

## 13. Edge Cases
- **Student transferred mid-day/mid-period:** roster attribution follows the effective transfer date (ATT-003/ENR-005).
- **Marking before enrollment finalized:** blocked — only active enrollments appear (ATT-003).
- **Approved leave vs marked absent conflict:** leave wins (excused), with review flag if ambiguous (ATT-005).
- **Retroactive holiday after marking:** marked attendance neutralized for % purposes, audited (ATT-006).
- **Time zone at day boundaries:** "today" resolves in the institute/shift time zone, not the server's (ATT-004/SESS).
- **Two teachers marking the same period (co-teaching):** ownership/serialization prevents conflicting double-marks.
- **Half-day / late thresholds:** configurable; a late arrival past a threshold may count as half-day or absent per policy.
- **Chronic absence of a minor:** escalates to a safeguarding flag, not just a low percentage (ATT-008).
- **Period mode with a free/library period:** non-instructional periods excluded from denominators.

## 14. Failure Scenarios
- **Marking for an inactive enrollment:** rejected (ATT-003).
- **Out-of-window/future marking:** blocked unless authorized backdate (ATT-004).
- **Notification channel down:** absence alert retried via queue; safeguarding flag never suppressed (ATT-008).
- **Concurrent marking conflict:** serialized; last legitimate submission wins with audit, no silent overwrite.
- **Projection lag in summaries at peak:** summaries may briefly lag; eligibility decisions use confirmed data, not stale reads.

## 15. Exception Handling Rules
- Ownership, enrollment, window, and calendar validated before a mark is accepted.
- Corrections require reason and respect record state; closed sessions need governed reopen.
- Absence notifications and safeguarding flags are best-effort-delivered but always recorded, even if delivery fails.
- Eligibility overrides are explicit, authorized, and audited — never implicit.

## 16. Compliance Considerations
- **Safeguarding:** timely absence notification and chronic-absence flagging support child-protection duties — a core obligation for institutions serving minors.
- **Mandated records:** some jurisdictions legally require retained attendance records; retention honors this.
- **Fairness:** documented marking windows, leave integration, and audited corrections defend attendance-based decisions (e.g., exam eligibility).
- **Privacy:** attendance is sensitive minor data, scoped, access-controlled, and custody-restriction-aware.

## 17. Future Considerations
- Biometric / RFID / facial attendance with strong minor-data safeguards and consent.
- Geofenced or device-based marking for distributed/online learning.
- Predictive chronic-absence analytics feeding early-intervention (consent-bound).
- Automated parent self-service leave requests reducing false absences.
