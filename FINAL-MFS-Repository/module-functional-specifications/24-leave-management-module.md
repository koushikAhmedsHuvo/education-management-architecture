# 24 — Leave Management Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`LEV-001…009`), and Use Case Repository (`UC-LEV-001…018`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Run the leave engine: configurable leave types/policies, balance accrual/carry-forward/encashment, holiday/sandwich duration computation, manager-routed approval with escalation/delegation/timeout, attendance/payroll integration, governed backdating, coverage coordination, and the **student-leave sub-domain (owner of the C-04 boundary)**.

**Business Goal.** Ensure leave requests are routed, decided, and never stall — reflected correctly in attendance and payroll — while owning the distinct, balance-free student-leave workflow that Attendance merely reflects.

**Scope.** Leave types/policies; balance accrual/carry-forward/encashment; holiday/weekend/sandwich computation; manager-routed approval; escalation/delegation/timeout; attendance & payroll integration; backdated/exceptional leave (governed); coverage/substitution coordination; student leave (distinct, balance-free).

**Out of Scope.** Attendance marking/reflection mechanics (Attendance module — reflects approved leave). Payroll computation (HR module — consumes leave). Workflow execution internals (Workflow Engine). Substitute assignment mechanics (Teacher module — Leave coordinates).

---

## 2. Actors

**Primary Actors.** Staff Member (requests), Manager/Approver (approves via reporting line), HR Administrator (policy), Student/Guardian (student leave), System (balance, duration, routing, escalation).

**Secondary Actors.** Workflow Engine (approval/escalation/delegation/timeout), Staff hierarchy (approver resolution), Attendance & Payroll/HR modules (integration), Teacher module (coverage), Audit, Reporting, Notification.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Configurable leave types/policies | Configure types, accrual, holidays, sandwich, routing (versioned). | High | Configuration Engine |
| FR-002 | Balance accrual/carry-forward/encashment | Accrue, carry forward (capped), encash per policy. | High | HR (encashment) |
| FR-003 | Duration computation | Compute days excluding holidays/weekends with sandwich rules. | High | Calendar/Config |
| FR-004 | Manager-routed approval | Route to the requester's manager (reporting line); SoD. | Critical | Staff (STF-006), Workflow |
| FR-005 | Escalation/delegation/timeout | Stall-proof routing; no silent auto-approval of sensitive steps. | High | Workflow (WFL-005/006/007) |
| FR-006 | Attendance & payroll integration | Approved leave → excused attendance + payroll treatment. | High | Attendance, HR |
| FR-007 | Backdated/exceptional leave (governed) | Governed, reasoned, audited backdating. | High | Workflow, AUTHZ-009 |
| FR-008 | Coverage/substitution coordination | Coordinate a substitute for approved teacher leave. | Medium | Teacher (TCH-005) |
| FR-009 | Student leave (balance-free) | Distinct student-leave workflow; Attendance reflects only (C-04). | High | Attendance (ATT-005) |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Configurable leave | Types/policies/accrual. | Fits any HR policy. |
| Balance management | Accrual/carry/encash. | Accurate entitlements. |
| Duration computation | Holiday/sandwich rules. | Correct day counts. |
| Manager-routed approval | Reporting-line routing. | Right approver. |
| Stall-proof routing | Escalation/delegation/timeout. | Never stuck. |
| Attendance/payroll integration | Excused + pay treatment. | Consistency. |
| Governed backdating | Reasoned/audited. | Controlled exceptions. |
| Student leave (C-04) | Balance-free, owned here. | Clean boundary. |

---

## 5. Screens

Request Leave; Leave Approval (manager); Leave Calendar; Balance & Accrual; Duration Preview; Student Leave (guardian); Coverage/Substitution; Backdated/Exceptional Leave; Leave Type/Policy Configuration; Leave Balance & Utilization Report; Bulk/Import Leave; Leave Export; Leave Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Request Leave | Select Type/Dates, Preview Duration, Submit | — |
| Leave Approval | Approve, Reject, Return, Delegate | Bulk Approve |
| Balance & Accrual | View Balance, Encash, Carry Forward | — |
| Student Leave | Request (guardian), Approve (class-teacher) | — |
| Coverage | Assign Substitute | — |
| Backdated Leave | Request Override (reason), Submit (→ approval) | — |
| Policy Config | Configure Types/Accrual/Holidays/Sandwich/Routing, Save (versioned) | — |
| Utilization Report | Run, Filter, Export | Export |

---

## 7. Forms

**Request Leave** — `type` (select), `fromDate`/`toDate` (required), `reason` (text). Validation: balance sufficient (LEV-002 / UC-LEV-017); no overlap (LEV-001 / UC-LEV-018); duration excludes holidays/sandwich (LEV-003).

**Leave Approval** — `decision` (approve/reject/return), `reason`. Validation: approver from reporting line (LEV-004/STF-006); requester ≠ approver (SoD); reflects in attendance/payroll on approval (LEV-006).

**Student Leave** — `student` (own child), `fromDate`/`toDate`, `reason`. Validation: balance-free (LEV-009); requester authorized (GRD-N-004); Attendance reflects excused (ATT-005); custody-restricted blocked (GRD-N-006).

**Backdated/Exceptional** — `date` (past), `reason` (text, required). Validation: governed/approved/audited (LEV-007).

**Policy Config** — `types`, `accrual`, `carryForwardCap`, `holidays`, `sandwichRule`, `approvalRouting`. Validation: typed/versioned (CFG-002/004).

---

## 8. Search & Filter Requirements

**Leave:** by staff/student, type, status (requested/approved/rejected/escalated), date range, manager. Sorting: date/status. Pagination: server-side, 25 default. Scope-bound; staff see own, managers see reports'.

---

## 9. Table Requirements

**Leave table:** Requester, Type, Dates, Days, Status, Approver, Balance. Sorting on Date/Status. Filtering as above. Export (governed). Bulk: approve, import.

---

## 10. Workflow Requirements

**Trigger events:** request, approve/reject/return, escalate/delegate/timeout, balance change, student-leave request, coverage. **Status changes:** leave `REQUESTED → APPROVED/REJECTED/RETURNED → (escalated)`. **Approvals:** manager-routed with escalation/delegation/timeout; sensitive steps never auto-approve (WFL-007). **Notifications:** request/decision, escalation, student-leave approval, coverage. **Audit:** requests, decisions, escalations/delegations, backdating (immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Request leave | authenticated staff (own) |
| Approve leave | `leave.approve` (resolved via hierarchy) |
| Manage balance/accrual | `leave.balance.manage` |
| Configure types/policies | `leave.config.manage` |
| Backdated/exceptional | `leave.backdate` (elevated) |
| Approve student leave | `student.leave.approve` |
| View leave/balance (scoped) | `leave.view` |
| Import/export | `leave.import`, `leave.export` |

Approver resolves via reporting hierarchy (STF-006/WFL-004); SoD enforced.

---

## 12. Business Rule References

LEV-001 (configurable leave types & policies), LEV-002 (balance accrual/carry-forward/encashment), LEV-003 (holiday/weekend & sandwich computation), LEV-004 (manager-routed approval), LEV-005 (escalation, delegation & timeout), LEV-006 (attendance & payroll integration), LEV-007 (backdated & exceptional leave governed), LEV-008 (coverage/substitution coordination), LEV-009 (student leave — distinct, balance-free). Cross-cutting: STF-006 (reporting hierarchy), WFL-004/005/006/007 (routing/escalation/delegation/safe-timeout), ATT-005 (excused reflection — C-04), HR-004 (payroll), TCH-005 (substitution), GRD-N-004/006 (student-leave custody), AUTHZ-009, AUD-001.

## 13. Use Case References

UC-LEV-001 (Request — balance + eligibility), UC-LEV-002 (Approve — manager-routed), UC-LEV-003 (Escalation/Delegation/Timeout), UC-LEV-004 (Compute Duration — holidays, sandwich), UC-LEV-005 (Reflect in Attendance & Payroll), UC-LEV-006 (Manage Balance), UC-LEV-007 (Student Leave — balance-free), UC-LEV-008 (Coordinate Coverage), UC-LEV-009 (View — scoped), UC-LEV-010 (Configure Types/Policies), UC-LEV-011 (Backdated/Exceptional), UC-LEV-012 (Search), UC-LEV-013 (Balance & Utilization Report), UC-LEV-014 (Bulk/Import), UC-LEV-015 (Export), UC-LEV-016 (Approval Workflow), UC-LEV-017 (Insufficient Balance Blocked), UC-LEV-018 (Overlapping Leave Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Request leave (balance/eligibility-checked) | POST | Staff |
| Approve / reject / return leave | POST | Manager |
| Escalate / delegate (timeout-driven) | POST | System / Manager |
| Compute duration (holidays/sandwich) | GET | System |
| Manage balance (accrual/carry/encash) | POST | HR |
| Request / approve student leave | POST | Guardian / Class-Teacher |
| Coordinate coverage/substitution | POST | Coordinator |
| Backdated/exceptional leave | POST | HR (→ approval) |
| Configure types/policies | GET/PUT | HR |
| Balance & utilization report | GET | Admin |
| Bulk / import / export leave | POST/GET | Admin |

Approval routes via reporting hierarchy (LEV-004/STF-006); sensitive steps never silently auto-approve (LEV-005/WFL-007); student leave is balance-free and reflected by Attendance (LEV-009/ATT-005).

---

## 15. Database Requirements

**Entities:** `LeaveType`/`LeavePolicy` (via Config), `LeaveRequest` (requester, type, dates, days, status), `LeaveBalance` (accrual, carry-forward, encashment), `LeaveDecision` (approver, outcome), `StudentLeave` (balance-free), `Coverage` (substitute link). **Relationships:** Staff 1—* LeaveRequest; LeaveRequest 1—1 Decision; Staff 1—1 LeaveBalance; Student 1—* StudentLeave. **Indexes:** index(LeaveRequest.requesterId, status), unique(LeaveRequest overlap guard per staff/date), index(StudentLeave.studentId). Manager resolved via ReportingLine (STF-006); student leave balance-free (LEV-009).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Request submitted/decided; escalation/delegation; student-leave approval. |
| In-App | Approval inbox; escalations; coverage assignments. |
| SMS/Push | Decision alerts (minimized); student-leave approval to guardian. |

Student-leave notices custody-aware (NOT-002/GRD-N-006); escalation never silent (WFL-007).

---

## 17. Audit Requirements

Log: requests, decisions (approver, outcome), escalations/delegations/timeouts, balance changes (accrual/carry/encash), backdated/exceptional leave, student-leave approvals, coverage. Record who/when/before/after. Decisions and escalations carry full lineage. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Leave balance/utilization, Pending approvals, Leave-type distribution, Backdated-leave log, Student-leave summary. **Exports:** governed leave export. **Dashboards:** leave overview (pending, escalations, balances, coverage gaps).

---

## 19. Error Handling

**Validation:** insufficient balance, overlapping leave → specific errors (UC-LEV-017/018). **Permission:** self-approval → SoD block; student leave by non-guardian/custody-restricted → blocked. **Workflow:** manager vacancy/timeout → escalation (never silent auto-approve, WFL-007). **System:** Attendance/payroll integration unavailable → reflection deferred, decision recorded.

---

## 20. Edge Cases

**Concurrent updates:** two approvers acting → idempotent single decision. **Duplicate data:** overlapping/duplicate leave blocked. **Partial failures:** bulk import partial → per-row report. **Rollback:** decision reversed → status restored, attendance/payroll reflection corrected. **Stall race:** SLA breach + late manager action → first valid decision wins; no-target timeout → hold-and-flag (never auto-approve).

---

## 21. Acceptance Criteria

**Functional.** Leave types/policies are configurable and versioned; balances accrue/carry-forward/encash per policy; duration excludes holidays/weekends with sandwich rules; requests route to the requester's manager (SoD) with escalation/delegation/timeout and never silently auto-approve; approved leave reflects as excused attendance and correct payroll treatment; backdated/exceptional leave is governed; student leave is balance-free and owned here while Attendance only reflects it (C-04).

**Business.** Leave never stalls when a manager is unavailable; entitlements and durations are accurate; attendance and payroll stay consistent with leave; the student-leave boundary is clean with no duplicate ownership.

---

## 22. Future Enhancements

Team leave calendars and conflict detection; auto-accrual schedulers; configurable approval chains by leave type/duration; leave-encashment automation at year-end; mobile leave requests/approvals; predictive coverage suggestions; compensatory-off tracking.
