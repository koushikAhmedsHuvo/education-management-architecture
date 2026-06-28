# 30 — File Management Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint (D46 private storage + signed URLs + scan), Business Rules Catalog (`FILE-001…008`), and Use Case Repository (`UC-FILE-001…015`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Store and serve files safely: private storage with no public access, upload validation, malware scan before availability, subject-scoped custody-aware access, short-lived scoped signed URLs, document versioning with metadata hygiene, immutable/reproducible generated official documents, and retention/secure-deletion/quota.

**Business Goal.** Ensure every file — student documents, uploads, and generated marksheets/receipts/payslips — is private, scanned, custody-respecting, and (for official documents) immutable and reproducible, with minors' EXIF/GPS stripped.

**Scope.** Private storage (no public URLs); upload validation (type/size); malware scan + quarantine before availability; subject-scoped custody-aware access; short-lived scoped signed URLs; versioning + metadata stripping (minors' EXIF/GPS); immutable reproducible generated official documents; retention/secure-deletion/quota. Cross-cutting service consumed by all modules.

**Out of Scope.** Document content/meaning (consuming modules own it; File stores/serves). Custody source of truth (Guardian — File consumes GRD-N-006). Generation source data integrity (Result/Fee/HR finalize; File renders the immutable artifact). Audit storage (Audit module).

---

## 2. Actors

**Primary Actors.** Uploader (staff/guardian/student — uploads documents), System (scan, signed-URL issuance, generation, retention), File Administrator (types/required-docs/quota config).

**Secondary Actors.** All consuming modules (store/generate/serve), Guardian module (custody), malware-scan service, Storage (S3/MinIO), Notification, Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Private storage, no public access | No public URLs; all access mediated. | Critical | Storage (D46) |
| FR-002 | Upload validation | Validate type and size on upload. | High | Config |
| FR-003 | Malware scan before availability | Scan and quarantine before a file is accessible. | Critical | Scan service |
| FR-004 | Subject-scoped custody-aware access | Access limited to authorized, custody-permitted parties. | Critical | Guardian (GRD-N-006) |
| FR-005 | Short-lived scoped signed URLs | Serve only via short-lived, scoped signed URLs. | Critical | Storage |
| FR-006 | Versioning + metadata hygiene | Version documents; strip metadata (minors' EXIF/GPS). | High | — |
| FR-007 | Immutable reproducible official docs | Generated docs (marksheet/receipt/payslip) immutable + reproducible. | Critical | Result/Fee/HR |
| FR-008 | Retention, secure deletion & quota | Enforce retention, secure deletion, per-scope quota. | High | Retention/C-08 |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Private storage | No public access. | Data protection. |
| Upload validation | Type/size gates. | Safety; integrity. |
| Scan-before-available | Quarantine first. | Malware protection. |
| Custody-aware access | Restricted parties blocked. | Safeguarding. |
| Signed URLs | Short-lived, scoped. | Controlled serving. |
| Metadata hygiene | EXIF/GPS stripped. | Minor-location privacy. |
| Immutable official docs | Reproducible artifacts. | Defensible records. |
| Retention/quota | Lifecycle + limits. | Compliance; cost control. |

---

## 5. Screens

File Browser (scoped); Upload (validated); File Detail / Versions; Signed-URL Access; Generated-Document View (immutable); Required-Document Checklist; File Type / Required-Docs Configuration; Retention / Secure-Deletion / Quota; Storage & Access Report; Bulk Upload / Generation; File Search; Quarantine (malware).

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| File Browser | Upload, Open, Search/Filter, Download (signed-URL) | Bulk Upload |
| Upload | Select, Validate (type/size), Submit (→ scan) | Bulk Upload |
| File Detail/Versions | View Versions, Download, Replace (new version) | — |
| Generated Document | View, Download (signed-URL), Reprint (same artifact) | Bulk Generate |
| Required-Doc Checklist | View Status, Upload Missing | — |
| Config | Set Types/Required-Docs/Quota, Save (versioned) | — |
| Retention | Apply Retention, Secure-Delete (governed) | Bulk Apply |
| Storage Report | Run, Filter, Export | Export |
| Quarantine | Review, Release (if clean), Discard | Bulk Discard |

---

## 7. Forms

**Upload** — `file` (binary), `type` (required-doc category), `subject` (student/staff). Validation: allowed type/size (FILE-002); scanned before available (FILE-003 / UC-FILE-013 quarantine); metadata stripped (FILE-006); stored privately (FILE-001).

**Generate Official Document** — `type` (marksheet/receipt/payslip), `sourceRef`. Validation: source finalized/immutable (FILE-007 / UC-FILE-015 — generation from non-finalized data blocked); reproducible from stamped versions; immutable artifact.

**Signed-URL Access** — `file`, `requester`. Validation: subject-scoped + custody-aware (FILE-004/GRD-N-006); short-lived scoped URL (FILE-005); public/unauthorized blocked (FILE-001 / UC-FILE-014).

**Config** — `allowedTypes`, `requiredDocs`, `quota`, `retention`. Validation: typed/versioned (CFG-002/004); retention respects C-08.

---

## 8. Search & Filter Requirements

**Files:** by subject (student/staff), type, status (scanning/available/quarantined), generated vs uploaded, date, version. Sorting: date/type. Pagination: server-side, 25 default. Scope + custody enforced; restricted parties see nothing.

---

## 9. Table Requirements

**File table:** Name, Subject, Type, Status, Version, Generated?, Uploaded By, Date. Sorting on Date/Type. Filtering as above. Download via signed-URL only (no direct links). Export (governed — metadata, not bulk content). Bulk: upload, generate.

---

## 10. Workflow Requirements

**Trigger events:** upload, scan complete (clean/infected), generate, version, signed-URL request, retention/deletion, quota check. **Status changes:** file `UPLOADING → SCANNING → AVAILABLE/QUARANTINED → ARCHIVED/DELETED`; generated docs are terminal-immutable. **Approvals:** secure deletion and quarantine release governed. **Notifications:** document available, required-doc missing, quarantine alert. **Audit:** uploads, scans (result), generation, access (signed-URL issuance), deletions (immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Upload file | `file.upload` |
| Access/download (signed-URL) | `file.access` (subject-scoped, custody-aware) |
| Generate official document | `file.generate` |
| Manage versions | `file.version.manage` |
| Configure types/required-docs/quota | `file.config.manage` |
| Manage retention/secure-deletion | `file.retention.manage` |
| Release/discard quarantine | `file.quarantine.manage` |
| Storage/access report | `file.report.view` |

Access is subject-scoped and custody-aware (FILE-004); files served only via signed URLs (FILE-005).

---

## 12. Business Rule References

FILE-001 (private storage, no public access), FILE-002 (upload validation — type/size), FILE-003 (malware scan before availability), FILE-004 (subject-scoped, custody-aware access), FILE-005 (short-lived scoped signed URLs), FILE-006 (versioning & metadata hygiene), FILE-007 (generated official documents immutable & reproducible), FILE-008 (retention, secure deletion & quota). Cross-cutting: D46 (private storage/signed URLs/scan), GRD-N-006 (custody restrictions), RES-001/FEE-003/HR-003 (reproducible source for generated docs), CFG-004 (config versioning), C-08 (retention precedence), AUD-001.

## 13. Use Case References

UC-FILE-001 (Upload, Scan & Make Available), UC-FILE-002 (Access via Signed URL — custody-aware), UC-FILE-003 (Generate Official Document — immutable, reproducible), UC-FILE-004 (Version & Strip Metadata), UC-FILE-005 (Retention, Secure Deletion & Quota), UC-FILE-006 (View/List — scoped), UC-FILE-007 (Configure Types & Required Documents), UC-FILE-008 (Search), UC-FILE-009 (Storage & Access Report), UC-FILE-010 (Bulk Upload/Generation), UC-FILE-011 (Export/Download — signed-URL), UC-FILE-012 (Required-Document Checklist), UC-FILE-013 (Malware Detected — quarantine), UC-FILE-014 (Public-Exposure/Unauthorized Access Blocked), UC-FILE-015 (Generation From Non-Finalized Data Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Upload file (→ scan) | POST | Uploader |
| Request signed URL (custody-aware) | POST | Authorized requester |
| Generate official document | POST | System / Module |
| Version document (strip metadata) | POST | Uploader |
| List / view files (scoped) | GET | Authorized roles |
| Required-document checklist | GET | Admin |
| Configure types/required-docs/quota | GET/PUT | Admin |
| Apply retention / secure-delete | POST | Admin (→ governed) |
| Release / discard quarantine | POST | Admin |
| Storage & access report | GET | Admin |

Files are never public — access only via short-lived scoped signed URLs after a clean scan (FILE-001/003/005); official documents generate only from finalized data and are immutable (FILE-007).

---

## 15. Database Requirements

**Entities:** `File` (subjectRef, type, status, storageKey, version, generated, checksum), `FileVersion` (history), `GeneratedDocument` (sourceRef, stamped versions, immutable, checksum), `Quarantine` (scan result), `RetentionPolicy`/`Quota` (via Config), `SignedUrlGrant` (scope, expiry). **Relationships:** Subject 1—* File; File 1—* FileVersion; File 1—1 GeneratedDocument (if generated). **Indexes:** index(File.subjectRef, type), index(File.status), unique(GeneratedDocument.sourceRef, type, version), index(SignedUrlGrant.expiry). Private storage, no public ACL (FILE-001); checksums + immutability for generated docs (FILE-007); metadata stripped at ingest (FILE-006).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Document available (signed-URL); required-document missing; generation complete. |
| In-App | Upload/scan status; quarantine alerts; quota warnings. |
| SMS/Push | Optional document-ready alert (minimized). |

Access links are short-lived signed URLs (FILE-005); custody-aware routing (NOT-002/GRD-N-006).

---

## 17. Audit Requirements

Log: uploads (type/size/subject), scan results (clean/infected → quarantine), generation (source, stamped versions, checksum), version replacements (metadata-stripped), signed-URL issuance (who/scope/expiry), access, retention/secure-deletion. Record who/when. Generated-document immutability and access are first-class audit events. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Storage utilization (by scope), Access log (signed-URL issuance), Required-document completeness, Quarantine/malware incidents, Retention/deletion. **Exports:** governed metadata export (not bulk content). **Dashboards:** file health (storage, quota, quarantine, generation volume).

---

## 19. Error Handling

**Validation:** invalid type/size, generation from non-finalized data → specific errors (UC-FILE-015). **Permission:** public/unauthorized/custody-restricted access → blocked + audited (UC-FILE-014). **Workflow:** quarantine release/secure-deletion pending → governed. **System:** scan service unavailable → file held in quarantine (never auto-available); storage unavailable → upload fails cleanly (no partial).

---

## 20. Edge Cases

**Concurrent updates:** concurrent version uploads → serialized versions, latest current. **Duplicate data:** identical re-upload → dedup by checksum (optional) or new version. **Partial failures:** bulk upload/generation partial → per-file report. **Rollback:** generation from later-revised source → new immutable version; prior artifact retained (reproducible). **Scan race:** access requested during scan → denied until clean (FILE-003).

---

## 21. Acceptance Criteria

**Functional.** Files are stored privately with no public access and served only via short-lived scoped signed URLs; uploads are type/size-validated and malware-scanned (quarantined) before becoming available; access is subject-scoped and custody-aware (restricted parties blocked); documents are versioned with metadata (EXIF/GPS) stripped; generated official documents are immutable, reproducible from stamped source versions, and never generated from non-finalized data; retention, secure deletion, and quota are enforced.

**Business.** Every file — uploaded or generated — is private, safe, and custody-respecting; minors' location metadata never leaks; official documents are tamper-proof and reproducible; storage is governed and compliant.

---

## 22. Future Enhancements

Client-side encryption; configurable virus-scan engines; document watermarking/QR verification for official docs; OCR/auto-classification of uploads; deduplication across the deployment; tiered/cold storage for retention; preview rendering without download; per-document access expiry policies.
