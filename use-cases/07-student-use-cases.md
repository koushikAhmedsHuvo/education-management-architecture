# 07 — Student Use Cases

Transforms the Student business rules (`STU-001`…`STU-008`) into use cases. The student is the central minor-protected entity: immutable identity, validated profile, sensitive-data masking, mandatory guardianship, lifecycle status, and archive-not-delete.

## 1. Primary Actors
Student Administrator / Registrar (manages records), Teacher (scoped read of own students), Student (self-service, age-appropriate), Guardian (children-only access).

## 2. Secondary Actors
System (identity, dedup, masking, status, retention), Configuration Engine (dynamic profile fields), Guardian module (mandatory linkage), File module (photo/documents), Audit & Notification services.

## 3. Goals
Maintain a single accurate record per student with an immutable ID; capture validated core + dynamic profile data; protect sensitive/minor data via masking and need-to-know; guarantee every minor has a guardian; reflect lifecycle status from enrollment/outcomes; preserve history through archive/anonymize rather than deletion.

## 4. User Journeys
- **Record creation:** typically via admission conversion (UC-ADM-005) — dedup-checked, immutable ID, guardian linked for minors. Manual creation by registrar follows the same guards.
- **Profile maintenance:** registrar edits official fields under control; staff/students/guardians view scoped, masked data.
- **Lifecycle:** status moves prospective → active → (inactive/withdrawn) → alumni, driven by enrollment and outcomes, not set arbitrarily.
- **Exit & retention:** withdrawal/graduation archives the record; lawful erasure anonymizes the erasable subset while retaining statutory records (Conflict C-08).

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core/CRUD | UC-STU-001 | Create Student Record (dedup, immutable ID) | Critical |
| CRUD | UC-STU-002 | View Student Profile (masked, scoped) | High |
| CRUD | UC-STU-003 | Update Official Fields (controlled) | High |
| CRUD | UC-STU-004 | Update Dynamic/Custom Fields | Medium |
| Core | UC-STU-005 | Manage Lifecycle Status | High |
| Core | UC-STU-006 | Link / Manage Guardians (mandatory for minors) | Critical |
| Core | UC-STU-007 | Archive / Anonymize Student (retention) | Critical |
| Admin | UC-STU-008 | Manage Student Photo & Documents | Medium |
| Approval | UC-STU-009 | Approve Sensitive-Field / Status Change | Medium |
| Search | UC-STU-010 | Search Students (scoped) | High |
| Reporting | UC-STU-011 | Student Roster & Demographics Report | Medium |
| Bulk | UC-STU-012 | Bulk Student Update | Medium |
| Import | UC-STU-013 | Import Students | Medium |
| Export | UC-STU-014 | Export Student Data (governed, masked) | High |
| Workflow | UC-STU-015 | Data-Subject Access / Erasure Request | High |
| Exception | UC-STU-016 | Duplicate Student Detected | High |
| Exception | UC-STU-017 | Minor Without Guardian Blocked | Critical |

---

## 6. Detailed Specifications (high-value use cases)

### UC-STU-001 — Create Student Record (dedup, immutable ID)
- **Module:** Student · **Priority:** Critical
- **Actors:** Registrar / System (primary)
- **Goal:** Create exactly one record per real student, with a permanent identifier.
- **Description:** Runs duplicate detection before creating; assigns an immutable student ID and a scope-unique student/roll number; enforces guardian linkage for minors.
- **Business Rules Applied:** STU-001, STU-002, STU-003, STU-006, STU-008.
- **Preconditions:** Authorized creator (or admission conversion); required core fields available.
- **Trigger:** Admission conversion (UC-ADM-005) or manual registrar creation.
- **Main Success Scenario:**
  1. System runs duplicate-student detection on identifying attributes (STU-003).
  2. If no match, System creates the record with an immutable student ID (STU-001) and validated core + dynamic fields (STU-002).
  3. System enforces a guardian link for minors (STU-006).
  4. System assigns a scope-unique student/roll number (STU-008).
  5. Student exists (status per source: prospective/active).
- **Alternative Flows:** A1) Match found → link to the existing student instead of creating (UC-STU-016).
- **Exception Flows:** E1) Minor without a guardian → block creation/activation (UC-STU-017). E2) Required core field invalid → reject.
- **Validation Rules:** Dedup performed; immutable ID assigned; core fields valid; guardian present for minors; number unique in scope (STU-001/002/003/006/008).
- **Permissions Required:** `student.manage` (manual) / system (conversion).
- **Notifications Triggered:** Onboarding/welcome to guardian (per preference).
- **Audit Events Generated:** `STUDENT_CREATED` (id) or `STUDENT_LINKED`.
- **Data Created:** Student record; profile; student number.
- **Data Updated:** None (or guardian linkage).
- **Data Deleted:** None.
- **Post Conditions:** One canonical student exists with a permanent ID and (for minors) a guardian.
- **Related Use Cases:** UC-ADM-005, UC-STU-006, UC-STU-016.
- **Acceptance Criteria:**
  - Given identifying attributes matching no existing student, When created, Then a new record with an immutable ID and unique number is produced.
  - Given a match to an existing student, When creation is attempted, Then the existing record is linked (no duplicate).
  - Given a minor without a guardian, When creation/activation is attempted, Then it is blocked.
- **Edge Case Analysis:**
  - *Invalid Input:* invalid core/custom field → reject with field errors.
  - *Permission Failure:* lacks `student.manage` → 403.
  - *Concurrent Update:* two creations for the same person → dedup yields one (or flags for merge).
  - *Duplicate Data:* core scenario — STU-003 prevents duplicates.
  - *System Failure:* atomic creation; ID never reused even on rollback.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* new student; link-to-existing on match.
  - *Negative:* minor without guardian; invalid field; duplicate.
  - *Boundary:* near-match fuzzy dedup threshold; number uniqueness at scope edge.

### UC-STU-002 — View Student Profile (masked, scoped)
- **Module:** Student · **Priority:** High
- **Actors:** Teacher / Registrar / Student / Guardian (primary), System
- **Goal:** Show only the data the viewer is authorized to see, with sensitive fields masked.
- **Description:** The same record renders differently per role: registrar (full, need-to-know), teacher (academic subset for own students), guardian (own children), student (own, age-appropriate) — sensitive fields masked unless explicitly authorized.
- **Business Rules Applied:** STU-005, STU-002, AUTHZ-002, AUTHZ-003, GRD-N-004.
- **Preconditions:** Authenticated viewer with a relationship/role to the student.
- **Trigger:** Viewer opens a student profile.
- **Main Success Scenario:**
  1. System resolves the viewer's role/relationship and scope/ownership (AUTHZ-002/003).
  2. System assembles the profile, applying field-level masking per need-to-know (STU-005).
  3. Viewer sees the authorized, masked view.
- **Alternative Flows:** A1) Registrar with explicit need-to-know unmasks specific sensitive fields (audited).
- **Exception Flows:** E1) Custody-restricted guardian → excluded from access (GRD-N-006). E2) Out-of-scope/non-owner → denied/empty.
- **Validation Rules:** Scope + ownership pass; masking applied per role; custody restrictions honored (STU-005, GRD-N-004/006).
- **Permissions Required:** `student.view` (scoped) / guardian-children / self.
- **Notifications Triggered:** None routine; sensitive unmasking may alert.
- **Audit Events Generated:** `STUDENT_VIEWED` (sensitive views/unmasking recorded).
- **Data Created/Updated/Deleted:** None.
- **Post Conditions:** Authorized, masked view rendered; sensitive access audited.
- **Related Use Cases:** UC-STU-003, UC-STU-014.
- **Acceptance Criteria:**
  - Given a teacher, When they view a student they teach, Then they see the academic subset with sensitive identifiers masked.
  - Given a guardian, When they view a non-child, Then access is denied.
  - Given a custody-restricted party, When they attempt access, Then they are excluded.
- **Edge Case Analysis:**
  - *Invalid Input:* unknown student id → not-found (no existence leak across scope).
  - *Permission Failure:* out-of-scope/non-owner → denied.
  - *Concurrent Update:* viewing during an edit → consistent read (no torn data).
  - *Duplicate Data:* N/A.
  - *System Failure:* masking failure → fail closed (withhold sensitive).
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* each role sees its authorized view.
  - *Negative:* out-of-scope; non-child guardian; custody-restricted.
  - *Boundary:* registrar unmasking exactly one sensitive field (audited).

### UC-STU-006 — Link / Manage Guardians (mandatory for minors)
- **Module:** Student · **Priority:** Critical
- **Actors:** Registrar (primary), System, Guardian module
- **Goal:** Guarantee every minor always has at least one active guardian and manage relationships safely.
- **Description:** Links one or more guardians (with roles, financial-responsibility, custody flags); enforces the always-≥1-guardian invariant for minors; changes are reasoned, dated, audited.
- **Business Rules Applied:** STU-006, GRD-N-002, GRD-N-003, GRD-N-005, GRD-N-006, GRD-N-007.
- **Preconditions:** Student exists; for minors a guardian is mandatory.
- **Trigger:** Registrar links/updates guardians (or admission conversion).
- **Main Success Scenario:**
  1. Registrar links a guardian (existing or new) with a role (GRD-N-005) and flags (financial-responsible, custody).
  2. System validates and records the relationship (dated, reasoned, GRD-N-007).
  3. For minors, System enforces ≥1 active guardian at all times (GRD-N-002).
  4. Guardian gains children-only scoped access (GRD-N-003/004).
- **Alternative Flows:** A1) Reuse an existing guardian across siblings (GRD-N-003). A2) Assign emergency-contact priority (GRD-N-007).
- **Exception Flows:** E1) Removing the last guardian of a minor → blocked (UC-STU-017). E2) Custody/contact restriction → enforced on access and notifications (GRD-N-006).
- **Validation Rules:** ≥1 active guardian for minors; relationship dated/reasoned; custody flags enforced (STU-006, GRD-N-002/006/007).
- **Permissions Required:** `student.guardian.manage`.
- **Notifications Triggered:** `GUARDIAN_LINKED/CHANGED` to affected parties (respecting restrictions).
- **Audit Events Generated:** `GUARDIAN_LINKED/UNLINKED/UPDATED` (reason, date, actor).
- **Data Created:** Guardian links; guardian record (if new).
- **Data Updated:** Relationship roles/flags.
- **Data Deleted:** None (links closed, not hard-deleted).
- **Post Conditions:** Minor has ≥1 active guardian; relationships accurate and audited.
- **Related Use Cases:** UC-GRD-N-001, UC-STU-017.
- **Acceptance Criteria:**
  - Given a minor, When guardian changes are made, Then at least one active guardian always remains.
  - Given siblings, When a guardian is linked, Then one guardian record serves multiple students.
  - Given a custody restriction, When set, Then the restricted party is excluded from access and notifications.
- **Edge Case Analysis:**
  - *Invalid Input:* invalid relationship/role rejected.
  - *Permission Failure:* lacks `student.guardian.manage` → 403.
  - *Concurrent Update:* simultaneous guardian edits → serialized; invariant preserved.
  - *Duplicate Data:* duplicate guardian → reuse, not re-create (GRD-N-003).
  - *System Failure:* atomic; never leaves a minor guardian-less.
  - *Workflow Failure:* guardianship change requiring approval pends (GRD-N-007).
- **QA Coverage:**
  - *Positive:* link/replace guardian; sibling reuse; emergency priority.
  - *Negative:* remove last guardian (blocked); custody-restricted access.
  - *Boundary:* swapping the sole guardian (add-before-remove ordering).

### UC-STU-007 — Archive / Anonymize Student (retention)
- **Module:** Student · **Priority:** Critical
- **Actors:** Registrar / Compliance Officer (primary), System
- **Goal:** End-of-life a student record without losing required history, honoring retention precedence.
- **Description:** On exit, the record is archived (read-only). On lawful erasure, the erasable subset (profile, contacts, media) is anonymized/deleted while statutory records (transcripts, fee ledger, audit) are retained per Conflict C-08; legal hold overrides.
- **Business Rules Applied:** STU-007, GRD-N-008, AUD-008 (and Cross-Cutting P4).
- **Preconditions:** Student exited/withdrawn/graduated, or a verified erasure request.
- **Trigger:** Lifecycle exit or a data-subject erasure request.
- **Main Success Scenario:**
  1. On exit, System archives the record (read-only, preserved).
  2. On lawful erasure, System applies the Retention Precedence Order: legal hold > statutory retention > erasure.
  3. System anonymizes/deletes the erasable subset; retains transcripts/financial/audit (audit pseudonymized).
  4. The action is recorded; affected files securely deleted (FILE-008).
- **Alternative Flows:** A1) Legal hold present → erasure suspended and recorded.
- **Exception Flows:** E1) Hard-delete attempt → forbidden (STU-007). E2) Erasure of statutory-retained data → not performed until retention elapses.
- **Validation Rules:** No hard delete; precedence applied; legal hold honored (STU-007, P4, AUD-008).
- **Permissions Required:** `student.archive` / `compliance.erasure.manage` (elevated).
- **Notifications Triggered:** Erasure/archival confirmation to the requester/compliance.
- **Audit Events Generated:** `STUDENT_ARCHIVED`, `STUDENT_ANONYMIZED` (scope), `LEGAL_HOLD_APPLIED` (if any).
- **Data Created:** None.
- **Data Updated:** Record archived; erasable subset anonymized.
- **Data Deleted:** Erasable personal data/files (securely); statutory data retained.
- **Post Conditions:** History preserved per law; PII minimized where lawful; fully audited.
- **Related Use Cases:** UC-STU-015, UC-FILE (secure deletion).
- **Acceptance Criteria:**
  - Given a lawful erasure with no hold, When processed, Then profile/contacts/media are anonymized while transcript/fee/audit are retained (audit pseudonymized).
  - Given a legal hold, When erasure is requested, Then it is suspended and recorded.
  - Given any path, When invoked, Then no hard delete of a data-bearing record occurs.
- **Edge Case Analysis:**
  - *Invalid Input:* erasure without verification rejected.
  - *Permission Failure:* lacks elevated permission → 403.
  - *Concurrent Update:* erasure during active enrollment → blocked/deferred until exit.
  - *Duplicate Data:* repeated erasure idempotent.
  - *System Failure:* partial anonymization recoverable; never half-erased inconsistent state.
  - *Workflow Failure:* compliance approval pends.
- **QA Coverage:**
  - *Positive:* archive on graduation; lawful erasure with retention split.
  - *Negative:* hold blocks erasure; hard-delete attempt; statutory data preserved.
  - *Boundary:* erasure exactly at retention expiry of a statutory record.

---

## 7. Compact Specifications (routine use cases)

- **UC-STU-003 — Update Official Fields (controlled)** · *High* · Rules: STU-004. Edit name/DOB/official identifiers under control (some require approval/audit). *Audit:* `STUDENT_OFFICIAL_FIELD_CHANGED`. *Edge:* immutable ID never editable; sensitive change may need approval (UC-STU-009). *QA:* controlled edit; immutable-ID edit blocked.
- **UC-STU-004 — Update Dynamic/Custom Fields** · *Medium* · Rules: STU-002, CFG-010. Edit config-defined fields (schema-validated). *Edge:* values validated against the field schema/version. *QA:* valid value; schema violation rejected.
- **UC-STU-005 — Manage Lifecycle Status** · *High* · Rules: STU-008. Status driven by enrollment/outcomes (prospective/active/inactive/withdrawn/alumni). *Audit:* `STUDENT_STATUS_CHANGED`. *Edge:* status changes are outcome-driven, not arbitrary; invalid transitions blocked. *QA:* valid transition; arbitrary set blocked.
- **UC-STU-008 — Manage Student Photo & Documents** · *Medium* · Rules: STU-005, FILE-001/004/006. Upload/replace photo & documents (private, scanned, metadata-stripped). *Edge:* minors' media stripped of EXIF; signed-URL access only. *QA:* upload+scan; metadata stripped; scoped access.
- **UC-STU-009 — Approve Sensitive-Field / Status Change** · *Medium* · Rules: STU-004, AUTHZ-009, WFL-004. Approver gates sensitive official-field/status changes. *Edge:* SoD; escalation. *QA:* approval gates; self-approval blocked.
- **UC-STU-010 — Search Students (scoped)** · *High* · Rules: AUTHZ-002/003, REP-002. Scoped/owned search; teacher sees own students. *Edge:* never returns out-of-scope students; masked previews. *QA:* scope/ownership respected; masking in results.
- **UC-STU-011 — Student Roster & Demographics Report** · *Medium* · Rules: REP-002/003. Scoped roster/demographics; sensitive fields masked. *Edge:* minors'/sensitive data masked unless need-to-know. *QA:* counts accurate; masking enforced.
- **UC-STU-012 — Bulk Student Update** · *Medium* · Rules: STU-004. Batch update permitted fields (governed). *Edge:* per-row validation; immutable fields excluded; partial-failure report. *QA:* clean bulk; invalid rows; immutable-field protection.
- **UC-STU-013 — Import Students** · *Medium* · Rules: STU-001/003/006/008. Import with dedup, immutable IDs, guardian enforcement for minors. *Edge:* duplicates linked; minors without guardian flagged. *QA:* clean import; duplicate link; minor-without-guardian flagged.
- **UC-STU-014 — Export Student Data (governed, masked)** · *High* · Rules: REP-005, STU-005. Governed, scoped, masked export; bulk PII gated/audited. *Permissions:* `report.export` (elevated for bulk). *Edge:* sensitive/minor data controlled; full audit. *QA:* scoped export; PII gated; masking applied.
- **UC-STU-015 — Data-Subject Access / Erasure Request** · *High* · Rules: STU-007, REP (subject access), P4. Verified subject-access (compile own data) or erasure (UC-STU-007). *Edge:* mixed data → only the subject's; precedence applied. *QA:* subject-access accuracy; erasure precedence.
- **UC-STU-016 — Duplicate Student Detected (Exception)** · *High* · Rules: STU-003. Match found → link/merge instead of create. *QA:* dedup link; merge integrity; no duplicate ID.
- **UC-STU-017 — Minor Without Guardian Blocked (Exception)** · *Critical* · Rules: STU-006, GRD-N-002. Creation/activation/last-guardian-removal blocked for minors. *QA:* each path blocked; remedied by linking a guardian.

## 8. Module-level QA & Edge Themes
- **Minors-first:** masking (STU-005), mandatory guardianship (STU-006/GRD-N-002), and custody enforcement (GRD-N-006) are the headline suites — no weaker path anywhere.
- **Identity integrity:** immutable ID (STU-001) and dedup (STU-003) prevent duplicate/merged-wrong records.
- **Archive-not-delete & retention precedence (C-08/P4):** assert no hard delete; verify the erasure/retention split.
- **Scope + masking in reads/exports:** every view, search, report, and export respects scope, ownership, and field masking.
