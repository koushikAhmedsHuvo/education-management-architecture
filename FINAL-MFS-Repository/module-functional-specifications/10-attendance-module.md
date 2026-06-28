# 10 — Attendance Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`ATT-001…009`), and Use Case Repository (`UC-ATT-001…017`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Record and report student presence: configurable mode/statuses, owner/granularity by mode, marking only for active enrollments within a controlled window, excused-not-absent reflection of approved leave, holiday exclusion, governed corrections, absence/safeguarding notifications, and summaries gating exam eligibility.

**Business Goal.** Produce accurate, fair, defensible attendance that drives exam eligibility and early safeguarding signals, without becoming a back-dating or correction loophole.

**Scope.** Mode/status configuration (daily/period); owner + granularity by mode; marking within a window; backdating control; reflection of approved leave as excused (this module reflects, Leave owns — C-04); holiday/non-instructional exclusion; governed corrections; absence + safeguarding notifications; summaries and exam-eligibility threshold.

**Out of Scope.** Student leave request/approval workflow (Leave Management owns it — ATT-005 only reflects). Staff attendance (HR module). Exam eligibility decision execution (Examination module — consumes the threshold). Notification delivery mechanics (Notification module).

---

## 2. Actors

**Primary Actors.** Subject-Teacher / Class-Teacher (marks attendance for owned scope), Attendance Administrator (config, corrections approval), System (window enforcement, summaries, holiday exclusion).

**Secondary Actors.** Enrollment module (active-enrollment source), Leave module (approved leave), Examination module (eligibility consumer), Workflow Engine (correction approval), Notification (absence/safeguarding), Configuration Engine (mode/threshold), Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Configurable mode & statuses | Configure daily/period mode and status set (present/absent/late/excused). | High | Configuration Engine |
| FR-002 | Mode-driven owner & granularity | Mode determines who marks and at what granularity. | High | FR-001, Section/Subject ownership |
| FR-003 | Active-enrollment-only marking | Allow marking only for active enrollments. | Critical | Enrollment |
| FR-004 | Marking window & backdating control | Enforce a marking window; backdating only via governed override. | High | Config, Workflow |
| FR-005 | Excused-not-absent reflection | Reflect approved leave as excused, never absent. | High | Leave (C-04) |
| FR-006 | Holiday exclusion | Exclude holidays/non-instructional days from attendance/eligibility. | High | Calendar/Config |
| FR-007 | Governed corrections | Corrections are reasoned, approved, audited (no silent edits). | High | Workflow, AUTHZ-009 |
| FR-008 | Absence & safeguarding notifications | Notify guardians of absence; raise safeguarding signal on patterns. | High | Notification |
| FR-009 | Summaries & eligibility threshold | Compute attendance % and exam-eligibility against threshold. | High | Examination |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Configurable modes | Daily or per-period with status sets. | Fits any institution type. |
| Windowed marking | Time-boxed marking, controlled backdating. | Prevents retroactive manipulation. |
| Active-only marking | Only enrolled students markable. | Accurate, scoped records. |
| Excused reflection | Approved leave shows excused. | Fair treatment; consistent with Leave. |
| Holiday exclusion | Non-instructional days ignored. | Correct percentages. |
| Governed corrections | Approved, audited changes. | Integrity; no silent edits. |
| Safeguarding signal | Pattern-based alerts. | Early child-welfare intervention. |
| Eligibility summaries | %-based exam gating. | Policy-driven fairness. |

---

## 5. Screens

Mark Attendance (roster, windowed); Student Attendance (history); Class/Section Attendance Overview; Attendance Correction; Holiday / Non-Instructional Day Management; Mode/Status/Threshold Configuration; Backdated/Window-Override Marking; Attendance Report & Defaulter List; Bulk/Import Attendance; Attendance Export; Correction Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Mark Attendance | Mark Present/Absent/Late/Excused, Save, Mark-All-Present, Submit | Mark All / Bulk Status |
| Student Attendance | View History, Filter by Range/Status | — |
| Section Overview | Drill to Roster, Export, View Summary | Export |
| Attendance Correction | Propose Correction (reason), Submit (→ approval) | — |
| Holiday Management | Add Holiday, Edit, Remove, Import Calendar | Bulk Import |
| Configuration | Set Mode/Statuses/Threshold/Window, Save (versioned) | — |
| Backdated Marking | Request Override (reason), Mark, Submit (→ approval) | — |
| Report & Defaulters | Run, Filter, Export | Export |

---

## 7. Forms

**Mark Attendance** — `date`/`period` (auto/required), `section/subject` (owned), per-student `status` (select from configured set). Validation: within window (ATT-004); only active enrollments (ATT-003); marker owns the scope (ATT-002/SEC-004/SUB-004); holiday → marking suppressed (ATT-006).

**Attendance Correction** — `student` (required), `date/period` (required), `newStatus` (select), `reason` (text, required). Validation: governed correction (ATT-007); routed to approval; original preserved (P5).

**Holiday/Non-Instructional Day** — `date(s)` (required), `type` (holiday/event/exam), `scope` (campus/class). Validation: excluded from attendance/eligibility (ATT-006).

**Configuration** — `mode` (daily/period), `statuses` (multi), `markingWindow` (structured), `backdatingPolicy` (structured), `eligibilityThreshold` (number %). Validation: typed/versioned (CFG-002/004); threshold 0–100.

**Backdated/Override Marking** — `date` (past), `reason` (text, required). Validation: window-override governed/approved (ATT-004); audited.

---

## 8. Search & Filter Requirements

**Attendance:** by student, section/class, subject, date range, status, marker. **Defaulters:** by threshold breach, class/section. Sorting: date/student/status. Pagination: server-side, 50 default (high volume). Scope + ownership enforced (teachers see own scope; guardians see own children).

---

## 9. Table Requirements

**Roster table:** Roll No., Student, Status (editable in-window), Last Marked. **History table:** Date/Period, Status, Marked By, Corrected?. **Defaulter table:** Student, Attendance %, Threshold, Eligibility. Sorting on Date/%/Status. Filtering as above. Export (governed). Bulk: status set, import.

---

## 10. Workflow Requirements

**Trigger events:** mark/submit, correction request, backdate-override, holiday change, summary computation, absence pattern. **Status changes:** attendance record `MARKED → CORRECTED (governed)`; correction `REQUESTED → APPROVED/REJECTED`. **Approvals:** corrections and window-overrides via Workflow Engine. **Notifications:** absence (to guardians, custody-aware), safeguarding signal (to admins), correction outcomes. **Audit:** marking, corrections (before/after), overrides, config (immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Mark attendance (owned scope) | `attendance.mark` |
| View attendance (scoped) | `attendance.view` |
| Correct attendance | `attendance.correct` (→ approval) |
| Approve correction | `attendance.correction.approve` |
| Backdate/override marking | `attendance.override` |
| Manage holidays | `attendance.holiday.manage` |
| Configure mode/threshold | `attendance.config.manage` |
| Report/export | `attendance.report.view`, `attendance.export` |

Marking is ownership-scoped (ATT-002); corrections enforce SoD.

---

## 12. Business Rule References

ATT-001 (configurable mode & statuses), ATT-002 (mode determines owner & granularity), ATT-003 (active enrollments only), ATT-004 (marking window & backdating control), ATT-005 (approved leave reflects as excused — C-04), ATT-006 (holidays excluded), ATT-007 (corrections governed & audited), ATT-008 (absence notification & safeguarding signal), ATT-009 (summaries & exam-eligibility threshold). Cross-cutting: ENR-008 (active enrollment), LEV-009 (student leave owner), SEC-004/SUB-004 (marking ownership), EXM-003 (eligibility consumer), WFL-002/004 (correction approval), NOT-002 (custody-aware), AUD-001.

## 13. Use Case References

UC-ATT-001 (Mark — windowed), UC-ATT-002 (Approved Leave → Excused), UC-ATT-003 (Absence & Safeguarding Signal), UC-ATT-004 (Compute Summary & Eligibility), UC-ATT-005 (Correct — governed), UC-ATT-006 (View roster/student), UC-ATT-007 (Configure Mode/Statuses/Threshold), UC-ATT-008 (Backdated/Override), UC-ATT-009 (Manage Holidays), UC-ATT-010 (Approve Correction), UC-ATT-011 (Search), UC-ATT-012 (Report & Defaulter List), UC-ATT-013 (Bulk/Import), UC-ATT-014 (Export), UC-ATT-015 (Correction Workflow), UC-ATT-016 (Marking Outside Window Blocked), UC-ATT-017 (Marking Inactive Enrollment Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Get markable roster (windowed, scoped) | GET | Teacher |
| Submit attendance | POST | Teacher |
| Get student/section attendance | GET | Authorized roles |
| Propose correction | POST | Teacher (→ approval) |
| Approve/reject correction | POST | Approver |
| Backdate/window-override marking | POST | Admin (→ approval) |
| Manage holidays / non-instructional days | POST/PUT/DELETE | Admin |
| Configure mode/statuses/threshold | GET/PUT | Admin |
| Compute/get attendance summary & eligibility | GET | Admin/Examination |
| Report & defaulter list | GET | Admin |
| Bulk / import / export attendance | POST/GET | Admin |

Marking validates window + active enrollment + ownership server-side (ATT-002/003/004).

---

## 15. Database Requirements

**Entities:** `AttendanceRecord` (studentId, enrollmentId, date/period, status, markedBy, corrected), `AttendanceCorrection` (old/new, reason, approver), `Holiday` (date, type, scope), `AttendanceConfig` (mode/statuses/threshold/window via Config), `AttendanceSummary` (computed % per student/period). **Relationships:** Enrollment 1—* AttendanceRecord; AttendanceRecord 1—* Correction. **Indexes:** unique(AttendanceRecord.enrollmentId, date, period), index(AttendanceRecord.sectionId, date), index(AttendanceRecord.studentId, date), index(Holiday.date, scope). High-volume: partition by session/period; summaries materialized.

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Daily/period absence to guardians; defaulter/eligibility-risk notices. |
| SMS | Same-day absence alert (minimized) to permitted guardian. |
| Push | Absence/late alerts. |
| In-App | Correction approvals; safeguarding signals to admins. |

Absence notices are custody-aware (NOT-002); safeguarding-pattern signals are mandatory-category (NOT-003); content minimized (NOT-004).

---

## 17. Audit Requirements

Log: marking (who/when/scope), corrections (before/after, reason, approver), backdated/override marking, holiday changes, config changes (threshold/window), summary recomputation. Record who/when/before/after. Corrections and overrides are first-class audit events. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Daily/period attendance, Defaulter list (below threshold), Student attendance trend, Exam-eligibility status, Safeguarding pattern alerts. **Exports:** governed attendance export. **Dashboards:** attendance health (marking-completion, defaulters, eligibility-at-risk).

---

## 19. Error Handling

**Validation:** marking outside window, inactive enrollment, marking on holiday → specific errors (UC-ATT-016/017). **Permission:** non-owner marking → 403. **Workflow:** correction/override pending approval → clear state. **System:** Leave service unavailable → leave reflection deferred (not marked absent); summary recompute failure → flagged, source-consistent eligibility.

---

## 20. Edge Cases

**Concurrent updates:** co-teacher concurrent marking → ownership-scoped; last in-window save wins, audited. **Duplicate data:** re-marking same student/date/period → overwrite in-window (not duplicate row). **Partial failures:** bulk/import partial → per-row report. **Rollback:** correction approval reversed → prior status restored, history intact. **Leave race:** leave approved after absence marked → reflection corrects absent→excused (UC-ATT-002).

---

## 21. Acceptance Criteria

**Functional.** Marking is allowed only within the window and only for active enrollments; approved leave reflects as excused, never absent; holidays are excluded from attendance and eligibility; corrections are reasoned, approved, and audited; absence triggers custody-aware notifications and pattern-based safeguarding signals; summaries compute attendance % and exam eligibility against the configured threshold.

**Business.** Attendance is accurate, fair, and defensible; it cannot be silently back-dated or edited; it drives exam eligibility consistently and surfaces child-welfare concerns early.

---

## 22. Future Enhancements

Biometric/RFID/QR marking; geofenced mobile marking; auto-absence escalation ladders; predictive at-risk (attendance + performance); period-level timetable integration; parent acknowledgement of absence notices; configurable safeguarding thresholds per jurisdiction.
