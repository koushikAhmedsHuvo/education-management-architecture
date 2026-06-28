# 23 — HR Use Cases

Transforms the HR business rules (`HR-001`…`HR-008`) into use cases. Organizational HR: configurable department/designation structure, probation-to-confirmation, versioned salary, staff attendance, payroll integrity with separation of duties, governed separation/settlement, and history-linked rehire.

## 1. Primary Actors
HR Administrator (structure, lifecycle, payroll), Payroll Officer (computes payroll), Finance Approver (payroll approval — SoD).

## 2. Secondary Actors
System (lifecycle, salary versioning, payroll computation, settlement), Staff/Leave/Attendance modules (inputs), Workflow Engine (payroll/separation approvals, SoD), Payment/Finance (disbursement), Audit service.

## 3. Goals
Configure departments/designations; manage probation→confirmation; version salary structures (reproducible payslips); track payroll-relevant staff attendance; compute payroll deterministically with strict SoD (compute ≠ approve ≠ disburse); govern separation, clearance, and final settlement; treat rehire as new, history-linked employment.

## 4. User Journeys
- **Structure:** HR configures departments/designations and maps staff.
- **Lifecycle:** new hire on probation → confirmation on milestone (HR-002).
- **Pay:** salary structure (versioned) + staff attendance + approved leave → payroll computed → SoD approval → disbursement → reproducible payslips.
- **Exit/return:** governed separation with clearance and final settlement; rehire creates new employment linked to history.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-HR-001 | Configure Department / Designation Structure | High |
| Core | UC-HR-002 | Probation → Confirmation Lifecycle | High |
| Core | UC-HR-003 | Manage Versioned Salary Structure | High |
| Core | UC-HR-004 | Record Staff Attendance (payroll-relevant) | High |
| Core | UC-HR-005 | Compute Payroll (deterministic) | Critical |
| Core | UC-HR-006 | Approve & Disburse Payroll (SoD) | Critical |
| Core | UC-HR-007 | Separation, Clearance & Final Settlement | High |
| Core | UC-HR-008 | Rehire (new employment, history-linked) | Medium |
| CRUD | UC-HR-009 | View Payslip / HR Records (scoped) | Medium |
| Approval | UC-HR-010 | Approve Salary Change | High |
| Search | UC-HR-011 | Search HR Records | Low |
| Reporting | UC-HR-012 | Payroll & HR Reports | High |
| Bulk | UC-HR-013 | Bulk Payroll Run | Critical |
| Import/Export | UC-HR-014 | Import / Export HR Data (governed) | Medium |
| Workflow | UC-HR-015 | Payroll / Separation Workflow | High |
| Exception | UC-HR-016 | Payroll Self-Approval (SoD) Blocked | Critical |
| Exception | UC-HR-017 | Edit Finalized Payslip Blocked | Critical |

---

## 6. Detailed Specifications (high-value use cases)

### UC-HR-005 — Compute Payroll (deterministic)
- **Module:** HR · **Priority:** Critical
- **Actors:** Payroll Officer (primary), System
- **Goal:** Compute payroll deterministically from the versioned salary structure, attendance, and approved leave.
- **Description:** Computes gross→deductions→net per staff using the effective versioned salary structure, payroll-relevant attendance (HR-004), and approved leave (LEV-006); stamps the salary-structure version for reproducible payslips.
- **Business Rules Applied:** HR-005, HR-003, HR-004, LEV-006, CFG-004.
- **Preconditions:** Effective salary structure; attendance/leave finalized for the period; payroll period open.
- **Trigger:** Payroll officer runs computation.
- **Main Success Scenario:**
  1. System resolves the effective, versioned salary structure per staff (HR-003).
  2. System incorporates payroll-relevant attendance (HR-004) and approved leave (LEV-006).
  3. System computes gross, deductions, and net deterministically (HR-005).
  4. System stamps the salary-structure/config version for reproducibility (CFG-004/P6).
- **Alternative Flows:** A1) Recompute after a correction (re-stamped).
- **Exception Flows:** E1) Missing salary structure/attendance → block (no silent defaults). E2) Period closed → governed reopen.
- **Validation Rules:** Effective structure resolved; deterministic computation; version stamped (HR-003/005).
- **Permissions Required:** `payroll.compute`.
- **Notifications Triggered:** None until approved/disbursed.
- **Audit Events Generated:** `PAYROLL_COMPUTED` (period, version).
- **Data Created:** Draft payroll (unapproved).
- **Data Updated:** Payroll store.
- **Data Deleted:** None.
- **Post Conditions:** Deterministic, reproducible payroll ready for SoD approval.
- **Related Use Cases:** UC-HR-006, UC-HR-003, UC-LEV-006.
- **Acceptance Criteria:**
  - Given an effective salary structure and finalized attendance/leave, When computed, Then payroll is deterministic and stamped with the structure version.
  - Given the same inputs recomputed, When re-run, Then identical results are produced.
  - Given missing inputs, When computed, Then it is blocked (no silent defaults).
- **Edge Case Analysis:**
  - *Invalid Input:* missing structure/attendance → blocked.
  - *Permission Failure:* lacks `payroll.compute` → 403.
  - *Concurrent Update:* recompute during correction → serialized; re-stamped.
  - *Duplicate Data:* idempotent computation.
  - *System Failure:* deterministic re-run; recoverable.
  - *Workflow Failure:* N/A (approval separate).
- **QA Coverage:**
  - *Positive:* compute; recompute reproducibility; leave/attendance effects.
  - *Negative:* missing inputs; closed period.
  - *Boundary:* zero-attendance period; mid-period salary change (version boundary).

### UC-HR-006 — Approve & Disburse Payroll (SoD)
- **Module:** HR · **Priority:** Critical
- **Actors:** Payroll Officer, Finance Approver, Disbursing Officer, System
- **Goal:** Approve and disburse payroll with strict separation of duties (compute ≠ approve ≠ disburse).
- **Description:** Computed payroll is approved by a different authority and disbursed by a third role; self-approval is impossible; finalized payslips are immutable (corrections via governed adjustment).
- **Business Rules Applied:** HR-006, HR-005, AUTHZ-009, PAY-001, FILE-007 (Cross-Cutting P9/P5).
- **Preconditions:** Payroll computed; approver and disbursing roles distinct from the computer.
- **Trigger:** Officer submits computed payroll for approval.
- **Main Success Scenario:**
  1. Payroll officer submits the computed run.
  2. A different finance approver reviews and approves (SoD: approver ≠ computer, HR-006).
  3. A disbursing role releases payment (third party where required); payslips generated (immutable, reproducible, FILE-007).
  4. Staff are notified; finalized payslips are locked.
- **Alternative Flows:** A1) Approval returns the run for correction (recompute then re-approve).
- **Exception Flows:** E1) Computer = approver → blocked (UC-HR-016). E2) Post-finalization payslip edit → governed adjustment only (UC-HR-017).
- **Validation Rules:** SoD enforced (compute/approve/disburse distinct); payslips immutable; reproducible (HR-006, P9/P5).
- **Permissions Required:** `payroll.approve`, `payroll.disburse` (distinct from `payroll.compute`).
- **Notifications Triggered:** `PAYSLIP_AVAILABLE` to staff; approval notices.
- **Audit Events Generated:** `PAYROLL_APPROVED`, `PAYROLL_DISBURSED`.
- **Data Created:** Payslips; disbursement records.
- **Data Updated:** Payroll status → finalized.
- **Data Deleted:** None.
- **Post Conditions:** Payroll approved, disbursed, and locked; SoD upheld; payslips reproducible.
- **Related Use Cases:** UC-HR-005, UC-PAY-001, UC-HR-016.
- **Acceptance Criteria:**
  - Given computed payroll, When the computer attempts to approve it, Then SoD blocks self-approval.
  - Given approval by a distinct authority, When approved and disbursed, Then immutable payslips are generated.
  - Given a finalized payslip, When an edit is attempted, Then it is blocked (governed adjustment only).
- **Edge Case Analysis:**
  - *Invalid Input:* approving an uncomputed run blocked.
  - *Permission Failure:* self-approval blocked (AUTHZ-009/HR-006).
  - *Concurrent Update:* double approval → idempotent.
  - *Duplicate Data:* duplicate disbursement prevented (idempotent, PAY-008).
  - *System Failure:* atomic approve+disburse+payslip; no partial.
  - *Workflow Failure:* approver vacancy → escalation; no auto-approve.
- **QA Coverage:**
  - *Positive:* compute→approve→disburse by distinct roles.
  - *Negative:* self-approval; finalized-payslip edit; double disbursement.
  - *Boundary:* approval at workflow timeout (escalation).

### UC-HR-007 — Separation, Clearance & Final Settlement
- **Module:** HR · **Priority:** High
- **Actors:** HR Administrator (primary), Finance, System
- **Goal:** Exit a staff member with governed clearance and an accurate final settlement.
- **Description:** Runs the clearance checklist (handover, assets, dues), computes final settlement (salary, leave encashment, deductions), and disburses under governance; ties into staff archival (STF-008).
- **Business Rules Applied:** HR-007, LEV-002 (encashment), STF-008, TCH-006.
- **Preconditions:** Separation initiated (UC-STF-008); clearance items defined.
- **Trigger:** Separation effective.
- **Main Success Scenario:**
  1. System runs the clearance checklist (handover TCH-006, assets, dues).
  2. System computes final settlement (pending salary, leave encashment LEV-002, deductions).
  3. Settlement is approved (SoD) and disbursed; final payslip/settlement document issued.
  4. Staff record archived (STF-008).
- **Alternative Flows:** A1) Disputed settlement → held pending resolution.
- **Exception Flows:** E1) Pending handover/dues → block settlement completion. E2) Negative settlement (net owed by staff) → recovery per policy.
- **Validation Rules:** Clearance complete; settlement accurate; approved; archive-not-delete (HR-007, STF-008).
- **Permissions Required:** `hr.settlement.manage` (+ approval).
- **Notifications Triggered:** Settlement notice; final payslip to staff.
- **Audit Events Generated:** `FINAL_SETTLEMENT_COMPUTED/DISBURSED`.
- **Data Created:** Settlement record; final payslip.
- **Data Updated:** Status; archived record.
- **Data Deleted:** None.
- **Post Conditions:** Clean, governed exit with accurate settlement; history preserved.
- **Related Use Cases:** UC-STF-008, UC-LEV-002, UC-HR-006.
- **Acceptance Criteria:**
  - Given separation, When settlement runs, Then clearance is required and the settlement (incl. leave encashment) is computed accurately.
  - Given pending handover/dues, When settling, Then completion is blocked until resolved.
  - Given a finalized settlement, When archived, Then statutory records are retained (no hard delete).
- **Edge Case Analysis:**
  - *Invalid Input:* settlement without clearance blocked.
  - *Permission Failure:* unauthorized → 403; self-approval blocked.
  - *Concurrent Update:* settlement + final payroll → consistent.
  - *Duplicate Data:* duplicate settlement prevented.
  - *System Failure:* atomic; no partial settlement.
  - *Workflow Failure:* approval/clearance pends.
- **QA Coverage:**
  - *Positive:* clean settlement with encashment.
  - *Negative:* pending clearance; negative settlement recovery.
  - *Boundary:* mid-month separation pro-rata; zero leave balance.

---

## 7. Compact Specifications (routine use cases)

- **UC-HR-001 — Configure Department / Designation Structure** · *High* · Rules: HR-001, CFG-004. Configure departments/designations (versioned). *Permissions:* `hr.config.manage`. *Audit:* structure change. *Edge:* historical mappings preserved. *QA:* config; history fidelity.
- **UC-HR-002 — Probation → Confirmation Lifecycle** · *High* · Rules: HR-002, STF-004. Manage probation→confirmation on milestone. *Audit:* `STAFF_CONFIRMED`. *Edge:* probation gates some capabilities; confirmation governed. *QA:* confirm; gated capabilities.
- **UC-HR-003 — Manage Versioned Salary Structure** · *High* · Rules: HR-003, CFG-004. Define salary structures, versioned, effective-dated. *Permissions:* `payroll.config.manage`. *Edge:* future-dated change applies forward; payslips keep their version. *QA:* versioned; effective-dating; reproducibility.
- **UC-HR-004 — Record Staff Attendance (payroll-relevant)** · *High* · Rules: HR-004, LEV-006. Track staff attendance distinct from student attendance; feeds payroll. *Edge:* approved leave reflected; LOP for unapproved absence. *QA:* attendance→payroll; leave reflection.
- **UC-HR-008 — Rehire (new employment, history-linked)** · *Medium* · Rules: HR-008, STF-003. Rehire as new employment linked to prior history. *Audit:* `STAFF_REHIRED`. *Edge:* new employment record; history visible. *QA:* rehire-link; no duplicate identity.
- **UC-HR-009 — View Payslip / HR Records (scoped)** · *Medium* · Rules: AUTHZ-003, STF-007, FILE-005. Staff see own payslips (signed URL); HR scoped. *Edge:* others' salary not visible. *QA:* scope; own-payslip.
- **UC-HR-010 — Approve Salary Change** · *High* · Rules: HR-003, AUTHZ-009, WFL-004. Approver gates salary changes (SoD). *Edge:* self-approval blocked; versioned. *QA:* approval; SoD.
- **UC-HR-011 — Search HR Records** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-HR-012 — Payroll & HR Reports** · *High* · Rules: REP-002/003. Payroll registers, cost reports (masked, scoped). *Edge:* salary data need-to-know. *QA:* totals; masking.
- **UC-HR-013 — Bulk Payroll Run** · *Critical* · Rules: HR-005/006, async. Run payroll for all staff (deterministic, async, SoD-gated). *Edge:* resumable; idempotent; partial recovery. *QA:* large run; reproducibility; SoD on the batch.
- **UC-HR-014 — Import / Export HR Data (governed)** · *Medium* · Rules: REP-005, STF-007. Governed, masked import/export. *Permissions:* `report.export` (elevated). *Edge:* salary/PII controlled. *QA:* scoped; gated; masked.
- **UC-HR-015 — Payroll / Separation Workflow** · *High* · Rules: WFL-002/004, HR-006/007. Version-pinned payroll/separation approval. *QA:* pinning; SoD; escalation.
- **UC-HR-016 — Payroll Self-Approval (SoD) Blocked (Exception)** · *Critical* · Rules: HR-006, AUTHZ-009. Computer cannot approve own payroll. *QA:* self-approval blocked; distinct approver allowed.
- **UC-HR-017 — Edit Finalized Payslip Blocked (Exception)** · *Critical* · Rules: HR-003 (P5). Finalized payslips immutable; corrections via governed adjustment. *QA:* edit blocked; adjustment path.

## 8. Module-level QA & Edge Themes
- **Payroll SoD (HR-006 / P9):** compute ≠ approve ≠ disburse, self-approval impossible — the headline financial-control suite.
- **Payroll determinism & reproducibility (HR-005 / P6):** identical inputs → identical payroll; payslips stamped to the salary-structure version.
- **Payslip immutability (HR-003 / P5):** finalized payslips never edited; governed adjustment only.
- **Governed settlement (HR-007):** clearance + handover gate final settlement; statutory retention on archival.
- **No silent defaults:** missing salary/attendance blocks computation rather than guessing.
