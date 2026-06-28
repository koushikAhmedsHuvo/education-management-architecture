# 06 — Admission Use Cases

Transforms the Admission business rules (`ADM-001`…`ADM-008`) into use cases. The funnel from applicant to enrolled student: configurable forms, version-pinned fair decisions, capacity discipline, and integrity-checked conversion.

## 1. Primary Actors
Applicant / Guardian (submits applications), Admissions Officer (reviews, evaluates), Admissions Administrator (configures forms, criteria, capacity), Approver (decision authority).

## 2. Secondary Actors
System (application numbering, idempotency, capacity, conversion), Workflow Engine (version-pinned decision workflow), Configuration Engine (form schema, criteria), Payment module (admission fee gating), Notification & Audit services.

## 3. Goals
Accept applications via configurable forms; evaluate fairly and consistently under version-pinned rules; control seats without over-filling sections; manage a waitlist; gate on admission fee where required; convert approved applicants into a Student + Enrollment with zero data loss or duplication.

## 4. User Journeys
- **Apply:** guardian completes the configured application form → submits → receives a unique application number (idempotent submission) → tracks status.
- **Decide:** officer reviews → criteria/merit evaluation → version-pinned approval workflow → offer / waitlist / reject, each applicant judged under the rules in force at submission.
- **Accept & convert:** offer accepted → admission fee paid (if gated) → system converts to Student + Enrollment, re-checking the section hard cap at conversion.
- **Exit paths:** applicant withdraws/declines; rejected/withdrawn applications are terminal; re-application is a new application.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-ADM-001 | Submit Application (idempotent) | Critical |
| Core | UC-ADM-002 | Evaluate Application (criteria/merit) | High |
| Core | UC-ADM-003 | Admission Decision (version-pinned workflow) | Critical |
| Core | UC-ADM-004 | Accept Offer & Pay Admission Fee | High |
| Core | UC-ADM-005 | Convert Approved Applicant → Student + Enrollment | Critical |
| Core | UC-ADM-006 | Waitlist Management & Promotion | High |
| CRUD | UC-ADM-007 | Configure Application Form (schema) | High |
| CRUD | UC-ADM-008 | View / Track Application | Medium |
| CRUD | UC-ADM-009 | Update Application (pre-submission / officer correction) | Medium |
| CRUD | UC-ADM-010 | Withdraw / Decline Application | Medium |
| Approval | UC-ADM-011 | Approve / Reject Decision (SoD) | High |
| Search | UC-ADM-012 | Search / Filter Applications | Medium |
| Reporting | UC-ADM-013 | Admission Funnel & Intake Report | Medium |
| Bulk | UC-ADM-014 | Bulk Decision / Bulk Convert | Medium |
| Import | UC-ADM-015 | Import Applications | Low |
| Export | UC-ADM-016 | Export Applicant Data (governed) | Medium |
| Workflow | UC-ADM-017 | Decision Workflow (return-for-correction) | High |
| Admin | UC-ADM-018 | Configure Capacity / Criteria / Fee Gating | High |
| Exception | UC-ADM-019 | Capacity Exceeded at Conversion | High |
| Exception | UC-ADM-020 | Duplicate / Re-Application on Terminal State | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-ADM-001 — Submit Application (idempotent)
- **Module:** Admission · **Priority:** Critical
- **Actors:** Applicant/Guardian (primary), System
- **Goal:** Capture a complete, validated application and issue a unique application number exactly once.
- **Description:** The applicant completes the configured form; on submit the system validates against the form schema and issues a unique application number, with idempotent handling of duplicate submits (double-click, retry).
- **Business Rules Applied:** ADM-001, ADM-002, CFG-010.
- **Preconditions:** Admission open for the target session/level; form schema published.
- **Trigger:** Applicant submits the application form.
- **Main Success Scenario:**
  1. Applicant completes the configured form (core + dynamic fields).
  2. System validates entries against the form schema (types, required, custom validation).
  3. System issues a unique application number and persists the application in `SUBMITTED`.
  4. System acknowledges with the application number and tracking instructions.
- **Alternative Flows:** A1) Save draft and resume later (`DRAFT`). A2) Required documents uploaded (File module) as part of the form.
- **Exception Flows:** E1) Schema validation fails → reject with field-level errors, no number issued. E2) Duplicate submit (same idempotency key) → returns the original application number, no duplicate (ADM-002).
- **Validation Rules:** All schema-required fields valid; idempotent submission keyed to prevent duplicates; admission window open (ADM-001/002).
- **Permissions Required:** Public/applicant submission (or guardian portal); officer-assisted entry needs `admission.application.manage`.
- **Notifications Triggered:** `APPLICATION_RECEIVED` (application number) to applicant/guardian.
- **Audit Events Generated:** `APPLICATION_SUBMITTED` (application number).
- **Data Created:** Application record; uploaded documents.
- **Data Updated:** None (or draft → submitted).
- **Data Deleted:** None.
- **Post Conditions:** Application exists in `SUBMITTED` with a unique number; entry into the funnel.
- **Related Use Cases:** UC-ADM-002, UC-ADM-003, UC-ADM-008.
- **Acceptance Criteria:**
  - Given a valid completed form, When submitted, Then a unique application number is issued and the application is `SUBMITTED`.
  - Given a double submission, When the same submission is retried, Then the same application number is returned with no duplicate created.
  - Given a schema-invalid form, When submitted, Then it is rejected with field errors and no number is issued.
- **Edge Case Analysis:**
  - *Invalid Input:* missing required/invalid custom field → field-level rejection.
  - *Permission Failure:* officer-assisted entry without permission → 403.
  - *Concurrent Update:* simultaneous duplicate submits → idempotency yields one application.
  - *Duplicate Data:* same applicant applying twice deliberately → allowed as distinct applications unless policy dedups (ties to STU-003 at conversion).
  - *System Failure:* submit transactional; partial submit never issues a number.
  - *Workflow Failure:* N/A (decision workflow starts later).
- **QA Coverage:**
  - *Positive:* valid submit; draft-resume-submit; with documents.
  - *Negative:* schema-invalid; closed admission window; unauthorized assisted entry.
  - *Boundary:* exactly-minimum required fields; idempotency window edge.

### UC-ADM-003 — Admission Decision (version-pinned workflow)
- **Module:** Admission · **Priority:** Critical
- **Actors:** Admissions Officer (primary), Approver, System
- **Goal:** Decide an application under the rules in force when it was submitted (fairness).
- **Description:** The decision runs on a version-pinned workflow so mid-cycle policy/criteria changes never alter in-flight applications; outcomes are offer / waitlist / reject.
- **Business Rules Applied:** ADM-003, ADM-006, WFL-002, WFL-004.
- **Preconditions:** Application in a reviewable state; decision workflow published; evaluation done (UC-ADM-002).
- **Trigger:** Officer advances the application to decision.
- **Main Success Scenario:**
  1. Officer initiates the decision; the workflow instance pins the workflow version (and applicable criteria version).
  2. The workflow resolves approver(s) with SoD (requester ≠ approver).
  3. Approver records offer / waitlist / reject with rationale.
  4. System sets the application state accordingly and notifies the applicant.
- **Alternative Flows:** A1) Return-for-correction routes back to the applicant (bounded rounds, UC-ADM-017). A2) Conditional offer pending documents/fee.
- **Exception Flows:** E1) Approver unavailable/timeout → escalation (never silent auto-approve). E2) Criteria/policy changed mid-cycle → in-flight application keeps its pinned version (ADM-003).
- **Validation Rules:** Pinned version governs; SoD enforced; decision rationale recorded (ADM-003, WFL-002/004).
- **Permissions Required:** `admission.application.review`; decision needs `admission.decision.approve`.
- **Notifications Triggered:** `ADMISSION_OFFER` / `ADMISSION_WAITLISTED` / `ADMISSION_REJECTED` to applicant/guardian.
- **Audit Events Generated:** `ADMISSION_DECISION` (outcome, pinned version, approver), workflow transitions.
- **Data Created:** Decision record; workflow instance.
- **Data Updated:** Application state.
- **Data Deleted:** None.
- **Post Conditions:** Application has a decision; applicant notified; full audit with pinned version.
- **Related Use Cases:** UC-ADM-002, UC-ADM-004, UC-ADM-017.
- **Acceptance Criteria:**
  - Given an application submitted under criteria v2, When criteria change to v3 mid-cycle, Then this application is still decided under v2.
  - Given a decision step, When the officer who reviewed is the only approver, Then SoD prevents self-approval and routes to another authority.
  - Given a decision, When recorded, Then the applicant is notified and the outcome is audited with the pinned version.
- **Edge Case Analysis:**
  - *Invalid Input:* decision without rationale (if required) blocked.
  - *Permission Failure:* review without decision authority cannot approve.
  - *Concurrent Update:* two approvers acting → workflow idempotent; one decision.
  - *Duplicate Data:* re-deciding a decided application blocked (terminal/locked).
  - *System Failure:* decision recorded transactionally with audit (outbox).
  - *Workflow Failure:* approver vacancy/timeout → escalate; no auto-approve (WFL-007).
- **QA Coverage:**
  - *Positive:* offer; waitlist; reject; conditional offer.
  - *Negative:* self-approval (SoD); mid-cycle criteria change (pinned); decide-twice.
  - *Boundary:* decision exactly at workflow timeout (escalation triggers).

### UC-ADM-005 — Convert Approved Applicant → Student + Enrollment
- **Module:** Admission · **Priority:** Critical
- **Actors:** Admissions Officer (primary), System
- **Goal:** Turn an accepted offer into a real Student and an active Enrollment, atomically, respecting the section hard cap.
- **Description:** On acceptance (and fee, if gated), the system creates/links a Student (dedup-checked, immutable ID) and an Enrollment in the target section instance — re-checking the section's hard capacity at the moment of conversion (Conflict C-01).
- **Business Rules Applied:** ADM-007, ADM-004, STU-001, STU-003, STU-006, ENR-001, ENR-002, ENR-003.
- **Preconditions:** Application `OFFER_ACCEPTED`; admission fee satisfied if gated (ADM-006); target section instance exists.
- **Trigger:** Officer (or automated step) converts the accepted application.
- **Main Success Scenario:**
  1. System runs duplicate-student detection (STU-003); links to an existing student or creates a new one with an immutable ID (STU-001).
  2. System enforces mandatory guardian linkage for minors (STU-006).
  3. System **re-checks the section hard cap** (ENR-003); if a seat is available, it creates the Enrollment (ENR-001) ensuring one active enrollment per session (ENR-002).
  4. System assigns a roll number (ENR-004) and sets the application to `CONVERTED`.
  5. The student becomes `ACTIVE` for the session.
- **Alternative Flows:** A1) Returning student → re-admission links to the existing student record (ENR-008-style). A2) Bulk conversion of a cohort (UC-ADM-014).
- **Exception Flows:** E1) Section full at conversion (offer buffer over-filled) → conversion blocked; applicant routed to waitlist/another section (UC-ADM-019, Conflict C-01). E2) Duplicate student detected → merge/link rather than create (STU-003). E3) Minor without guardian → block until a guardian is linked (STU-006).
- **Validation Rules:** Offer accepted; fee satisfied if gated; section seat available at conversion (hard cap wins); dedup performed; guardian present for minors (ADM-004/006/007, ENR-003, STU-006).
- **Permissions Required:** `admission.convert` / `enrollment.manage`.
- **Notifications Triggered:** `ENROLLMENT_CONFIRMED` to guardian; welcome/onboarding notice.
- **Audit Events Generated:** `APPLICATION_CONVERTED`, `STUDENT_CREATED`/`STUDENT_LINKED`, `ENROLLMENT_CREATED`.
- **Data Created:** Student (if new); Enrollment; roll number.
- **Data Updated:** Application → `CONVERTED`; student status → `ACTIVE`; section active count.
- **Data Deleted:** None.
- **Post Conditions:** Student exists with an active enrollment; section count consistent with the hard cap; application terminal-converted.
- **Related Use Cases:** UC-ADM-004, UC-ADM-019, UC-STU-001, UC-ENR-001.
- **Acceptance Criteria:**
  - Given an accepted offer with an available seat, When converted, Then a Student and an active Enrollment are created atomically and the application is `CONVERTED`.
  - Given offers over-filled the section (buffer), When more accept than seats exist, Then conversion respects the hard cap and excess applicants are waitlisted/rerouted (not over-enrolled).
  - Given a returning applicant matching an existing student, When converted, Then the existing student is linked (no duplicate).
  - Given a minor without a guardian, When conversion is attempted, Then it is blocked until a guardian is linked.
- **Edge Case Analysis:**
  - *Invalid Input:* conversion of a non-accepted application blocked.
  - *Permission Failure:* lacks `admission.convert` → 403.
  - *Concurrent Update:* two conversions racing for the last seat → only one succeeds (hard-cap enforcement); the other is rerouted.
  - *Duplicate Data:* duplicate student prevented by STU-003; re-running conversion is idempotent (no second enrollment, ENR-002).
  - *System Failure:* atomic Student+Enrollment creation; no orphan student without enrollment or vice versa.
  - *Workflow Failure:* if fee gating is pending, conversion waits (not partial).
- **QA Coverage:**
  - *Positive:* new-student conversion; returning-student link; bulk cohort convert.
  - *Negative:* full section; non-accepted application; minor without guardian; duplicate student.
  - *Boundary:* the last available seat under concurrent conversion; buffer exactly equal to declines.

---

## 7. Compact Specifications (routine use cases)

- **UC-ADM-002 — Evaluate Application (criteria/merit)** · *High* · Rules: ADM-003. Officer scores/ranks against configured criteria (version-pinned). *Permissions:* `admission.application.review`. *Audit:* evaluation recorded. *Edge:* criteria version pinned at submission; incomplete docs flagged. *QA:* scoring correctness; pinned criteria; missing-doc handling.
- **UC-ADM-004 — Accept Offer & Pay Admission Fee** · *High* · Rules: ADM-006, PAY. Applicant accepts; admission fee gating enforced before conversion. *Notifications:* `OFFER_ACCEPTED`, payment receipt. *Edge:* unpaid gated fee blocks conversion; offer expiry. *QA:* accept+pay; unpaid block; expired offer.
- **UC-ADM-006 — Waitlist Management & Promotion** · *High* · Rules: ADM-005, ADM-004. Maintain ordered waitlist; promote when a seat frees (hard cap). *Audit:* `WAITLIST_PROMOTED`. *Edge:* promotion respects hard cap; ordering integrity. *QA:* promote on vacancy; ordering; no over-promotion.
- **UC-ADM-007 — Configure Application Form (schema)** · *High* · Rules: ADM-001, CFG-010. Admin defines core+dynamic fields, required docs, validations (versioned). *Permissions:* `admission.config.manage`. *Edge:* schema change versioned; historical applications keep their schema version. *QA:* valid schema; invalid field def; version fidelity.
- **UC-ADM-008 — View / Track Application** · *Medium* · Rules: AUTHZ-003, GRD-N-004. Applicant/guardian tracks own application; officer sees scoped applications. *Edge:* guardian sees only their applicant. *QA:* own-only visibility; officer scope.
- **UC-ADM-009 — Update Application (pre-submission / officer correction)** · *Medium* · Rules: ADM-001, ADM-008. Edit before submission; post-submission officer corrections governed/audited. *Edge:* terminal applications immutable. *QA:* pre-submit edit; governed correction; terminal edit blocked.
- **UC-ADM-010 — Withdraw / Decline Application** · *Medium* · Rules: ADM-008. Applicant withdraws or declines an offer → terminal. *Audit:* `APPLICATION_WITHDRAWN/DECLINED`. *Edge:* frees a seat → waitlist promotion. *QA:* withdraw; decline frees seat.
- **UC-ADM-011 — Approve / Reject Decision (SoD)** · *High* · Rules: ADM-003, AUTHZ-009, WFL-004. Approver (≠ reviewer) finalizes. *Edge:* self-approval blocked; escalation. *QA:* approval; SoD; timeout escalate.
- **UC-ADM-012 — Search / Filter Applications** · *Medium* · Rules: REP-002, AUTHZ-002. Scoped search by status/criteria/intake. *Edge:* scope-filtered. *QA:* scoped results; filter correctness.
- **UC-ADM-013 — Admission Funnel & Intake Report** · *Medium* · Rules: REP-002. Funnel metrics (submitted→offered→converted), conversion rates. *Edge:* as-of freshness; scoped. *QA:* funnel counts match audit.
- **UC-ADM-014 — Bulk Decision / Bulk Convert** · *Medium* · Rules: ADM-003, ADM-007, ENR-003. Batch decide/convert a cohort, hard-cap enforced per seat. *Edge:* partial-failure report; per-seat capacity. *QA:* clean batch; capacity overflow; idempotent re-run.
- **UC-ADM-015 — Import Applications** · *Low* · Rules: ADM-001, ADM-002. Import applications (schema-validated, idempotent). *Edge:* duplicate/invalid rows reported. *QA:* clean import; duplicates; invalid rows.
- **UC-ADM-016 — Export Applicant Data (governed)** · *Medium* · Rules: REP-005, STU-005. Governed, scoped export; sensitive/minor data controlled. *Permissions:* `report.export` (elevated for bulk PII). *Edge:* bulk PII gated/audited. *QA:* scoped export; unauthorized denied; PII masked.
- **UC-ADM-017 — Decision Workflow (return-for-correction)** · *High* · Rules: ADM-003, WFL-011. Bounded send-back loop for corrections. *Edge:* round limit; escalates on breach. *QA:* return-correct-resubmit; loop bound.
- **UC-ADM-018 — Configure Capacity / Criteria / Fee Gating** · *High* · Rules: ADM-004, ADM-005, ADM-006, CFG-004. Set intake capacity, buffer (offer-stage), criteria, fee gating. *Edge:* buffer is offer-stage only (C-01). *QA:* config applies; buffer vs hard-cap semantics.
- **UC-ADM-019 — Capacity Exceeded at Conversion (Exception)** · *High* · Rules: ADM-004, ENR-003 (Conflict C-01). Conversion blocked when section full; reroute/waitlist. *QA:* hard cap wins; reroute path.
- **UC-ADM-020 — Duplicate / Re-Application on Terminal State (Exception)** · *High* · Rules: ADM-008, STU-003. Re-application is a new application; terminal ones immutable; dedup at conversion. *QA:* re-apply creates new; terminal immutable; dedup link.

## 8. Module-level QA & Edge Themes
- **Fairness (pinned versions):** the headline negative suite — mid-cycle criteria/workflow changes must never affect in-flight applications (ADM-003 / WFL-002).
- **Capacity discipline (C-01):** offer-stage buffer vs enrollment hard cap is the critical boundary; test concurrent conversions for the last seat.
- **Conversion integrity:** atomic Student+Enrollment, dedup (STU-003), and mandatory guardian for minors (STU-006) — no orphans, no duplicates, no unguarded minors.
- **Idempotency:** submission (ADM-002) and conversion must tolerate retries without duplication.
