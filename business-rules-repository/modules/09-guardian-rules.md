# 09 — Guardian Business Rules

## 1. Module Purpose
Govern the **guardian** — the responsible adult linked to a student, who receives communications, exercises parental consent for a minor, accesses the parent portal for their linked children only, and may handle fees. Because most students are minors, the guardian relationship is a **child-safety and access-control boundary**, not merely contact data. This module is the source of truth for guardian-of-record, relationship types, consent, financial responsibility, and the strict "a guardian sees only their own children" ownership rule.

## 2. Actors
- **Guardian** — the linked adult (parent, legal guardian, sponsor); accesses only their children's data.
- **Institute / Campus Administrator** — manages guardian records and links.
- **Admission Officer** — establishes guardian links during admission (Doc 07).
- **System** — enforces linkage validity, consent, ownership scoping, and custody/notification constraints.

## 3. Use Cases

**Use Case ID:** UC-GRD-01 — Link a Guardian to a Student
**Actors:** Admission Officer / Administrator, System
**Description:** Associate a responsible adult with a student, defining relationship and responsibilities.
**Preconditions:** Student exists; actor authorized; guardian identity captured.
**Main Flow:** 1) Capture/select guardian (name, relationship, contact, optional account). 2) Define responsibilities (primary contact, financial responsibility, portal access, custody flags). 3) Create the link; designate a guardian-of-record. 4) For a minor, record parental consent where required.
**Alternative Flow:** A1) Reuse an existing guardian for siblings (one guardian → many students).
**Exception Flow:** E1) Removing the last guardian of a minor → blocked (GRD-N-002). E2) Conflicting custody/contact restriction → enforce (GRD-N-006).
**Post Conditions:** Guardian link active; ownership/notification scope established; audited.
**Business Rules Applied:** GRD-N-001, GRD-N-002, GRD-N-003.

**Use Case ID:** UC-GRD-02 — Guardian Portal Access to Linked Children
**Actors:** Guardian, System
**Description:** A guardian views information for their linked children only.
**Preconditions:** Guardian has an active account and ≥1 active link with portal-access enabled.
**Main Flow:** 1) Guardian authenticates. 2) System resolves the set of linked children (ownership). 3) Guardian sees only those children's permitted data.
**Exception Flow:** E1) Attempt to access a non-linked student → denied (GRD-N-004). E2) A link revoked → access to that child ends immediately.
**Post Conditions:** Scoped access enforced; sensitive access optionally audited.
**Business Rules Applied:** GRD-N-004, GRD-N-005.

**Use Case ID:** UC-GRD-03 — Update Guardianship (Add/Remove/Change)
**Actors:** Administrator, System
**Description:** Maintain guardian links over time (new guardian, removal, custody change).
**Preconditions:** Actor authorized; change reason captured for sensitive changes.
**Main Flow:** 1) Add/remove/modify a guardian link with effective date and reason. 2) System enforces the minor-must-retain-a-guardian invariant and custody constraints. 3) Notifications and portal access re-resolve.
**Exception Flow:** E1) Change that would orphan a minor → blocked. E2) Court-ordered restriction → applied and enforced (GRD-N-006).
**Post Conditions:** Guardianship updated; access/notification re-resolved; audited with reason.
**Business Rules Applied:** GRD-N-002, GRD-N-006, GRD-N-007.

## 4. Business Rules

**Rule ID:** GRD-N-001
**Rule Name:** Guardian-of-Record Designation
**Description:** Each student has exactly one primary guardian-of-record, with optional additional guardians.
**Priority:** High
**Category:** Identity / responsibility
**Preconditions:** Guardian linkage.
**Business Rule:** Exactly one active guardian is the primary guardian-of-record (default contact and consent authority); others are secondary. Roles per guardian are explicit (primary contact, financial responsible, portal access, emergency contact) and may differ.
**System Action:** Enforce a single primary; allow multiple secondaries with explicit roles.
**Validation:** Exactly one active primary per student.
**Failure Behavior:** Reject a second primary; require demotion of the existing one first.
**Audit Requirement:** Log `GUARDIAN_LINKED` and `GUARDIAN_OF_RECORD_CHANGED`.
**Example Scenario:** A father is primary guardian-of-record; an aunt is a secondary emergency contact.
**Related Rules:** GRD-N-002, GRD-N-003.

**Rule ID:** GRD-N-002
**Rule Name:** A Minor Must Always Have ≥1 Active Guardian
**Description:** Guardian removal cannot leave a minor student with no responsible adult.
**Priority:** Critical
**Category:** Child safety
**Preconditions:** Guardian-link removal for a minor.
**Business Rule:** The system blocks any change that would leave a minor with zero active guardian links. Removing the only/last guardian requires first adding a replacement.
**System Action:** Validate post-change guardian count for minors before applying.
**Validation:** Student minor status (DOB vs majority); resulting active guardian count ≥1.
**Failure Behavior:** Block the removal; prompt to add a replacement guardian.
**Audit Requirement:** Log `GUARDIAN_REMOVAL_BLOCKED` (minor-protection).
**Example Scenario:** A guardian transfer requires the new guardian to be linked before the old one is removed.
**Related Rules:** STU-006, GRD-N-007.

**Rule ID:** GRD-N-003
**Rule Name:** One Guardian, Many Students (Sibling Reuse)
**Description:** A single guardian record links to multiple students (siblings) without duplication.
**Priority:** Medium
**Category:** Data quality
**Preconditions:** A guardian already exists and is linked to another student.
**Business Rule:** Guardians are reusable across students; siblings share one guardian record, so contact/consent updates propagate consistently and the portal shows all linked children together.
**System Action:** Link the existing guardian; do not create duplicates.
**Validation:** Guardian identity matched (avoid accidental duplicates).
**Failure Behavior:** Prompt to reuse on a likely-duplicate guardian rather than silently duplicating.
**Audit Requirement:** Log multi-student links.
**Example Scenario:** A parent of three students updates their phone once; all three records reflect it.
**Related Rules:** GRD-N-001, STU-003.

**Rule ID:** GRD-N-004
**Rule Name:** Strict Children-Only Ownership Scope
**Description:** A guardian can access only the data of students they are actively linked to.
**Priority:** Critical
**Category:** Access control (minors)
**Preconditions:** Guardian accesses student data (portal or API).
**Business Rule:** The ownership predicate restricts a guardian to their linked children only; cross-child access is impossible. Revoking a link ends access to that child immediately (permissions re-resolve).
**System Action:** Resolve linked-children set; filter every guardian-facing query to it.
**Validation:** Active link exists for each accessed student.
**Failure Behavior:** Non-linked access denied/filtered; sensitive attempts audited.
**Audit Requirement:** Log `GUARDIAN_ACCESS_DENIED` for non-linked attempts.
**Example Scenario:** A guardian cannot view a classmate of their child, only their own child.
**Related Rules:** AUTHZ-003, GRD-N-002.

**Rule ID:** GRD-N-005
**Rule Name:** Role-Scoped Guardian Capabilities
**Description:** What a guardian can do (view results, pay fees, receive which notifications) depends on their explicit roles on the link.
**Priority:** Medium
**Category:** Authorization
**Preconditions:** Guardian performs an action on a linked child.
**Business Rule:** Capabilities derive from link roles: only the financial-responsible guardian sees/pays fees by default; portal-access flag controls login; notification routing follows designated contacts. A secondary emergency contact may have read-limited access.
**System Action:** Apply per-link role capabilities.
**Validation:** Action permitted by the guardian's role on that link.
**Failure Behavior:** Out-of-role action denied.
**Audit Requirement:** Log sensitive guardian actions (e.g., fee payment).
**Example Scenario:** A sponsor handles fees but does not receive academic notifications, per their assigned roles.
**Related Rules:** GRD-N-001, FEE/PAY (Docs 17–18).

**Rule ID:** GRD-N-006
**Rule Name:** Custody / Contact Restrictions Are Enforced
**Description:** Court-ordered or institution-recorded custody and contact restrictions constrain access and communication.
**Priority:** Critical
**Category:** Child safety / legal
**Preconditions:** A custody or contact restriction is recorded for a student-guardian relationship.
**Business Rule:** Where a restriction exists (e.g., a non-custodial party barred from contact or data), the system enforces it: no portal access, no notifications, no data visibility to the restricted party — overriding default guardian capabilities.
**System Action:** Apply restriction flags that override capabilities and notifications.
**Validation:** Restriction recorded with authority/reference; enforced consistently.
**Failure Behavior:** Restricted access/communication blocked; attempts audited and flagged.
**Audit Requirement:** Log `CUSTODY_RESTRICTION_APPLIED/ENFORCED`.
**Example Scenario:** A court order bars one parent from accessing records; the system blocks that party entirely while the custodial parent retains access.
**Related Rules:** GRD-N-004, GRD-N-005, Compliance.

**Rule ID:** GRD-N-007
**Rule Name:** Guardianship Changes Are Reasoned, Dated, and Audited
**Description:** Adding, removing, or changing guardianship is a governed change with a reason and effective date.
**Priority:** High
**Category:** Governance
**Preconditions:** A guardianship change.
**Business Rule:** Sensitive guardianship changes (primary change, removal, custody updates) require a recorded reason, an effective date, and full audit; they re-resolve access and notifications immediately. May require approval where configured.
**System Action:** Apply the change with reason/date; re-resolve ownership and notifications.
**Validation:** Reason captured for sensitive changes; minor-guardian invariant preserved.
**Failure Behavior:** Unjustified sensitive change rejected.
**Audit Requirement:** Log `GUARDIANSHIP_CHANGED` with before/after, reason, actor.
**Example Scenario:** Changing the primary guardian after a custody ruling is recorded with the ruling reference.
**Related Rules:** GRD-N-002, GRD-N-006.

**Rule ID:** GRD-N-008
**Rule Name:** Parental Consent for Minors
**Description:** Processing a minor's data relies on recorded parental/guardian consent where required, withdrawable.
**Priority:** Critical
**Category:** Compliance (minors / GDPR)
**Preconditions:** Consent-requiring processing of a minor's data (e.g., certain communications, optional data, media use).
**Business Rule:** Consent (and its scope and withdrawal) is recorded against the guardian-of-record; consent-gated processing checks for valid consent; withdrawal stops the gated processing going forward.
**System Action:** Record consent/withdrawal; gate processing on consent state.
**Validation:** Consent present and in-scope for the processing; not withdrawn.
**Failure Behavior:** Consent-gated processing blocked without valid consent.
**Audit Requirement:** Log `CONSENT_GRANTED/WITHDRAWN` with scope.
**Example Scenario:** A school cannot publish a minor's photo without recorded guardian consent, and stops if consent is withdrawn.
**Related Rules:** STU-005, Doc 30 (media), Compliance.

## 5. Validation Rules
- Exactly one active primary guardian-of-record per student.
- Minor → ≥1 active guardian at all times (invariant).
- Guardian reused across siblings (avoid duplicates); identity matched on link.
- Guardian-facing access strictly limited to linked children.
- Capabilities limited by per-link roles; custody restrictions override.
- Sensitive guardianship changes require reason + effective date; consent recorded where required.

## 6. State Machine

**State Name:** PROSPECTIVE
**Description:** Guardian captured during admission, link not yet finalized.
**Allowed Transitions:** → ACTIVE (link finalized on conversion); → DISCARDED (application abandoned).
**Forbidden Transitions:** → REVOKED (nothing to revoke yet).
**System Actions:** Hold guardian data with the application.

**State Name:** ACTIVE
**Description:** Live guardian link with defined roles.
**Allowed Transitions:** → SUSPENDED (temporary restriction); → REVOKED (relationship ended); role/primary changes in place.
**Forbidden Transitions:** removal that orphans a minor (GRD-N-002).
**System Actions:** Drive ownership, notifications, capabilities.

**State Name:** SUSPENDED
**Description:** Temporarily restricted (e.g., pending custody verification).
**Allowed Transitions:** → ACTIVE (restored); → REVOKED.
**Forbidden Transitions:** access while suspended.
**System Actions:** Suspend access/notifications; retain link.

**State Name:** REVOKED
**Description:** Link ended (custody change, removal, relationship ended).
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** silent reactivation (re-link is a new, audited link).
**System Actions:** End access/notifications immediately; retain history.

**State Name:** ARCHIVED
**Description:** Historical guardian link, retained.
**Allowed Transitions:** none (terminal).
**Forbidden Transitions:** edits/hard-delete.
**System Actions:** Read-only retention.

## 7. Status Definitions
Link: `PROSPECTIVE` · `ACTIVE` · `SUSPENDED` · `REVOKED` · `ARCHIVED`. Per-link role flags: `PRIMARY_CONTACT`, `FINANCIAL_RESPONSIBLE`, `PORTAL_ACCESS`, `EMERGENCY_CONTACT`. Restriction flags: `CUSTODY_RESTRICTED`, `CONTACT_BARRED`.

## 8. Workflow Rules
- Guardian linkage during admission is part of conversion (Doc 07); standalone changes are admin actions.
- Sensitive guardianship changes (primary change, removal, custody) may require approval via the Workflow Engine (configurable) and always require a reason.
- Custody restrictions, once recorded with authority, are enforced immediately and cannot be casually removed (governed change).

## 9. Permission Rules
- `guardian.link.manage` — create/modify/remove guardian links within scope.
- `guardian.custody.manage` — record/modify custody and contact restrictions (narrowly granted).
- `guardian.view` — read guardian records (scoped; guardians read only their own profile).
- Guardians themselves have no admin permissions; their access is purely ownership-scoped to linked children.

## 10. Notification Rules
- Notifications route to designated contacts per link roles and consent, **excluding** custody-restricted parties (GRD-N-006).
- `GUARDIAN_LINKED/REVOKED` → notify the affected guardian(s) and admins.
- `GUARDIAN_OF_RECORD_CHANGED` and `CUSTODY_RESTRICTION_APPLIED` → notify relevant parties per policy (respecting restrictions).
- Consent requests/confirmations routed to the guardian-of-record.

## 11. Audit Requirements
Mandatory: `GUARDIAN_LINKED`, `GUARDIAN_OF_RECORD_CHANGED`, `GUARDIAN_ROLE_CHANGED`, `GUARDIAN_SUSPENDED/REVOKED`, `GUARDIAN_REMOVAL_BLOCKED`, `CUSTODY_RESTRICTION_APPLIED/ENFORCED`, `CONSENT_GRANTED/WITHDRAWN`, `GUARDIAN_ACCESS_DENIED` (non-linked). With actor, guardian, student, reason (for sensitive changes), timestamp.

## 12. Data Retention Rules
- Guardian records and links retained while any linked student is active and per retention thereafter (needed for historical contact/consent record).
- Consent records retained to evidence lawful processing (and its withdrawal) for the required period.
- Custody/restriction records retained per legal need; handled with heightened confidentiality.
- Anonymization of a guardian follows retention/erasure once no active obligation remains, preserving non-identifying integrity.

## 13. Edge Cases
- **Divorced/separated parents:** distinct guardian links with possibly different roles and custody restrictions; the system must enforce one parent's restriction without blocking the other (GRD-N-006).
- **Student reaches majority:** guardian access may need to end or require the now-adult student's consent — handle the minor→adult transition explicitly (links to STU edge case).
- **Guardian who is also a staff member:** their guardian (children-only) access and their staff (role-based) access are separate scopes — must not bleed into each other.
- **Single guardian for many siblings:** one update propagates; but removing that guardian is blocked if it would orphan any linked minor (GRD-N-002 per child).
- **Emergency contact who is not a data-access guardian:** has contact role without portal/data access.
- **Disputed guardianship:** suspend the link pending verification rather than guess; never expose a minor's data to an unverified claimant.
- **Consent withdrawn mid-year for media:** previously published media handling per policy; future processing stops (GRD-N-008).
- **Guardian shared across institutes in the deployment:** still children-only scoped; sees each child within that child's institute, never crosses to unrelated students.

## 14. Failure Scenarios
- **Orphaning a minor:** hard-blocked (GRD-N-002); replacement required first.
- **Restriction not enforced due to misconfiguration:** treated as a serious incident; restrictions fail closed (deny on uncertainty).
- **Duplicate guardian created for siblings:** prevented by match-and-reuse; if it happens, a merge process consolidates (audited).
- **Consent state ambiguous:** consent-gated processing blocked until consent is unambiguous.

## 15. Exception Handling Rules
- Custody/contact restrictions fail **closed** — on any doubt, deny the restricted party.
- Removal that would orphan a minor is rejected with a clear message.
- Non-linked guardian access is denied and audited, never partially served.
- Sensitive changes without a reason are rejected.

## 16. Compliance Considerations
- **Child safety first:** custody enforcement, mandatory guardianship, and children-only scoping are core protections, not optional features.
- **Parental consent (GDPR / minors):** consent recorded, scoped, withdrawable, and evidenced; consent-gated processing strictly enforced (GRD-N-008).
- **Confidentiality of custody data:** treated as highly sensitive, access-restricted, audited.
- **Majority transition:** lawful basis and access shift when a student becomes an adult — explicitly designed for.

## 17. Future Considerations
- Self-service guardian verification and document-backed custody management.
- Granular per-guardian consent dashboards (what each has consented to).
- Adult-student data-control handover at majority.
- Guardian-to-guardian visibility controls in shared-custody arrangements.
