# 06 — Student Business Rules

## 1. Module Purpose
Govern the **student** entity — the central person record of the institution and its most sensitive data subject (a minor in most cases). This module owns the student's identity, profile (core + dynamic custom fields), unique identification, status lifecycle across sessions, and the cross-cutting protections that apply because students are minors. It is distinct from Admission (how a person *becomes* a student, Doc 07) and Enrollment (how a student is *placed* in a session's structure, Doc 08).

## 2. Actors
- **Institute / Campus Administrator** — creates and manages student records within scope.
- **Admission Officer** — produces student records via the admission flow (Doc 07).
- **Teacher** — reads (not edits) profile data for their own students (ownership-scoped).
- **Student** — reads their own profile (self-ownership).
- **Guardian** — reads their linked child's profile (relationship-ownership, Doc 09).
- **System** — enforces uniqueness, scoping, custom-field validation, and minor-data protections.

## 3. Use Cases

**Use Case ID:** UC-STU-01 — Create / Materialize a Student Record
**Actors:** Admission Officer / Administrator, System
**Description:** A person becomes a persistent student record with a unique institutional ID.
**Preconditions:** Admission approved (Doc 07) **or** an admin with `student.record.manage` performs a direct creation (migration/manual case); active scope established.
**Main Flow:** 1) System assembles core fields + validated dynamic custom fields. 2) System assigns a unique student ID per the configured scheme. 3) Student record created in `PRE_ENROLLED`/`ACTIVE` state and scoped to institute (and campus). 4) Guardian links established (Doc 09).
**Alternative Flow:** A1) Bulk import creates many records, each validated and ID-assigned.
**Exception Flow:** E1) Duplicate-candidate detected (same name/DOB/guardian) → flag for review (STU-003). E2) Custom-field validation fails → reject the record.
**Post Conditions:** Student exists, uniquely identified, scoped, audited; ready for enrollment.
**Business Rules Applied:** STU-001, STU-002, STU-003, STU-006.

**Use Case ID:** UC-STU-02 — Update Student Profile
**Actors:** Administrator (edit) / Teacher, Student, Guardian (read), System
**Description:** Maintain a student's profile data over time.
**Preconditions:** Actor authorized for the field set; student `ACTIVE`.
**Main Flow:** 1) Actor edits permitted fields. 2) System validates against definitions. 3) Changes versioned and audited; sensitive-field changes flagged.
**Alternative Flow:** A1) A guardian/student requests a correction → routed to an admin (read-only roles cannot self-edit official fields).
**Exception Flow:** E1) Edit of a locked/official field (e.g., legal name) by an unauthorized role → denied. E2) Edit on a graduated/archived student → blocked (STU-007).
**Post Conditions:** Profile updated and audited, or change rejected.
**Business Rules Applied:** STU-004, STU-005, STU-007.

**Use Case ID:** UC-STU-03 — Deactivate / Withdraw / Graduate a Student
**Actors:** Administrator, System
**Description:** Move a student out of active operation while preserving history.
**Preconditions:** Actor holds `student.record.manage`; a terminal-ish event occurs (withdrawal, graduation, long inactivity).
**Main Flow:** 1) Admin sets the appropriate status with reason. 2) System preserves all academic/financial history immutably. 3) Access and operations adjust accordingly.
**Exception Flow:** E1) Withdrawal with outstanding dues/records → warn per policy. E2) Hard-delete attempt → forbidden (STU-007).
**Post Conditions:** Status changed; history retained; audited.
**Business Rules Applied:** STU-007, STU-008.

## 4. Business Rules

**Rule ID:** STU-001
**Rule Name:** Unique Student Identifier
**Description:** Every student has a unique, configurable, immutable institutional ID.
**Priority:** Critical
**Category:** Identity
**Preconditions:** Student creation.
**Business Rule:** A unique student ID (format configurable per client/institute — prefix, sequence, year segment) is assigned at creation, unique within the deployment, and never reused or re-edited. It is distinct from any government ID.
**System Action:** Generate the next ID per the scheme atomically; lock it.
**Validation:** Uniqueness across the deployment; format matches the configured scheme.
**Failure Behavior:** Reject creation on ID collision; never silently reuse a retired ID.
**Audit Requirement:** Log `STUDENT_CREATED` with the assigned ID.
**Example Scenario:** "SCH-2026-000142" identifies a student permanently, even after graduation.
**Related Rules:** STU-002, STU-003.

**Rule ID:** STU-002
**Rule Name:** Core vs Dynamic Profile Fields
**Description:** Student profile is core fields plus institute-defined dynamic custom fields, all validated.
**Priority:** High
**Category:** Configuration / data quality
**Preconditions:** Profile create/edit.
**Business Rule:** Core fields (legal name, DOB, gender, guardians, student ID) are always present. Additional fields are dynamic custom fields defined per institute via the Configuration Engine and validated against their definitions; "schemaless" never means "unvalidated."
**System Action:** Validate core + dynamic fields on every write; store dynamic values as validated JSONB on the record.
**Validation:** Core required-field presence; dynamic values valid against field definitions (type, range, options, required).
**Failure Behavior:** Reject the write on any validation failure; report the offending field.
**Audit Requirement:** Log `STUDENT_PROFILE_UPDATED` with changed fields (values for sensitive fields masked).
**Example Scenario:** One institute adds a "blood group" field and a "previous school" field; both validate on entry.
**Related Rules:** CFG (Configuration Engine), STU-004.

**Rule ID:** STU-003
**Rule Name:** Duplicate-Student Detection
**Description:** The system surfaces likely duplicates before creating a redundant student record.
**Priority:** High
**Category:** Data integrity
**Preconditions:** Student creation (admission or manual/import).
**Business Rule:** On creation, the system checks for likely duplicates (matching name + DOB + guardian, or matching government ID where captured) within the institute and flags candidates for human review rather than silently creating or silently blocking.
**System Action:** Run the duplicate heuristic; present candidates; require explicit confirm-as-new or merge.
**Validation:** Heuristic fields available; reviewer decision recorded.
**Failure Behavior:** Proceed only on explicit reviewer confirmation; never auto-merge.
**Audit Requirement:** Log `DUPLICATE_FLAGGED` and the resolution (`CONFIRMED_NEW`/`MERGED`).
**Example Scenario:** A re-admitting student matches an existing record; the officer is prompted before a second record is created.
**Related Rules:** STU-001, ADM (admission).

**Rule ID:** STU-004
**Rule Name:** Official-Field Edit Control
**Description:** Legally/operationally significant fields are edit-restricted and change-tracked.
**Priority:** High
**Category:** Integrity / governance
**Preconditions:** Edit of an official field (legal name, DOB, guardian-of-record, government ID).
**Business Rule:** Official fields can be changed only by authorized roles, may require supporting justification, and every change is versioned with before/after. Students/guardians (read-only roles) request changes; they do not self-edit official fields.
**System Action:** Gate the edit by permission; capture justification; version the change.
**Validation:** Actor authorized; justification captured where configured; new value valid.
**Failure Behavior:** Unauthorized or unjustified edit rejected.
**Audit Requirement:** Log `STUDENT_OFFICIAL_FIELD_CHANGED` with before/after and justification.
**Example Scenario:** Correcting a misspelled legal name is an authorized, justified, audited change — not a casual edit.
**Related Rules:** STU-002, STU-005, AUTHZ-003.

**Rule ID:** STU-005
**Rule Name:** Sensitive-Data Minimization & Masking
**Description:** Sensitive student PII is minimized, access-controlled, masked in low-trust contexts, and field-encrypted where most sensitive.
**Priority:** Critical
**Category:** Privacy (minors)
**Preconditions:** Sensitive fields (government ID, medical notes, contact details, photo) are stored or displayed.
**Business Rule:** Collect only what definitions require; the most sensitive identifiers are field-encrypted; display is masked except to roles with explicit need; photos and IDs are access-controlled like documents (signed URLs).
**System Action:** Encrypt designated fields; mask on display by role; gate media via signed URLs.
**Validation:** Field-sensitivity classification applied; role need-to-know enforced.
**Failure Behavior:** Unauthorized access masked/denied; never expose raw sensitive identifiers broadly.
**Audit Requirement:** Log access to highly sensitive fields where configured (`SENSITIVE_FIELD_ACCESSED`).
**Example Scenario:** A teacher sees a student's name and class but a masked government ID; the registrar sees the full value.
**Related Rules:** STU-002, Doc 30 (File Management), Compliance.

**Rule ID:** STU-006
**Rule Name:** Mandatory Guardian Linkage for Minors
**Description:** A minor student must have at least one responsible guardian linked.
**Priority:** Critical
**Category:** Child safety / integrity
**Preconditions:** Student creation/activation where the student is a minor.
**Business Rule:** A student classified as a minor (by DOB against the configured age of majority) must have ≥1 active guardian link with a defined relationship and contact before enrollment is finalized. The guardian-of-record drives notifications and portal access (Doc 09).
**System Action:** Block enrollment finalization for a minor with no guardian link.
**Validation:** Age computed from DOB; ≥1 guardian link present and valid.
**Failure Behavior:** Block finalization; prompt to add a guardian.
**Audit Requirement:** Log `GUARDIAN_LINK_REQUIRED_BLOCK` when enforced.
**Example Scenario:** A 9-year-old cannot be fully enrolled until a parent/guardian is linked.
**Related Rules:** STU-005, ENR (enrollment), Doc 09 (Guardian).

**Rule ID:** STU-007
**Rule Name:** Archive/Anonymize, Never Hard-Delete
**Description:** Student records are never hard-deleted; they are withdrawn/graduated/archived and, when lawful, anonymized.
**Priority:** Critical
**Category:** Data integrity / compliance
**Preconditions:** Student removal requested.
**Business Rule:** A student that holds any academic/financial history cannot be hard-deleted. Lifecycle moves to `WITHDRAWN`/`GRADUATED`/`ARCHIVED`, preserving records (transcripts, ledgers). Erasure is retention-driven anonymization with export-before-purge, retaining non-identifying integrity references.
**System Action:** Transition status; preserve history read-only; anonymize only per retention/erasure policy.
**Validation:** Detect dependent history; archive instead of delete.
**Failure Behavior:** Hard-delete of a data-bearing student rejected.
**Audit Requirement:** Log `STUDENT_WITHDRAWN/GRADUATED/ARCHIVED/ANONYMIZED` with reason.
**Example Scenario:** An alumnus's transcript remains retrievable years later; a lawful erasure removes PII while keeping anonymized integrity links.
**Related Rules:** STU-008, SESS-005, Doc 30, Compliance.

**Rule ID:** STU-008
**Rule Name:** Status Reflects Lifecycle, Driven by Enrollment & Outcomes
**Description:** Student status is derived from real events (admission, enrollment, promotion, withdrawal, graduation), not set arbitrarily.
**Priority:** High
**Category:** Lifecycle integrity
**Preconditions:** A lifecycle event occurs.
**Business Rule:** Status transitions follow the state machine and are caused by domain events; an admin cannot jump a student to an arbitrary state that contradicts their enrollment/results (e.g., "GRADUATED" without reaching the final level) except via an explicit, audited override.
**System Action:** Derive/guard status transitions from events; require justification for overrides.
**Validation:** Transition allowed by the state machine and consistent with enrollment/results.
**Failure Behavior:** Inconsistent transition blocked or requires override.
**Audit Requirement:** Log every `STUDENT_STATUS_CHANGED` with cause (event/override).
**Example Scenario:** A student becomes `ACTIVE` only upon finalized enrollment, not on record creation alone.
**Related Rules:** STU-007, ENR, SESS-006.

## 5. Validation Rules
- Student ID: unique deployment-wide, scheme-valid, immutable.
- Core fields present; dynamic fields valid against definitions.
- DOB present and plausible (drives minor classification and guardian requirement).
- Minor → ≥1 valid guardian link before enrollment finalization.
- Official-field edits gated by permission (+ justification where configured).
- Operations on `GRADUATED`/`ARCHIVED` students blocked unless via governed correction.

## 6. State Machine

**State Name:** PRE_ENROLLED
**Description:** Record materialized (post-admission or manual) but not yet enrolled in a session.
**Allowed Transitions:** → ACTIVE (enrollment finalized); → WITHDRAWN (abandoned before enrollment); → ARCHIVED.
**Forbidden Transitions:** → GRADUATED.
**System Actions:** Require guardian link for minors before allowing ACTIVE.

**State Name:** ACTIVE
**Description:** Currently enrolled and operating in a session.
**Allowed Transitions:** → INACTIVE (long absence/suspension); → WITHDRAWN; → GRADUATED (final level, via promotion); → TRANSFERRED_OUT.
**Forbidden Transitions:** → PRE_ENROLLED.
**System Actions:** Permit academic operations; drive attendance/exam ownership.

**State Name:** INACTIVE
**Description:** Temporarily not operating (extended leave, suspension, non-payment hold).
**Allowed Transitions:** → ACTIVE (reinstated); → WITHDRAWN; → ARCHIVED.
**Forbidden Transitions:** → GRADUATED.
**System Actions:** Suspend operational participation; retain data.

**State Name:** WITHDRAWN
**Description:** Left before completion.
**Allowed Transitions:** → ACTIVE (re-admission may instead create a fresh enrollment, audited); → ARCHIVED.
**Forbidden Transitions:** silent reactivation.
**System Actions:** Preserve history; close active enrollments.

**State Name:** GRADUATED
**Description:** Completed the final level; exited the structure.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** → ACTIVE (alumni re-entry is a new admission).
**System Actions:** Mark completion; retain transcript permanently.

**State Name:** TRANSFERRED_OUT
**Description:** Moved to another institution (outside this deployment) or institute.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** → ACTIVE (return is a new admission/enrollment).
**System Actions:** Issue transfer certificate (Doc 08/30); retain history.

**State Name:** ARCHIVED
**Description:** Long-term retained, read-only.
**Allowed Transitions:** → ANONYMIZED (retention/erasure only).
**Forbidden Transitions:** hard-delete (if history exists).
**System Actions:** Read-only; retain per class.

**State Name:** ANONYMIZED
**Description:** PII removed per erasure; terminal.
**Allowed Transitions:** none.
**Forbidden Transitions:** all.
**System Actions:** Strip PII; keep non-identifying integrity references.

## 7. Status Definitions
`PRE_ENROLLED` · `ACTIVE` · `INACTIVE` · `WITHDRAWN` · `GRADUATED` · `TRANSFERRED_OUT` · `ARCHIVED` · `ANONYMIZED`. (Minor/Adult is a derived classification from DOB, not a status.)

## 8. Workflow Rules
- Direct student creation outside admission requires elevated permission and is audited (migration/manual path).
- Withdrawal and status overrides may require approval via the Workflow Engine (configurable) given their impact.
- Official-field corrections may route through an approval workflow where configured (SoD: requester ≠ approver).
- Re-admission of a `WITHDRAWN`/`GRADUATED` student creates a fresh enrollment, never a silent reactivation.

## 9. Permission Rules
- `student.record.view` — read student records (scoped + ownership: teacher→own students, etc.).
- `student.record.manage` — create/edit student records within scope.
- `student.official_field.manage` — edit official fields (narrowly granted).
- `student.sensitive.view` — view unmasked sensitive fields (need-to-know).
- `student.status.manage` — change lifecycle status / overrides.
- Students see only themselves; guardians see only linked children (Doc 09); teachers see only assigned students.

## 10. Notification Rules
- `STUDENT_CREATED` → notify guardians (welcome, per preference) and the admission officer.
- `STUDENT_OFFICIAL_FIELD_CHANGED` (legal name/DOB/guardian) → notify guardians.
- `STUDENT_WITHDRAWN/GRADUATED/TRANSFERRED_OUT` → notify guardians with relevant documents.
- `STUDENT_STATUS_CHANGED` to INACTIVE (e.g., non-payment hold) → notify guardians per policy.

## 11. Audit Requirements
Mandatory: `STUDENT_CREATED`, `STUDENT_PROFILE_UPDATED`, `STUDENT_OFFICIAL_FIELD_CHANGED`, `SENSITIVE_FIELD_ACCESSED` (where configured), `DUPLICATE_FLAGGED`/resolution, `STUDENT_STATUS_CHANGED`, `STUDENT_WITHDRAWN/GRADUATED/TRANSFERRED_OUT/ARCHIVED/ANONYMIZED`, guardian-link required blocks. With actor, student ID, scope, before/after (sensitive masked), timestamp.

## 12. Data Retention Rules
- Academic records (transcripts, results) and financial ledgers retained long-term per legal minimums — typically years beyond graduation.
- Profile and contact PII retained only as long as lawful/necessary; subject to erasure → `ANONYMIZED`.
- Photos/IDs follow the file-retention class with stricter handling (minors).
- Withdrawn/graduated students' records retained, not purged early; anonymization is export-before-purge.

## 13. Edge Cases
- **Re-admitting a former student:** duplicate detection (STU-003) prevents a second master record; re-entry is a new enrollment under the existing student ID.
- **Student who turns 18 mid-enrollment:** minor→adult reclassification may change guardian-access and consent semantics; handle the transition explicitly (does the adult student now control their own data?).
- **Twins / same-guardian same-DOB:** duplicate heuristic must not falsely merge siblings; require human confirmation.
- **Student with no government ID (young minor):** government ID is optional; the institutional student ID is the canonical key.
- **Profile edit on a closed-session context:** current profile is editable, but historical session records remain immutable (a name change doesn't rewrite past marksheets — those reflect the name at issuance).
- **Guardian removed leaving a minor with none:** blocked — a minor must retain ≥1 guardian (STU-006).
- **Photo of a minor:** never public; signed-URL + scoped access only (STU-005).

## 14. Failure Scenarios
- **ID generation race:** atomic sequence prevents two students getting the same ID under concurrent creation.
- **Custom-field definition changed after data exists:** existing values validated against the version they were created under; promotion-to-column handled by the engine, not by silent data loss.
- **Bulk import partial failure:** per-row validation; valid rows succeed, invalid rows reported — never a half-imported corrupt batch without a report.
- **Anonymization requested while legal hold active:** erasure blocked under legal hold; recorded.

## 15. Exception Handling Rules
- Validation failures reject the specific write with the offending field identified.
- Status transitions inconsistent with enrollment/results are blocked or require an audited override.
- Unauthorized sensitive-field access is masked/denied, never partially leaked.
- Hard-delete of a data-bearing student is rejected with an explicit archive-instead message.

## 16. Compliance Considerations
- **Minors-first:** the entire module treats students as minors by default — mandatory guardianship, masking, field encryption, restricted media, heightened audit.
- **GDPR-aligned rights:** access/export/erasure/rectification map to profile read/export, anonymization, and official-field correction; consent (esp. parental) governs processing.
- **Age-of-majority transition:** consent and access semantics adjust when a student becomes an adult — explicitly handled.
- **Record permanence vs erasure:** balanced — academic/financial integrity retained while personal data is erasable where lawful.

## 17. Future Considerations
- Student-controlled data after majority (self-service consent management).
- Cross-institute student identity within a deployment (a student moving between this client's institutes without losing history).
- Health/special-needs modules with stricter access classes.
- Biometric/photo-based identity (with strong minor-data safeguards) for attendance/exams.
