# 30 — File Management Business Rules

## 1. Module Purpose
Govern **files and documents** — uploads (student photos, admission documents, staff records) and generated artifacts (marksheets, certificates, receipts, payslips, admit cards). Files are stored in **private object storage** and accessed only via **short-lived, scoped, signed URLs** (never public); uploads are **validated and malware-scanned** before availability; every file is **scoped to its subject** and inherits that subject's access rules — so a minor's photo or document is protected exactly as their record is. Generated official documents are **version-stamped and reproducible**. This module closes the loop on the document/media protections referenced throughout the catalog.

## 2. Actors
- **Uploader** — staff/guardian/applicant who uploads documents.
- **Authorized Viewer** — a role/relationship permitted to access a file (subject-scoped, custody-aware).
- **Generating Modules** — produce official documents (results, finance, exam, HR).
- **Administrator** — configures file types, required documents, quotas, retention.
- **System** — validates, scans, stores privately, issues signed URLs, scopes access, version-stamps, retains/purges, and audits.

## 3. Use Cases

**Use Case ID:** UC-FILE-01 — Upload, Scan & Make Available
**Actors:** Uploader, System
**Description:** Safely ingest a document and make it available only after scanning.
**Preconditions:** File type/size policy configured; uploader authorized for the subject.
**Main Flow:** 1) Uploader submits a file for a subject/purpose. 2) System validates type/size and stores it privately in a quarantined state. 3) Malware scan runs; on clean, the file becomes available; metadata (esp. minors' photos) is stripped per policy. 4) The file inherits the subject's access scope.
**Alternative Flow:** A1) Re-upload creates a new version (FILE-006).
**Exception Flow:** E1) Disallowed type/oversize → reject (FILE-002). E2) Malware detected → quarantine, never serve (FILE-003).
**Post Conditions:** Clean file available, subject-scoped; audited.
**Business Rules Applied:** FILE-001, FILE-002, FILE-003, FILE-004.

**Use Case ID:** UC-FILE-02 — Access a File via Signed URL
**Actors:** Authorized Viewer, System
**Description:** Grant time-limited, scoped access to a private file.
**Preconditions:** Viewer authorized for the subject (scope + ownership, custody-aware).
**Main Flow:** 1) Viewer requests a file. 2) System verifies authorization (subject scope/ownership, custody restrictions). 3) Issues a short-lived, scoped signed URL. 4) Access is audited.
**Exception Flow:** E1) Unauthorized/custody-restricted → denied (FILE-004). E2) Expired URL → re-request required.
**Post Conditions:** Time-limited access granted to an authorized viewer; audited.
**Business Rules Applied:** FILE-004, FILE-005.

**Use Case ID:** UC-FILE-03 — Generate an Official Document
**Actors:** Generating Module, System
**Description:** Produce a version-stamped, reproducible official document.
**Preconditions:** Source data finalized (e.g., published result, issued receipt).
**Main Flow:** 1) The module generates the document from finalized, version-stamped data (grading/fee/config versions). 2) Stored privately, immutable, with provenance. 3) Delivered via signed URL; reproducible on demand.
**Exception Flow:** E1) Source not finalized → block generation.
**Post Conditions:** Immutable, reproducible official document; audited.
**Business Rules Applied:** FILE-007.

## 4. Business Rules

**Rule ID:** FILE-001
**Rule Name:** Private Storage, No Public Access
**Description:** All files are stored privately; there is no public URL — access is only via signed URLs.
**Priority:** Critical
**Category:** Security (minors/privacy)
**Preconditions:** Any file stored.
**Business Rule:** Files live in private object storage with encryption at rest; there are no publicly accessible file URLs; access is exclusively through short-lived signed URLs issued after authorization.
**System Action:** Store privately/encrypted; never expose direct/public links.
**Validation:** No public ACLs; encryption at rest enabled.
**Failure Behavior:** Reject any public-exposure configuration.
**Audit Requirement:** Log file storage.
**Example Scenario:** A student's photo is never on a guessable public URL; only authorized, signed access works.
**Related Rules:** FILE-004, STU-005, architecture D46.

**Rule ID:** FILE-002
**Rule Name:** Upload Validation (Type, Size)
**Description:** Uploads are validated against configured allowed types and size limits.
**Priority:** High
**Category:** Integrity / safety
**Preconditions:** Upload.
**Business Rule:** Allowed file types and size limits are configurable per purpose; disallowed types (e.g., executables) and oversize files are rejected; content type is verified (not just extension) to prevent disguised files.
**System Action:** Validate true content type and size against policy.
**Validation:** Type allowed; size within limit; content matches claimed type.
**Failure Behavior:** Reject invalid uploads with a clear reason.
**Audit Requirement:** Log rejected uploads.
**Example Scenario:** A disguised executable renamed to .pdf is detected and rejected.
**Related Rules:** FILE-003, CFG (type policy).

**Rule ID:** FILE-003
**Rule Name:** Malware Scan Before Availability
**Description:** Files are scanned for malware and made available only when clean; detections are quarantined.
**Priority:** Critical
**Category:** Security
**Preconditions:** Upload stored.
**Business Rule:** Uploaded files are quarantined until a malware scan passes; clean files become available; detections are quarantined and never served, and the uploader/admin is alerted.
**System Action:** Quarantine, scan, release-on-clean or quarantine-on-detection.
**Validation:** Scan completes before availability.
**Failure Behavior:** Never serve unscanned/infected files; quarantine and alert.
**Audit Requirement:** Log `FILE_SCANNED` (result); `FILE_QUARANTINED`.
**Example Scenario:** An infected attachment is quarantined and never reaches a viewer.
**Related Rules:** FILE-001, FILE-002.

**Rule ID:** FILE-004
**Rule Name:** Subject-Scoped, Custody-Aware Access
**Description:** A file inherits its subject's access rules; access requires subject scope/ownership and respects custody restrictions.
**Priority:** Critical
**Category:** Authorization (minors)
**Preconditions:** File access.
**Business Rule:** A file is bound to a subject (student, staff, application); accessing it requires the same scope + ownership as the subject's record (AUTHZ-002/003), and custody/contact restrictions (GRD-N-006) apply — a barred party cannot access a minor's documents. A guardian accesses only their child's files.
**System Action:** Authorize against the subject's access rules; enforce custody restrictions before issuing a signed URL.
**Validation:** Viewer authorized for the subject; not restricted.
**Failure Behavior:** Deny unauthorized/restricted access; no signed URL issued.
**Audit Requirement:** Log `FILE_ACCESS_GRANTED/DENIED` with subject.
**Example Scenario:** A teacher can view their student's submitted document but not another class's; a custody-barred parent cannot view a minor's files.
**Related Rules:** AUTHZ-003, GRD-N-006, STU-005.

**Rule ID:** FILE-005
**Rule Name:** Short-Lived, Scoped Signed URLs
**Description:** Access URLs are short-lived, single-purpose, and scoped; leakage exposure is minimized.
**Priority:** High
**Category:** Security
**Preconditions:** Access granted.
**Business Rule:** Signed URLs expire quickly (configurable, short), are scoped to the specific file and operation, and are not reusable beyond their window; a new request is needed after expiry. This bounds the damage of an accidentally shared link.
**System Action:** Issue time-boxed, scoped signed URLs; expire promptly.
**Validation:** Expiry short; scope specific.
**Failure Behavior:** Reject expired/over-scoped URL use.
**Audit Requirement:** Signed-URL issuance recorded.
**Example Scenario:** A marksheet link shared by accident stops working within minutes.
**Related Rules:** FILE-001, FILE-004.

**Rule ID:** FILE-006
**Rule Name:** Document Versioning & Metadata Hygiene
**Description:** Re-uploads create versions; privacy-sensitive metadata is stripped, especially for minors' media.
**Priority:** Medium
**Category:** Integrity / privacy
**Preconditions:** Re-upload or media upload.
**Business Rule:** Replacing a document creates a new version (prior retained per policy, not silently overwritten); EXIF/location and other sensitive metadata are stripped from media (e.g., minors' photos) to prevent privacy leakage.
**System Action:** Version on re-upload; strip sensitive metadata.
**Validation:** Versioning applied; metadata stripped per policy.
**Failure Behavior:** Block silent overwrite; never retain location metadata on minors' photos.
**Audit Requirement:** Log file versions.
**Example Scenario:** A re-uploaded certificate creates v2; a student photo has its GPS metadata removed.
**Related Rules:** STU-005, FILE-004.

**Rule ID:** FILE-007
**Rule Name:** Generated Official Documents Are Immutable & Reproducible
**Description:** System-generated documents (marksheets, certificates, receipts, payslips, admit cards) are version-stamped, immutable, and reproducible.
**Priority:** High
**Category:** Integrity / reproducibility
**Preconditions:** Document generation.
**Business Rule:** Official documents are generated from finalized, version-stamped source data (grading/fee/config versions) and are immutable; regenerating reproduces the same document; a revision (e.g., result revision RES-006) produces a new versioned document, with the prior retained.
**System Action:** Generate from stamped data; store immutable; reproduce on demand.
**Validation:** Source finalized; provenance stamped.
**Failure Behavior:** Block generation from non-finalized data; no silent document edits.
**Audit Requirement:** Log `DOCUMENT_GENERATED` with provenance.
**Example Scenario:** A reprinted 2025 marksheet matches the original because it reproduces from the stamped data.
**Related Rules:** RES-006, GRD-007, FEE-003, HR-003.

**Rule ID:** FILE-008
**Rule Name:** Retention, Secure Deletion & Quota
**Description:** Files follow retention classes with secure deletion; storage quotas apply per deployment.
**Priority:** Medium
**Category:** Lifecycle / cost
**Preconditions:** Retention/deletion or storage limits.
**Business Rule:** Files are retained per their class (e.g., transcripts long, transient uploads short); deletion is secure (not just dereferenced) and, on subject anonymization (STU-007/STF-008), associated personal files are removed/stripped; storage quotas per deployment are enforced with alerts. Legal hold overrides deletion.
**System Action:** Apply retention; securely delete; enforce quotas; honor legal hold.
**Validation:** Retention class set; deletion secure; quota tracked.
**Failure Behavior:** Block premature deletion under hold; alert near quota.
**Audit Requirement:** Log `FILE_DELETED/PURGED` and quota events.
**Example Scenario:** On lawful erasure, a former student's photo and documents are securely deleted while their transcript is retained per academic retention.
**Related Rules:** STU-007, STF-008, AUD-008, Compliance.

## 5. Validation Rules
- Files stored privately/encrypted; no public access; access only via short-lived scoped signed URLs.
- Uploads validated (true type, size); malware-scanned before availability; infected quarantined.
- Access requires subject scope/ownership; custody restrictions enforced.
- Re-uploads versioned; sensitive metadata stripped (esp. minors' media).
- Generated documents immutable, version-stamped, reproducible.
- Retention/secure-deletion/quota enforced; legal hold overrides deletion.

## 6. State Machine

**State Name:** UPLOADED (QUARANTINED)
**Description:** Received, stored privately, awaiting scan.
**Allowed Transitions:** → AVAILABLE (clean); → QUARANTINED (infected); → REJECTED (invalid).
**Forbidden Transitions:** serving before scan.
**System Actions:** Validate; scan; strip metadata on release.

**State Name:** AVAILABLE
**Description:** Clean, subject-scoped, accessible via signed URLs.
**Allowed Transitions:** → SUPERSEDED (new version); → ARCHIVED; → DELETED (retention/erasure).
**Forbidden Transitions:** public exposure.
**System Actions:** Issue signed URLs to authorized viewers; audit access.

**State Name:** QUARANTINED
**Description:** Malware detected; never served.
**Allowed Transitions:** → DELETED (after handling).
**Forbidden Transitions:** availability.
**System Actions:** Isolate; alert; block access.

**State Name:** SUPERSEDED
**Description:** Replaced by a newer version; prior retained per policy.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** silent overwrite.
**System Actions:** Keep version history.

**State Name:** ARCHIVED / DELETED / PURGED
**Description:** Aged/retained, securely deleted, or purged post-retention.
**Allowed Transitions:** ARCHIVED → DELETED/PURGED (retention, no hold).
**Forbidden Transitions:** deletion under legal hold; insecure deletion.
**System Actions:** Retain/secure-delete per class.

## 7. Status Definitions
`UPLOADED` (quarantined) · `AVAILABLE` · `QUARANTINED` · `SUPERSEDED` · `ARCHIVED` · `DELETED` · `PURGED` · `REJECTED`. Document classes: `UPLOAD` · `GENERATED_OFFICIAL` (immutable). 

## 8. Workflow Rules
- Required-document checklists (admission, onboarding) gate their processes (links ADM, HR).
- Generated-document creation is triggered by finalized source events (publish, issue).
- Malware detection triggers a quarantine/alert handling path.
- Erasure triggers secure deletion of personal files (subject to legal hold).

## 9. Permission Rules
- `file.upload` — upload for an authorized subject/purpose.
- `file.access` — access files (subject scope/ownership; custody-aware).
- `file.manage` — configure types/quotas/retention.
- `document.generate` — produce official documents (system/role-driven).
- All access constrained by the subject's scope/ownership (FILE-004), regardless of file permission.

## 10. Notification Rules
- `FILE_QUARANTINED` (malware) → alert uploader/admin.
- Generated document ready (marksheet/certificate/receipt) → notify the subject/guardian with a signed link.
- Required-document missing → remind the relevant party.
- Quota-threshold alerts → notify admins.

## 11. Audit Requirements
Mandatory: `FILE_UPLOADED`, `FILE_SCANNED`/`FILE_QUARANTINED`, `FILE_ACCESS_GRANTED/DENIED` (subject), signed-URL issuance, `FILE_VERSIONED`, `DOCUMENT_GENERATED` (provenance), `FILE_DELETED/PURGED`, quota events. Access to minors' media/documents is especially audited.

## 12. Data Retention Rules
- Files retained per class (transcripts/certificates long; transient uploads short).
- Generated official documents retained per academic/financial retention; reproducible from stamped data.
- Personal files securely deleted on lawful erasure (STU-007/STF-008); legal hold overrides.
- Quotas per deployment; archival tiering for cost without losing required documents.

## 13. Edge Cases
- **Malware detected:** quarantined, never served (FILE-003).
- **Disguised file type:** caught by true-content-type validation (FILE-002).
- **Leaked signed URL:** short expiry + scope limits exposure (FILE-005).
- **Minor's photo metadata:** GPS/EXIF stripped (FILE-006).
- **Subject anonymized:** personal files securely deleted/stripped; required retained records kept (FILE-008/STU-007).
- **Re-upload:** versioned, not silently overwritten (FILE-006).
- **Reprint of an official document:** reproduces the original from stamped data (FILE-007).
- **Custody-barred party:** denied file access (FILE-004/GRD-N-006).
- **Quota exceeded:** alert and govern; never silently drop required documents.

## 14. Failure Scenarios
- **Public-exposure config:** rejected (FILE-001).
- **Unscanned/infected serve:** prevented (FILE-003).
- **Unauthorized/restricted access:** denied; no URL (FILE-004).
- **Expired/over-scoped URL:** rejected (FILE-005).
- **Generation from non-finalized data:** blocked (FILE-007).
- **Deletion under legal hold:** blocked (FILE-008).

## 15. Exception Handling Rules
- Files are never served before passing validation and scanning.
- Access fails closed on authorization/custody doubt; no signed URL issued.
- Generated documents come only from finalized, stamped data and are immutable.
- Deletion is secure and hold-aware; metadata hygiene protects minors.

## 16. Compliance Considerations
- **Child safety/privacy:** private storage, signed URLs, custody-aware access, and metadata stripping protect minors' media/documents.
- **Consent for media:** publishing/using minors' media respects recorded consent (GRD-N-008).
- **Data protection:** secure deletion on erasure; legal hold; minimization.
- **Document integrity:** immutable, reproducible official documents support academic/financial trust and verification.

## 17. Future Considerations
- Verifiable documents (QR/cryptographic signatures) for tamper-proof certificates/transcripts.
- On-the-fly watermarking for sensitive exports.
- DLP scanning of uploads for sensitive-data leakage.
- Client-side/end-to-end encryption options for the most sensitive documents.
