# Mark Corrections

## 1. Table Purpose

Governed post-lock correction, original preserved (EXM-007) — backs `marks.api.md` #14, #15–16.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `mark_id` | `UUID` | No | — |
| `previous_mark` | `NUMERIC(6,2)` | Yes | — |
| `previous_special_outcome` | `mark_special_outcome` | Yes | — |
| `new_mark` | `NUMERIC(6,2)` | Yes | — |
| `new_special_outcome` | `mark_special_outcome` | Yes | — |
| `reason` | `TEXT` | No | — |
| `status` | `change_request_status (reuses the Wave-5 ENUM: PENDING_APPROVAL, APPROVED, REJECTED)` | No | 'PENDING_APPROVAL' |
| `requested_by` | `UUID` | No | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`mark_id → marks(id)` RESTRICT; `requested_by → users(id)` RESTRICT; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK — though application logic typically prevents two simultaneously-pending corrections on the same mark, this isn't a hard DB constraint, since a rejected correction followed by a fresh one is a legitimate sequence.

## 6. Indexes

`(status)` — the approvals inbox.

## 7. Relationships

Many-to-one with `marks`; on approval, this is the row that retains the "original" value (`previous_mark`/`previous_special_outcome`) once `marks.mark` itself is updated to `new_mark` — this is the table that gives EXM-007's "original preserved" its actual storage, since `marks` itself (§4) was deliberately *not* made append-only.

## 8. Soft Delete Strategy

Standard; retained indefinitely as the disclosed correction history.

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

**Derived, via `mark_id → mark_sheets → exams → institute_id`.** No direct institute column.

## 11. Notes for TypeORM Entity Design

Approval-commit is a single transaction: snapshot the current `marks` row's values into `previous_mark`/`previous_special_outcome` (if not already captured at proposal time — capturing at proposal time, as modeled here, is preferable, since it reflects the value *as of the request*, not whatever it happens to be at decision time), update `marks` to the new value, and — conditionally — enqueue a `result_revisions` row referencing the affected `results` row(s) for this student/exam.

---

## 12. Performance Considerations

**Very low volume** (post-lock corrections should be rare exceptions, not routine). No performance concerns.