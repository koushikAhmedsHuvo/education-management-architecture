# Documents

## 1. Table Purpose

The generic, polymorphic document/file table (FILE-003…007) — backs `student.api.md` #16–19 now, and will be reused unchanged by Examination's transcripts/admit-cards and Finance's receipts in later waves, per §0.3.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `entity_type` | `VARCHAR(64)` | No | — |
| `entity_id` | `UUID` | No | — |
| `document_type` | `VARCHAR(64)` | No | — |
| `storage_key` | `VARCHAR(512)` | No | — |
| `version` | `INTEGER` | No | 1 |
| `scan_status` | `document_scan_status (ENUM: PENDING, CLEAN, QUARANTINED)` | No | 'PENDING' |
| `immutable` | `BOOLEAN` | No | false |
| `uploaded_by` | `UUID` | No | — |
| `uploaded_at` | `TIMESTAMPTZ` | No | now() |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`uploaded_by → users(id)` RESTRICT. **No typed FK on `(entity_type, entity_id)`** — the same deliberate polymorphic-association trade-off as `audit_log`, justified for the identical reason: a generic document table referencing every possible owning entity with typed FKs is unworkable.

## 5. Unique Constraints

None beyond the PK — multiple versions of the same `document_type` for the same entity are expected and intentional (replace creates a new version row, never overwrites).

## 6. Indexes

`(entity_type, entity_id, document_type, version DESC)` — the dominant "this entity's documents, latest version first" query.

## 7. Relationships

Polymorphic, read by every later wave that needs file storage.

## 8. Soft Delete Strategy

Standard `deleted_at`; a "delete" via `student.api.md` #19 is a real soft-delete here (unlike most history-bearing tables in this schema, an uploaded document genuinely can be removed — it isn't load-bearing historical truth the way a mark or a payment is), gated by the governed-reason requirement the API specifies, not by a DB constraint.

## 9. Audit Fields

| Column | Type | Nullable | Default Value |
|---|---|---|---|
| `created_at` | `TIMESTAMPTZ` | No | `now()` |
| `created_by` | `UUID` | Yes | — |
| `updated_at` | `TIMESTAMPTZ` | No | `now()` |
| `updated_by` | `UUID` | Yes | — |
| `deleted_at` | `TIMESTAMPTZ` | Yes | — |
| `deleted_by` | `UUID` | Yes | — |

## 10. Multi-Institute Isolation Strategy

**Polymorphic/contextual**, same shape as `audit_log` — no institute column at all; isolation is enforced entirely by the application resolving `(entity_type, entity_id)` back to its owning institute before authorizing any read.

## 11. Notes for TypeORM Entity Design

The malware-scan worker (an `async_jobs`-tracked background process, Wave 1) flips `PENDING → CLEAN`/`QUARANTINED` asynchronously after upload — model the upload endpoint's immediate response as `{ id, status: 'SCAN_PENDING' }` exactly as `student.api.md` #16 already specifies, never as a synchronous scan. Photo uploads additionally strip EXIF/GPS metadata **before** this row is even written (FILE-006) — that stripping happens in the upload-handling service, not as a DB concern.

---

## 12. Performance Considerations

**Potentially very high volume at scale** (every student photo, every generated marksheet, every receipt, every transcript across the entire system's lifetime, with versioning multiplying uploaded-document counts further). The `(entity_type, entity_id, document_type, version DESC)` index is the dominant access path; this table stores only `storage_key` metadata, never file bytes, keeping individual rows small regardless of the underlying file size — a deliberate design choice that keeps this table's own performance characteristics independent of document size.