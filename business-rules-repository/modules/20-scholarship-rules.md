# 20 — Scholarship Business Rules

## 1. Module Purpose
Govern **scholarships** — structured, often **funded**, criteria-based awards that support a student's education, distinct from discounts (rule/discretionary fee reductions, Doc 19) and waivers (targeted forgiveness). A scholarship has an **award lifecycle** (application → selection → award → disbursement → renewal/revocation), may have a **funding source/sponsor** with accountability obligations, may cover fees and/or pay a **stipend** beyond fees, and is typically **performance-linked** (renewal conditions like GPA/attendance). This module emphasizes fair selection, sponsor accountability, disbursement integrity, and renewal/revocation governance.

## 2. Actors
- **Scholarship Committee / Coordinator** — defines programs, selects awardees, manages renewal/revocation.
- **Finance Administrator** — handles disbursement and fund accounting.
- **Sponsor / Donor (external)** — funds awards; receives accountability reporting (no direct system access by default).
- **Student / Guardian** — apply, view award status (own).
- **System** — manages the award lifecycle, eligibility, disbursement, renewal checks, fund tracking, and audit.

## 3. Use Cases

**Use Case ID:** UC-SCH-01 — Define a Scholarship Program & Select Awardees
**Actors:** Scholarship Committee, System
**Description:** Create a program with criteria/funding and select recipients fairly.
**Preconditions:** Actor holds `scholarship.program.manage`; funding source recorded.
**Main Flow:** 1) Define the program (type, criteria, coverage, stipend, funding source, slots, renewal conditions). 2) Eligible students apply or are nominated. 3) Committee reviews and selects via a governed (often multi-member) workflow. 4) Awards are granted.
**Alternative Flow:** A1) Auto-eligibility shortlist by merit/need feeding committee review.
**Exception Flow:** E1) Slots exceeded → waitlist. E2) Selection bias/SoD concern → committee/governance controls (SCH-003).
**Post Conditions:** Awards granted with provenance; audited; sponsor-reportable.
**Business Rules Applied:** SCH-001, SCH-002, SCH-003.

**Use Case ID:** UC-SCH-02 — Disburse a Scholarship
**Actors:** Finance Administrator, System
**Description:** Apply the award to fees and/or pay a stipend, drawing on the fund.
**Preconditions:** Award active; funding available.
**Main Flow:** 1) System applies fee coverage as a transparent reduction (like a funded discount) and/or schedules a stipend payout. 2) Fund balance is decremented and tracked against the sponsor. 3) Disbursement recorded with provenance.
**Exception Flow:** E1) Insufficient fund balance → block/flag (SCH-005). E2) Coverage exceeds fees → excess is a stipend, not a negative fee (SCH-004).
**Post Conditions:** Disbursement applied; fund tracked; audited.
**Business Rules Applied:** SCH-004, SCH-005, SCH-006.

**Use Case ID:** UC-SCH-03 — Renew or Revoke
**Actors:** Scholarship Committee, System
**Description:** Continue or end an award based on performance conditions.
**Preconditions:** Renewal/revocation conditions defined; period boundary or trigger reached.
**Main Flow:** 1) System evaluates renewal conditions (GPA/attendance/conduct) at the defined checkpoint. 2) Meets conditions → renewed; fails → review for revocation/probation. 3) Decision recorded; student/guardian and sponsor informed.
**Exception Flow:** E1) Borderline case → probation per policy. E2) Sponsor withdraws funding → governed wind-down (SCH-007).
**Post Conditions:** Award renewed/revoked/probationary; audited.
**Business Rules Applied:** SCH-007, SCH-008.

## 4. Business Rules

**Rule ID:** SCH-001
**Rule Name:** Scholarship Is a Structured, Funded Award (Distinct from Discount/Waiver)
**Description:** A scholarship is a criteria-based award with a lifecycle and (usually) a funding source, modeled distinctly from discounts/waivers.
**Priority:** Critical
**Category:** Domain clarity / integrity
**Preconditions:** Program/award definition.
**Business Rule:** A scholarship has type (merit/need/sports/sponsored/full/partial), coverage (which fee heads and/or stipend), a funding source, slots, and renewal conditions. It is tracked separately from discounts/waivers in records and reporting, even though fee-coverage mechanics resemble a funded discount.
**System Action:** Model scholarships with their own lifecycle/entities; classify distinctly.
**Validation:** Program fields complete; funding source recorded.
**Failure Behavior:** Reject incomplete programs.
**Audit Requirement:** Log `SCHOLARSHIP_PROGRAM_DEFINED`.
**Example Scenario:** A donor-funded merit scholarship covers full tuition plus a monthly stipend, tracked against the donor's fund.
**Related Rules:** DSC-001 (distinction), SCH-004.

**Rule ID:** SCH-002
**Rule Name:** Defined Eligibility & Fair, Slot-Bounded Selection
**Description:** Selection uses defined criteria, respects slot limits, and is governed to ensure fairness.
**Priority:** High
**Category:** Fairness
**Preconditions:** Applications/nominations exist.
**Business Rule:** Eligibility criteria (merit thresholds, need indicators) are defined and transparent; selection respects available slots; over-subscription waitlists fairly; decisions are recorded with the basis. Selection is committee/workflow-governed for high-value awards.
**System Action:** Evaluate eligibility; enforce slots; record selection basis.
**Validation:** Criteria/slots defined; basis captured.
**Failure Behavior:** Block over-slot awards; waitlist; require recorded basis.
**Audit Requirement:** Log `SCHOLARSHIP_AWARDED` with basis and committee decision.
**Example Scenario:** 10 need-based slots are awarded by defined criteria; the 11th qualified applicant is waitlisted.
**Related Rules:** SCH-003, ADM-005 (waitlist parallel).

**Rule ID:** SCH-003
**Rule Name:** Governed Selection & Conflict-of-Interest Controls
**Description:** Award selection is governed (committee/approval) with conflict-of-interest and SoD safeguards.
**Priority:** High
**Category:** Governance / integrity
**Preconditions:** Selection of awardees.
**Business Rule:** High-value or discretionary selection requires committee/multi-approver decisions; a selector with a conflict of interest (e.g., a relative applicant) is excluded from that decision; SoD separates selection from disbursement.
**System Action:** Enforce committee workflow; flag/exclude conflicts; separate select vs disburse.
**Validation:** Committee constituted; conflicts declared; SoD enforced.
**Failure Behavior:** Block conflicted or single-actor high-value awards.
**Audit Requirement:** Log committee decisions and conflict exclusions.
**Example Scenario:** A committee member related to an applicant recuses from that decision, recorded in audit.
**Related Rules:** AUTHZ-009, SCH-002.

**Rule ID:** SCH-004
**Rule Name:** Coverage, Stipend & No-Negative-Fee Rule
**Description:** Scholarship coverage reduces fees transparently; any benefit beyond fees is an explicit stipend, never a negative fee.
**Priority:** High
**Category:** Integrity
**Preconditions:** Disbursement.
**Business Rule:** Fee coverage applies as a transparent reduction (funded) up to the covered heads; benefit exceeding fees (or pure stipends) is paid as an explicit stipend disbursement, not as a negative charge. Coverage scope (which heads) is defined.
**System Action:** Apply fee coverage transparently; pay excess/stipend as a payout.
**Validation:** Coverage scope defined; stipend handled as payout.
**Failure Behavior:** Block negative-fee outcomes; route excess to stipend.
**Audit Requirement:** Log fee-coverage and stipend disbursements distinctly.
**Example Scenario:** A full scholarship covers all fees and additionally pays a living stipend, recorded as two distinct flows.
**Related Rules:** DSC-004 (cap parallel), PAY (stipend payout), SCH-005.

**Rule ID:** SCH-005
**Rule Name:** Fund Tracking & Sponsor Accountability
**Description:** Awards draw on a tracked fund; disbursements are accountable to the funding source.
**Priority:** High
**Category:** Fund integrity
**Preconditions:** A funded scholarship disburses.
**Business Rule:** Each program's fund balance is tracked; disbursements decrement it; over-disbursement beyond available funds is blocked/flagged; sponsor-accountability reporting summarizes awards and spend without exposing minors' data beyond consented scope.
**System Action:** Maintain fund ledger; block over-spend; generate sponsor reports.
**Validation:** Fund balance ≥ disbursement; reporting scope consented.
**Failure Behavior:** Block over-fund disbursement; flag shortfalls.
**Audit Requirement:** Log fund movements and sponsor reports.
**Example Scenario:** A donor receives a report of how their fund was awarded, with student data shared only per consent.
**Related Rules:** SCH-004, Compliance (data sharing).

**Rule ID:** SCH-006
**Rule Name:** Scholarship + Discount/Waiver Interaction
**Description:** How scholarships combine with discounts/waivers is explicit and capped.
**Priority:** Medium
**Category:** Fairness / control
**Preconditions:** A scholarship student also qualifies for discounts/waivers.
**Business Rule:** The interaction policy is explicit: typically a scholarship covers net fees after structural category pricing, and additional discounts may or may not stack per policy; the total benefit cannot exceed total fees plus any defined stipend, and a combined cap applies.
**System Action:** Apply per the defined interaction order; enforce combined caps.
**Validation:** Interaction policy + combined cap defined.
**Failure Behavior:** Block combinations exceeding policy.
**Audit Requirement:** Combined-benefit computation recorded.
**Example Scenario:** A scholarship student's additional sibling discount is disallowed (or capped) per the configured policy.
**Related Rules:** DSC-005, FEE-006.

**Rule ID:** SCH-007
**Rule Name:** Renewal & Revocation Are Condition-Based and Governed
**Description:** Continuation depends on defined performance conditions; revocation/wind-down is governed and fair.
**Priority:** High
**Category:** Lifecycle / fairness
**Preconditions:** Renewal checkpoint or revocation trigger.
**Business Rule:** Renewal conditions (e.g., maintain GPA/attendance/conduct) are defined and evaluated at checkpoints; failing may lead to probation or revocation per policy with notice and (often) an appeal. Sponsor funding withdrawal triggers a governed wind-down protecting the student where possible.
**System Action:** Evaluate conditions; apply renew/probation/revoke; handle sponsor withdrawal.
**Validation:** Conditions defined; results available; due process followed.
**Failure Behavior:** No silent revocation; notice/appeal per policy.
**Audit Requirement:** Log `SCHOLARSHIP_RENEWED/PROBATION/REVOKED` with basis.
**Example Scenario:** A merit scholarship lapses to probation when GPA dips below the threshold, with a recovery period.
**Related Rules:** RES (GPA), ATT (attendance), SCH-008.

**Rule ID:** SCH-008
**Rule Name:** Award Changes Never Rewrite History
**Description:** Renewal/revocation/adjustment affect future disbursements; past disbursements remain valid records.
**Priority:** Medium
**Category:** Integrity
**Preconditions:** An award status change.
**Business Rule:** Revoking or changing an award stops/adjusts future disbursement but does not retroactively reverse legitimately disbursed past benefit (unless fraud is established via a governed process). History is preserved.
**System Action:** Apply changes forward; preserve past disbursement records.
**Validation:** Change is forward-looking unless fraud governed.
**Failure Behavior:** Block silent retroactive clawback.
**Audit Requirement:** Log forward changes; any clawback governed.
**Example Scenario:** A revoked scholarship stops next term's coverage but the current term already disbursed stands.
**Related Rules:** FEE-003, SCH-007.

## 5. Validation Rules
- Program complete (type/criteria/coverage/funding/slots/renewal) before activation.
- Selection within slots, criteria-based, conflict-free, governed for high value.
- Coverage transparent; excess is stipend; never negative fee.
- Fund balance ≥ disbursement; sponsor reporting consented.
- Scholarship/discount interaction explicit and capped.
- Renewal/revocation condition-based, governed, forward-looking.

## 6. State Machine

**State Name:** APPLIED / NOMINATED
**Description:** Candidate in consideration.
**Allowed Transitions:** → UNDER_REVIEW; → WITHDRAWN.
**Forbidden Transitions:** disbursement before award.
**System Actions:** Collect application; eligibility check.

**State Name:** UNDER_REVIEW
**Description:** Committee evaluating.
**Allowed Transitions:** → AWARDED; → WAITLISTED; → REJECTED.
**Forbidden Transitions:** award beyond slots.
**System Actions:** Governed selection; conflict checks.

**State Name:** AWARDED / ACTIVE
**Description:** Granted and disbursing per schedule.
**Allowed Transitions:** → RENEWED; → PROBATION; → REVOKED; → COMPLETED.
**Forbidden Transitions:** negative-fee disbursement; over-fund spend.
**System Actions:** Disburse (coverage/stipend); track fund; evaluate conditions.

**State Name:** PROBATION
**Description:** Conditions partially unmet; recovery period.
**Allowed Transitions:** → ACTIVE (recovered); → REVOKED (failed).
**Forbidden Transitions:** silent continuation without review.
**System Actions:** Hold/limit disbursement per policy; re-evaluate.

**State Name:** REVOKED
**Description:** Ended for unmet conditions/sponsor withdrawal.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** retroactive clawback without governance.
**System Actions:** Stop future disbursement; preserve past records.

**State Name:** COMPLETED / ARCHIVED
**Description:** Award fulfilled or historical.
**Allowed Transitions:** COMPLETED → ARCHIVED.
**Forbidden Transitions:** edits.
**System Actions:** Retain records; final sponsor reporting.

## 7. Status Definitions
`APPLIED`/`NOMINATED` · `UNDER_REVIEW` · `WAITLISTED` · `AWARDED`/`ACTIVE` · `PROBATION` · `RENEWED` · `REVOKED` · `COMPLETED` · `ARCHIVED` · `REJECTED`/`WITHDRAWN`. Types: `MERIT` · `NEED` · `SPORTS` · `SPONSORED` · `FULL` · `PARTIAL`.

## 8. Workflow Rules
- High-value/discretionary selection runs a committee/multi-approver workflow with conflict-of-interest controls (SCH-003).
- Disbursement is separated from selection (SoD).
- Renewal evaluation is scheduled at checkpoints; revocation/probation follow due process with notice/appeal.
- Sponsor-funding withdrawal triggers a governed wind-down.

## 9. Permission Rules
- `scholarship.program.manage` — define programs/funding/criteria.
- `scholarship.select` — committee selection (governed; conflict-controlled).
- `scholarship.disburse` — disburse awards (finance; SoD from selection).
- `scholarship.renew_revoke` — manage renewal/revocation.
- `scholarship.view` — read awards (students/guardians own; staff scoped; sponsor via reports).

## 10. Notification Rules
- `SCHOLARSHIP_AWARDED/REJECTED/WAITLISTED` → notify applicant/guardian.
- `SCHOLARSHIP_RENEWED/PROBATION/REVOKED` → notify student/guardian with basis and any appeal path.
- Disbursement confirmations → guardian; fund/sponsor reports → sponsor (consented scope).
- Never expose other applicants' data to a family.

## 11. Audit Requirements
Mandatory: `SCHOLARSHIP_PROGRAM_DEFINED`, `SCHOLARSHIP_AWARDED` (basis, committee), conflict exclusions, disbursements (coverage/stipend), fund movements, `SCHOLARSHIP_RENEWED/PROBATION/REVOKED` (basis), sponsor reports. Selection fairness and fund accountability must be fully evidenced.

## 12. Data Retention Rules
- Scholarship awards, selections, disbursements, and fund ledgers retained per financial/academic retention and sponsor-agreement terms.
- Selection-basis records retained to evidence fairness.
- Sponsor data-sharing governed by consent and agreement; minors' data shared only as consented.
- Anonymization post-retention preserves non-identifying fund integrity.

## 13. Edge Cases
- **Coverage exceeds fees:** excess is an explicit stipend, never a negative fee (SCH-004).
- **Scholarship + discount overlap:** governed by interaction policy and combined cap (SCH-006).
- **Renewal failure mid-year:** probation/revocation forward-looking; current disbursed term stands (SCH-007/SCH-008).
- **Sponsor withdraws funding:** governed wind-down protecting the student where possible (SCH-007).
- **Awardee withdraws/transfers:** future disbursement stops; pro-rata/clawback only via governance.
- **Conflict of interest in selection:** selector recuses; recorded (SCH-003).
- **Fund shortfall:** disbursement blocked/flagged before over-spend (SCH-005).
- **External-sponsor reporting vs minors' privacy:** data shared only within consented scope.

## 14. Failure Scenarios
- **Over-slot/conflicted award:** blocked (SCH-002/SCH-003).
- **Over-fund disbursement:** blocked (SCH-005).
- **Negative-fee outcome:** prevented; excess routed to stipend (SCH-004).
- **Silent revocation/clawback:** prevented; governed with notice (SCH-007/SCH-008).
- **Combined benefit over cap:** blocked (SCH-006).

## 15. Exception Handling Rules
- Selection is governed, conflict-controlled, slot-bounded, and basis-recorded.
- Disbursement is fund-checked, transparent, and SoD-separated from selection.
- Renewal/revocation follow due process; changes are forward-looking; clawback is governed.
- Sponsor reporting respects minors' privacy and consent.

## 16. Compliance Considerations
- **Fair selection & due process:** governed, conflict-controlled selection and renewal/appeal protect against bias and unfair revocation.
- **Fund & sponsor accountability:** tracked funds and consented reporting meet donor obligations without violating minors' privacy.
- **Financial integrity:** transparent coverage/stipend flows and audit support statutory finance audit.
- **Minors' data:** shared with sponsors only within consented scope.

## 17. Future Considerations
- Public scholarship application portal with document upload.
- Automated need/merit scoring feeding committee shortlists.
- Multi-year award modeling with automated renewal checkpoints.
- Donor portals with privacy-preserving impact dashboards.
