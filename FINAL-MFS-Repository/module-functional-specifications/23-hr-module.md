# 23 — HR Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`HR-001…008`), and Use Case Repository (`UC-HR-001…017`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Organizational HR and payroll: configurable department/designation structure, probation-to-confirmation lifecycle, versioned salary structures, payroll-relevant staff attendance, payroll computation integrity with strict separation of duties, governed separation/settlement, and history-linked rehire.

**Business Goal.** Run accurate, reproducible, fraud-resistant payroll (compute ≠ approve ≠ disburse) with immutable payslips, and govern the employment lifecycle from confirmation through separation.

**Scope.** Department/designation structure; probation→confirmation; versioned salary structures (reproducible payslips); staff attendance (payroll-relevant); deterministic payroll computation; payroll SoD; governed separation/clearance/final settlement; rehire as new history-linked employment.

**Out of Scope.** Base staff record (Staff module). Teacher specialization (Teacher module). Leave balances/approval (Leave module — feeds payroll). Payment-gateway disbursement mechanics (Payment/integration). Config versioning (Configuration Engine).

---

## 2. Actors

**Primary Actors.** HR Administrator (structure, lifecycle), Payroll Officer (computes payroll), Finance Approver (approves — SoD), Disbursing Officer (disburses — SoD), System (lifecycle, salary versioning, payroll computation, settlement).

**Secondary Actors.** Staff/Leave/Attendance modules (inputs), Workflow Engine (payroll/separation approvals, SoD), Payment/Finance (disbursement), File (payslips), Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Department/designation structure | Configure departments/designations (versioned). | High | Configuration Engine |
| FR-002 | Probation→confirmation | Manage probation→confirmation on milestone. | High | Staff (STF-004) |
| FR-003 | Versioned salary structure | Define salary structures, versioned, effective-dated. | High | Configuration Engine |
| FR-004 | Staff attendance (payroll-relevant) | Track staff attendance distinct from student; feeds payroll. | High | Leave (LEV-006) |
| FR-005 | Deterministic payroll computation | Compute payroll reproducibly, stamped to salary version. | Critical | FR-003, Leave |
| FR-006 | Payroll SoD | Compute ≠ approve ≠ disburse; immutable finalized payslips. | Critical | Workflow, AUTHZ-009 |
| FR-007 | Separation, clearance & settlement | Governed clearance + accurate final settlement. | High | Staff (STF-008), Leave |
| FR-008 | Rehire (history-linked) | Rehire as new employment linked to prior history. | Medium | Staff (STF-003) |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Dept/designation structure | Configurable org. | Flexible HR model. |
| Probation lifecycle | Confirmation milestone. | Governed onboarding. |
| Versioned salary | Reproducible payslips. | Defensible payroll. |
| Staff attendance | Payroll-relevant. | Accurate pay. |
| Deterministic payroll | Reproducible. | Correctness. |
| Payroll SoD | Compute/approve/disburse split. | Fraud prevention. |
| Governed settlement | Clearance + accurate. | Clean exits. |
| History-linked rehire | New, linked. | Continuity. |

---

## 5. Screens

Department/Designation Structure; Probation→Confirmation; Salary Structure (versioned); Staff Attendance; Payroll Computation; Payroll Approval & Disbursement; Payslip View; Separation/Clearance/Settlement; Rehire; Payroll & HR Reports; Bulk Payroll Run; HR Data Import/Export; Payroll/Separation Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Dept/Designation | Configure, Save (versioned) | — |
| Probation→Confirmation | Confirm (milestone) | Bulk Confirm |
| Salary Structure | Define, Publish Version, Effective-Date | — |
| Staff Attendance | Record, Reflect Leave, View | Bulk Import |
| Payroll Computation | Compute, Recompute, View Provenance | Bulk Run |
| Approval & Disbursement | Approve (distinct), Disburse (distinct), Generate Payslips | — |
| Payslip | View, Download (signed URL) | — |
| Separation/Settlement | Run Clearance, Compute Settlement, Disburse | — |
| Payroll Report | Run, Filter, Export | Export |

---

## 7. Forms

**Salary Structure** — components (`name`, `amount/formula`), `effectiveDate`, `version`. Validation: typed/versioned (CFG-004); effective forward; payslips keep version (HR-003/P6).

**Payroll Computation** — `period`, `scope`. Validation: effective salary structure resolved (HR-003); attendance/approved leave incorporated (HR-004/LEV-006); deterministic, version-stamped (HR-005); missing inputs block (no silent defaults).

**Payroll Approval/Disbursement** — `approve` (distinct role), `disburse` (distinct role). Validation: compute ≠ approve ≠ disburse (HR-006/AUTHZ-009 / UC-HR-016); finalized payslips immutable (UC-HR-017); reproducible (FILE-007).

**Separation/Settlement** — clearance checklist, `settlement` (salary, leave encashment, deductions). Validation: clearance + handover complete (HR-007/TCH-006); leave encashment (LEV-002); approved (SoD); archive-not-delete.

**Rehire** — `priorStaff` (link). Validation: new employment, history-linked (HR-008/STF-003).

---

## 8. Search & Filter Requirements

**HR/Payroll:** by staff, department/designation, period, payroll status, settlement status. Sorting: period/staff. Pagination: server-side, 25 default. Scope-bound; salary masked.

---

## 9. Table Requirements

**Payroll table:** Staff, Period, Gross, Deductions, Net, Status, Payslip. Salary columns masked/need-to-know (UC-HR-012). Sorting on Period/Net. Filtering as above. Export (governed, masked). Bulk: run.

---

## 10. Workflow Requirements

**Trigger events:** structure config, confirmation, salary change, compute, approve, disburse, separation, settlement, rehire. **Status changes:** payroll `COMPUTED → APPROVED → DISBURSED → FINALIZED`; staff `PROBATION → CONFIRMED`. **Approvals:** payroll (SoD), salary change, separation via Workflow Engine. **Notifications:** payslip available, confirmation, settlement. **Audit:** computation (provenance), approval, disbursement, settlement (immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Configure dept/designation | `hr.config.manage` |
| Manage probation/confirmation | `hr.lifecycle.manage` |
| Manage salary structure | `payroll.config.manage` |
| Compute payroll | `payroll.compute` |
| Approve payroll | `payroll.approve` (distinct) |
| Disburse payroll | `payroll.disburse` (distinct) |
| Approve salary change | `payroll.salary.approve` |
| Manage separation/settlement | `hr.settlement.manage` |
| View payslip/HR (scoped) | `hr.view` |
| Import/export | `hr.import`, `hr.export` |

Compute/approve/disburse are distinct permissions held by distinct roles (HR-006).

---

## 12. Business Rule References

HR-001 (department/designation structure), HR-002 (probation-to-confirmation), HR-003 (versioned salary structure), HR-004 (staff attendance payroll-relevant), HR-005 (payroll computation integrity), HR-006 (payroll separation of duties), HR-007 (governed separation, clearance & final settlement), HR-008 (rehire is new employment, history-linked). Cross-cutting: STF-004/008 (status/separation), TCH-006 (handover), LEV-002/006 (encashment/payroll integration), PAY-001/008 (disbursement), FILE-007 (immutable payslip), WFL-002/004, AUTHZ-009 (SoD), CFG-004, AUD-001.

## 13. Use Case References

UC-HR-001 (Configure Dept/Designation), UC-HR-002 (Probation→Confirmation), UC-HR-003 (Versioned Salary Structure), UC-HR-004 (Staff Attendance), UC-HR-005 (Compute Payroll — deterministic), UC-HR-006 (Approve & Disburse — SoD), UC-HR-007 (Separation, Clearance & Settlement), UC-HR-008 (Rehire — history-linked), UC-HR-009 (View Payslip/HR — scoped), UC-HR-010 (Approve Salary Change), UC-HR-011 (Search), UC-HR-012 (Payroll & HR Reports), UC-HR-013 (Bulk Payroll Run), UC-HR-014 (Import/Export — governed), UC-HR-015 (Payroll/Separation Workflow), UC-HR-016 (Payroll Self-Approval Blocked), UC-HR-017 (Edit Finalized Payslip Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Configure dept/designation | POST/PUT | HR Admin |
| Confirm (probation→confirmation) | POST | HR Admin |
| Define / version salary structure | POST/PUT | HR Admin |
| Record staff attendance | POST | HR Admin |
| Compute / recompute payroll | POST | Payroll Officer |
| Approve payroll (distinct) | POST | Finance Approver |
| Disburse payroll (distinct) | POST | Disbursing Officer |
| Approve salary change | POST | Approver |
| Separation, clearance & settlement | POST | HR Admin (→ approval) |
| Rehire (history-linked) | POST | HR Admin |
| Get payslip (signed URL) | GET | Staff (own) |
| Payroll & HR reports | GET | Admin |
| Bulk payroll / import / export | POST/GET | Admin |

Compute, approve, and disburse are enforced as distinct roles (HR-006/AUTHZ-009); finalized payslips immutable (HR-003/FILE-007).

---

## 15. Database Requirements

**Entities:** `Department`/`Designation` (via Config), `SalaryStructure` (version, effectiveDate), `StaffAttendance` (payroll-relevant), `Payroll` (period, gross/deductions/net, status, provenance{salaryVer, configVer}), `Payslip` (immutable, file ref), `Settlement` (clearance, encashment, deductions), `EmploymentHistory` (rehire links). **Relationships:** Staff 1—* Payroll; Payroll 1—1 Payslip; Staff 1—* EmploymentHistory. **Indexes:** index(Payroll.staffId, period), unique(Payslip.payrollId), index(SalaryStructure.effectiveDate). SoD enforced at workflow/permission layer (HR-006); payslips immutable (HR-003); provenance stamped (P6).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Payslip available; confirmation; settlement. |
| In-App | Payroll approvals; salary-change approvals; separation tasks. |
| SMS/Push | Payslip-ready alert (minimized). |

Payslips via signed URL (FILE-005); salary content need-to-know.

---

## 17. Audit Requirements

Log: structure/salary changes (version), confirmation, payroll computation (provenance), approval, disbursement, settlement, rehire. Record who/when/before/after. Payroll SoD steps and settlement are first-class audit events; finalized payslip edits blocked. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Payroll register, Cost-to-company, Department/designation distribution, Attrition/separation, Settlement register. **Exports:** governed masked payroll/HR export. **Dashboards:** HR/payroll overview (run status, pending approvals, settlement queue).

---

## 19. Error Handling

**Validation:** missing salary/attendance, self-approval, finalized-payslip edit → specific errors (UC-HR-016/017). **Permission:** compute=approve → SoD block; salary unmask without need-to-know → masked. **Workflow:** payroll/settlement pending → not finalized until approved. **System:** Leave/attendance inputs unavailable → compute blocked (no silent defaults); closed period → governed reopen.

---

## 20. Edge Cases

**Concurrent updates:** recompute during correction → serialized, re-stamped. **Duplicate data:** idempotent computation; duplicate disbursement prevented (PAY-008). **Partial failures:** bulk run partial → resumable, per-staff report. **Rollback:** approval returns run → recompute then re-approve. **Mid-period race:** salary change mid-period → version boundary handled; payslip stamps the version used.

---

## 21. Acceptance Criteria

**Functional.** Departments/designations and salary structures are versioned; probation→confirmation is governed; payroll computes deterministically from the effective salary version with attendance/approved leave and stamps provenance; compute, approve, and disburse are distinct roles (self-approval blocked); finalized payslips are immutable; separation runs clearance + accurate settlement (incl. leave encashment); rehire is new, history-linked employment.

**Business.** Payroll is accurate, reproducible, and fraud-resistant via SoD; payslips are defensible and immutable; exits are clean and settled; the employment lifecycle is governed end to end.

---

## 22. Future Enhancements

Tax/statutory-deduction engines per jurisdiction; payslip templates per client; bank-file generation for disbursement; loan/advance management; appraisal/increment workflows; provident-fund/gratuity tracking; payroll analytics and forecasting.
