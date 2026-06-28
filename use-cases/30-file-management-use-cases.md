# 30 — File Management Use Cases

Transforms the File Management business rules (`FILE-001`…`FILE-008`) into use cases. The final module: private storage with signed-URL-only access, upload validation, malware scanning before availability, subject-scoped custody-aware access, short-lived scoped URLs, document versioning with metadata hygiene, immutable reproducible generated documents, and retention/secure-deletion/quota.

## 1. Primary Actors
Uploader (staff/guardian/applicant), Authorized Viewer (role/relationship-permitted), Generating Modules (results/finance/exam/HR), File Administrator (types, quotas, retention).

## 2. Secondary Actors
System (validation, scan, private storage, signed URLs, versioning, retention), Guardian module (custody), Configuration Engine (file types/required docs), Audit service, object storage.

## 3. Goals
Store files privately (no public URLs); validate uploads (type/size, true content type); scan for malware before availability; scope access to the file's subject with custody-awareness; deliver via short-lived scoped signed URLs; version documents and strip privacy-sensitive metadata; generate immutable, reproducible official documents; enforce retention, secure deletion, and quotas.

## 4. User Journeys
- **Upload:** an uploader submits a document → validated → quarantined and scanned → made available on clean → metadata stripped (esp. minors' photos) → scoped to the subject.
- **Access:** an authorized viewer requests a file → custody/scope checked → short-lived signed URL issued → access audited.
- **Generate:** a module produces an official document (marksheet/receipt/payslip/admit card) from finalized data → immutable, reproducible → delivered via signed URL.
- **Lifecycle:** files follow retention; secure deletion on erasure; legal hold overrides; quotas enforced.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-FILE-001 | Upload, Scan & Make Available | Critical |
| Core | UC-FILE-002 | Access File via Signed URL (custody-aware) | Critical |
| Core | UC-FILE-003 | Generate Official Document (immutable, reproducible) | High |
| Core | UC-FILE-004 | Version Document & Strip Metadata | Medium |
| Core | UC-FILE-005 | Retention, Secure Deletion & Quota | High |
| CRUD | UC-FILE-006 | View / List Files (scoped) | Medium |
| Admin | UC-FILE-007 | Configure File Types & Required Documents | Medium |
| Search | UC-FILE-008 | Search Files | Low |
| Reporting | UC-FILE-009 | Storage & Access Report | Low |
| Bulk | UC-FILE-010 | Bulk Upload / Generation | Medium |
| Export | UC-FILE-011 | Export / Download (signed-URL) | Medium |
| Workflow | UC-FILE-012 | Required-Document Checklist | Medium |
| Exception | UC-FILE-013 | Malware Detected (quarantine) | Critical |
| Exception | UC-FILE-014 | Public-Exposure / Unauthorized Access Blocked | Critical |
| Exception | UC-FILE-015 | Generation From Non-Finalized Data Blocked | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-FILE-001 — Upload, Scan & Make Available
- **Module:** File Management · **Priority:** Critical
- **Actors:** Uploader (primary), System
- **Goal:** Safely ingest a document and make it available only after validation and a clean malware scan.
- **Description:** Validates type/size (true content type), stores privately in a quarantined state, scans for malware, releases on clean, strips privacy-sensitive metadata (esp. minors' photos), and scopes the file to its subject.
- **Business Rules Applied:** FILE-001, FILE-002, FILE-003, FILE-004, FILE-006.
- **Preconditions:** File type/size policy configured; uploader authorized for the subject.
- **Trigger:** Uploader submits a file for a subject/purpose.
- **Main Success Scenario:**
  1. System validates type/size and true content type (FILE-002).
  2. System stores the file privately in a quarantined state (FILE-001).
  3. System runs a malware scan; on clean, the file becomes available (FILE-003).
  4. System strips privacy-sensitive metadata (EXIF/GPS for minors' photos, FILE-006) and scopes the file to its subject (FILE-004).
- **Alternative Flows:** A1) Re-upload creates a new version (UC-FILE-004).
- **Exception Flows:** E1) Disallowed type/oversize → reject (FILE-002). E2) Malware detected → quarantine, never serve (UC-FILE-013).
- **Validation Rules:** True content type allowed; size within limit; scanned before availability; metadata stripped; subject-scoped (FILE-002/003/004/006).
- **Permissions Required:** `file.upload` (authorized for the subject).
- **Notifications Triggered:** `FILE_QUARANTINED` (on detection); upload confirmation.
- **Audit Events Generated:** `FILE_UPLOADED`, `FILE_SCANNED` / `FILE_QUARANTINED`.
- **Data Created:** File (quarantined→available); metadata.
- **Data Updated:** File status.
- **Data Deleted:** Stripped metadata.
- **Post Conditions:** Clean file available, subject-scoped, privately stored.
- **Related Use Cases:** UC-FILE-002, UC-FILE-013, UC-STU-008.
- **Acceptance Criteria:**
  - Given a valid file, When uploaded, Then it is scanned and made available only when clean.
  - Given a disguised executable (wrong content type), When uploaded, Then it is rejected.
  - Given a minor's photo, When uploaded, Then EXIF/GPS metadata is stripped.
- **Edge Case Analysis:**
  - *Invalid Input:* disallowed type/oversize rejected.
  - *Permission Failure:* uploader not authorized for the subject → 403.
  - *Concurrent Update:* duplicate upload → versioned, not duplicated.
  - *Duplicate Data:* re-upload → new version (FILE-006).
  - *System Failure:* never serve unscanned; atomic availability flip.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* upload→scan→available; metadata stripped.
  - *Negative:* disguised type; oversize; malware.
  - *Boundary:* file exactly at size limit; scan-in-progress access blocked.

### UC-FILE-002 — Access File via Signed URL (custody-aware)
- **Module:** File Management · **Priority:** Critical
- **Actors:** Authorized Viewer (primary), System
- **Goal:** Grant time-limited, scoped access to a private file, honoring the subject's access rules and custody restrictions.
- **Description:** Access requires the same scope + ownership as the file's subject record, with custody/contact restrictions enforced; on success, a short-lived, single-purpose, scoped signed URL is issued; access is audited.
- **Business Rules Applied:** FILE-004, FILE-005, FILE-001, GRD-N-006.
- **Preconditions:** Viewer authorized for the subject (scope + ownership, custody-aware).
- **Trigger:** Viewer requests a file.
- **Main Success Scenario:**
  1. System verifies authorization against the file's subject (scope/ownership, FILE-004).
  2. System enforces custody/contact restrictions (GRD-N-006).
  3. System issues a short-lived, scoped signed URL (FILE-005).
  4. The access is audited.
- **Alternative Flows:** A1) Generated official document accessed the same way.
- **Exception Flows:** E1) Unauthorized/custody-restricted → denied, no URL (UC-FILE-014). E2) Expired URL → re-request required.
- **Validation Rules:** Subject scope/ownership; custody enforced; URL short-lived/scoped; audited (FILE-004/005).
- **Permissions Required:** `file.access` (subject scope/ownership; custody-aware).
- **Notifications Triggered:** None routine.
- **Audit Events Generated:** `FILE_ACCESS_GRANTED/DENIED` (subject); signed-URL issuance.
- **Data Created:** Signed URL (ephemeral).
- **Data Updated:** None.
- **Data Deleted:** None.
- **Post Conditions:** Time-limited access granted to an authorized viewer; audited.
- **Related Use Cases:** UC-FILE-001, UC-GRD-N-004.
- **Acceptance Criteria:**
  - Given an authorized viewer, When they request a file, Then a short-lived signed URL is issued and the access is audited.
  - Given a custody-restricted party, When they request a minor's file, Then access is denied (no URL).
  - Given an expired URL, When used, Then it is rejected (re-request required).
- **Edge Case Analysis:**
  - *Invalid Input:* request for non-existent file → not-found (no leak).
  - *Permission Failure:* out-of-scope/non-owner → denied; custody-restricted → denied.
  - *Concurrent Update:* access during re-version → consistent (latest/specified version).
  - *Duplicate Data:* N/A.
  - *System Failure:* fail closed (no URL on doubt).
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* authorized access; generated-doc access.
  - *Negative:* unauthorized; custody-restricted; expired URL.
  - *Boundary:* access exactly at URL-expiry edge.

### UC-FILE-003 — Generate Official Document (immutable, reproducible)
- **Module:** File Management · **Priority:** High
- **Actors:** Generating Module (primary), System
- **Goal:** Produce a version-stamped, immutable, reproducible official document from finalized data.
- **Description:** Official documents (marksheets, certificates, receipts, payslips, admit cards) are generated from finalized, version-stamped source data; stored immutable with provenance; reproducible on demand; a revision produces a new version with the prior retained.
- **Business Rules Applied:** FILE-007, RES-006, FEE-003, HR-003 (Cross-Cutting P6).
- **Preconditions:** Source data finalized (published result, issued receipt/payslip).
- **Trigger:** A module generates an official document.
- **Main Success Scenario:**
  1. The module generates the document from finalized, version-stamped data (FILE-007).
  2. System stores it immutable with provenance (rule/config/grade/salary versions).
  3. The document is delivered via signed URL and reproducible on demand.
- **Alternative Flows:** A1) Revision (e.g., result revision RES-006) → new versioned document; prior retained.
- **Exception Flows:** E1) Source not finalized → block generation (UC-FILE-015).
- **Validation Rules:** Source finalized; provenance stamped; immutable; reproducible (FILE-007).
- **Permissions Required:** `document.generate` (system/role-driven).
- **Notifications Triggered:** Document-ready notice (with signed link).
- **Audit Events Generated:** `DOCUMENT_GENERATED` (provenance).
- **Data Created:** Immutable official document.
- **Data Updated:** None (revision → new version).
- **Data Deleted:** None.
- **Post Conditions:** Immutable, reproducible official document available.
- **Related Use Cases:** UC-RES-004, UC-FEE-002, UC-HR-006.
- **Acceptance Criteria:**
  - Given finalized data, When generated, Then the document is immutable and reproduces identically on demand.
  - Given a revision, When produced, Then a new versioned document is created and the prior retained.
  - Given non-finalized source, When generation is attempted, Then it is blocked.
- **Edge Case Analysis:**
  - *Invalid Input:* generation from draft data blocked.
  - *Permission Failure:* unauthorized generation → 403.
  - *Concurrent Update:* re-generate → reproduces identical (idempotent).
  - *Duplicate Data:* idempotent generation.
  - *System Failure:* atomic; provenance recorded.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* generate; reproduce; revision-versioned.
  - *Negative:* non-finalized source; silent edit (must not occur).
  - *Boundary:* reprint of an old document (reproduces from stamp).

---

## 7. Compact Specifications (routine use cases)

- **UC-FILE-004 — Version Document & Strip Metadata** · *Medium* · Rules: FILE-006, STU-005. Re-upload → new version (prior retained); strip EXIF/GPS from media. *Edge:* no silent overwrite; minors' metadata stripped. *QA:* versioning; metadata hygiene.
- **UC-FILE-005 — Retention, Secure Deletion & Quota** · *High* · Rules: FILE-008, STU-007, AUD-008. Apply retention; secure-delete on erasure; enforce quotas; legal hold overrides. *Audit:* `FILE_DELETED/PURGED`. *Edge:* statutory docs retained; hold blocks deletion. *QA:* retention; secure delete; quota; hold.
- **UC-FILE-006 — View / List Files (scoped)** · *Medium* · Rules: FILE-004, AUTHZ-003. Scoped/owned listing; custody-aware. *QA:* scope; custody.
- **UC-FILE-007 — Configure File Types & Required Documents** · *Medium* · Rules: FILE-002, CFG-010. Configure allowed types/sizes and required-document sets per process. *Permissions:* `file.manage`. *Edge:* required-doc gating (UC-FILE-012). *QA:* config; required-set.
- **UC-FILE-008 — Search Files** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-FILE-009 — Storage & Access Report** · *Low* · Rules: REP-002, FILE-008. Storage usage, access patterns (scoped). *Edge:* minors'-media access especially audited. *QA:* usage accuracy; access audit.
- **UC-FILE-010 — Bulk Upload / Generation** · *Medium* · Rules: FILE-001/003/007. Bulk upload (scanned) / bulk generate official docs (async). *Edge:* per-file scan; resumable. *QA:* bulk scan; bulk generate.
- **UC-FILE-011 — Export / Download (signed-URL)** · *Medium* · Rules: FILE-005, REP-005. Download via short-lived signed URL (governed). *Edge:* expiry; scoped. *QA:* signed-URL; expiry.
- **UC-FILE-012 — Required-Document Checklist** · *Medium* · Rules: FILE-002, ADM/HR. Gate processes on required documents (admission/onboarding). *Edge:* missing doc blocks/flags. *QA:* checklist gating; reminders.
- **UC-FILE-013 — Malware Detected (quarantine) (Exception)** · *Critical* · Rules: FILE-003. Infected files quarantined, never served; alerted. *QA:* quarantine; never served; alert.
- **UC-FILE-014 — Public-Exposure / Unauthorized Access Blocked (Exception)** · *Critical* · Rules: FILE-001/004. No public URLs; unauthorized/custody-restricted access denied. *QA:* no public access; unauthorized denied.
- **UC-FILE-015 — Generation From Non-Finalized Data Blocked (Exception)** · *High* · Rules: FILE-007. Official-document generation blocked until source finalized. *QA:* blocked; finalized allowed.

## 8. Module-level QA & Edge Themes
- **Private + signed-URL only (FILE-001/005 / FILE-014):** no public file URLs; access exclusively via short-lived scoped signed URLs — the headline security suite.
- **Scan before availability (FILE-003 / FILE-013):** infected files quarantined, never served.
- **Custody-aware subject access (FILE-004 / GRD-N-006):** files inherit the subject's access rules; minors' files strictly protected.
- **Immutable reproducible documents (FILE-007 / P6):** generated official documents reproduce from stamped data; revisions versioned.
- **Metadata hygiene + secure deletion (FILE-006/008 / C-08):** minors' EXIF stripped; erasure securely deletes erasable files while statutory docs are retained.
