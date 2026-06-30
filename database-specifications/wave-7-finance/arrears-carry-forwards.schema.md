# Arrears Carry-Forwards

## 1. Table Purpose

Explicit, recorded carry-forward of outstanding dues into a new session (FEE-008) — backs `fee-collection.api.md` #21.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `student_id` | `UUID` | No | — |
| `from_session_id` | `UUID` | No | — |
| `to_session_id` | `UUID` | No | — |
| `amount` | `NUMERIC(10,2)` | No | — |
| `carried_by` | `UUID` | No | — |
| `carried_at` | `TIMESTAMPTZ` | No | now() |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`student_id → students(id)` RESTRICT; `from_session_id → academic_sessions(id)` RESTRICT; `to_session_id → academic_sessions(id)` RESTRICT; `carried_by → users(id)` RESTRICT.

## 5. Unique Constraints

`(student_id, from_session_id, to_session_id) WHERE deleted_at IS NULL` — a given arrears amount is carried forward between two specific sessions exactly once.

## 6. Indexes

`(student_id)`; `(from_session_id, to_session_id)`.

## 7. Relationships

A per-student record, deliberately separate from `invoices`/`fines` — it documents the *decision* to carry forward, not a new charge itself (the actual new-session due amount it produces is realized as a regular `invoices` line in the destination session).

## 8. Soft Delete Strategy

Standard; retained permanently as the disclosed, explicit record FEE-008 requires.

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

**Direct, via `student_id → students → institute_id`.** No direct institute column.

## 11. Notes for TypeORM Entity Design

Naturally invoked alongside `06-academic-session-api.md`'s rollover (Wave 1) — the rollover job's per-student processing, for any student with outstanding dues, inserts one row here per carried amount, never silently folding it into the new session's regular invoice without this explicit record existing first.

---

## 12. Performance Considerations

**Low volume, written in a single concentrated burst per institute per rollover event** (the entire population of students with outstanding dues, processed together during session rollover). No special performance concerns beyond ensuring the rollover job batches this insert rather than looping per student.