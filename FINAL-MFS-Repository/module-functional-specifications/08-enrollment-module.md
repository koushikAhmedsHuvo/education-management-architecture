# 08 — Enrollment Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`ENR-001…008`), and Use Case Repository (`UC-ENR-001…018`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Bind a student to a specific session/level/section instance and manage that binding over time: single active enrollment per session, hard-cap section capacity, roll-number assignment, period-accurate transfers, withdrawal/exit with transfer certificate, and outcome-driven promotion.

**Business Goal.** Place every active student in exactly one correct section per session, never exceed section capacity, and preserve accurate enrollment history across transfers and year transitions.

**Scope.** Enrollment creation (bind to section instance, hard cap); one active enrollment per student per session; section capacity enforcement; roll-number assignment; inter-section transfer (history-preserving); withdrawal + clearance; cross-institution exit (withdrawal + transfer certificate); promotion → next-session enrollment by outcome; status-driven operational eligibility.

**Out of Scope.** Section/class definitions (Academic modules — enrollment consumes instances). Student master record (Student module). Promotion outcome computation (Result/Session modules — enrollment applies the outcome). Inter-campus moves (Campus module — distinct from section transfer).

---

## 2. Actors

**Primary Actors.** Registrar / Enrollment Administrator, Academic Coordinator (section placement), System (capacity enforcement, roll-number assignment, promotion application).

**Secondary Actors.** Session module (rollover/promotion), Section/Class modules (capacity, instances), Student module (subject), Result module (outcomes), Workflow Engine (transfer/withdrawal approval), File module (transfer certificate), Audit, Reporting, Notification.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Create enrollment | Bind a student to a session/level/section instance. | Critical | Session, Section |
| FR-002 | One active enrollment per session | Prevent double-enrollment within a session. | Critical | FR-001 |
| FR-003 | Section capacity (hard cap) | Enforce section seat limit at enrollment/transfer. | Critical | Section module |
| FR-004 | Roll-number assignment | Assign a unique roll number within section/session. | High | FR-001 |
| FR-005 | Inter-section transfer | Move a student between sections, preserving period-accurate history. | High | Workflow, Section |
| FR-006 | Withdrawal & clearance | Exit a student with clearance (dues, items). | High | Fee, File (TC) |
| FR-007 | Cross-institution exit | Treat external exit as withdrawal + transfer certificate. | High | File module |
| FR-008 | Promotion → next-session enrollment | Create next-session enrollment from outcome (advance/retain). | High | Session, Result |
| FR-009 | Status-driven eligibility | Enrollment status governs operational eligibility (attendance/exams/fees). | High | downstream modules |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Section binding | Student placed in exactly one section instance. | Accurate academic placement. |
| Hard-cap capacity | Section never over-enrolled. | Class-size integrity. |
| Roll numbers | Unique within section/session. | Operational identity. |
| History-preserving transfer | Period-accurate section history. | Correct attendance/marks attribution. |
| Withdrawal + clearance + TC | Governed exit with certificate. | Clean separation; portability. |
| Outcome-driven promotion | Next-session enrollment by result. | Correct year transitions. |
| Status-driven eligibility | One status gates downstream ops. | Consistent operational control. |

---

## 5. Screens

Enrollment List; Enrollment Detail; Create Enrollment (section placement); Edit Enrollment; Inter-Section Transfer; Withdrawal & Clearance; Cross-Institution Exit (TC); Roll-Number Assignment; Promotion → Next Session; Enrollment & Capacity Report; Bulk Enrollment / Section Rebalance; Enrollment Import; Enrollment Export; Transfer/Withdrawal Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Enrollment List | Create, Open, Search/Filter, Export | Bulk Enroll, Rebalance |
| Enrollment Detail | Edit, Transfer Section, Withdraw, Assign Roll No., View History | — |
| Create Enrollment | Select Session/Section, Validate Capacity, Assign Roll No., Save | — |
| Inter-Section Transfer | Select Target Section, Reason, Validate Capacity, Submit (→ approval) | Bulk Transfer |
| Withdrawal & Clearance | Run Clearance, Generate TC, Confirm (→ approval) | — |
| Promotion | Apply Outcome, Set Target Section, Confirm | Bulk Promote |
| Capacity Report | Run, Filter, Export | Export |

---

## 7. Forms

**Create Enrollment** — `student` (search-select, required), `session` (select, required, current/admission-open), `level/class` (select, required), `section` (select, required), `rollNumber` (auto/assignable). Validation: no existing active enrollment this session (ENR-002); section capacity available (ENR-003); roll number unique in section/session (ENR-004); bind to a valid session instance (ENR-001).

**Inter-Section Transfer** — `targetSection` (select, required, capacity-available), `effectiveDate` (date, required), `reason` (text, required). Validation: capacity hard cap (ENR-003); history preserved period-accurately (ENR-005); routed to approval.

**Withdrawal & Clearance** — `reason` (select/text, required), `effectiveDate` (date, required), `clearanceItems` (checklist). Validation: clearance complete per policy; transfer certificate generated for cross-institution exit (ENR-006); status → withdrawn (ENR-008).

**Promotion** — `outcome` (advance/retain), `targetSection` (for advance). Validation: outcome from Result/Session (ENR-007); target capacity respected.

---

## 8. Search & Filter Requirements

**Enrollments:** by student, session, class/level, section, roll number, status (active/transferred/withdrawn/promoted/graduated), campus. Sorting: roll number/section/status. Pagination: server-side, 25 default. Scope + ownership enforced.

---

## 9. Table Requirements

**Enrollment table:** Roll No., Student, Class/Section, Session, Status, Enrolled Date, Campus. Sorting on Roll No./Section/Status. Filtering as above. Export (governed). Bulk: Enroll, Transfer, Promote, Rebalance.

---

## 10. Workflow Requirements

**Trigger events:** enroll, transfer, withdraw, exit (TC), promote, roll-number (re)assignment. **Status changes:** `ACTIVE → (TRANSFERRED within / WITHDRAWN / PROMOTED / GRADUATED)`. **Approvals:** inter-section transfer, withdrawal, cross-institution exit via Workflow Engine. **Notifications:** enrollment confirmation, transfer, withdrawal/TC, promotion (custody-aware, to guardians). **Audit:** all bindings, transfers (period-accurate), withdrawals, promotions (immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| View enrollment(s) | `enrollment.view` |
| Create enrollment | `enrollment.create` |
| Update enrollment | `enrollment.update` |
| Transfer section | `enrollment.transfer.execute` |
| Withdraw / exit (TC) | `enrollment.withdraw` |
| Assign roll number | `enrollment.rollnumber.manage` |
| Promote | `enrollment.promote` |
| Approve transfer/withdrawal | `enrollment.transfer.approve` |
| Import/export | `enrollment.import`, `enrollment.export` |

All capabilities scope-bound (institute/campus); transfers/withdrawals governed.

---

## 12. Business Rule References

ENR-001 (bind to session/level/section instance), ENR-002 (one active enrollment per session), ENR-003 (section capacity enforcement), ENR-004 (roll-number assignment), ENR-005 (transfer preserves period-accurate history), ENR-006 (cross-institution exit = withdrawal + TC), ENR-007 (promotion creates next-session enrollment by outcome), ENR-008 (status drives operational eligibility). Cross-cutting: SESS-004/006 (session instances/promotion), SEC-003 (section capacity source), WFL-002/004 (approvals), FILE-007 (TC immutable), AUD-001, NOT-002.

## 13. Use Case References

UC-ENR-001 (Create — hard cap), UC-ENR-002 (View), UC-ENR-003 (Update placement), UC-ENR-004 (Inter-Section Transfer), UC-ENR-005 (Withdrawal & Clearance), UC-ENR-006 (Promotion → Next-Session), UC-ENR-007 (Re-Enrollment/Re-Admission), UC-ENR-008 (Assign/Reassign Roll Number), UC-ENR-009 (Approve Transfer/Withdrawal), UC-ENR-010 (Search), UC-ENR-011 (Enrollment & Capacity Report), UC-ENR-012 (Bulk Enrollment/Rebalance), UC-ENR-013/014 (Import/Export), UC-ENR-015 (Withdrawal/Transfer Workflow), UC-ENR-016 (Double-Enrollment Blocked), UC-ENR-017 (Section Capacity Full), UC-ENR-018 (Cross-Institution Exit — TC).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Create enrollment (capacity-checked) | POST | Registrar |
| Get / list enrollments | GET | Authorized roles |
| Update enrollment (placement) | PUT | Registrar |
| Transfer section (single/bulk) | POST | Registrar (→ approval) |
| Withdraw / cross-institution exit (TC) | POST | Registrar (→ approval) |
| Assign / reassign roll number | POST | Registrar |
| Promote → next-session enrollment | POST | Registrar |
| Enrollment & capacity report | GET | Admin |
| Import / export enrollments | POST/GET | Admin |

Capacity checks and roll-number assignment are transactional to prevent over-enrollment/collisions (ENR-003/004).

---

## 15. Database Requirements

**Entities:** `Enrollment` (studentId, sessionId, classId, sectionId, rollNumber, status), `EnrollmentHistory` (period-accurate section spans), `Withdrawal` (reason, clearance, TC ref), `PromotionLink` (from/to enrollment). **Relationships:** Student 1—* Enrollment; Enrollment *—1 Section instance; Enrollment 1—* History; Enrollment 1—1 Withdrawal (if exited). **Indexes:** unique(Enrollment.studentId, sessionId where active) (ENR-002), unique(Enrollment.sectionId, rollNumber, sessionId) (ENR-004), index(Enrollment.sectionId, status), index(EnrollmentHistory.enrollmentId, period). Capacity enforced via section-count check under transaction (ENR-003).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Enrollment confirmation; transfer; withdrawal + TC; promotion outcome. |
| In-App | Placement/roll-number; pending transfer/withdrawal approvals. |
| SMS/Push | Minimized enrollment/transfer alerts to permitted guardians. |

Custody-aware routing (NOT-002); promotion notifications batched (NOT-006).

---

## 17. Audit Requirements

Log: enrollment create (section, roll number), section transfers (from/to, period, approver), withdrawal/exit (reason, TC ref), promotion (outcome, from/to), roll-number changes. Record who/when/before/after. Period-accurate history is itself the audit of placement. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Enrollment by class/section, Capacity utilization/vacancies, Transfer history, Withdrawal/attrition, Promotion outcomes. **Exports:** governed enrollment export. **Dashboards:** section fill rates, capacity alerts, attrition trends.

---

## 19. Error Handling

**Validation:** double-enrollment, section full, duplicate roll number → specific errors (UC-ENR-016/017). **Permission:** out-of-scope enrollment → 403/not-found. **Workflow:** transfer/withdrawal pending approval → clear state. **System:** capacity service contention → transactional cap; promotion target missing → blocked (deferred to Session UC-SESS-018).

---

## 20. Edge Cases

**Concurrent updates:** two enrollments for one student/session → one active (ENR-002). **Duplicate data:** duplicate roll number → blocked/reassigned. **Partial failures:** bulk enrollment/rebalance partial → per-record report, completed rows stand. **Rollback:** transfer approval reversed → student remains in source section, history accurate. **Capacity race:** last seat contested → one enrolls, other rejected (ENR-003).

---

## 21. Acceptance Criteria

**Functional.** A student has exactly one active enrollment per session; section capacity hard caps are never exceeded; roll numbers are unique within section/session; section transfers preserve period-accurate history; withdrawal runs clearance and (for external exit) issues a transfer certificate; promotion creates the correct next-session enrollment by outcome; enrollment status gates downstream operational eligibility.

**Business.** Every active student is correctly and uniquely placed; class sizes stay within limits; enrollment history remains accurate across transfers and years, so attendance, marks, and fees attribute correctly.

---

## 22. Future Enhancements

Optimized auto-section-balancing; seat-waitlist within sections; configurable roll-number schemes; self-service transfer requests (governed); capacity forecasting; integrated TC verification/QR; mid-session re-streaming workflows.
