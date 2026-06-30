# Enrollments

## 1. Table Purpose

The append-only placement record (ENR-001…008), per §0.4 — backs `enrollment.api.md` #1–6, #14.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `student_id` | `UUID` | No | — |
| `session_id` | `UUID` | No | — |
| `section_id` | `UUID` | No | — |
| `roll_number` | `INTEGER` | No | — |
| `period_from` | `TIMESTAMPTZ` | No | now() |
| `period_to` | `TIMESTAMPTZ` | Yes | — |
| `change_type` | `enrollment_change_type (ENUM: INITIAL, TRANSFER, PROMOTION, WITHDRAWAL)` | No | 'INITIAL' |
| `status` | `enrollment_status (ENUM: ACTIVE, SUPERSEDED, WITHDRAWN, GRADUATED)` | No | 'ACTIVE' |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`student_id → students(id)` RESTRICT; `session_id → academic_sessions(id)` RESTRICT; `section_id → sections(id)` RESTRICT (Wave 4).

## 5. Unique Constraints

**`(student_id, session_id) WHERE period_to IS NULL AND status = 'ACTIVE' AND deleted_at IS NULL`** — the precise DB-level enforcement of ENR-002 ("one active enrollment per student per session"); **`(section_id, roll_number) WHERE period_to IS NULL AND deleted_at IS NULL`** — ENR-004's roll-number uniqueness, scoped to currently-active occupants of that section only (a transferred-out student's old roll number is correctly available for reuse the moment their row closes).

## 6. Indexes

`(student_id, session_id)` — covered by the partial unique constraint below, no separate index needed; `(section_id, status) WHERE period_to IS NULL` — the capacity-count hot path (§10's TypeORM notes); `(student_id, period_from DESC)` — a student's full placement history, ordered.

## 7. Relationships

Many rows per student over time (their full placement history); many rows per section at any moment (its current roster, via the `period_to IS NULL` filter).

## 8. Soft Delete Strategy

Standard `deleted_at`, essentially unused — `status`/`period_to` together form the complete, queried lifecycle, identical rationale to every other append-only table in this schema.

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

**Direct, mandatory.** `institute_id` is denormalized directly onto this table specifically (rather than derived purely through `section_id`) precisely *for* performance — see Performance Considerations below; this is one of the few deliberate denormalizations in the schema, justified by how hot the uniqueness check it supports is.

## 11. Notes for TypeORM Entity Design

A transfer (governed via `enrollment_change_requests`, §11 below — note the *next* table, not this one) executes as: `UPDATE enrollments SET period_to = $effectiveDate, status = 'SUPERSEDED' WHERE id = $currentId; INSERT INTO enrollments (student_id, session_id, section_id, roll_number, period_from, change_type) VALUES (...)` — both statements in one transaction, identical discipline to Wave 4's class-teacher reassignment pattern (§4 of that file), for the identical reason: a crash between them would leave a student transiently un-enrolled or double-enrolled.

---

## 12. Performance Considerations

**One of the highest-volume, highest-criticality tables in the schema** — every student's placement history, growing every session, every transfer, every promotion (never updated in place, only appended). The two partial unique indexes — `(student_id, session_id) WHERE period_to IS NULL` and `(section_id, roll_number) WHERE period_to IS NULL` — are the schema's most safety-critical concurrency guarantees in Wave 5 and must remain genuine, dedicated indexes; the `(section_id, status) WHERE period_to IS NULL` index is the capacity-count hot path checked on every new enrollment and every admission conversion across the whole system. This table is a strong long-term partitioning candidate (by `session_id` or `period_from`) once multi-year history accumulates at scale.