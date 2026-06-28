# 20 — Scholarship Use Cases

Transforms the Scholarship business rules (`SCH-001`…`SCH-008`) into use cases. A structured, funded award distinct from discount/waiver: defined eligibility and fair slot-bounded selection, governed selection with conflict-of-interest controls, coverage/stipend with a no-negative-fee rule, fund tracking and sponsor accountability, condition-based renewal/revocation, and history-preserving award changes.

## 1. Primary Actors
Scholarship Administrator (defines programs, manages funds), Selection Committee (evaluates and selects), Sponsor (funds, accountability view).

## 2. Secondary Actors
System (eligibility, slot bounding, COI checks, fund tracking, coverage application), Workflow Engine (committee selection, SoD/COI), Fee/Discount modules (coverage application), Finance (fund accounting), Audit service.

## 3. Goals
Define funded scholarship programs distinct from discounts; select fairly within bounded slots; govern selection with conflict-of-interest controls; apply coverage/stipend without driving fees negative; track funds and sponsor accountability; renew/revoke on conditions under governance; never rewrite award history.

## 4. User Journeys
- **Establish:** admin defines a program (eligibility, slots, coverage, fund source, sponsor) → fund balance tracked.
- **Select:** applicants evaluated → committee selects within slots via a governed workflow with COI controls (a committee member related to an applicant recuses) → awards granted.
- **Apply:** coverage reduces fees (interacting with discounts per SCH-006) without negative net; stipends disbursed.
- **Sustain:** renewal/revocation evaluated against conditions (e.g., GPA) under governance; changes preserve history; sponsor sees accountability reports.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-SCH-001 | Define Scholarship Program (funded, slot-bounded) | High |
| Core | UC-SCH-002 | Evaluate Eligibility & Shortlist | High |
| Core | UC-SCH-003 | Govern Selection (slots, COI controls) | Critical |
| Core | UC-SCH-004 | Apply Coverage / Stipend (no negative fee) | High |
| Core | UC-SCH-005 | Renew / Revoke Award (condition-based) | High |
| Core | UC-SCH-006 | Track Fund & Sponsor Accountability | High |
| CRUD | UC-SCH-007 | View Scholarship / Award (scoped) | Medium |
| Admin | UC-SCH-008 | Configure Eligibility, Slots & Coverage | High |
| Approval | UC-SCH-009 | Approve Selection / Disbursement (SoD) | Critical |
| Search | UC-SCH-010 | Search Scholarships / Awards | Low |
| Reporting | UC-SCH-011 | Award & Fund Utilization Report (sponsor) | Medium |
| Bulk | UC-SCH-012 | Bulk Award / Disbursement | Medium |
| Import/Export | UC-SCH-013 | Import / Export Program Data | Low |
| Workflow | UC-SCH-014 | Selection / Renewal Workflow | High |
| Exception | UC-SCH-015 | Over-Slot Award Blocked | High |
| Exception | UC-SCH-016 | Conflict-of-Interest Selection Blocked | Critical |

---

## 6. Detailed Specifications (high-value use cases)

### UC-SCH-003 — Govern Selection (slots, COI controls)
- **Module:** Scholarship · **Priority:** Critical
- **Actors:** Selection Committee (primary), Approver, System
- **Goal:** Select awardees fairly within bounded slots, with conflict-of-interest controls and SoD.
- **Description:** A governed, version-pinned committee workflow selects within the program's slot limit; committee members with a conflict of interest (related to an applicant) are recused; selection and approval are separated.
- **Business Rules Applied:** SCH-002, SCH-003, AUTHZ-009, WFL-002, WFL-004.
- **Preconditions:** Program defined with slots; shortlist prepared; committee constituted.
- **Trigger:** Committee runs selection.
- **Main Success Scenario:**
  1. System presents eligible, shortlisted applicants within the slot limit (SCH-002).
  2. The version-pinned selection workflow runs; COI checks recuse conflicted members (SCH-003).
  3. The committee selects within slots; selection ≠ final approval (SoD).
  4. An authorized approver ratifies; awards are granted within slots.
- **Alternative Flows:** A1) Quorum-based committee decision (WFL-008). A2) Waitlist for over-subscription.
- **Exception Flows:** E1) Selecting beyond slots → blocked (UC-SCH-015). E2) A conflicted member acting → blocked/recused (UC-SCH-016).
- **Validation Rules:** Within slots; COI recusal enforced; SoD (selection ≠ approval); version-pinned (SCH-002/003, AUTHZ-009).
- **Permissions Required:** `scholarship.select` (committee) + `scholarship.approve` (SoD).
- **Notifications Triggered:** `SCHOLARSHIP_AWARDED` to awardees/guardians; committee notifications.
- **Audit Events Generated:** `SELECTION_RECORDED`, `COI_RECUSAL`, `AWARD_GRANTED`.
- **Data Created:** Awards; selection records.
- **Data Updated:** Slot counts; fund commitments.
- **Data Deleted:** None.
- **Post Conditions:** Fair, within-slot awards granted; COI and SoD enforced; fully audited.
- **Related Use Cases:** UC-SCH-002, UC-SCH-004, UC-SCH-009.
- **Acceptance Criteria:**
  - Given a slot limit, When selection runs, Then no more than the available slots are awarded.
  - Given a committee member related to an applicant, When selecting, Then they are recused from that decision.
  - Given selection, When finalized, Then a separate approver ratifies (SoD).
- **Edge Case Analysis:**
  - *Invalid Input:* selecting an ineligible applicant blocked.
  - *Permission Failure:* selector also approving → SoD block.
  - *Concurrent Update:* two selections for the last slot → one succeeds (slot bound).
  - *Duplicate Data:* duplicate award to the same student prevented.
  - *System Failure:* atomic; slot counts consistent.
  - *Workflow Failure:* approver vacancy → escalation; no auto-grant.
- **QA Coverage:**
  - *Positive:* within-slot selection; quorum decision; waitlist.
  - *Negative:* over-slot; COI member acting; self-approval.
  - *Boundary:* last slot under concurrency; committee quorum edge.

### UC-SCH-004 — Apply Coverage / Stipend (no negative fee)
- **Module:** Scholarship · **Priority:** High
- **Actors:** System (primary), Finance Administrator
- **Goal:** Apply scholarship coverage to fees (and disburse stipends) without driving the net fee negative, interacting correctly with discounts.
- **Description:** Coverage reduces applicable fees per the program; combined with discounts/waivers per SCH-006 and clamped to a non-negative net (SCH-004); stipends are disbursed and fund-tracked.
- **Business Rules Applied:** SCH-004, SCH-006, DSC-004, FEE-002.
- **Preconditions:** Active award; applicable fees; fund available.
- **Trigger:** Invoice generation / coverage application.
- **Main Success Scenario:**
  1. System applies coverage to applicable fee heads per the program (SCH-004).
  2. System resolves interaction with discounts/waivers (SCH-006) and clamps net ≥ 0 (SCH-004/DSC-004).
  3. Coverage appears as a transparent line; any stipend is disbursed and fund-tracked (SCH-005).
- **Alternative Flows:** A1) Partial coverage (percentage/capped). A2) Stipend-only award (no fee coverage).
- **Exception Flows:** E1) Coverage exceeding fees → clamped to zero, remainder per policy (not negative). E2) Insufficient fund → flagged (UC-SCH-006).
- **Validation Rules:** Net fee ≥ 0; interaction rules applied; fund sufficient; transparent (SCH-004/006).
- **Permissions Required:** System; `scholarship.disburse` for stipends (+ approval).
- **Notifications Triggered:** `SCHOLARSHIP_APPLIED`, `STIPEND_DISBURSED`.
- **Audit Events Generated:** `COVERAGE_APPLIED`, `STIPEND_DISBURSED`.
- **Data Created:** Coverage line; disbursement record.
- **Data Updated:** Net fee; fund balance.
- **Data Deleted:** None.
- **Post Conditions:** Coverage applied transparently, net non-negative; fund updated.
- **Related Use Cases:** UC-FEE-002, UC-DSC-004, UC-SCH-006.
- **Acceptance Criteria:**
  - Given coverage exceeding the applicable fees, When applied, Then the net fee is zero (never negative).
  - Given a scholarship plus a discount, When both apply, Then SCH-006 governs the combined result.
  - Given a stipend, When disbursed, Then the fund balance decreases and it is tracked.
- **Edge Case Analysis:**
  - *Invalid Input:* coverage on a non-applicable head ignored/blocked.
  - *Permission Failure:* stipend disbursement without permission → 403.
  - *Concurrent Update:* coverage + discount racing → deterministic net ≥ 0.
  - *Duplicate Data:* duplicate coverage prevented.
  - *System Failure:* atomic; fund consistent.
  - *Workflow Failure:* disbursement approval pends.
- **QA Coverage:**
  - *Positive:* full/partial coverage; stipend; scholarship+discount.
  - *Negative:* over-coverage (clamp); insufficient fund; duplicate.
  - *Boundary:* coverage equal to fees (net zero); fund exactly sufficient.

---

## 7. Compact Specifications (routine use cases)

- **UC-SCH-001 — Define Scholarship Program (funded, slot-bounded)** · *High* · Rules: SCH-001, SCH-005. Define program (eligibility, slots, coverage, fund, sponsor) distinct from discount. *Permissions:* `scholarship.config.manage`. *Audit:* `PROGRAM_DEFINED`. *Edge:* funded + slot-bounded; sponsor linked. *QA:* define; fund link; slot set.
- **UC-SCH-002 — Evaluate Eligibility & Shortlist** · *High* · Rules: SCH-002. Evaluate applicants against defined eligibility; shortlist within fairness rules. *Edge:* objective criteria; ranked shortlist. *QA:* eligibility; shortlist correctness.
- **UC-SCH-005 — Renew / Revoke Award (condition-based)** · *High* · Rules: SCH-007, SCH-008. Renew/revoke against conditions (e.g., GPA) under governance; history preserved. *Audit:* `AWARD_RENEWED/REVOKED`. *Edge:* condition-based; never rewrites history. *QA:* renew on met; revoke on unmet; history intact.
- **UC-SCH-006 — Track Fund & Sponsor Accountability** · *High* · Rules: SCH-005. Track fund balance, commitments, disbursements; sponsor accountability. *Edge:* over-commitment flagged; sponsor-scoped view. *QA:* fund accuracy; over-commit flag.
- **UC-SCH-007 — View Scholarship / Award (scoped)** · *Medium* · Rules: AUTHZ-003, GRD-N-005. Scoped view; awardee/guardian see own award. *QA:* scope respected.
- **UC-SCH-008 — Configure Eligibility, Slots & Coverage** · *High* · Rules: SCH-002/004, CFG-004. Configure criteria, slots, coverage (versioned). *Edge:* coverage interaction (SCH-006) coherent. *QA:* config; coherence.
- **UC-SCH-009 — Approve Selection / Disbursement (SoD)** · *Critical* · Rules: SCH-003, AUTHZ-009, WFL-004. Approver (≠ selector) approves selection/disbursement. *Edge:* SoD; COI; escalation. *QA:* approval; SoD; COI.
- **UC-SCH-010 — Search Scholarships / Awards** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-SCH-011 — Award & Fund Utilization Report (sponsor)** · *Medium* · Rules: REP-002, SCH-005. Award/fund utilization for sponsors (scoped). *Edge:* sponsor sees own program only. *QA:* utilization accuracy; sponsor scope.
- **UC-SCH-012 — Bulk Award / Disbursement** · *Medium* · Rules: SCH-003/004. Bulk award/disburse within slots/fund. *Edge:* slot/fund enforced; partial-failure report. *QA:* clean bulk; slot/fund limits.
- **UC-SCH-013 — Import / Export Program Data** · *Low* · Rules: CFG-002, REP-005. Import/export program/award data (validated, governed). *QA:* valid import; scoped export.
- **UC-SCH-014 — Selection / Renewal Workflow** · *High* · Rules: WFL-002/004/008, SCH-003. Version-pinned committee workflow with quorum/COI. *QA:* pinning; quorum; COI; SoD.
- **UC-SCH-015 — Over-Slot Award Blocked (Exception)** · *High* · Rules: SCH-002. Awards cannot exceed slots. *QA:* over-slot blocked; waitlist.
- **UC-SCH-016 — Conflict-of-Interest Selection Blocked (Exception)** · *Critical* · Rules: SCH-003. Conflicted committee member recused/blocked. *QA:* COI recusal; clean selection allowed.

## 8. Module-level QA & Edge Themes
- **Distinct from discount/waiver (SCH-001):** scholarships are funded, slot-bounded awards with fund accounting — not fee reductions; the interaction with discounts (SCH-006) is explicit.
- **Governed selection + COI (SCH-003 / P9):** slot bounding, conflict-of-interest recusal, and SoD (selection ≠ approval) are the headline control suite.
- **No negative fee (SCH-004):** coverage clamps net fees at zero, interacting correctly with discounts.
- **Fund accountability (SCH-005):** over-commitment is flagged; sponsors see scoped utilization.
- **History-preserving changes (SCH-008):** renewals/revocations never rewrite award history.
