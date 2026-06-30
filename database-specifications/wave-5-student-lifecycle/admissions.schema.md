# Admissions (Applications)

## 1. Table Purpose

The admission application lifecycle (ADM-001…007) — backs `admission.api.md` #1–17.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `session_id` | `UUID` | No | — |
| `level_class_id` | `UUID` | No | — |
| `application_number` | `VARCHAR(32)` | Yes | — |
| `form_id` | `UUID` | No | — |
| `form_version` | `INTEGER` | No | — |
| `field_values` | `JSONB` | No | '{}' |
| `document_ids` | `UUID[]` | No | '{}' |
| `stage` | `admission_stage (ENUM: DRAFT, SUBMITTED, UNDER_REVIEW, RETURNED_FOR_CORRECTION, WAITLISTED, APPROVED, ENROLLED, REJECTED, WITHDRAWN, EXPIRED)` | No | 'DRAFT' |
| `return_count` | `SMALLINT` | No | 0 |
| `assigned_officer_id` | `UUID` | Yes | — |
| `evaluation` | `JSONB` | Yes | — |
| `evaluation_rubric_version` | `INTEGER` | Yes | — |
| `waitlist_rank` | `SMALLINT` | Yes | — |
| `offer_expires_at` | `TIMESTAMPTZ` | Yes | — |
| `admission_fee_status` | `admission_fee_status (ENUM: NOT_REQUIRED, PENDING, SETTLED)` | No | 'NOT_REQUIRED' |
| `submitted_at` | `TIMESTAMPTZ` | Yes | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT; `session_id → academic_sessions(id)` RESTRICT; `level_class_id → classes(id)` RESTRICT; `form_id → dynamic_forms(id)` RESTRICT; `assigned_officer_id → users(id)` RESTRICT; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

`(application_number) WHERE application_number IS NOT NULL AND deleted_at IS NULL` — ADM-002's unique application number, assigned only on submit (a `DRAFT` row legitimately has no number yet, hence the partial predicate rather than `NOT NULL`).

## 6. Indexes

`(institute_id, session_id, stage)`; `(level_class_id, session_id, stage)` — the officer triage list's dominant filter; `(assigned_officer_id, stage)`.

## 7. Relationships

One application converts (at most once) into one `students` row, tracked via `students.converted_from_application_id`; many applications per (institute, session, level).

## 8. Soft Delete Strategy

Standard, though `stage ∈ {REJECTED, WITHDRAWN, EXPIRED}` is the disclosed terminal lifecycle.

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

**Direct, mandatory.** `institute_id` is NOT NULL; the `(level_class_id, session_id, stage)` index additionally scopes by the academic structure dimension, since the officer triage screen filters by both institute and intake level/session simultaneously.

## 11. Notes for TypeORM Entity Design

**Conversion** (`admission.api.md` #16) is the single most transaction-critical write in this entire wave — it must, in one transaction: verify `stage = 'APPROVED'`, verify `admission_fee_status = 'SETTLED'` (or fee-gating waived), verify a guardian link exists if the resulting student will be a minor, insert the new `students` row, insert the new `enrollments` row (capacity-checked against **currently active enrollments**, not other pending approvals — ADM-005's "confirmed-enrollment basis"), and update `stage = 'ENROLLED'` plus `students.converted_from_application_id` — any failure rolls back the entire set, leaving the application cleanly `APPROVED` and retryable, exactly as `admission.api.md` #16 itself specifies.

---

## 12. Performance Considerations

**Highly seasonal, bursty volume** — admission applications cluster heavily around intake periods rather than arriving steadily, unlike most operational tables in this schema. Expect significant write-load spikes during admission season specifically; the `(institute_id, session_id, stage)` and `(level_class_id, session_id, stage)` indexes are both sized for that bursty officer-triage access pattern, not steady-state load. The conversion transaction (§9's TypeORM notes) is this table's single most expensive write — it touches three tables atomically and should be monitored for lock contention during high-volume bulk-conversion windows specifically.