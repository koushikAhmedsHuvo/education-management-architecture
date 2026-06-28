# 22 — Staff Business Rules

> **Prefix note:** To keep rule IDs unique and testable, the HR-phase modules use distinct prefixes — Staff `STF`, Teacher `TCH`, HR/Payroll `HR`, Leave `LEV` — refining the README's grouped "HR" label.

## 1. Module Purpose
Govern the **staff** record — the canonical person-of-the-institution employment record on which Teacher (Doc 21), HR processes (Doc 23), and Leave (Doc 24) all build. Staff owns employee identity, employment details (designation, department, type, reporting line), institute/campus scoping, the staff↔user-account link for system access, and the lifecycle from onboarding to separation. A **Teacher is a Staff member with teaching assignments**; this module is the base, Doc 21 is the specialization. Staff data includes sensitive PII (salary, national ID) and is protected accordingly.

## 2. Actors
- **HR Administrator** — creates and manages staff records, employment details, reporting lines.
- **Institute / Campus Administrator** — manages staff within their scope.
- **Reporting Manager** — the staff member a report's leave/approvals route to.
- **Staff Member** — reads/maintains permitted parts of their own record.
- **System** — enforces uniqueness, scoping, the staff↔user link, lifecycle, and sensitive-data protection.

## 3. Use Cases

**Use Case ID:** UC-STF-01 — Create a Staff Record & Link a User Account
**Actors:** HR Administrator, System
**Description:** Onboard a staff member as a record with system access.
**Preconditions:** Actor holds `staff.record.manage`; institute scope set.
**Main Flow:** 1) Capture identity + employment (designation, department, type, join date, reporting manager). 2) System assigns a unique employee ID and links a user account (invitation-based activation, AUTH-011). 3) Assign role/memberships (Doc 02) per the staff member's function. 4) Record enters onboarding/probation.
**Alternative Flow:** A1) Bulk staff import, each validated and ID-assigned.
**Exception Flow:** E1) Duplicate (same national ID) → flag for review (STF-003). E2) Reporting manager out of scope → reject.
**Post Conditions:** Staff record + linked account exist, scoped, audited.
**Business Rules Applied:** STF-001, STF-002, STF-003, STF-006.

**Use Case ID:** UC-STF-02 — Manage Assignments & Multi-Campus Service
**Actors:** HR/Institute Administrator, System
**Description:** Assign a staff member to departments/campuses, possibly more than one.
**Preconditions:** Staff active; targets valid in scope.
**Main Flow:** 1) Assign department/designation and campus(es). 2) For multi-campus service, create explicit multi-assignments (no silent cross-campus access). 3) Assignments drive scoped access and (for teachers) ownership.
**Exception Flow:** E1) Cross-institute assignment → separate employment, not a silent span (STF-005).
**Post Conditions:** Assignments recorded; scoped access resolved; audited.
**Business Rules Applied:** STF-005, STF-007.

**Use Case ID:** UC-STF-03 — Suspend / Separate a Staff Member
**Actors:** HR Administrator, System
**Description:** Suspend or offboard a staff member, revoking access and preserving history.
**Preconditions:** Actor holds `staff.lifecycle.manage`; reason recorded.
**Main Flow:** 1) Set suspension/separation with reason and effective date. 2) System revokes system access (AUTH-005), reassigns or flags open duties (teaching assignments, approvals). 3) Record moves to suspended/separated; history preserved.
**Exception Flow:** E1) Separation with open ownership (active sections/pending approvals) → require reassignment/handover (STF-008). E2) Hard-delete attempt → forbidden (STF-008).
**Post Conditions:** Access revoked; duties handed over; record retained; audited.
**Business Rules Applied:** STF-004, STF-008.

## 4. Business Rules

**Rule ID:** STF-001
**Rule Name:** Unique Employee Identifier
**Description:** Each staff member has a unique, configurable, immutable employee ID, distinct from any government ID.
**Priority:** Critical
**Category:** Identity
**Preconditions:** Staff creation.
**Business Rule:** A unique employee ID (configurable scheme) is assigned at creation, unique within the deployment, never reused or edited.
**System Action:** Generate per scheme atomically; lock.
**Validation:** Uniqueness; format valid.
**Failure Behavior:** Reject on collision; no reuse of retired IDs.
**Audit Requirement:** Log `STAFF_CREATED` with employee ID.
**Example Scenario:** "EMP-2026-0042" identifies a staff member permanently, including after separation.
**Related Rules:** STF-003, STF-008.

**Rule ID:** STF-002
**Rule Name:** Staff–User Account Linkage
**Description:** A staff member who needs system access is linked to exactly one user account; identity and employment are distinct but bound.
**Priority:** High
**Category:** Access integration
**Preconditions:** Staff needs system access.
**Business Rule:** The staff record links to one user account (Doc 01); authentication is via that account, authorization via scoped memberships (Doc 02). Separation revokes the account's access (AUTH-005) but the staff history persists.
**System Action:** Create/link the account via invitation-based activation; bind lifecycle.
**Validation:** One active account per staff for access; activation per AUTH-011.
**Failure Behavior:** Block duplicate accounts for one staff member.
**Audit Requirement:** Log `STAFF_ACCOUNT_LINKED`.
**Example Scenario:** A teacher's record links to their login; resigning disables the login while the record remains.
**Related Rules:** AUTH-011, AUTH-005, AUTHZ-007.

**Rule ID:** STF-003
**Rule Name:** Duplicate-Staff Detection
**Description:** The system surfaces likely duplicates before creating a redundant staff record.
**Priority:** High
**Category:** Data integrity
**Preconditions:** Staff creation/import.
**Business Rule:** On creation, check for likely duplicates (national ID, or name + DOB) within the deployment; flag for review rather than silently duplicating or blocking (handles rehire correctly).
**System Action:** Run the duplicate heuristic; require confirm-as-new or link to existing.
**Validation:** Heuristic fields present; reviewer decision recorded.
**Failure Behavior:** Proceed only on explicit confirmation; never auto-merge.
**Audit Requirement:** Log `STAFF_DUPLICATE_FLAGGED` and resolution.
**Example Scenario:** A rehired former employee matches an existing record; HR is prompted before a second record is created.
**Related Rules:** STF-001, HR-008 (rehire).

**Rule ID:** STF-004
**Rule Name:** Employment-Status Lifecycle Drives Access
**Description:** Staff status (onboarding/active/suspended/separated) governs system access and operational eligibility.
**Priority:** High
**Category:** Lifecycle integrity
**Preconditions:** A status change.
**Business Rule:** Only `ACTIVE` (incl. probation) staff have operational access and can hold assignments; suspension/separation revoke access (AUTH-005) and end live assignment authority while preserving history.
**System Action:** Gate access/assignment by status; re-resolve on change.
**Validation:** Transition allowed by the state machine; effective date valid.
**Failure Behavior:** Block operational actions by non-active staff.
**Audit Requirement:** Log `STAFF_STATUS_CHANGED` with reason.
**Example Scenario:** A suspended staff member cannot log in or be assigned new duties.
**Related Rules:** AUTH-005, STF-008, HR (lifecycle).

**Rule ID:** STF-005
**Rule Name:** Institute/Campus Scoping & Multi-Assignment
**Description:** Staff are scoped to an institute (and campus); cross-campus/cross-institute service requires explicit assignment.
**Priority:** High
**Category:** Scope
**Preconditions:** Staff assignment.
**Business Rule:** A staff member belongs to an institute; serving multiple campuses needs explicit multi-campus assignment (no silent span). Serving multiple institutes in the deployment is modeled as distinct employment/memberships, not one record silently crossing institutes.
**System Action:** Resolve scoped access from explicit assignments.
**Validation:** Assignments reference active in-scope targets.
**Failure Behavior:** Block silent cross-scope access.
**Audit Requirement:** Log assignment changes.
**Example Scenario:** A teacher serving two campuses has two explicit campus assignments, each scoping their access.
**Related Rules:** CAMP-004, INST-006, TCH-002.

**Rule ID:** STF-006
**Rule Name:** Reporting Hierarchy
**Description:** Each staff member (except the top) has a reporting manager, driving approvals, escalation, and delegation.
**Priority:** High
**Category:** Organization
**Preconditions:** Staff record with a manager.
**Business Rule:** The reporting line determines leave/approval routing (Doc 24), escalation targets, and delegation defaults; cycles in the hierarchy are forbidden.
**System Action:** Maintain an acyclic reporting graph; route approvals accordingly.
**Validation:** Manager in scope; no reporting cycle.
**Failure Behavior:** Reject cyclic/invalid reporting lines.
**Audit Requirement:** Log `REPORTING_MANAGER_CHANGED`.
**Example Scenario:** A teacher's leave routes to their head of department per the reporting line.
**Related Rules:** LEV (approval), WFL (escalation/delegation), AUTHZ-009.

**Rule ID:** STF-007
**Rule Name:** Sensitive Staff Data Protection
**Description:** Sensitive staff PII (salary, national ID, contact) is access-controlled, masked, and field-encrypted where most sensitive.
**Priority:** Critical
**Category:** Privacy
**Preconditions:** Sensitive fields stored/displayed.
**Business Rule:** Salary and national ID are field-encrypted and visible only to authorized HR/finance roles (need-to-know); masked elsewhere; access to highly sensitive fields may be audited.
**System Action:** Encrypt designated fields; mask by role; gate access.
**Validation:** Sensitivity classification applied; role need-to-know enforced.
**Failure Behavior:** Mask/deny unauthorized access.
**Audit Requirement:** Log `SENSITIVE_STAFF_FIELD_ACCESSED` where configured.
**Example Scenario:** A department head sees a teacher's contact but not their salary; payroll sees salary.
**Related Rules:** HR-005 (payroll), STU-005 (parallel), Compliance.

**Rule ID:** STF-008
**Rule Name:** Archive/Anonymize, Never Hard-Delete; Handover on Separation
**Description:** Staff records are never hard-deleted; separation requires handover of open duties and preserves history.
**Priority:** Critical
**Category:** Integrity / continuity
**Preconditions:** Separation/removal.
**Business Rule:** A staff member with history cannot be hard-deleted; they are separated/archived, access revoked, and open ownership (teaching assignments, pending approvals, delegations) reassigned/handed over so nothing is orphaned. Erasure is retention-driven anonymization with export-before-purge.
**System Action:** Require handover; revoke access; preserve history; anonymize only per retention.
**Validation:** Open duties reassigned before finalizing separation.
**Failure Behavior:** Block hard-delete; block separation leaving orphaned ownership.
**Audit Requirement:** Log `STAFF_SEPARATED/ARCHIVED/ANONYMIZED` and handover.
**Example Scenario:** A resigning teacher's sections are reassigned and pending approvals delegated before their access ends.
**Related Rules:** AUTH-005, TCH-005, HR-007, LEV.

## 5. Validation Rules
- Employee ID unique, scheme-valid, immutable.
- One active linked user account per staff for access.
- Duplicate detection on national ID before creation.
- Status governs access/assignment eligibility.
- Multi-campus/institute service via explicit assignment only.
- Reporting graph acyclic; sensitive fields protected; separation requires handover.

## 6. State Machine

**State Name:** ONBOARDING
**Description:** Record created; account activation/setup in progress.
**Allowed Transitions:** → ACTIVE (incl. probation); → WITHDRAWN (offer declined).
**Forbidden Transitions:** holding live assignments before active.
**System Actions:** Invitation activation; provisional setup.

**State Name:** ACTIVE
**Description:** Employed and operational (probation is an HR sub-state, Doc 23).
**Allowed Transitions:** → SUSPENDED; → NOTICE_PERIOD; → SEPARATED.
**Forbidden Transitions:** → ONBOARDING.
**System Actions:** Permit assignments/access; drive ownership for teachers.

**State Name:** SUSPENDED
**Description:** Temporarily disabled (investigation, policy).
**Allowed Transitions:** → ACTIVE (reinstate); → SEPARATED.
**Forbidden Transitions:** new assignments while suspended.
**System Actions:** Revoke access; retain data; pause assignments.

**State Name:** NOTICE_PERIOD
**Description:** Resignation/termination notice in progress.
**Allowed Transitions:** → SEPARATED.
**Forbidden Transitions:** → ACTIVE (withdrawal of resignation is a governed exception).
**System Actions:** Plan handover; wind down assignments.

**State Name:** SEPARATED
**Description:** Employment ended; access revoked; history retained.
**Allowed Transitions:** → ARCHIVED; → ANONYMIZED (retention).
**Forbidden Transitions:** silent reactivation (rehire is new employment).
**System Actions:** Finalize handover/settlement; revoke access.

**State Name:** ARCHIVED / ANONYMIZED
**Description:** Long-term retained / PII removed per erasure.
**Allowed Transitions:** ARCHIVED → ANONYMIZED.
**Forbidden Transitions:** edits/hard-delete (history exists).
**System Actions:** Read-only retention / PII stripping.

## 7. Status Definitions
`ONBOARDING` · `ACTIVE` · `SUSPENDED` · `NOTICE_PERIOD` · `SEPARATED` · `ARCHIVED` · `ANONYMIZED`. Employment sub-states (HR, Doc 23): `PROBATION` · `CONFIRMED`.

## 8. Workflow Rules
- Onboarding may run an approval/checklist workflow (Doc 23).
- Suspension/termination for cause is governed (approval + reason), given its impact.
- Separation enforces a handover checklist (assignments, approvals, assets) before finalization.
- Reporting-line changes re-route pending approvals appropriately.

## 9. Permission Rules
- `staff.record.manage` — create/edit staff records.
- `staff.lifecycle.manage` — suspend/separate/reinstate.
- `staff.sensitive.view` — view salary/national ID (HR/finance need-to-know).
- `staff.view` — read staff directory (scoped; staff read own record).
- Scoped to institute/campus; campus admins manage their campus's staff.

## 10. Notification Rules
- `STAFF_CREATED`/account activation → notify the staff member (onboarding).
- `STAFF_STATUS_CHANGED` (suspension/separation) → notify the staff member and their manager.
- `REPORTING_MANAGER_CHANGED` → notify affected parties.
- Sensitive-data access alerts to HR security channel where configured.

## 11. Audit Requirements
Mandatory: `STAFF_CREATED`, `STAFF_ACCOUNT_LINKED`, assignment changes, `REPORTING_MANAGER_CHANGED`, `STAFF_STATUS_CHANGED` (reason), `STAFF_SEPARATED/ARCHIVED/ANONYMIZED`, handover records, `SENSITIVE_STAFF_FIELD_ACCESSED`. With actor, employee ID, scope, before/after.

## 12. Data Retention Rules
- Employment records retained per labor-law/statutory retention (often years post-separation).
- Payroll-relevant data retained per finance/tax retention (HR, Doc 23).
- Sensitive PII minimized and erasable post-retention via anonymization, preserving non-identifying employment integrity.
- Legal-hold overrides erasure during disputes.

## 13. Edge Cases
- **Staff who is also a guardian:** their staff (role-based) access and guardian (children-only) access are separate scopes that must not bleed (links GRD-N).
- **Rehire of a former employee:** duplicate detection prevents a second master record; rehire is new employment under the existing person, not silent reactivation (STF-003/HR-008).
- **Multi-campus/institute service:** explicit assignments; access never silently spans scope (STF-005).
- **Separation with open teaching duties:** handover required; sections reassigned/substituted before access ends (STF-008/TCH-005).
- **Suspended staff with pending approvals:** approvals delegate/escalate; nothing stalls indefinitely (links LEV/WFL).
- **Contract expiry:** contract-type staff auto-flag near expiry for renewal/separation decision.
- **Staff serving as a student's reporting context (small institutions):** roles kept distinct.

## 14. Failure Scenarios
- **Hard-delete of a staff with history:** rejected; archive instead (STF-008).
- **Separation orphaning ownership:** blocked until handover (STF-008).
- **Duplicate employee ID / record:** prevented (STF-001/STF-003).
- **Cyclic reporting line:** rejected (STF-006).
- **Unauthorized sensitive-data access:** masked/denied (STF-007).

## 15. Exception Handling Rules
- Lifecycle transitions validated, transactional, and audited.
- Separation is gated on handover; no orphaned assignments/approvals.
- Sensitive data fails closed (mask/deny on doubt).
- Cross-scope access requires explicit assignment; never inferred.

## 16. Compliance Considerations
- **Labor law:** employment records and retention honor statutory requirements.
- **Data protection:** staff PII (salary/national ID) protected, minimized, erasable post-retention.
- **Continuity & accountability:** handover and retained history support audits and operational continuity.
- **Separation of personas:** staff-vs-guardian scope separation protects minors' data boundaries.

## 17. Future Considerations
- Org-chart visualization and succession planning.
- Skills/competency registry feeding assignment and substitution.
- Self-service staff profile and document management.
- Cross-institute talent sharing within a deployment (governed).
