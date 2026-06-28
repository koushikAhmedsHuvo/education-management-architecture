# 08 — Enrollment Use Cases

Transforms the Enrollment business rules (`ENR-001`…`ENR-008`) into use cases. Enrollment binds a student to a session/level/section instance — the operative leaf that drives eligibility, capacity, roll numbers, transfers, and promotion.

## 1. Primary Actors
Enrollment Administrator / Registrar (manages enrollments), Academic Coordinator (section placement).

## 2. Secondary Actors
System (binding, capacity, roll numbers, transfer history, promotion), Admission module (conversion source), Session module (rollover), Finance (clearance on exit), Notification & Audit services.

## 3. Goals
Bind each active student to exactly one active enrollment per session; enforce section hard capacity; assign unique roll numbers; move students between sections/campuses with period-accurate history; handle withdrawal/exit with clearance and a transfer certificate; drive operational eligibility (attendance, exams, fees) from enrollment status; promote to the next session by outcome.

## 4. User Journeys
- **Enroll:** from admission conversion (UC-ADM-005) or registrar action → student bound to a section instance with a roll number, within the hard cap.
- **Move:** transfer between sections (rebalancing) or campuses (UC-CAMP-005) → dated, history-preserving.
- **Exit:** withdrawal → clearance (dues, library, returns) → transfer certificate → enrollment closed.
- **Advance:** session rollover (UC-SESS-004) creates the next-session enrollment by outcome (advance/retain/graduate).

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core/CRUD | UC-ENR-001 | Create Enrollment (bind to section, hard cap) | Critical |
| CRUD | UC-ENR-002 | View Enrollment(s) | Medium |
| CRUD | UC-ENR-003 | Update Enrollment (placement/details) | Medium |
| Core | UC-ENR-004 | Inter-Section Transfer | High |
| Core | UC-ENR-005 | Withdrawal & Clearance (exit) | High |
| Core | UC-ENR-006 | Promotion → Next-Session Enrollment | Critical |
| Core | UC-ENR-007 | Re-Enrollment / Re-Admission of Returning Student | Medium |
| Admin | UC-ENR-008 | Assign / Reassign Roll Number | Medium |
| Approval | UC-ENR-009 | Approve Transfer / Withdrawal | Medium |
| Search | UC-ENR-010 | Search Enrollments | Medium |
| Reporting | UC-ENR-011 | Enrollment & Capacity Report | Medium |
| Bulk | UC-ENR-012 | Bulk Enrollment / Section Rebalance | High |
| Import | UC-ENR-013 | Import Enrollments | Low |
| Export | UC-ENR-014 | Export Enrollment Data | Low |
| Workflow | UC-ENR-015 | Withdrawal / Transfer Workflow | Medium |
| Exception | UC-ENR-016 | Double-Enrollment Blocked | High |
| Exception | UC-ENR-017 | Section Capacity Full | High |
| Exception | UC-ENR-018 | Cross-Institution Exit (withdrawal + TC) | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-ENR-001 — Create Enrollment (bind to section, hard cap)
- **Module:** Enrollment · **Priority:** Critical
- **Actors:** Registrar / System (primary)
- **Goal:** Bind a student to a specific session/level/section instance within the hard capacity, with one active enrollment per session.
- **Description:** Creates the enrollment leaf, enforcing one-active-per-session and the section hard cap (the authoritative limit at enrollment per Conflict C-01), and assigns a roll number.
- **Business Rules Applied:** ENR-001, ENR-002, ENR-003, ENR-004, ENR-008.
- **Preconditions:** Active student; target section instance exists in the session; seat available.
- **Trigger:** Admission conversion or registrar enrollment.
- **Main Success Scenario:**
  1. System verifies the student has no active enrollment for the session (ENR-002).
  2. System checks the section hard capacity (ENR-003); if a seat exists, proceeds.
  3. System binds the student to the section instance (ENR-001) and assigns a unique roll number (ENR-004).
  4. Enrollment is `ACTIVE`, enabling attendance/exam/fee eligibility (ENR-008).
- **Alternative Flows:** A1) Placement into a specific section by the coordinator; else auto-balance.
- **Exception Flows:** E1) Existing active enrollment for the session → block (UC-ENR-016). E2) Section full → block/reroute (UC-ENR-017).
- **Validation Rules:** No active enrollment for the session; seat within hard cap; section in the same institute/campus scope (ENR-001/002/003).
- **Permissions Required:** `enrollment.manage`.
- **Notifications Triggered:** `ENROLLMENT_CONFIRMED` to guardian.
- **Audit Events Generated:** `ENROLLMENT_CREATED` (student, section, roll number).
- **Data Created:** Enrollment; roll number.
- **Data Updated:** Section active count; student status → active.
- **Data Deleted:** None.
- **Post Conditions:** One active enrollment; section count ≤ hard cap; eligibility enabled.
- **Related Use Cases:** UC-ADM-005, UC-ENR-004, UC-ENR-016.
- **Acceptance Criteria:**
  - Given an active student with no session enrollment and an available seat, When enrolled, Then a single active enrollment with a unique roll number is created.
  - Given a student already enrolled for the session, When re-enrolled, Then it is blocked.
  - Given a full section, When enrollment is attempted, Then it is blocked or rerouted (hard cap).
- **Edge Case Analysis:**
  - *Invalid Input:* enrolling an inactive/non-existent student rejected.
  - *Permission Failure:* lacks `enrollment.manage` → 403.
  - *Concurrent Update:* two enrollments racing for the last seat → only one succeeds (hard cap).
  - *Duplicate Data:* duplicate enrollment for the session prevented (ENR-002).
  - *System Failure:* atomic; roll number not consumed on failure.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* enroll; placement; auto-balance.
  - *Negative:* double-enrollment; full section; cross-scope section.
  - *Boundary:* last seat under concurrency; roll-number uniqueness at scope edge.

### UC-ENR-005 — Withdrawal & Clearance (exit)
- **Module:** Enrollment · **Priority:** High
- **Actors:** Registrar (primary), Finance (clearance), System, Guardian (notified)
- **Goal:** Exit a student cleanly with clearance and a transfer certificate.
- **Description:** Withdrawal runs a clearance checklist (dues, library, asset returns), then closes the enrollment and issues a transfer certificate; cross-institution exit is always withdrawal + TC (not a "transfer").
- **Business Rules Applied:** ENR-006, ENR-008, FEE (clearance), FILE-007 (TC document).
- **Preconditions:** Active enrollment; withdrawal requested; clearance items defined.
- **Trigger:** Guardian/registrar initiates withdrawal.
- **Main Success Scenario:**
  1. Registrar initiates withdrawal with a reason and effective date.
  2. System runs the clearance checklist (outstanding dues, returns).
  3. On clearance, System closes the enrollment (`WITHDRAWN`) and updates student status.
  4. System generates a transfer certificate (immutable, reproducible) for the family.
- **Alternative Flows:** A1) Withdrawal may require approval (UC-ENR-015). A2) Clearance waiver via governed exception.
- **Exception Flows:** E1) Outstanding dues/items → block completion until cleared or waived. E2) Cross-institution move → handled as withdrawal + TC (UC-ENR-018), never an internal transfer.
- **Validation Rules:** Clearance satisfied (or waived); effective date valid; TC generated (ENR-006).
- **Permissions Required:** `enrollment.withdraw` (+ finance clearance).
- **Notifications Triggered:** `WITHDRAWAL_COMPLETED`, transfer-certificate-ready to guardian.
- **Audit Events Generated:** `ENROLLMENT_WITHDRAWN` (reason), `TRANSFER_CERTIFICATE_ISSUED`.
- **Data Created:** Transfer certificate document.
- **Data Updated:** Enrollment → `WITHDRAWN`; student status; section seat freed.
- **Data Deleted:** None.
- **Post Conditions:** Enrollment closed; seat freed (waitlist may promote); TC issued; history retained.
- **Related Use Cases:** UC-ENR-018, UC-ADM-006 (waitlist), UC-FILE-007.
- **Acceptance Criteria:**
  - Given satisfied clearance, When withdrawal completes, Then the enrollment closes, a TC is issued, and the seat is freed.
  - Given outstanding dues, When withdrawal is attempted, Then completion is blocked until cleared or waived.
  - Given a cross-institution exit, When processed, Then it is withdrawal + TC, not an internal transfer.
- **Edge Case Analysis:**
  - *Invalid Input:* withdrawal without reason (if required) rejected.
  - *Permission Failure:* lacks `enrollment.withdraw` → 403.
  - *Concurrent Update:* withdrawal during fee posting → clearance reflects latest balance.
  - *Duplicate Data:* re-withdrawing a closed enrollment idempotent.
  - *System Failure:* TC generation from finalized data; no partial close.
  - *Workflow Failure:* approval/clearance pends; enrollment stays active until complete.
- **QA Coverage:**
  - *Positive:* clean withdrawal + TC; seat freed → waitlist promote.
  - *Negative:* outstanding dues block; cross-institution path.
  - *Boundary:* withdrawal on a term boundary; clearance waiver (governed).

### UC-ENR-006 — Promotion → Next-Session Enrollment
- **Module:** Enrollment · **Priority:** Critical
- **Actors:** System (primary, via rollover), Registrar (oversight)
- **Goal:** Create the next-session enrollment for each student by academic outcome.
- **Description:** Driven by session rollover (UC-SESS-004), creates the next enrollment: advance → next level instance; retain → same level's new-session instance; graduate → exit. Atomic and reversible within the window.
- **Business Rules Applied:** ENR-007, ENR-002, ENR-003, SESS-006.
- **Preconditions:** Rollover initiated; outcomes finalized; target instances exist.
- **Trigger:** Session rollover batch.
- **Main Success Scenario:**
  1. For each active enrollment, System reads the outcome.
  2. System creates the appropriate next-session enrollment (advance/retain), enforcing one-active-per-session and the hard cap.
  3. Graduating students exit; withdrawn excluded.
  4. Prior enrollments close as completed; next-session enrollments become active when the session is current.
- **Alternative Flows:** A1) Manual override of an individual outcome (audited).
- **Exception Flows:** E1) Target level missing → block (UC-SESS-018). E2) Capacity at target full → overflow handling per policy (reroute/expand).
- **Validation Rules:** Outcome finalized; target instance exists with capacity; one-active-per-session preserved (ENR-007/002/003).
- **Permissions Required:** `enrollment.promote` (system/registrar).
- **Notifications Triggered:** Promotion outcome to guardians (per preference).
- **Audit Events Generated:** `ENROLLMENT_PROMOTED/RETAINED/GRADUATED`.
- **Data Created:** Next-session enrollments.
- **Data Updated:** Prior enrollments → completed.
- **Data Deleted:** None.
- **Post Conditions:** Cohort enrolled for the next session by outcome; reversible within the window.
- **Related Use Cases:** UC-SESS-004, UC-SESS-012.
- **Acceptance Criteria:**
  - Given an advancing student, When promotion runs, Then a next-level next-session enrollment is created.
  - Given a retained student, When promotion runs, Then they are enrolled in the same level's new-session instance.
  - Given target capacity exhaustion, When promoting, Then overflow is handled per policy (never silent over-enroll).
- **Edge Case Analysis:**
  - *Invalid Input:* promotion without finalized outcome blocked.
  - *Permission Failure:* lacks `enrollment.promote` → 403.
  - *Concurrent Update:* re-run promotion → idempotent (no double next-enrollment).
  - *Duplicate Data:* one-active-per-session prevents duplicates (ENR-002).
  - *System Failure:* atomic/recoverable; no half-promoted cohort.
  - *Workflow Failure:* rollover approval gates the batch.
- **QA Coverage:**
  - *Positive:* advance; retain; graduate.
  - *Negative:* missing target; unfinalized outcome; capacity overflow.
  - *Boundary:* full-cohort retain; single-student promotion; rollback window edge.

---

## 7. Compact Specifications (routine use cases)

- **UC-ENR-002 — View Enrollment(s)** · *Medium* · Rules: AUTHZ-002/003. Scoped read; teacher sees own sections; guardian sees children. *QA:* scope/ownership respected.
- **UC-ENR-003 — Update Enrollment (placement/details)** · *Medium* · Rules: ENR-001, ENR-004. Adjust placement/roll within scope; binding to student/session immutable. *Audit:* `ENROLLMENT_UPDATED`. *Edge:* cannot rebind to a different student. *QA:* placement edit; immutable-binding protection.
- **UC-ENR-004 — Inter-Section Transfer** · *High* · Rules: ENR-005, ENR-003. Move between sections (same session) preserving history; target hard cap enforced. *Audit:* `SECTION_TRANSFER`. *Edge:* target full blocks; period attribution preserved. *QA:* transfer; full target; history fidelity.
- **UC-ENR-007 — Re-Enrollment / Re-Admission of Returning Student** · *Medium* · Rules: ENR-008, STU-003. Returning student links to the existing record; new enrollment created. *Edge:* dedup to existing student; gap period handled. *QA:* re-enroll links; no duplicate student.
- **UC-ENR-008 — Assign / Reassign Roll Number** · *Medium* · Rules: ENR-004. Assign/reassign unique roll within section/session. *Edge:* uniqueness in scope; reassignment audited. *QA:* unique assignment; duplicate blocked.
- **UC-ENR-009 — Approve Transfer / Withdrawal** · *Medium* · Rules: WFL-004. Approver gates transfer/withdrawal. *Edge:* SoD; escalation. *QA:* approval gates; timeout escalates.
- **UC-ENR-010 — Search Enrollments** · *Medium* · Rules: AUTHZ-002, REP-002. Scoped search by section/status/session. *QA:* scope respected.
- **UC-ENR-011 — Enrollment & Capacity Report** · *Medium* · Rules: REP-002, ENR-003. Section fill levels, capacity utilization, vacancies. *Edge:* counts consistent with hard cap; as-of freshness. *QA:* counts match; capacity accuracy.
- **UC-ENR-012 — Bulk Enrollment / Section Rebalance** · *High* · Rules: ENR-001/003/005. Batch enroll or rebalance sections (capacity-enforced, history-preserving). *Edge:* per-seat capacity; partial-failure report. *QA:* clean bulk; overflow; rebalance history.
- **UC-ENR-013 — Import Enrollments** · *Low* · Rules: ENR-001/002/003. Import with one-active-per-session and capacity checks. *Edge:* duplicates/over-capacity rows reported. *QA:* clean import; duplicate; over-capacity.
- **UC-ENR-014 — Export Enrollment Data** · *Low* · Rules: REP-005. Governed, scoped export. *QA:* scoped export.
- **UC-ENR-015 — Withdrawal / Transfer Workflow** · *Medium* · Rules: WFL-002/004. Version-pinned workflow for exit/transfer. *QA:* pinning; SoD; escalation.
- **UC-ENR-016 — Double-Enrollment Blocked (Exception)** · *High* · Rules: ENR-002. One active enrollment per student per session. *QA:* second active blocked; cross-session allowed.
- **UC-ENR-017 — Section Capacity Full (Exception)** · *High* · Rules: ENR-003 (Conflict C-01). Enrollment/transfer blocked at hard cap; reroute/waitlist. *QA:* hard cap wins; reroute path.
- **UC-ENR-018 — Cross-Institution Exit (Exception)** · *High* · Rules: ENR-006. Cross-institution move = withdrawal + transfer certificate, not internal transfer. *QA:* TC issued; not modeled as transfer.

## 8. Module-level QA & Edge Themes
- **Capacity at the leaf (C-01):** the section hard cap is the authoritative limit at enrollment, transfer, and promotion — concurrency for the last seat is the key suite.
- **One-active-per-session (ENR-002):** double-enrollment prevention across enroll/import/promotion.
- **Period-accurate history:** transfers and promotions must preserve prior-period attribution.
- **Eligibility coupling (ENR-008):** enrollment status correctly gates attendance/exam/fee operations downstream.
