# 23 — HR Business Rules

> **Prefix note:** HR-phase prefixes are distinct — Staff `STF`, Teacher `TCH`, HR/Payroll `HR`, Leave `LEV`.

## 1. Module Purpose
Govern **HR processes and policies** that operate on staff records (Doc 22): recruitment/onboarding, the department/designation structure, the employment lifecycle (probation to confirmation to separation), **staff attendance** (a distinct concept from student attendance), **payroll configuration and basic payroll** (full payroll runs are deferred per the delivery roadmap — this module owns the configuration, salary structure, and the controls that make payroll auditable when it lands), light performance management, and **offboarding with final settlement and clearance**. Sensitive compensation data and separation-of-duties on payroll are central concerns.

## 2. Actors
- **HR Administrator** — runs onboarding, lifecycle, staff attendance policy, performance, offboarding.
- **Payroll Preparer** — configures/prepares payroll (cannot also approve it — SoD).
- **Payroll Approver / Finance Authority** — approves payroll (distinct from preparer).
- **Reporting Manager** — confirms probation, conducts appraisals, certifies clearance.
- **Staff Member** — self-service profile, payslips, attendance, appraisal acknowledgement.
- **System** — enforces lifecycle, staff-attendance rules, payroll SoD, settlement, and audit.

## 3. Use Cases

**Use Case ID:** UC-HR-01 — Onboard & Confirm a Staff Member
**Actors:** HR Administrator, Reporting Manager, System
**Description:** Run onboarding and move a staff member through probation to confirmation.
**Preconditions:** Staff record created (Doc 22); onboarding checklist configured.
**Main Flow:** 1) HR runs the onboarding checklist (documents, role/membership setup, payroll setup). 2) Staff enters `PROBATION`. 3) At probation end, the manager confirms or extends. 4) Confirmed staff are `CONFIRMED`.
**Alternative Flow:** A1) Probation extended once per policy with reason.
**Exception Flow:** E1) Incomplete onboarding → block confirmation. E2) Probation failure → governed exit (HR-007).
**Post Conditions:** Staff onboarded/confirmed; audited.
**Business Rules Applied:** HR-001, HR-002.

**Use Case ID:** UC-HR-02 — Configure & Prepare Payroll (SoD)
**Actors:** Payroll Preparer, Approver, System
**Description:** Define salary structure and prepare a payroll for approval, with duties separated.
**Preconditions:** Salary structure configured; staff attendance/leave finalized for the period.
**Main Flow:** 1) Preparer configures salary structure (basic, allowances, deductions) and prepares the period's payroll, integrating attendance/leave (HR-004/LEV). 2) System computes gross/deductions/net. 3) A different authority approves (SoD). 4) Approved payroll is locked; payslips issued.
**Exception Flow:** E1) Preparer = approver → blocked (HR-006). E2) Attendance/leave not finalized → block preparation.
**Post Conditions:** Payroll approved/locked; payslips issued; audited.
**Business Rules Applied:** HR-003, HR-004, HR-005, HR-006.

**Use Case ID:** UC-HR-03 — Offboard with Clearance & Final Settlement
**Actors:** HR Administrator, Reporting Manager, Finance, System
**Description:** Separate a staff member with clearance and final settlement.
**Preconditions:** Separation initiated (Doc 22); clearance items configured.
**Main Flow:** 1) System runs the clearance checklist (asset return, handover TCH-006, dues, library). 2) Final settlement computes pending salary, leave encashment (LEV), deductions. 3) Settlement is approved (SoD) and paid; access fully revoked (AUTH-005).
**Exception Flow:** E1) Outstanding dues/un-returned assets → withhold/adjust per policy. E2) Termination for cause → governed process with documentation.
**Post Conditions:** Cleared, settled, access revoked, archived; audited.
**Business Rules Applied:** HR-007, HR-008.

## 4. Business Rules

**Rule ID:** HR-001
**Rule Name:** Configurable Department/Designation Structure
**Description:** Departments and designations are configurable per institute and underpin assignment, hierarchy, and payroll.
**Priority:** Medium
**Category:** Configuration
**Preconditions:** HR structure setup.
**Business Rule:** Departments and designations (with grade/level) are configurable; staff assignments reference them; the structure informs reporting lines (STF-006) and salary bands (HR-005).
**System Action:** Maintain the structure; validate assignments against it.
**Validation:** Department/designation valid in scope.
**Failure Behavior:** Reject assignments to undefined departments/designations.
**Audit Requirement:** Log structure changes.
**Example Scenario:** A "Senior Teacher" designation in the "Science Department" maps to a salary band.
**Related Rules:** STF-005, STF-006, HR-005.

**Rule ID:** HR-002
**Rule Name:** Probation-to-Confirmation Lifecycle
**Description:** New staff serve a configurable probation; confirmation (or governed extension/exit) is an explicit, recorded decision.
**Priority:** High
**Category:** Lifecycle
**Preconditions:** Onboarding complete.
**Business Rule:** Probation duration is configurable; confirmation requires a manager decision; extension is limited and reasoned; probation failure follows a governed exit. Some entitlements (e.g., certain leave accruals) may differ during probation (LEV).
**System Action:** Track probation; prompt confirmation; apply probation-specific policies.
**Validation:** Probation period defined; decision recorded.
**Failure Behavior:** Block silent auto-confirmation without a decision where policy requires it.
**Audit Requirement:** Log `PROBATION_CONFIRMED/EXTENDED/FAILED`.
**Example Scenario:** A teacher is confirmed after a six-month probation on the manager's recommendation.
**Related Rules:** STF-004, HR-007, LEV.

**Rule ID:** HR-003
**Rule Name:** Versioned Salary Structure
**Description:** Salary structures (components, bands) are configurable, effective-dated, and versioned; changes never silently rewrite past payslips.
**Priority:** High
**Category:** Payroll integrity
**Preconditions:** Salary configuration.
**Business Rule:** A salary structure defines components (basic, allowances, deductions) with rules; it is effective-dated and versioned; issued payslips reference the structure version in effect — historical payslips are immutable.
**System Action:** Version and effective-date structures; stamp the version on payslips.
**Validation:** Components valid; no overlapping effective periods per scope.
**Failure Behavior:** Reject overlaps; require new versions for changes.
**Audit Requirement:** Log `SALARY_STRUCTURE_PUBLISHED`.
**Example Scenario:** A mid-year allowance change applies forward; prior payslips stand.
**Related Rules:** HR-005, FEE-001 (parallel pattern), STF-007.

**Rule ID:** HR-004
**Rule Name:** Staff Attendance Is Distinct and Payroll-Relevant
**Description:** Staff attendance is a separate concept from student attendance and integrates with leave and payroll.
**Priority:** High
**Category:** Integrity / integration
**Preconditions:** Staff attendance tracked.
**Business Rule:** Staff attendance (check-in/out, working days, late/absence) is modeled separately from student attendance (Doc 10); approved leave (LEV) reflects correctly (not absence); unpaid leave/absence feeds payroll deductions. Biometric/device capture is a future channel.
**System Action:** Track staff attendance; integrate leave; feed payroll.
**Validation:** Attendance source valid; leave cross-referenced.
**Failure Behavior:** Ambiguous attendance flagged for HR review, not silently penalized.
**Audit Requirement:** Log staff-attendance corrections (governed, like ATT-007).
**Example Scenario:** A staff member on approved leave is not docked pay; unauthorized absence may be.
**Related Rules:** LEV, HR-005, ATT (distinct).

**Rule ID:** HR-005
**Rule Name:** Payroll Computation Integrity
**Description:** Payroll computes deterministically from the salary structure, attendance/leave, and statutory rules, with provenance.
**Priority:** High
**Category:** Payroll integrity
**Preconditions:** Period attendance/leave finalized; structure resolved.
**Business Rule:** Net pay = earnings (per structure) − deductions (statutory + unpaid-leave + others), computed deterministically; the payslip records the structure version and inputs (provenance). Full payroll-run scale-out is deferred per roadmap, but the computation/audit model is fixed now.
**System Action:** Compute deterministically; stamp provenance; generate payslips.
**Validation:** Inputs finalized; structure resolved; statutory rules current.
**Failure Behavior:** Block computation on unfinalized inputs.
**Audit Requirement:** Payslip records provenance; `PAYROLL_COMPUTED` logged.
**Example Scenario:** A payslip reproducibly reflects basic + allowances − tax − one unpaid-leave day.
**Related Rules:** HR-003, HR-004, HR-006.

**Rule ID:** HR-006
**Rule Name:** Payroll Separation of Duties
**Description:** The preparer of a payroll cannot also approve it; approval is a distinct authority.
**Priority:** Critical
**Category:** Financial control / fraud prevention
**Preconditions:** Payroll approval.
**Business Rule:** Payroll preparation and approval are separated (SoD, AUTHZ-009); approval is required before payslips issue/pay; self-approval is impossible; approved payroll is locked (immutable, corrections via governed adjustment like FEE-003).
**System Action:** Enforce preparer ≠ approver; lock on approval.
**Validation:** Approver distinct and authorized; payroll complete.
**Failure Behavior:** Block self-approval; block payout without approval.
**Audit Requirement:** Log `PAYROLL_PREPARED/APPROVED/LOCKED` with distinct actors.
**Example Scenario:** The HR officer preparing payroll cannot approve it; the finance head does.
**Related Rules:** AUTHZ-009, HR-005, FEE-003.

**Rule ID:** HR-007
**Rule Name:** Governed Separation, Clearance & Final Settlement
**Description:** Offboarding runs clearance and final settlement under governance; termination-for-cause is documented.
**Priority:** High
**Category:** Lifecycle / integrity
**Preconditions:** Separation.
**Business Rule:** Separation requires a clearance checklist (asset return, handover TCH-006, dues, system access), a final settlement (pending pay, leave encashment LEV, deductions) approved under SoD, and full access revocation (AUTH-005). Termination for cause follows a documented, governed process.
**System Action:** Run clearance; compute/approve settlement; revoke access; archive.
**Validation:** Clearance complete; settlement approved; handover done.
**Failure Behavior:** Block finalization with incomplete clearance/orphaned duties.
**Audit Requirement:** Log clearance, `FINAL_SETTLEMENT_APPROVED`, access revocation.
**Example Scenario:** A resigning teacher returns assets, completes handover, and receives an approved final settlement before access ends.
**Related Rules:** STF-008, TCH-006, LEV, AUTH-005.

**Rule ID:** HR-008
**Rule Name:** Rehire Is New Employment, History-Linked
**Description:** Rehiring a former staff member creates a new employment record linked to the existing person, not a silent reactivation.
**Priority:** Medium
**Category:** Integrity
**Preconditions:** A former (separated) staff member returns.
**Business Rule:** Duplicate detection (STF-003) links to the existing person; a new employment period begins (new join date, fresh probation/entitlements per policy) while prior employment history is preserved and linked.
**System Action:** Link to the person; open a new employment period; preserve prior history.
**Validation:** Identity matched; new period created.
**Failure Behavior:** Block silent reactivation of an old employment record.
**Audit Requirement:** Log `STAFF_REHIRED` linking periods.
**Example Scenario:** A teacher rejoining after two years starts fresh probation, with their prior tenure visible in history.
**Related Rules:** STF-003, STF-008, HR-002.

## 5. Validation Rules
- Departments/designations defined before assignment.
- Probation tracked; confirmation/extension/exit decisions recorded.
- Salary structures versioned/effective-dated; payslips immutable.
- Staff attendance integrates leave; ambiguity flagged not auto-penalized.
- Payroll deterministic with provenance; preparer ≠ approver; approved payroll locked.
- Separation gated on clearance/handover/settlement; rehire is new employment.

## 6. State Machine
*(Employment lifecycle, extending Staff Doc 22.)*

**State Name:** ONBOARDING
**Description:** Onboarding checklist in progress.
**Allowed Transitions:** → PROBATION.
**Forbidden Transitions:** payroll before setup complete.
**System Actions:** Run checklist; set up payroll/access.

**State Name:** PROBATION
**Description:** Probationary employment.
**Allowed Transitions:** → CONFIRMED; → PROBATION (extended once); → SEPARATED (probation failure).
**Forbidden Transitions:** silent auto-confirm where a decision is required.
**System Actions:** Apply probation policies; prompt confirmation.

**State Name:** CONFIRMED
**Description:** Confirmed employment.
**Allowed Transitions:** → SUSPENDED; → NOTICE_PERIOD; → SEPARATED.
**Forbidden Transitions:** → PROBATION.
**System Actions:** Full entitlements; normal payroll.

**State Name:** NOTICE_PERIOD
**Description:** Separation notice; clearance/handover underway.
**Allowed Transitions:** → SEPARATED.
**Forbidden Transitions:** → CONFIRMED (withdrawal is a governed exception).
**System Actions:** Clearance checklist; settlement prep.

**State Name:** SEPARATED
**Description:** Employment ended; settled; access revoked.
**Allowed Transitions:** → ARCHIVED; rehire opens a NEW period (HR-008).
**Forbidden Transitions:** silent reactivation.
**System Actions:** Finalize settlement; archive.

**State Name:** ARCHIVED
**Description:** Historical employment record.
**Allowed Transitions:** none (terminal).
**Forbidden Transitions:** edits/hard-delete.
**System Actions:** Retain per labor/tax retention.

## 7. Status Definitions
`ONBOARDING` · `PROBATION` · `CONFIRMED` · `SUSPENDED` · `NOTICE_PERIOD` · `SEPARATED` · `ARCHIVED`. Payroll: `DRAFT` · `PREPARED` · `APPROVED` · `LOCKED` · `PAID`.

## 8. Workflow Rules
- Onboarding/confirmation, probation extension, and termination-for-cause are governed, recorded decisions.
- Payroll runs a prepare → approve workflow with mandatory SoD (HR-006).
- Offboarding runs clearance → settlement → access-revocation in sequence (HR-007).
- Staff-attendance corrections and leave approvals follow governed flows (HR-004/LEV).

## 9. Permission Rules
- `hr.structure.manage` — departments/designations/salary structures.
- `hr.lifecycle.manage` — onboarding/probation/confirmation/separation.
- `hr.payroll.prepare` — prepare payroll (cannot approve).
- `hr.payroll.approve` — approve payroll (cannot be the preparer; SoD).
- `hr.staff_attendance.manage` — staff attendance policy/corrections.
- `hr.view` — read HR data (scoped; staff read own payslips/attendance).

## 10. Notification Rules
- Onboarding tasks/confirmation → notify staff and manager.
- `PAYROLL_APPROVED`/payslip issued → notify staff (payslip available).
- Probation confirmation/extension, appraisal outcomes → notify staff.
- Offboarding clearance tasks and `FINAL_SETTLEMENT_APPROVED` → notify staff and managers.

## 11. Audit Requirements
Mandatory: HR-structure changes, `PROBATION_CONFIRMED/EXTENDED/FAILED`, `SALARY_STRUCTURE_PUBLISHED`, staff-attendance corrections, `PAYROLL_PREPARED/APPROVED/LOCKED/PAID` (distinct actors), `FINAL_SETTLEMENT_APPROVED`, `STAFF_REHIRED`. Payroll and settlement are financial events — fully attributable with SoD evidence.

## 12. Data Retention Rules
- Employment, payroll, and settlement records retained per labor/tax/statutory retention (often multi-year).
- Payslips and salary-structure versions retained for reproducibility.
- Sensitive compensation data minimized, field-encrypted (STF-007), erasable post-retention.
- Legal-hold overrides erasure during disputes.

## 13. Edge Cases
- **Probation extension limit:** bounded per policy; not indefinite (HR-002).
- **Partial-month payroll (mid-period join/leave):** pro-rated deterministically with provenance (HR-005).
- **Unpaid leave vs approved paid leave:** only unpaid affects pay; approved paid leave does not (HR-004/LEV).
- **Termination for cause:** governed, documented; settlement may differ per policy/law (HR-007).
- **Rehire:** new employment period, history-linked; fresh entitlements (HR-008).
- **Payroll preparer also a manager who approves leave:** payroll approval SoD still applies separately (HR-006).
- **Salary change mid-cycle:** effective-dated; current cycle uses the in-effect version (HR-003).
- **Staff attendance device outage:** ambiguity flagged for HR, not auto-deducted (HR-004).

## 14. Failure Scenarios
- **Self-approved payroll:** blocked (HR-006).
- **Payout without approval:** blocked (HR-006).
- **Edit to a locked payslip:** rejected; governed adjustment only (HR-003/HR-006).
- **Separation with incomplete clearance/handover:** blocked (HR-007/TCH-006).
- **Silent rehire reactivation:** blocked (HR-008).
- **Payroll on unfinalized attendance/leave:** blocked (HR-005).

## 15. Exception Handling Rules
- Payroll requires finalized inputs, deterministic computation, provenance, and SoD; otherwise it blocks.
- Lifecycle decisions are explicit and recorded; no silent auto-progression where a decision is required.
- Offboarding is sequential and gated; nothing is orphaned.
- Sensitive compensation data fails closed.

## 16. Compliance Considerations
- **Labor & tax law:** lifecycle, payroll, settlement, and retention honor statutory requirements.
- **Financial control:** payroll SoD, locking, and provenance prevent payroll fraud.
- **Data protection:** compensation data protected, minimized, erasable post-retention.
- **Fair process:** governed probation/termination/appraisal support fair-employment obligations.

## 17. Future Considerations
- Full payroll-run scale-out with statutory tax engines and bank disbursement files (deferred per roadmap).
- Biometric/device staff attendance integration.
- Structured performance-appraisal cycles with goals/ratings.
- Self-service HR portal (leave, payslips, documents, declarations).
