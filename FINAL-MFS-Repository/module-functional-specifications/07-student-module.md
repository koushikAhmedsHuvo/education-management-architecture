# 07 — Student Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`STU-001…008`), and Use Case Repository (`UC-STU-001…017`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Maintain the authoritative student master record: unique immutable identity, core vs dynamic profile fields, duplicate prevention, controlled official-field edits, sensitive-data minimization/masking, mandatory guardian linkage for minors, lifecycle status, and archive-not-delete.

**Business Goal.** Hold one accurate, privacy-protected record per student that every academic, financial, and reporting module references — with heightened protection for minors' data.

**Scope.** Student CRUD with immutable student ID; core (official) vs dynamic (config-defined) fields; duplicate detection; controlled official-field editing; sensitive-data masking; mandatory ≥1 guardian for minors; lifecycle status driven by enrollment/outcomes; photo/document management; archive/anonymize with retention; data-subject access/erasure.

**Out of Scope.** Enrollment placement (Enrollment module). Guardian records themselves (Guardian module — this module links them). Admission/conversion (Admission module — creates the student). Result/attendance data (respective modules, scoped to the student).

---

## 2. Actors

**Primary Actors.** Registrar / Student-Records Administrator, Campus/Institute Administrator (scoped), Student/Guardian (limited self-view), System (dedup, masking, status derivation).

**Secondary Actors.** Admission module (creates students on conversion), Guardian module (linkage), Enrollment/Result modules (status drivers), File module (photo/documents), Configuration Engine (dynamic fields), Audit, Reporting, Notification.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Unique immutable student ID | Assign a permanent, non-reusable student identifier. | Critical | Config (ID scheme) |
| FR-002 | Core vs dynamic fields | Maintain fixed official fields + config-defined dynamic fields. | High | Configuration Engine |
| FR-003 | Duplicate detection | Detect likely duplicates on create/import; link rather than duplicate. | High | FR-001 |
| FR-004 | Controlled official-field edits | Restrict/approve edits to official fields (name, DOB, etc.). | High | Workflow/AUTHZ-009 |
| FR-005 | Sensitive-data masking | Mask/minimize sensitive fields by role need-to-know. | Critical | Authorization, Reporting |
| FR-006 | Mandatory guardian for minors | A minor must always have ≥1 active guardian. | Critical | Guardian module |
| FR-007 | Lifecycle status | Derive status from enrollment/outcomes (prospective/active/alumni/withdrawn/archived). | High | Enrollment, Result |
| FR-008 | Photo & document management | Manage student photo/documents (private, scanned, metadata-stripped). | Medium | File module |
| FR-009 | Archive/anonymize, never delete | Archive with retention; anonymize erasable data on lawful erasure. | High | Retention/Audit |
| FR-010 | Data-subject access/erasure | Fulfil subject access and lawful erasure (children-only for guardians). | Medium | Reporting, Audit |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Student master record | One immutable, authoritative record. | Single source of truth. |
| Core + dynamic fields | Official fields + configurable custom fields. | Flexibility without schema churn. |
| Duplicate prevention | Detect/link on create/import. | Clean, deduplicated data. |
| Controlled official edits | Governed changes to official fields. | Record integrity; audit trail. |
| Sensitive-data masking | Role-based minimization. | Minor-data protection; privacy. |
| Mandatory guardian (minors) | Always ≥1 active guardian. | Safeguarding; contactability. |
| Archive + erasure | Retention-aware lifecycle. | Compliance; data-subject rights. |

---

## 5. Screens

Student List; Student Detail (profile/masked); Student Create; Student Edit (official-field controlled); Dynamic Fields Edit; Lifecycle Status; Guardians (link/manage); Student Photo & Documents; Archive/Anonymize; Data-Subject Access/Erasure; Student Roster & Demographics Report; Student Import; Student Data Export; Sensitive-Change Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Student List | Create, Open, Search/Filter, Export | Bulk Update, Bulk Import |
| Student Detail | Edit, Manage Guardians, Manage Documents, Change Status, Archive | — |
| Student Edit | Save (official → approval), Save Dynamic Fields | — |
| Guardians | Link Guardian, Set Financial-Responsible, Set Restrictions, Unlink | — |
| Photo & Documents | Upload, Replace (version), Download (signed URL), Delete (governed) | — |
| Archive/Anonymize | Archive, Anonymize (erasure), Confirm with Reason | — |
| Data-Subject Access | Generate Access Report, Process Erasure | — |
| Roster Report | Run, Filter, Export | Export |

---

## 7. Forms

**Create Student** — official: `firstName`/`lastName` (text, required), `dob` (date, required), `gender` (select), `studentId` (auto, immutable). Dynamic: config-defined fields (CFG-010). Validation: dedup check (STU-003); minor requires guardian linkage path (STU-006); ID immutable (STU-001).

**Edit Official Fields** — name/DOB/official identifiers. Validation: controlled edit (STU-004) — sensitive changes routed to approval (UC-STU-009); immutable ID never editable.

**Dynamic Fields** — config-defined typed fields. Validation: engine-validated per schema (CFG-010).

**Link Guardian** — `guardian` (search-select or create), `relationship` (select), `isFinancialResponsible` (toggle), `custodyRestrictions` (structured, optional). Validation: minor always retains ≥1 active guardian (STU-006 / GRD-N-002); restrictions enforced downstream (GRD-N-006).

**Archive/Anonymize** — `reason` (text, required), `mode` (archive | anonymize-erasure). Validation: never hard-delete (STU-007); statutory data retained; legal hold blocks erasure (C-08).

---

## 8. Search & Filter Requirements

**Students:** by student ID, name, status, campus, class/section (via enrollment), guardian, admission year. Sorting: name/ID/status. Pagination: server-side, 25 default. Scope + ownership enforced (teachers see own students; guardians see own children — masked).

---

## 9. Table Requirements

**Student table:** Photo, Student ID, Name, Status, Campus, Class/Section, Guardian, Updated. Sensitive columns masked per role (STU-005). Sorting on Name/ID/Status. Filtering as above. Export (governed, masked — UC-STU-014). Bulk: Update, Import.

---

## 10. Workflow Requirements

**Trigger events:** create (from conversion), official-field change, status change, guardian link/unlink, archive/anonymize, erasure request. **Status changes:** `PROSPECTIVE → ACTIVE → (ALUMNI/WITHDRAWN) → ARCHIVED` (driven by enrollment/outcomes, STU-008). **Approvals:** sensitive official-field/status changes (SoD). **Notifications:** guardian-linkage, status changes (custody-aware). **Audit:** all official-field/status/guardian/archival changes (before/after, immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| View student (masked) | `student.view` |
| Create student | `student.create` |
| Edit official fields | `student.official.update` (→ approval) |
| Edit dynamic fields | `student.dynamic.update` |
| Manage status | `student.status.manage` |
| Manage guardians link | `student.guardian.manage` |
| Manage documents | `student.document.manage` |
| Archive/anonymize | `student.archive`, `student.anonymize` |
| Subject access/erasure | `student.subject_access` |
| Import/export | `student.import`, `student.export` |

Sensitive fields require need-to-know permission to unmask (STU-005).

---

## 12. Business Rule References

STU-001 (unique student ID), STU-002 (core vs dynamic fields), STU-003 (duplicate detection), STU-004 (official-field edit control), STU-005 (sensitive-data minimization & masking), STU-006 (mandatory guardian for minors), STU-007 (archive/anonymize, never hard-delete), STU-008 (status reflects lifecycle). Cross-cutting: GRD-N-002 (≥1 active guardian), CFG-010 (dynamic fields), AUTHZ-003 (ownership), FILE-001/003/006 (documents), AUD-008/C-08 (erasure precedence), REP-002/003 (scoped/masked reporting).

## 13. Use Case References

UC-STU-001 (Create — dedup, immutable ID), UC-STU-002 (View — masked, scoped), UC-STU-003 (Update Official — controlled), UC-STU-004 (Update Dynamic), UC-STU-005 (Lifecycle Status), UC-STU-006 (Link/Manage Guardians), UC-STU-007 (Archive/Anonymize), UC-STU-008 (Photo & Documents), UC-STU-009 (Approve Sensitive Change), UC-STU-010 (Search), UC-STU-011 (Roster & Demographics), UC-STU-012 (Bulk Update), UC-STU-013 (Import), UC-STU-014 (Export — governed, masked), UC-STU-015 (Subject Access/Erasure), UC-STU-016 (Duplicate Detected), UC-STU-017 (Minor Without Guardian Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Create student | POST | Registrar (or conversion) |
| Get / list students (masked, scoped) | GET | Authorized roles |
| Update official fields | PUT | Registrar (→ approval) |
| Update dynamic fields | PUT | Registrar |
| Change lifecycle status | POST | Registrar |
| Link / unlink guardian | POST/DELETE | Registrar |
| Upload / manage documents | POST | Registrar |
| Archive / anonymize student | POST | Registrar (elevated) |
| Subject access / erasure | POST | Compliance |
| Search students | GET | Authorized roles |
| Roster & demographics report | GET | Admin |
| Import / export students | POST/GET | Admin |

All reads apply scope + ownership + masking (STU-005, AUTHZ-002/003).

---

## 15. Database Requirements

**Entities:** `Student` (immutable studentId, core fields, status), `StudentDynamicField` (config-defined values), `StudentGuardianLink` (relationship, financial-responsible, restrictions), `StudentDocument` (file refs), `StudentArchive`/anonymization log. **Relationships:** Student *—* Guardian (via link); Student 1—* Enrollment; Student 1—* Document. **Indexes:** unique(Student.studentId), dedup index(lastName, dob, guardianPhone), index(Student.status, campusId), index(StudentGuardianLink.studentId), index(StudentGuardianLink.guardianId). Immutable ID; archive-not-delete enforced at data layer (STU-007).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Guardian-linkage confirmation; status changes (custody-aware); subject-access fulfilment. |
| In-App | Profile/status updates; pending sensitive-change approvals. |
| SMS/Push | Minimized status alerts to permitted guardians. |

Content minimized on low-assurance channels (NOT-004); restricted parties excluded (NOT-002/GRD-N-006).

---

## 17. Audit Requirements

Log: create, official-field changes (before/after, approver), dynamic-field changes, status changes, guardian link/unlink, document add/replace/delete, archive/anonymize (reason, retained vs erased). Record who/when/before/after. Sensitive-field access (unmasking) is auditable. Immutable via outbox; erasure pseudonymizes audit references (AUD-008).

---

## 18. Reporting Requirements

**Reports:** Student roster, Demographics (masked/aggregated), Status distribution, Guardian-linkage completeness (minors without guardian), Document completeness. **Exports:** governed, masked student export (PII-gated). **Dashboards:** student-records health (active count, minors-without-guardian alerts, pending approvals).

---

## 19. Error Handling

**Validation:** duplicate student, minor without guardian, invalid dynamic field → specific errors (UC-STU-016/017). **Permission:** unmask without need-to-know → masked; out-of-scope/non-owner → 403/not-found. **Workflow:** sensitive change pending approval → clear state. **System:** File scan pending → document unavailable; erasure under legal hold → blocked (C-08).

---

## 20. Edge Cases

**Concurrent updates:** two edits to a student → versioned; official-field change pends approval. **Duplicate data:** dedup match on create/import → link, not duplicate. **Partial failures:** archive with pending dependencies → handled/blocked per policy. **Rollback:** failed bulk update → per-row report, no partial silent writes. **Guardian race:** unlinking the last guardian of a minor → blocked (STU-006/GRD-N-002).

---

## 21. Acceptance Criteria

**Functional.** Each student has a unique, immutable ID and duplicates are prevented/linked; official-field edits are controlled and approved; sensitive fields are masked by role; a minor always has ≥1 active guardian; status reflects enrollment/outcomes; students are archived/anonymized, never hard-deleted; subject-access/erasure is fulfilled within retention/legal-hold precedence.

**Business.** One accurate, privacy-protected record per student underpins every other module; minors' data is structurally protected; the record remains correct and recoverable across the student lifecycle.

---

## 22. Future Enhancements

Biometric/ID-card integration; configurable dedup scoring; student self-service profile (limited); document-expiry tracking; family/household grouping views; consent-driven data-sharing with external bodies; ML-assisted duplicate resolution.
