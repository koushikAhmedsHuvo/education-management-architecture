# Async Jobs

## 1. Table Purpose

The single platform-wide tracking table for every deferred/background operation across all waves (bulk invite, bulk transfer, bulk mark-entry import, bulk compute, etc.) — backs the API conventions' shared `GET /api/v1/jobs/{jobId}` endpoint.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `job_type` | `VARCHAR(64)` | No | — |
| `institute_id` | `UUID` | Yes | — |
| `requested_by` | `UUID` | No | — |
| `idempotency_key` | `VARCHAR(128)` | No | — |
| `status` | `job_status (ENUM: QUEUED, RUNNING, COMPLETED, FAILED)` | No | 'QUEUED' |
| `progress_processed` | `INTEGER` | No | 0 |
| `progress_total` | `INTEGER` | Yes | — |
| `payload` | `JSONB` | No | — |
| `result` | `JSONB` | Yes | — |
| `started_at` | `TIMESTAMPTZ` | Yes | — |
| `completed_at` | `TIMESTAMPTZ` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`requested_by → users(id)` (RESTRICT); `institute_id → institutes(id)` (RESTRICT).

## 5. Unique Constraints

`(idempotency_key)` — globally unique, **not** soft-delete-partial (idempotency keys are client-generated UUIDs and must never collide regardless of row lifecycle); this is what makes "replay the same key" return the original job rather than starting a duplicate, the exact guarantee every bulk endpoint's contract promises.

## 6. Indexes

`(requested_by, status)`; `(idempotency_key)` UNIQUE (see below); `(institute_id, job_type, status)`.

## 7. Relationships

None inbound — `async_jobs` is a leaf table other tables do not reference (a job's *effects* land in the relevant domain tables directly; the job row itself is a process-tracking record, not a business entity other rows point to).

## 8. Soft Delete Strategy

`deleted_at` present but rarely used; completed jobs are retained for audit/debugging per a time-based retention policy (purge job, outside schema scope), not soft-deleted by user action.

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

**Direct, nullable.** `institute_id` is populated where the triggering bulk operation is institute-scoped; platform-level jobs (rare) may have it NULL. Job rows are only ever read by their own `requested_by` user or a caller holding the same permission the originating operation required — isolation here is ownership-based more than tenant-based, since a job is inherently a personal/operational record, not shared business data.

## 11. Notes for TypeORM Entity Design

`progress_total` nullable because some job types (e.g. a streaming CSV import) don't know their total row count until parsing completes. Use a `@Column({ type: 'jsonb' })` for `payload`/`result` with no typed interface enforced at the column level — validate shape in the service layer per `job_type`, since a single column can't usefully type-discriminate 20+ job shapes. Consider a BullMQ-backed worker writing status transitions back to this table rather than the table itself being the queue (the Architecture's documented "Background Jobs" pattern: Postgres is the durable record, Redis/BullMQ is the execution queue).

---

## 12. Performance Considerations

**Moderate write volume, short-lived rows.** Each bulk operation across all seven waves writes one row here plus periodic `progress_processed` updates during execution — the `(idempotency_key)` unique index is the critical hot-path lookup on every replay/retry. Most rows are read a handful of times shortly after creation and rarely again; a time-based archival/purge job (outside this schema's scope) should clear `COMPLETED`/`FAILED` rows past a retention window to keep the table small and the indexes hot in cache.