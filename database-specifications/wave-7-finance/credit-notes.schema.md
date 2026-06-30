# Credit Notes

## 1. Table Purpose

The sole correction path for an issued invoice (FEE-003/008) — backs `fee-collection.api.md` #11.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `invoice_id` | `UUID` | No | — |
| `amount` | `NUMERIC(10,2)` | No | — |
| `reason` | `TEXT` | No | — |
| `status` | `credit_note_status (ENUM: ISSUED, PENDING_APPROVAL)` | No | 'ISSUED' |
| `issued_by` | `UUID` | No | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`invoice_id → invoices(id)` RESTRICT; `issued_by → users(id)` RESTRICT; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK — multiple credit notes against one invoice are permitted (sequential partial adjustments).

## 6. Indexes

`(invoice_id)`.

## 7. Relationships

Many-to-one with `invoices`; the invoice's own `lines`/`total_net` are **never** modified by a credit note — its effect is computed at read time as `invoice.total_net − SUM(credit_notes.amount WHERE invoice_id = X AND status = 'ISSUED')`, and `invoices.status` transitions to `'ADJUSTED'` as a side effect, the only field on `invoices` a credit note is ever allowed to touch.

## 8. Soft Delete Strategy

Standard `deleted_at`; in practice never removed, retained as the permanent correction record.

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

**Derived, via `invoice_id → invoices → students → institute_id`.** No direct institute column.

## 11. Notes for TypeORM Entity Design

This table is the clearest, most literal embodiment of "correction without mutation" in this entire schema — `invoices` itself is genuinely immutable (§4), and every correction anyone will ever see is a `credit_notes` row layered transparently alongside it.

---

## 12. Performance Considerations

**Low volume relative to `invoices`** (corrections are the exception, not the norm). No special performance concerns.