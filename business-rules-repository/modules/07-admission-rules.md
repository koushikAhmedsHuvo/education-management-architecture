# 07 — Admission Business Rules

## 1. Module Purpose
Govern **admission** — the process by which a person (applicant) becomes a student. This is the first real consumer of the Workflow Engine (approval), the Configuration Engine (application forms are dynamic), and the File service (documents). Admission targets a specific academic session (which may be a future/admission-open session running alongside the current one, SESS-002) and ends by materializing a student record (Doc 06) and an enrollment (Doc 08). Admission is where data quality, fairness, and auditability are established for everything downstream.

## 2. Actors
- **Applicant / Guardian** — submits an application (often a guardian on behalf of a minor).
- **Admission Officer** — reviews, requests corrections, and processes applications.
- **Approver(s)** — role/committee approving admission per the workflow definition.
- **Accountant** — handles admission fee where applicable.
- **System** — enforces application validity, workflow, fairness controls, and conversion to student/enrollment.

## 3. Use Cases

**Use Case ID:** UC-ADM-01 — Submit an Application
**Actors:** Applicant/Guardian, System
**Description:** A prospective student's application is captured against a configurable form for a target session/level.
**Preconditions:** Admissions are open for the target session/level; the application form is configured.
**Main Flow:** 1) Applicant completes the dynamic application form. 2) Uploads required documents. 3) System validates fields and required documents. 4) Application enters `SUBMITTED` with a unique application number.
**Alternative Flow:** A1) Save as `DRAFT` and resume later. A2) Application fee required before submission acceptance (ADM-006).
**Exception Flow:** E1) Missing required field/document → block submission with specifics. E2) Admissions closed for the target → reject.
**Post Conditions:** Application recorded, numbered, audited; queued for review.
**Business Rules Applied:** ADM-001, ADM-002, ADM-006.

**Use Case ID:** UC-ADM-02 — Review & Decide
**Actors:** Admission Officer, Approver(s), System
**Description:** The application is reviewed and decided via the configured workflow.
**Preconditions:** Application `SUBMITTED`; reviewer authorized.
**Main Flow:** 1) Officer reviews; verifies documents. 2) Workflow routes through configured approval steps (single/committee). 3) Decision: `APPROVED` / `REJECTED` / `WAITLISTED` / `RETURNED_FOR_CORRECTION`.
**Alternative Flow:** A1) Return for correction → applicant updates → re-review. A2) Waitlist → later promotion to approved as seats free (ADM-005).
**Exception Flow:** E1) Capacity exceeded for the level → block approval or waitlist (ADM-004). E2) Duplicate applicant detected → flag (links to STU-003).
**Post Conditions:** Decision recorded with full audit and version-pinned workflow.
**Business Rules Applied:** ADM-003, ADM-004, ADM-005, ADM-007.

**Use Case ID:** UC-ADM-03 — Convert Approved Application to Student + Enrollment
**Actors:** Admission Officer, System
**Description:** An approved, fee-cleared application becomes a student record and a session enrollment.
**Preconditions:** Application `APPROVED`; admission fee settled (if required); guardian linkage available for minors.
**Main Flow:** 1) System materializes a student record (Doc 06) with a unique student ID. 2) Establishes guardian links (Doc 09). 3) Creates the enrollment in the target session/level (Doc 08). 4) Application → `ENROLLED` (terminal-success).
**Exception Flow:** E1) Minor without guardian link → block conversion (STU-006). E2) Unpaid mandatory admission fee → block (ADM-006). E3) Duplicate confirmed → resolve before conversion.
**Post Conditions:** Student + enrollment exist; application closed successfully; audited end-to-end.
**Business Rules Applied:** ADM-007, ADM-008, STU-006, ENR rules.

## 4. Business Rules

**Rule ID:** ADM-001
**Rule Name:** Configurable Application Form
**Description:** The admission application is a dynamic, per-institute configurable form, validated against its definitions.
**Priority:** High
**Category:** Configuration
**Preconditions:** Application capture.
**Business Rule:** The form (fields, required documents, validation, conditional sections) is defined via the Configuration Engine and may differ by institute/level/session. The same definition drives applicant UI and server validation.
**System Action:** Render and validate from the form definition; version-stamp the application against the form version used.
**Validation:** All required fields/documents present and valid per definition.
**Failure Behavior:** Block submission listing missing/invalid items.
**Audit Requirement:** Log `APPLICATION_SUBMITTED` with form version.
**Example Scenario:** A college's application asks for prior transcripts; a school's asks for a birth certificate — both configured, both validated.
**Related Rules:** ADM-002, CFG, STU-002.

**Rule ID:** ADM-002
**Rule Name:** Unique Application Number & Idempotent Submission
**Description:** Each application gets a unique number; duplicate/double submissions are prevented.
**Priority:** High
**Category:** Identity / integrity
**Preconditions:** Submission.
**Business Rule:** A unique application number (configurable scheme) is assigned on submission; the system prevents accidental double-submission of the same draft and detects re-applications.
**System Action:** Assign number atomically; idempotency on submit.
**Validation:** Uniqueness; one active application per applicant per target (configurable).
**Failure Behavior:** Reject duplicate submissions of the same draft.
**Audit Requirement:** Captured in `APPLICATION_SUBMITTED`.
**Example Scenario:** A double-click does not create two applications.
**Related Rules:** ADM-001, STU-003.

**Rule ID:** ADM-003
**Rule Name:** Admission Decision via Version-Pinned Workflow
**Description:** Every admission decision flows through the configured approval workflow, pinned to the workflow version in effect at submission.
**Priority:** Critical
**Category:** Workflow / fairness
**Preconditions:** Application `SUBMITTED`.
**Business Rule:** Decisions (approve/reject/waitlist/return) occur only through the workflow; the workflow version is pinned at submission so a mid-cycle policy change does not retroactively alter in-flight applications (fairness + WFL version-pinning).
**System Action:** Drive the FSM; record each step, decision, and actor.
**Validation:** Workflow definition valid; reviewer authorized; step order respected.
**Failure Behavior:** Out-of-band decisions rejected.
**Audit Requirement:** Log `APPLICATION_DECISION` per step with actor, authority, workflow version.
**Example Scenario:** Applications submitted under last month's criteria are decided under those criteria even after the policy updates.
**Related Rules:** ADM-005, WFL (Doc 28).

**Rule ID:** ADM-004
**Rule Name:** Capacity & Seat Control
**Description:** Admissions respect configurable seat capacity per level/section/session.
**Priority:** High
**Category:** Capacity
**Preconditions:** Approval/conversion against a level with a capacity limit.
**Business Rule:** When configured, the system enforces seat limits; approvals beyond capacity are blocked or routed to waitlist. Capacity counts confirmed enrollments, not mere approvals, to avoid over-commit (a configurable buffer is allowed).
**System Action:** Check live confirmed-enrollment count against capacity at approval/conversion.
**Validation:** Capacity defined; count current.
**Failure Behavior:** Block over-capacity approval or auto-waitlist per policy.
**Audit Requirement:** Log `CAPACITY_BLOCK` / `WAITLISTED` decisions.
**Example Scenario:** A class capped at 40 sends the 41st approved applicant to the waitlist.
**Related Rules:** ADM-005, SEC (section capacity, Doc 12).

**Rule ID:** ADM-005
**Rule Name:** Waitlist Management
**Description:** Waitlisted applicants are ordered and promoted fairly as seats free.
**Priority:** Medium
**Category:** Fairness
**Preconditions:** Capacity full; qualified applicants remain.
**Business Rule:** The waitlist is ordered by a configurable, transparent criterion (merit, application time, or policy); promotion to approved follows that order when a seat opens, with audit.
**System Action:** Maintain ordered waitlist; promote next on seat availability.
**Validation:** Ordering criterion defined; promotion respects order.
**Failure Behavior:** Out-of-order promotion blocked (or requires justified override).
**Audit Requirement:** Log `WAITLIST_PROMOTED` with the basis.
**Example Scenario:** When a student declines, the top waitlisted applicant is offered the seat.
**Related Rules:** ADM-004, ADM-003.

**Rule ID:** ADM-006
**Rule Name:** Admission Fee Gating
**Description:** Where an admission/application fee is required, it gates the relevant step.
**Priority:** High
**Category:** Finance integration
**Preconditions:** A fee is configured for application submission or admission confirmation.
**Business Rule:** Configurable per institute: an application fee may gate submission, and/or an admission fee may gate conversion to enrollment. Fee records integrate with Finance (Docs 17–18); waivers/discounts apply via their rules (Docs 19–20).
**System Action:** Generate the fee; block the gated step until settled or waived.
**Validation:** Fee configured; payment/waiver recorded.
**Failure Behavior:** Block the gated step on non-payment; never silently enroll unpaid where the fee is mandatory.
**Audit Requirement:** Log `ADMISSION_FEE_REQUIRED/SETTLED/WAIVED`.
**Example Scenario:** A student cannot be enrolled until the (non-waived) admission fee is paid.
**Related Rules:** ADM-007, FEE/PAY/DSC (Docs 17–20).

**Rule ID:** ADM-007
**Rule Name:** Conversion Integrity (Approved → Student + Enrollment)
**Description:** Conversion atomically creates the student and enrollment, with all preconditions met.
**Priority:** Critical
**Category:** Integrity
**Preconditions:** Application `APPROVED`; fee cleared; guardian linked (minor); seat available.
**Business Rule:** Conversion is transactional: it creates the student record (unique ID), establishes guardian links, and creates the enrollment in the target session/level — or it does none of these. No half-converted state.
**System Action:** Run conversion in one transaction; on any precondition failure, abort cleanly.
**Validation:** All preconditions (ADM-006, STU-006, ADM-004) satisfied.
**Failure Behavior:** Abort and report the unmet precondition; application stays `APPROVED` (not lost).
**Audit Requirement:** Log `APPLICATION_CONVERTED` linking application → student ID → enrollment.
**Example Scenario:** A minor's approved application converts to a student + enrollment only after a guardian is linked and the fee is paid.
**Related Rules:** STU-001, STU-006, ENR, ADM-006.

**Rule ID:** ADM-008
**Rule Name:** Terminal-State Immutability & Re-Application
**Description:** Decided applications are immutable; a new attempt is a new application.
**Priority:** Medium
**Category:** Integrity
**Preconditions:** Application reaches a terminal state (`ENROLLED`/`REJECTED`/`WITHDRAWN`/`EXPIRED`).
**Business Rule:** Terminal applications are read-only; an applicant who wants to try again files a new application (linked for history), preserving the original decision record.
**System Action:** Lock terminal applications; create new applications for re-attempts.
**Validation:** No edits to terminal applications.
**Failure Behavior:** Edits to terminal applications rejected.
**Audit Requirement:** Log re-application linkage.
**Example Scenario:** A rejected applicant re-applies next cycle under a new application number, with history preserved.
**Related Rules:** ADM-002, STU-003.

## 5. Validation Rules
- Application valid against the form definition (required fields/documents present).
- Admissions open for the target session/level at submission.
- One active application per applicant per target (configurable).
- Decisions only via the version-pinned workflow.
- Capacity respected at approval/conversion.
- Conversion preconditions (fee, guardian-for-minor, seat) all satisfied — transactional.

## 6. State Machine

**State Name:** DRAFT
**Description:** Application started, not submitted.
**Allowed Transitions:** → SUBMITTED; → EXPIRED (abandoned); → WITHDRAWN.
**Forbidden Transitions:** → APPROVED.
**System Actions:** Hold incomplete data; no number assigned until submit (or assign provisional).

**State Name:** SUBMITTED
**Description:** Complete application awaiting review.
**Allowed Transitions:** → UNDER_REVIEW; → RETURNED_FOR_CORRECTION; → WITHDRAWN.
**Forbidden Transitions:** → ENROLLED (must be approved first).
**System Actions:** Assign number; enter workflow; lock applicant-edited fields except on return.

**State Name:** UNDER_REVIEW
**Description:** In the approval workflow.
**Allowed Transitions:** → APPROVED; → REJECTED; → WAITLISTED; → RETURNED_FOR_CORRECTION.
**Forbidden Transitions:** → SUBMITTED.
**System Actions:** Route per workflow; record each step.

**State Name:** RETURNED_FOR_CORRECTION
**Description:** Sent back to the applicant for fixes.
**Allowed Transitions:** → SUBMITTED (resubmitted); → EXPIRED; → WITHDRAWN.
**Forbidden Transitions:** → APPROVED (must be re-reviewed).
**System Actions:** Unlock specified fields; track correction round.

**State Name:** WAITLISTED
**Description:** Qualified but no seat.
**Allowed Transitions:** → APPROVED (seat opens); → REJECTED/EXPIRED.
**Forbidden Transitions:** → ENROLLED directly (must be approved).
**System Actions:** Maintain ordered position.

**State Name:** APPROVED
**Description:** Accepted; pending conversion (fee/guardian/seat).
**Allowed Transitions:** → ENROLLED (conversion); → WITHDRAWN; → EXPIRED (offer lapses).
**Forbidden Transitions:** → REJECTED (decision already made; reversal is governed/audited).
**System Actions:** Generate admission fee if configured; await conversion preconditions.

**State Name:** ENROLLED
**Description:** Converted to student + enrollment (terminal success).
**Allowed Transitions:** none (terminal).
**Forbidden Transitions:** edits.
**System Actions:** Link application → student → enrollment; lock.

**State Name:** REJECTED / WITHDRAWN / EXPIRED
**Description:** Terminal non-success states.
**Allowed Transitions:** none (re-attempt = new application).
**Forbidden Transitions:** edits.
**System Actions:** Lock; preserve decision record.

## 7. Status Definitions
`DRAFT` · `SUBMITTED` · `UNDER_REVIEW` · `RETURNED_FOR_CORRECTION` · `WAITLISTED` · `APPROVED` · `ENROLLED` (terminal success) · `REJECTED` · `WITHDRAWN` · `EXPIRED` (terminal non-success).

## 8. Workflow Rules
- The admission approval workflow is configurable (single officer, committee, multi-step) and version-pinned at submission (ADM-003).
- Escalation/timeout rules apply to stalled reviews (Workflow Engine, Doc 28).
- SoD: where configured, the reviewer cannot also be the sole final approver of the same application.
- Capacity and fee gates are enforced at the approval/conversion steps, not bypassable by workflow shortcuts.

## 9. Permission Rules
- `admission.application.submit` — create/submit (applicant/guardian/officer).
- `admission.application.review` — review and progress applications.
- `admission.application.decide` — approve/reject/waitlist (per workflow role).
- `admission.application.convert` — perform conversion to student/enrollment.
- All scoped to institute/campus; reviewers see only their institute's applications.

## 10. Notification Rules
- `APPLICATION_SUBMITTED` → acknowledge to applicant/guardian with the application number.
- `RETURNED_FOR_CORRECTION` → notify applicant with required fixes.
- `APPLICATION_DECISION` (approved/rejected/waitlisted) → notify applicant/guardian.
- `WAITLIST_PROMOTED` → notify the promoted applicant with an offer deadline.
- `ADMISSION_FEE_REQUIRED` → notify with payment instructions.
- `APPLICATION_CONVERTED` → welcome the new student/guardian.

## 11. Audit Requirements
Mandatory: `APPLICATION_SUBMITTED` (with form version), `APPLICATION_DECISION` (per step, with workflow version + actor + authority), `RETURNED_FOR_CORRECTION`, `WAITLISTED`/`WAITLIST_PROMOTED`, `CAPACITY_BLOCK`, `ADMISSION_FEE_REQUIRED/SETTLED/WAIVED`, `APPLICATION_CONVERTED` (application→student→enrollment), terminal transitions. Decisions must be fully attributable for fairness/appeals.

## 12. Data Retention Rules
- Applications (including rejected/withdrawn) retained per policy — needed for fairness audits, appeals, and re-application linkage.
- Uploaded admission documents follow the file-retention class (minors-stricter); purged per retention with export-before-purge.
- Decision records retained long enough to defend admissions fairness (configurable, often multi-year).
- Converted applications link permanently to the resulting student for provenance.

## 13. Edge Cases
- **Guardian applies for a minor:** the guardian is the submitter; the minor is the subject — captured distinctly (the applicant ≠ the account holder).
- **Re-application after rejection:** new application, linked to prior history; original decision immutable (ADM-008).
- **Approved-but-never-converted:** the offer can expire; the seat is released back to capacity/waitlist.
- **Admissions to a future session while current runs:** applications target the future session explicitly (SESS-002) — operations must not confuse target vs current.
- **Document fraud / mismatch:** flagged at review; conversion blocked pending verification.
- **Duplicate applicant who is an existing student:** detected (STU-003); may be a re-enrollment, not a new student.
- **Capacity counted on approvals vs confirmed enrollments:** count confirmed to avoid phantom over-commit; configurable buffer for expected declines (ADM-004).
- **Fee paid but conversion fails (guardian missing):** fee is recorded; conversion retried after the guardian is added — money is never lost or orphaned.

## 14. Failure Scenarios
- **Conversion precondition fails mid-transaction:** atomic abort; application stays `APPROVED`; nothing half-created (ADM-007).
- **Workflow misconfiguration (no approver resolvable):** application holds in `UNDER_REVIEW`; escalation/alert rather than silent approval.
- **Payment gateway delay:** application waits in `APPROVED`; conversion proceeds on confirmed settlement.
- **Form definition changed mid-cycle:** in-flight applications keep their submitted form version (ADM-001); new applications use the new version.

## 15. Exception Handling Rules
- Submissions failing validation are rejected with specific missing/invalid items.
- Out-of-band or out-of-order decisions are rejected; all decisions flow through the workflow.
- Reversal of a terminal decision is a governed, audited exception, never a casual edit.
- Conversion is all-or-nothing; partial conversions are impossible by design.

## 16. Compliance Considerations
- **Fairness & non-discrimination:** transparent, ordered, fully-audited decisions defend against bias claims; version-pinning ensures consistent criteria within a cycle.
- **Minors & guardian consent:** guardian-submitted applications and mandatory guardian linkage reflect child-data protection.
- **Document handling:** admission documents (often identity documents of minors) are private, scanned, signed-URL-gated, retention-bound.
- **Right to explanation/appeal:** retained decision records support appeals and regulatory review.

## 17. Future Considerations
- Online public admission portal with payment-gateway integration (R4).
- Merit-list automation and entrance-test scoring feeding waitlist order.
- Cross-institute admission within a deployment (apply once, consider multiple institutes).
- Anti-fraud document verification (automated checks) with strong audit.
