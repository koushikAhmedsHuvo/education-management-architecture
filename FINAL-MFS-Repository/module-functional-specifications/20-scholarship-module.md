# 20 — Scholarship Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`SCH-001…008`), and Use Case Repository (`UC-SCH-001…016`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Manage funded, slot-bounded scholarship awards — distinct from discounts/waivers: defined eligibility and fair slot-bounded selection, governed selection with conflict-of-interest controls, coverage/stipend with no-negative-fee, fund tracking and sponsor accountability, condition-based renewal/revocation, and history-preserving award changes.

**Business Goal.** Award scholarships fairly within funded slots, with conflict-of-interest controls and SoD, tracking funds and sponsor accountability — never driving fees negative and never rewriting award history.

**Scope.** Scholarship program definition (funded, slot-bounded, sponsor); eligibility & fair slot-bounded selection; governed selection with COI controls; coverage/stipend application (no negative fee); fund tracking & sponsor accountability; condition-based renewal/revocation; history-preserving award changes; interaction with discounts (SCH-006).

**Out of Scope.** Fee reductions as discounts/waivers (Discount module — distinct). Fee structure (Fee module). Payment/disbursement mechanics (Payment/Fee). Config versioning (Configuration Engine).

---

## 2. Actors

**Primary Actors.** Scholarship Administrator (programs, funds), Selection Committee (evaluates, selects), Sponsor (funds, accountability view), Approver (selection/disbursement — SoD), System (eligibility, slot bounding, COI checks, fund tracking, coverage).

**Secondary Actors.** Fee/Discount modules (coverage/interaction), Workflow Engine (committee selection, SoD/COI), Finance (fund accounting), Audit, Reporting, Notification.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Define funded program | Define program (eligibility, slots, coverage, fund, sponsor), distinct from discount. | High | Configuration Engine |
| FR-002 | Eligibility & shortlist | Evaluate applicants against eligibility; fair shortlist. | High | FR-001 |
| FR-003 | Governed selection (slots, COI) | Select within slots with COI recusal and SoD. | Critical | Workflow, AUTHZ-009 |
| FR-004 | Coverage/stipend (no negative fee) | Apply coverage/stipend; net fee never negative. | High | Fee, Discount (SCH-006) |
| FR-005 | Renewal/revocation (condition-based) | Renew/revoke against conditions, governed, history-preserving. | High | Result/GPA, Workflow |
| FR-006 | Fund tracking & sponsor accountability | Track fund balance/commitments; sponsor-scoped reporting. | High | Finance |
| FR-007 | History-preserving award changes | Award changes never rewrite history. | High | Audit |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Funded programs | Slot-bounded, sponsor-linked. | Structured awards. |
| Fair selection | Eligibility + slots. | Equity. |
| COI controls | Recusal + SoD. | Integrity. |
| Coverage/stipend | No negative fee. | Correct net. |
| Fund tracking | Balance/commitments. | Sponsor accountability. |
| Condition renewal/revocation | Governed lifecycle. | Performance-linked awards. |
| History-preserving | No rewrite. | Trustworthy records. |

---

## 5. Screens

Scholarship Program List; Program Definition (funded, slot-bounded); Eligibility & Shortlist; Governed Selection (slots, COI); Coverage/Stipend; Renewal/Revocation; Fund & Sponsor Accountability; Award/Fund Utilization Report; Bulk Award/Disbursement; Import/Export Program Data; Selection/Renewal Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Program List | Define, Open, Search, Export | — |
| Program Definition | Set Eligibility/Slots/Coverage/Fund/Sponsor, Save | — |
| Eligibility & Shortlist | Evaluate, Shortlist, Rank | — |
| Governed Selection | Select (within slots), COI Recuse, Submit (→ approval) | Bulk Award |
| Coverage/Stipend | Apply Coverage, Disburse Stipend (→ approval) | Bulk Disburse |
| Renewal/Revocation | Renew (condition met), Revoke (condition unmet) | — |
| Fund Accountability | View Balance/Commitments, Sponsor Report | — |
| Utilization Report | Run, Filter, Export | Export |

---

## 7. Forms

**Program Definition** — `name`, `eligibility` (criteria), `slots` (number, required), `coverage` (%/amount/stipend), `fundSource`, `sponsor`. Validation: funded + slot-bounded (SCH-001/002); sponsor linked (SCH-005); distinct from discount.

**Selection** — committee `select` within slots, COI `recuse`. Validation: within slots (SCH-002 / UC-SCH-015); conflicted member recused (SCH-003 / UC-SCH-016); selection ≠ approval (SoD).

**Coverage/Stipend** — `coverage` application, `stipend` disbursement. Validation: net fee ≥ 0 (SCH-004/DSC-004); interaction with discounts (SCH-006); fund sufficient (SCH-005).

**Renewal/Revocation** — `condition` (e.g., GPA), `action` (renew/revoke), `reason`. Validation: condition-based, governed; history preserved (SCH-007/008).

---

## 8. Search & Filter Requirements

**Scholarships/Awards:** by program, student, sponsor, status (awarded/renewed/revoked), fund, eligibility. Sorting: program/date/status. Pagination: server-side, 25 default. Scope-bound; sponsor sees own program only.

---

## 9. Table Requirements

**Award table:** Student, Program, Coverage, Status, Sponsor, Fund Drawn, Renewal. Sorting on Program/Status. Filtering as above. Export (governed). Bulk: award/disburse.

---

## 10. Workflow Requirements

**Trigger events:** define program, shortlist, select (COI), award, coverage/stipend, renew/revoke. **Status changes:** award `SELECTED → APPROVED → ACTIVE → RENEWED/REVOKED`. **Approvals:** selection/disbursement via Workflow Engine (SoD, COI, quorum). **Notifications:** award, coverage, renewal/revocation (to awardees/guardians); sponsor reports. **Audit:** selection (COI recusals), awards, coverage, renewals/revocations (history-preserving, immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Define program/configure | `scholarship.config.manage` |
| Evaluate/shortlist/select | `scholarship.select` (committee) |
| Approve selection/disbursement | `scholarship.approve` (SoD) |
| Apply coverage/disburse stipend | `scholarship.disburse` |
| Renew/revoke | `scholarship.lifecycle.manage` |
| View award (scoped) | `scholarship.view` |
| Sponsor accountability view | `scholarship.sponsor.view` |
| Import/export | `scholarship.import`, `scholarship.export` |

Selection enforces COI + SoD (SCH-003); selection ≠ approval.

---

## 12. Business Rule References

SCH-001 (funded award, distinct from discount/waiver), SCH-002 (eligibility & slot-bounded selection), SCH-003 (governed selection & COI controls), SCH-004 (coverage/stipend & no-negative-fee), SCH-005 (fund tracking & sponsor accountability), SCH-006 (scholarship + discount/waiver interaction), SCH-007 (renewal & revocation condition-based & governed), SCH-008 (award changes never rewrite history). Cross-cutting: DSC-004 (no negative fee), FEE-002 (fee heads), WFL-002/004/008 (selection/quorum/SoD), AUTHZ-009, AUD-001, CFG-004.

## 13. Use Case References

UC-SCH-001 (Define Program — funded, slot-bounded), UC-SCH-002 (Evaluate Eligibility & Shortlist), UC-SCH-003 (Govern Selection — slots, COI), UC-SCH-004 (Apply Coverage/Stipend — no negative fee), UC-SCH-005 (Renew/Revoke), UC-SCH-006 (Track Fund & Sponsor Accountability), UC-SCH-007 (View — scoped), UC-SCH-008 (Configure Eligibility/Slots/Coverage), UC-SCH-009 (Approve Selection/Disbursement — SoD), UC-SCH-010 (Search), UC-SCH-011 (Award & Fund Utilization Report), UC-SCH-012 (Bulk Award/Disbursement), UC-SCH-013 (Import/Export), UC-SCH-014 (Selection/Renewal Workflow), UC-SCH-015 (Over-Slot Award Blocked), UC-SCH-016 (Conflict-of-Interest Selection Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Define / configure program | POST/PUT | Scholarship Admin |
| Evaluate eligibility / shortlist | POST | Committee |
| Govern selection (slots, COI) | POST | Committee (→ approval) |
| Approve selection / disbursement | POST | Approver |
| Apply coverage / disburse stipend | POST | Scholarship Admin |
| Renew / revoke award | POST | Scholarship Admin |
| Fund & sponsor accountability | GET | Admin / Sponsor |
| Award & fund utilization report | GET | Admin / Sponsor |
| Bulk award / import / export | POST/GET | Admin |

Selection enforces slots + COI recusal + SoD (SCH-002/003); coverage clamps net fee at zero (SCH-004).

---

## 15. Database Requirements

**Entities:** `ScholarshipProgram` (eligibility, slots, coverage, fund, sponsor), `Award` (studentId, programId, status, coverage), `Selection` (committee, COI recusals), `Fund` (balance, commitments, disbursements), `AwardChangeLog` (renew/revoke, reason, history), `Stipend` (disbursement). **Relationships:** Program 1—* Award; Program 1—1 Fund; Award 1—* AwardChangeLog. **Indexes:** index(Award.programId, status), index(Award.studentId), index(Fund.programId). Slot count enforced (SCH-002); history-preserving changes (SCH-008).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Award, coverage, renewal/revocation — to awardee/guardian; sponsor utilization reports. |
| In-App | Selection/disbursement approvals; COI recusal notices. |
| SMS/Push | Award confirmation (minimized). |

Routed to permitted guardian (GRD-N-005); sponsor sees own program only (SCH-005).

---

## 17. Audit Requirements

Log: program definition, selection (with COI recusals), awards, coverage/stipend disbursement, renewals/revocations, fund movements. Record who/when/before/after. Selection (COI/SoD) and award changes are first-class audit events; changes never rewrite history (SCH-008). Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Award/fund utilization (sponsor-scoped), Selection fairness/COI log, Renewal/revocation, Coverage value. **Exports:** governed export (sponsor-scoped). **Dashboards:** scholarship governance (slots used, fund balance, pending selections, COI flags).

---

## 19. Error Handling

**Validation:** over-slot award, COI member acting → specific errors (UC-SCH-015/016). **Permission:** selector approving own selection → SoD block. **Workflow:** selection/disbursement pending → not awarded until approved. **System:** insufficient fund → coverage flagged/blocked; coverage exceeding fees → clamped to zero (not negative).

---

## 20. Edge Cases

**Concurrent updates:** two selections for last slot → one succeeds (slot bound). **Duplicate data:** duplicate award to same student prevented. **Partial failures:** bulk award/disburse partial → per-record report; slot/fund enforced. **Rollback:** revocation reversed → award restored, history intact. **COI race:** conflicted member acting → recused/blocked (SCH-003).

---

## 21. Acceptance Criteria

**Functional.** Scholarships are funded, slot-bounded awards distinct from discounts; selection stays within slots with COI recusal and SoD (selection ≠ approval); coverage/stipend never drives net fee negative and interacts correctly with discounts; renewal/revocation is condition-based and governed; funds are tracked with sponsor accountability; award changes never rewrite history.

**Business.** Scholarships are awarded fairly and transparently within funded limits; conflicts of interest are controlled; sponsors have accountable visibility; award history is trustworthy and immutable.

---

## 22. Future Enhancements

Applicant scholarship portal; automated eligibility scoring; multi-round committee balloting; sponsor self-service dashboards; fund-forecasting; renewal auto-evaluation from results; external donor integration; impact reporting.
