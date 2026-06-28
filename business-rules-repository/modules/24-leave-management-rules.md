# 24 — Leave Management Business Rules

> **Prefix note:** HR-phase prefixes are distinct — Staff `STF`, Teacher `TCH`, HR/Payroll `HR`, Leave `LEV`.
> **Scope decision:** This module governs **staff leave** as its primary domain (balances, accrual, payroll integration, manager approval). **Student leave** is a deliberately *distinct* sub-domain — no balances, no payroll, approved by class-teacher/admin, integrating with attendance (ATT-005) — and is specified in its own rule (LEV-009) and section so the two are never conflated.

## 1. Module Purpose
Govern **leave** — requesting time off, approving it, tracking balances, and reflecting it in attendance and payroll. For **staff**, leave has types, accrual/carry-forward/encashment balances, a manager-routed approval workflow (with escalation/delegation/timeout via the Workflow Engine, Doc 28), coverage/substitution coordination (TCH-005), and payroll consequences (HR-004). For **students**, leave is a lighter request-and-approve flow that marks attendance as excused (ATT-005) without balances or pay.

## 2. Actors
- **Staff Member** — requests leave; views balances.
- **Reporting Manager / Approver** — approves/rejects within the reporting line (STF-006); delegates when away.
- **HR Administrator** — configures leave types/policies, handles encashment/settlement, governs backdated/exceptional leave.
- **Student / Guardian** — request student leave (distinct sub-domain).
- **Class Teacher / Academic Admin** — approve student leave.
- **System** — enforces balances, workflow routing, coverage, sandwich/holiday rules, and attendance/payroll integration.

## 3. Use Cases

**Use Case ID:** UC-LEV-01 — Apply for Staff Leave
**Actors:** Staff Member, Reporting Manager, System
**Description:** A staff member requests leave that is routed for approval and reflected downstream.
**Preconditions:** Leave types/balances configured; requester is active staff.
**Main Flow:** 1) Staff selects leave type, dates (full/half-day), reason. 2) System checks balance and overlaps; computes the leave-day count per holiday/weekend/sandwich rules. 3) Routes to the manager (escalation/delegation/timeout per WFL). 4) On approval, balance is debited, staff attendance reflects leave, and coverage/substitution is coordinated (TCH-005); payroll reflects unpaid types.
**Alternative Flow:** A1) Half-day or short leave. A2) Backdated leave via governed exception (LEV-007).
**Exception Flow:** E1) Insufficient balance → reject or route as unpaid per policy (LEV-002). E2) Manager unavailable → delegate/escalate (LEV-005).
**Post Conditions:** Leave approved/rejected; balance/attendance/payroll updated; audited.
**Business Rules Applied:** LEV-001, LEV-002, LEV-003, LEV-004, LEV-005, LEV-006.

**Use Case ID:** UC-LEV-02 — Cancel / Modify Approved Leave
**Actors:** Staff Member, Manager, System
**Description:** Change or cancel leave before/after it starts.
**Preconditions:** Approved leave exists.
**Main Flow:** 1) Staff requests cancellation/modification. 2) System re-credits balance for unused days, re-routes approval if needed, and reverses attendance/payroll effects.
**Exception Flow:** E1) Cancellation after the leave is partly availed → only unused days re-credited.
**Post Conditions:** Leave updated; balances/attendance corrected; audited.
**Business Rules Applied:** LEV-006, LEV-008.

**Use Case ID:** UC-LEV-03 — Request Student Leave (Distinct Sub-Domain)
**Actors:** Student/Guardian, Class Teacher/Admin, System
**Description:** A student requests leave that, if approved, marks attendance excused.
**Preconditions:** Student actively enrolled; student-leave enabled.
**Main Flow:** 1) Guardian/student submits a leave request (dates, reason). 2) Class teacher/admin approves. 3) Approved dates mark attendance excused/on-leave (ATT-005); no balance or payroll involved.
**Exception Flow:** E1) Overlap with exams → flagged per policy. E2) Excessive leave → safeguarding/academic review (links ATT-008).
**Post Conditions:** Student leave approved; attendance reflects excused; audited.
**Business Rules Applied:** LEV-009.

## 4. Business Rules

**Rule ID:** LEV-001
**Rule Name:** Configurable Leave Types & Policies
**Description:** Leave types and their policies (paid/unpaid, accrual, limits, eligibility) are configurable per institute.
**Priority:** High
**Category:** Configuration
**Preconditions:** Leave setup.
**Business Rule:** Types (casual, sick, earned/annual, maternity/paternity, unpaid, special) are configured with paid/unpaid status, accrual rules, per-request and annual limits, eligibility (e.g., probation restrictions), and documentation requirements (e.g., medical certificate over N days).
**System Action:** Resolve leave policy via the Configuration Engine; enforce per type.
**Validation:** Type/policy valid; eligibility consistent.
**Failure Behavior:** Reject undefined types; enforce eligibility.
**Audit Requirement:** Log leave-policy changes.
**Example Scenario:** Sick leave over three days requires a medical certificate per policy.
**Related Rules:** LEV-002, HR-002 (probation), CFG.

**Rule ID:** LEV-002
**Rule Name:** Balance Accrual, Carry-Forward & Encashment
**Description:** Leave balances accrue, may carry forward within caps, and may be encashable per policy.
**Priority:** High
**Category:** Entitlement integrity
**Preconditions:** Balance-bearing leave types.
**Business Rule:** Balances accrue per the configured schedule; carry-forward to the next year is capped per policy; lapse beyond the cap is explicit; encashment (e.g., on separation) follows defined rules. Balances never go negative without an explicit unpaid/leave-without-pay path.
**System Action:** Accrue, carry-forward (capped), lapse, and encash per policy; prevent silent negatives.
**Validation:** Accrual/cap/encashment rules defined; balance ≥ 0 (or explicit LWP).
**Failure Behavior:** Reject over-balance paid leave; route excess as unpaid per policy.
**Audit Requirement:** Log accrual, carry-forward, lapse, encashment.
**Example Scenario:** Unused earned leave carries forward up to 30 days; the rest lapses, recorded.
**Related Rules:** LEV-001, HR-007 (settlement encashment).

**Rule ID:** LEV-003
**Rule Name:** Holiday/Weekend & Sandwich-Leave Computation
**Description:** The chargeable leave-day count is computed per defined holiday/weekend and sandwich-leave rules.
**Priority:** High
**Category:** Computation (commonly disputed)
**Preconditions:** Leave-day computation.
**Business Rule:** Whether intervening weekends/holidays count toward leave (the "sandwich" rule) is configurable and applied deterministically; half-days are supported; the computed chargeable days are shown transparently before submission.
**System Action:** Compute chargeable days per the sandwich/holiday policy; show the breakdown.
**Validation:** Sandwich/holiday policy defined; calendar resolved.
**Failure Behavior:** Block computation if policy/calendar undefined.
**Audit Requirement:** Chargeable-day computation recorded with the leave.
**Example Scenario:** Leave Friday and Monday counts the weekend as leave only if the sandwich rule is enabled.
**Related Rules:** LEV-002, academic/HR calendar.

**Rule ID:** LEV-004
**Rule Name:** Manager-Routed Approval
**Description:** Staff leave routes to the approver per the reporting line, recorded with decision and reason.
**Priority:** High
**Category:** Workflow
**Preconditions:** Leave submitted.
**Business Rule:** Approval routes to the reporting manager (STF-006) or a configured approver chain; the decision (approve/reject) with reason is recorded; multi-level approval applies for long leave per policy.
**System Action:** Route per reporting line/policy; record decisions.
**Validation:** Approver authorized; routing valid.
**Failure Behavior:** Block out-of-band approvals.
**Audit Requirement:** Log `LEAVE_DECISION` with approver and reason.
**Example Scenario:** A week's leave needs the department head's approval; a single sick day needs the line manager's.
**Related Rules:** STF-006, LEV-005, WFL.

**Rule ID:** LEV-005
**Rule Name:** Escalation, Delegation & Timeout
**Description:** Pending leave that stalls escalates; an away approver delegates; timeouts are handled per the Workflow Engine.
**Priority:** High
**Category:** Workflow reliability
**Preconditions:** Approval routing.
**Business Rule:** If an approver does not act within a configured time, the request escalates (to the next level) or follows a timeout policy; an approver who is themselves on leave delegates approval authority so requests never stall.
**System Action:** Apply escalation/delegation/timeout via the Workflow Engine (Doc 28).
**Validation:** Escalation/delegation/timeout configured; delegate authorized.
**Failure Behavior:** No indefinite stalling; unresolved requests escalate/flag.
**Audit Requirement:** Log escalations and delegated approvals.
**Example Scenario:** A manager on vacation has approvals delegated to a deputy so staff leave is not blocked.
**Related Rules:** WFL (Doc 28), STF-006, LEV-004.

**Rule ID:** LEV-006
**Rule Name:** Attendance & Payroll Integration
**Description:** Approved leave reflects in staff attendance (not absence) and drives payroll for unpaid types.
**Priority:** High
**Category:** Integration
**Preconditions:** Leave approved.
**Business Rule:** Approved leave marks staff attendance as on-leave (HR-004), distinct from unauthorized absence; paid leave does not reduce pay; unpaid leave (or over-balance) feeds payroll deductions deterministically; reversal on cancellation re-corrects both.
**System Action:** Update attendance; feed payroll per paid/unpaid; reverse on cancellation.
**Validation:** Leave status authoritative; payroll period not yet locked (or governed adjustment).
**Failure Behavior:** If payroll is locked, corrections via governed adjustment (HR-006).
**Audit Requirement:** Log attendance/payroll effects of leave.
**Example Scenario:** Three approved paid sick days do not affect salary; one unpaid day is deducted.
**Related Rules:** HR-004, HR-005, HR-006.

**Rule ID:** LEV-007
**Rule Name:** Backdated & Exceptional Leave Are Governed
**Description:** Backdated or policy-exceeding leave requires elevated authorization and a reason.
**Priority:** Medium
**Category:** Governance
**Preconditions:** Backdated/exceptional leave.
**Business Rule:** Leave for past dates or beyond normal limits (e.g., emergency, extended medical) requires elevated approval and recorded justification; it still reconciles attendance/payroll correctly (governed adjustment if a period is locked).
**System Action:** Gate backdated/exceptional leave behind elevated approval; reconcile downstream.
**Validation:** Authorized; reason captured.
**Failure Behavior:** Reject ungoverned backdating.
**Audit Requirement:** Log `LEAVE_BACKDATED/EXCEPTIONAL` with justification.
**Example Scenario:** Emergency medical leave applied retroactively is approved by HR with documentation.
**Related Rules:** ATT-004 (parallel), HR-006.

**Rule ID:** LEV-008
**Rule Name:** Coverage / Substitution Coordination
**Description:** Approving a teacher's leave coordinates coverage so classes are not left unattended.
**Priority:** Medium
**Category:** Continuity
**Preconditions:** A teacher's leave is approved.
**Business Rule:** Teacher leave triggers substitute coordination (TCH-005) for affected periods; approval may be conditional on coverage availability per policy; coverage gaps are surfaced.
**System Action:** Link approved teacher leave to substitute assignment; surface gaps.
**Validation:** Coverage policy defined; substitutes resolvable.
**Failure Behavior:** Flag uncovered periods; conditional approval per policy.
**Audit Requirement:** Log coverage coordination.
**Example Scenario:** A teacher's approved leave automatically prompts substitute assignment for their classes.
**Related Rules:** TCH-005, LEV-004.

**Rule ID:** LEV-009
**Rule Name:** Student Leave Is a Distinct, Balance-Free Sub-Domain
**Description:** Student leave is requested by student/guardian, approved by class-teacher/admin, and only marks attendance excused — no balances, no payroll.
**Priority:** High
**Category:** Domain separation
**Preconditions:** Student-leave enabled; student actively enrolled.
**Business Rule:** Student leave has its own light workflow; approval marks the covered dates excused/on-leave in attendance (ATT-005); there are no leave balances, accrual, encashment, or payroll effects; excessive student leave may trigger academic/safeguarding review (ATT-008). It is never conflated with staff leave entitlements.
**System Action:** Run the student-leave flow; mark attendance excused on approval; flag excess.
**Validation:** Student enrolled; approver authorized (class teacher/admin).
**Failure Behavior:** Reject if not enrolled; flag exam-period or excessive leave per policy.
**Audit Requirement:** Log `STUDENT_LEAVE_REQUESTED/DECIDED` and attendance effect.
**Example Scenario:** A guardian requests three days' leave for a family event; on approval, those days show excused, not absent.
**Related Rules:** ATT-005, ATT-008, ENR-008.

## 5. Validation Rules
- Leave types/policies defined; eligibility (incl. probation) enforced.
- Balances accrue/carry-forward/encash per policy; no silent negatives (explicit LWP).
- Chargeable days computed per sandwich/holiday policy, shown transparently.
- Approval routes per reporting line; escalation/delegation/timeout configured.
- Approved leave integrates with attendance/payroll; cancellations reverse correctly.
- Backdated/exceptional leave governed; student leave is balance-free and attendance-only.

## 6. State Machine
*(Leave request lifecycle; applies to staff; student leave uses the simplified subset.)*

**State Name:** DRAFT
**Description:** Request being prepared.
**Allowed Transitions:** → SUBMITTED; → DISCARDED.
**Forbidden Transitions:** balance debit before approval.
**System Actions:** Compute chargeable days; check balance/overlap.

**State Name:** SUBMITTED
**Description:** Awaiting approval.
**Allowed Transitions:** → UNDER_APPROVAL; → CANCELLED (by requester before decision).
**Forbidden Transitions:** attendance/payroll effect before approval.
**System Actions:** Route to approver; start escalation timers.

**State Name:** UNDER_APPROVAL
**Description:** In the approval workflow (possibly multi-level/escalated).
**Allowed Transitions:** → APPROVED; → REJECTED; → CANCELLED.
**Forbidden Transitions:** indefinite pending (escalation applies).
**System Actions:** Escalate/delegate/timeout per WFL.

**State Name:** APPROVED
**Description:** Granted; downstream effects applied.
**Allowed Transitions:** → CANCELLED/MODIFIED (re-credit unused); → AVAILED.
**Forbidden Transitions:** silent reversal.
**System Actions:** Debit balance; mark attendance; feed payroll; coordinate coverage.

**State Name:** AVAILED
**Description:** Leave taken (period elapsed).
**Allowed Transitions:** → ARCHIVED; partial cancellation only for unused future days.
**Forbidden Transitions:** re-crediting availed days.
**System Actions:** Finalize attendance/payroll effects.

**State Name:** REJECTED / CANCELLED / ARCHIVED
**Description:** Not granted, withdrawn, or historical.
**Allowed Transitions:** to ARCHIVED.
**Forbidden Transitions:** edits.
**System Actions:** Re-credit unused balance; retain record.

## 7. Status Definitions
`DRAFT` · `SUBMITTED` · `UNDER_APPROVAL` · `APPROVED` · `AVAILED` · `REJECTED` · `CANCELLED` · `MODIFIED` · `ARCHIVED`. Student-leave subset: `SUBMITTED` · `APPROVED`/`REJECTED` · `CANCELLED`.

## 8. Workflow Rules
- Staff leave routes per reporting line with escalation/delegation/timeout (LEV-005, WFL Doc 28).
- Long/extended leave may require multi-level approval and documentation (LEV-001/LEV-004).
- Backdated/exceptional leave needs elevated authorization (LEV-007).
- Student leave runs a light class-teacher/admin approval, separate from staff workflows (LEV-009).

## 9. Permission Rules
- `leave.apply` — staff apply for own leave.
- `leave.approve` — approve within reporting line/delegation.
- `leave.policy.manage` — configure types/balances/policies.
- `leave.exceptional.manage` — backdated/exceptional governance, encashment.
- `student_leave.approve` — class-teacher/admin approve student leave.
- `leave.view` — read leave/balances (staff own; managers their reports; students/guardians own).

## 10. Notification Rules
- `LEAVE_SUBMITTED` → notify approver; `LEAVE_DECISION` → notify requester.
- Escalation/delegation → notify the new approver.
- Approved teacher leave → notify coverage coordinator/substitute (LEV-008).
- `STUDENT_LEAVE_DECIDED` → notify guardian; attendance reflects excused.
- Balance lapse/encashment notices to staff.

## 11. Audit Requirements
Mandatory: `LEAVE_SUBMITTED`, `LEAVE_DECISION` (approver/reason), escalations/delegations, balance accrual/carry-forward/lapse/encashment, attendance/payroll effects, `LEAVE_BACKDATED/EXCEPTIONAL`, `STUDENT_LEAVE_REQUESTED/DECIDED`. Leave touches pay and attendance — fully attributable.

## 12. Data Retention Rules
- Staff leave records and balances retained per labor/payroll retention.
- Leave affecting payroll retained with financial records.
- Student-leave records retained per academic/attendance retention (lighter; no financial tie).
- Anonymization follows the staff/student record; legal-hold overrides erasure.

## 13. Edge Cases
- **Sandwich leave:** weekends/holidays between leave days counted only if the rule is enabled — computed transparently (LEV-003), a classic dispute.
- **Insufficient balance:** rejected or routed as unpaid per policy; never silent negative (LEV-002).
- **Approver on leave:** delegation prevents stalling (LEV-005).
- **Leave during exams (teacher):** coverage coordination required; may be conditionally approved (LEV-008).
- **Cancellation after partial availment:** only unused future days re-credited (LEV-006/LEV-008).
- **Backdated emergency leave:** governed exception with documentation (LEV-007).
- **Maternity/statutory leave:** honors statutory minimums regardless of balance (LEV-001).
- **Encashment on separation:** computed in final settlement (HR-007/LEV-002).
- **Student leave vs absence:** approved student leave is excused, not absent (LEV-009/ATT-005); excessive leave triggers review (ATT-008).
- **Payroll already locked when leave changes:** governed adjustment, not silent edit (LEV-006/HR-006).

## 14. Failure Scenarios
- **Over-balance paid leave:** blocked or unpaid per policy (LEV-002).
- **Indefinite pending approval:** escalated/flagged (LEV-005).
- **Undefined sandwich/calendar policy:** computation blocked (LEV-003).
- **Out-of-band approval:** rejected (LEV-004).
- **Re-crediting availed days:** prevented (LEV-006).
- **Conflating student leave with staff entitlements:** prevented by domain separation (LEV-009).

## 15. Exception Handling Rules
- Balances never go silently negative; over-balance is explicit unpaid or rejected.
- Approvals never stall indefinitely (escalation/delegation/timeout).
- Downstream attendance/payroll effects reverse correctly on cancellation; locked periods use governed adjustment.
- Student leave is strictly attendance-only; never touches balances/payroll.

## 16. Compliance Considerations
- **Labor law:** statutory leave (maternity, etc.) honored regardless of balance; entitlements per law.
- **Fair process:** transparent computation, routed approvals, and governed exceptions support fair treatment.
- **Payroll accuracy:** correct paid/unpaid handling and audit support statutory payroll compliance.
- **Student welfare:** student-leave/attendance integration supports safeguarding (excused vs absent clarity).

## 17. Future Considerations
- Self-service leave portal with balance forecasting.
- Team leave calendars and conflict detection for coverage planning.
- Statutory leave-policy templates per jurisdiction.
- Automated student-leave from guardian app feeding attendance directly.
