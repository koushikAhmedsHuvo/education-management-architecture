# 24 — Leave Management Use Cases

Transforms the Leave Management business rules (`LEV-001`…`LEV-009`) into use cases. The leave engine: configurable types/policies, balance accrual/carry-forward/encashment, holiday/sandwich computation, manager-routed approval with escalation/delegation/timeout, attendance & payroll integration, governed backdating, coverage coordination, and the **student-leave sub-domain (owner of C-04)**.

## 1. Primary Actors
Staff Member (requests leave), Manager / Approver (approves via reporting line), HR Administrator (configures policy), Student/Guardian (student leave — balance-free).

## 2. Secondary Actors
System (balance, computation, routing, escalation), Workflow Engine (approval/escalation/delegation/timeout), Staff hierarchy (approver resolution), Attendance & Payroll modules (integration), Teacher module (coverage), Audit service.

## 3. Goals
Configure leave types/policies; accrue/carry-forward/encash balances; compute durations excluding holidays (with sandwich rules); route approval up the reporting line with escalation/delegation/timeout so nothing stalls; reflect approved leave in attendance and payroll; govern backdated/exceptional leave; coordinate coverage/substitution; manage student leave as a distinct, balance-free sub-domain (Attendance only reflects it).

## 4. User Journeys
- **Request (staff):** staff requests leave → balance/eligibility checked → routed to their manager (reporting line) → approved/escalated → reflected in attendance/payroll.
- **Cover:** approved teacher leave coordinates a substitute (TCH-005/LEV-008).
- **Stall-proofing:** if the manager doesn't act, the request escalates; if the manager is away, it delegates; timeouts never silently approve.
- **Student leave (C-04):** a guardian requests student leave → approved in this module (balance-free) → Attendance reflects it as excused (ATT-005).

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-LEV-001 | Request Leave (balance + eligibility) | High |
| Core | UC-LEV-002 | Approve Leave (manager-routed) | Critical |
| Core | UC-LEV-003 | Escalation / Delegation / Timeout | High |
| Core | UC-LEV-004 | Compute Duration (holidays, sandwich) | High |
| Core | UC-LEV-005 | Reflect in Attendance & Payroll | High |
| Core | UC-LEV-006 | Manage Balance (accrual/carry-forward/encashment) | High |
| Core | UC-LEV-007 | Student Leave (balance-free sub-domain) | High |
| Core | UC-LEV-008 | Coordinate Coverage / Substitution | Medium |
| CRUD | UC-LEV-009 | View Leave / Balance (scoped) | Medium |
| Admin | UC-LEV-010 | Configure Leave Types & Policies | High |
| Approval | UC-LEV-011 | Backdated / Exceptional Leave (governed) | High |
| Search | UC-LEV-012 | Search Leave Records | Low |
| Reporting | UC-LEV-013 | Leave Balance & Utilization Report | Medium |
| Bulk/Import | UC-LEV-014 | Bulk / Import Leave (opening balances) | Low |
| Export | UC-LEV-015 | Export Leave Data | Low |
| Workflow | UC-LEV-016 | Leave Approval Workflow | High |
| Exception | UC-LEV-017 | Insufficient Balance Blocked | High |
| Exception | UC-LEV-018 | Overlapping Leave Blocked | Medium |

---

## 6. Detailed Specifications (high-value use cases)

### UC-LEV-002 — Approve Leave (manager-routed)
- **Module:** Leave Management · **Priority:** Critical
- **Actors:** Manager / Approver (primary), Staff (requester), System
- **Goal:** Route a leave request to the right approver via the reporting line and record the decision.
- **Description:** The request routes to the requester's manager (from the staff hierarchy, STF-006); the approver decides; SoD ensures the requester cannot approve their own; the decision reflects in attendance/payroll.
- **Business Rules Applied:** LEV-004, LEV-006, STF-006, AUTHZ-009, WFL-004.
- **Preconditions:** A submitted leave request; reporting line defined.
- **Trigger:** Leave request submitted (UC-LEV-001).
- **Main Success Scenario:**
  1. System resolves the approver from the reporting hierarchy (STF-006/WFL-004).
  2. The manager reviews balance, coverage, and conflicts.
  3. The manager approves/rejects with a reason; SoD prevents self-approval.
  4. On approval, the leave reflects in attendance (excused) and payroll (LEV-006).
- **Alternative Flows:** A1) Multi-level approval for long leave. A2) Approval triggers coverage coordination (UC-LEV-008).
- **Exception Flows:** E1) Manager inactive/timeout → escalate (UC-LEV-003). E2) Requester is the manager → route to the next level (SoD).
- **Validation Rules:** Approver from hierarchy; SoD; balance sufficient (LEV-004/006, STF-006).
- **Permissions Required:** `leave.approve` (resolved via hierarchy).
- **Notifications Triggered:** `LEAVE_APPROVED/REJECTED` to requester; coverage notices.
- **Audit Events Generated:** `LEAVE_DECISION` (approver, outcome).
- **Data Created:** Decision record.
- **Data Updated:** Leave status; balance; attendance/payroll reflection.
- **Data Deleted:** None.
- **Post Conditions:** Decision recorded; approved leave reflected downstream.
- **Related Use Cases:** UC-LEV-001, UC-LEV-003, UC-LEV-005.
- **Acceptance Criteria:**
  - Given a request, When routed, Then it reaches the requester's manager per the reporting line.
  - Given the requester is the manager, When routing, Then it goes to the next level (SoD).
  - Given approval, When recorded, Then the leave reflects as excused in attendance and adjusts payroll.
- **Edge Case Analysis:**
  - *Invalid Input:* decision without reason (if required) blocked.
  - *Permission Failure:* self-approval blocked (AUTHZ-009).
  - *Concurrent Update:* two approvers acting → idempotent; one decision.
  - *Duplicate Data:* re-deciding a decided request blocked.
  - *System Failure:* decision atomic with reflection.
  - *Workflow Failure:* manager vacancy/timeout → escalation (UC-LEV-003).
- **QA Coverage:**
  - *Positive:* approve; reject; multi-level.
  - *Negative:* self-approval; decide-twice.
  - *Boundary:* approval at timeout edge (escalation triggers).

### UC-LEV-003 — Escalation / Delegation / Timeout
- **Module:** Leave Management · **Priority:** High
- **Actors:** System (primary), Manager / Delegate / Next-level approver
- **Goal:** Ensure leave requests never stall when an approver is slow, away, or absent.
- **Description:** If a step exceeds its SLA, it escalates to the next level; an away manager can delegate; timeouts never silently approve — the request escalates or holds (WFL-005/006/007).
- **Business Rules Applied:** LEV-005, WFL-005, WFL-006, WFL-007.
- **Preconditions:** A pending leave approval.
- **Trigger:** SLA breach, manager delegation, or manager unavailability.
- **Main Success Scenario:**
  1. On SLA breach, System escalates to the next-level approver (WFL-005).
  2. An away manager delegates authority to a deputy (bounded, attributed, WFL-006).
  3. Timeouts escalate or hold; sensitive approvals never auto-approve (WFL-007).
- **Alternative Flows:** A1) Delegation chain (bounded) when the delegate is also away.
- **Exception Flows:** E1) No escalation target → hold-and-flag (never auto-approve).
- **Validation Rules:** Escalation/delegation bounded; no silent auto-approval (LEV-005, WFL-005/006/007).
- **Permissions Required:** Resolved approver/delegate authority.
- **Notifications Triggered:** Escalation/delegation notices.
- **Audit Events Generated:** `LEAVE_ESCALATED/DELEGATED`.
- **Data Created:** Escalation/delegation records.
- **Data Updated:** Approver assignment.
- **Data Deleted:** None.
- **Post Conditions:** Request progresses or is safely flagged; nothing stalls.
- **Related Use Cases:** UC-LEV-002, UC-WFL (engine).
- **Acceptance Criteria:**
  - Given an unactioned request past SLA, When breached, Then it escalates to the next level.
  - Given an away manager, When they delegate, Then a bounded, attributed delegate can approve.
  - Given a timeout with no target, When reached, Then it holds-and-flags (never auto-approves).
- **Edge Case Analysis:**
  - *Invalid Input:* invalid delegation target rejected.
  - *Permission Failure:* delegate ineligible → blocked.
  - *Concurrent Update:* escalation + late manager action → first valid decision wins.
  - *Duplicate Data:* idempotent escalation.
  - *System Failure:* reliable escalation (outbox); no lost request.
  - *Workflow Failure:* delegate chain bounded.
- **QA Coverage:**
  - *Positive:* escalation; delegation; bounded chain.
  - *Negative:* silent auto-approval (must not occur); ineligible delegate.
  - *Boundary:* escalation exactly at SLA; last delegate in chain.

### UC-LEV-007 — Student Leave (balance-free sub-domain)
- **Module:** Leave Management · **Priority:** High
- **Actors:** Guardian / Student (primary), Class-Teacher / Admin (approver), System
- **Goal:** Manage student leave as a distinct, balance-free sub-domain whose approval Attendance merely reflects (Conflict C-04).
- **Description:** Student leave has its own request/approval lifecycle (no accrual/balance); on approval, Attendance reflects the period as excused (ATT-005) — this module owns the workflow; Attendance owns only the reflection.
- **Business Rules Applied:** LEV-009, ATT-005, GRD-N-004 (Conflict C-04).
- **Preconditions:** Active student enrollment; guardian/authorized requester.
- **Trigger:** Guardian/student requests leave.
- **Main Success Scenario:**
  1. Guardian submits a student-leave request (dates, reason) for their child (GRD-N-004).
  2. The request routes to the class-teacher/admin for approval (no balance check — balance-free).
  3. On approval, this module records the approved leave.
  4. Attendance reflects the dates as excused (ATT-005) — not absent.
- **Alternative Flows:** A1) Long medical leave with documentation.
- **Exception Flows:** E1) Overlap with exams/holidays handled per policy. E2) Custody-restricted requester → blocked (GRD-N-006).
- **Validation Rules:** Balance-free; requester authorized for the student; Attendance reflects, not owns (LEV-009, ATT-005).
- **Permissions Required:** Guardian (own child) / `student.leave.approve` (approver).
- **Notifications Triggered:** `STUDENT_LEAVE_APPROVED` to guardian; class-teacher notice.
- **Audit Events Generated:** `STUDENT_LEAVE_RECORDED/APPROVED`.
- **Data Created:** Student-leave record.
- **Data Updated:** Attendance reflection (excused).
- **Data Deleted:** None.
- **Post Conditions:** Student leave approved here; Attendance shows excused; boundary respected.
- **Related Use Cases:** UC-ATT-002, UC-GRD-N-003.
- **Acceptance Criteria:**
  - Given a guardian request for their child, When approved here, Then Attendance reflects the dates as excused (not absent).
  - Given student leave, When processed, Then no leave balance is consulted (balance-free).
  - Given a custody-restricted requester, When requesting, Then it is blocked.
- **Edge Case Analysis:**
  - *Invalid Input:* invalid date range rejected.
  - *Permission Failure:* requesting for a non-child → denied.
  - *Concurrent Update:* leave approval vs same-day attendance marking → excused reflection wins for the period.
  - *Duplicate Data:* duplicate leave for the same dates deduped.
  - *System Failure:* atomic record + reflection.
  - *Workflow Failure:* approval pends; attendance not pre-excused until approved.
- **QA Coverage:**
  - *Positive:* approve → excused reflection.
  - *Negative:* non-child request; custody-restricted; balance consulted (must not be).
  - *Boundary:* leave overlapping an exam day; approval after attendance already marked absent (correction to excused).

---

## 7. Compact Specifications (routine use cases)

- **UC-LEV-001 — Request Leave (balance + eligibility)** · *High* · Rules: LEV-001, LEV-002, LEV-006. Staff request with type/dates; balance and eligibility checked. *Audit:* `LEAVE_REQUESTED`. *Edge:* insufficient balance blocks (UC-LEV-017); overlap blocked (UC-LEV-018). *QA:* request; balance check; eligibility.
- **UC-LEV-004 — Compute Duration (holidays, sandwich)** · *High* · Rules: LEV-003. Compute leave days excluding holidays/weekends with sandwich rules. *Edge:* sandwich leave counts per policy; holiday exclusion. *QA:* duration accuracy; sandwich; holiday exclusion.
- **UC-LEV-005 — Reflect in Attendance & Payroll** · *High* · Rules: LEV-006, HR-004, ATT-005. Approved leave → excused attendance + payroll treatment (paid/unpaid). *Edge:* unpaid leave → LOP in payroll. *QA:* attendance reflection; payroll effect.
- **UC-LEV-006 — Manage Balance (accrual/carry-forward/encashment)** · *High* · Rules: LEV-002. Accrue, carry forward (capped), encash per policy. *Audit:* balance changes. *Edge:* carry-forward cap; encashment on separation (HR-007). *QA:* accrual; carry-forward cap; encashment.
- **UC-LEV-008 — Coordinate Coverage / Substitution** · *Medium* · Rules: LEV-008, TCH-005. Approved teacher leave coordinates a substitute. *Edge:* substitute attributed; reverts after (TCH-005). *QA:* coverage assigned; revert.
- **UC-LEV-009 — View Leave / Balance (scoped)** · *Medium* · Rules: AUTHZ-003. Staff see own; managers see reports' leave (scoped). *QA:* scope; own-balance.
- **UC-LEV-010 — Configure Leave Types & Policies** · *High* · Rules: LEV-001/002/003, CFG-004. Configure types, accrual, holidays, sandwich, approval routing (versioned). *Permissions:* `leave.config.manage`. *Edge:* policy versioned. *QA:* config; versioning.
- **UC-LEV-011 — Backdated / Exceptional Leave (governed)** · *High* · Rules: LEV-007, AUTHZ-009. Governed backdated/exceptional leave (elevated, reasoned, audited). *Edge:* limited window; approval. *QA:* governed backdate; ungoverned blocked.
- **UC-LEV-012 — Search Leave Records** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-LEV-013 — Leave Balance & Utilization Report** · *Medium* · Rules: REP-002. Balances, utilization, pending approvals. *Edge:* scoped; manager view of reports. *QA:* balance accuracy; pending list.
- **UC-LEV-014 — Bulk / Import Leave (opening balances)** · *Low* · Rules: LEV-002. Import opening balances/historical leave (validated). *Edge:* reconciled; invalid rows reported. *QA:* clean import; reconciliation.
- **UC-LEV-015 — Export Leave Data** · *Low* · Rules: REP-005. Governed, scoped export. *QA:* scoped export.
- **UC-LEV-016 — Leave Approval Workflow** · *High* · Rules: WFL-002/004/005/006, LEV-004/005. Version-pinned manager-routed workflow with escalation/delegation/timeout. *QA:* pinning; routing; escalation; SoD.
- **UC-LEV-017 — Insufficient Balance Blocked (Exception)** · *High* · Rules: LEV-002. Staff leave exceeding balance blocked (unless LWP policy). *QA:* blocked; LWP path.
- **UC-LEV-018 — Overlapping Leave Blocked (Exception)** · *Medium* · Rules: LEV-001. Overlapping leave for the same person blocked. *QA:* overlap blocked; adjacent allowed.

## 8. Module-level QA & Edge Themes
- **Manager-routed approval + stall-proofing (LEV-004/005 / WFL-005/006/007):** routing via the reporting line with escalation/delegation/timeout — and never a silent auto-approval — is the headline suite.
- **Student-leave boundary (LEV-009 / C-04):** this module owns the student-leave workflow; Attendance only reflects approved leave as excused — no duplicate ownership.
- **Duration computation (LEV-003):** holiday/weekend exclusion and sandwich rules — a precise correctness suite.
- **Attendance & payroll integration (LEV-006):** approved leave flows to excused attendance and correct paid/unpaid payroll treatment.
- **Governed backdating (LEV-007):** exceptional/backdated leave is elevated, reasoned, and audited.
