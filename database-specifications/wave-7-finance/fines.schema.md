# Fines

## 1. Table Purpose

Late fines as distinct, bounded charges — never silent invoice edits (FEE-007) — backs `fee-collection.api.md` #14–15.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `student_id` | `UUID` | No | — |
| `invoice_id` | `UUID` | No | — |
| `amount` | `NUMERIC(10,2)` | No | — |
| `basis` | `TEXT` | No | — |
| `status` | `fine_status (ENUM: APPLIED, PENDING_WAIVER_APPROVAL, WAIVED)` | No | 'APPLIED' |
| `waive_reason` | `TEXT` | Yes | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`student_id → students(id)` RESTRICT; `invoice_id → invoices(id)` RESTRICT; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK.

## 6. Indexes

`(invoice_id)`; `(student_id, status)`.

## 7. Relationships

Many-to-one with `invoices` (a fine is its own row, deliberately never merged into `invoices.lines`, per FEE-007's "added as a new line, never a silent edit" — here realized as a genuinely separate table rather than a JSONB line, for clean queryability of "all fines this period").

## 8. Soft Delete Strategy

Standard; `status = 'WAIVED'` is the disclosed lifecycle.

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

**Direct, via `student_id → students → institute_id`.** No direct institute column on this table, mirroring `invoices`' own student-scoped-access reasoning.

## 11. Notes for TypeORM Entity Design

Waiving a fine routes through the same `workflow_instance_id` governance pattern as everywhere else; on approval, `status → 'WAIVED'` and the invoice's effective due amount recomputes at read time, exactly as `credit_notes`' effect does.

---

## 12. Performance Considerations

**Moderate volume, concentrated around due-date boundaries** (fines apply in a burst whenever the dues-aging sweep runs past grace periods, not steadily). The `(invoice_id)` and `(student_id, status)` indexes support both the per-invoice and per-student access patterns this bursty write pattern produces.