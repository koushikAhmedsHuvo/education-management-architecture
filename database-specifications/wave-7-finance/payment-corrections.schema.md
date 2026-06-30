# Payment Corrections

## 1. Table Purpose

Governed reversal and mis-post correction, unified per §0.4 — backs `payment.api.md` #20–23.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `payment_id` | `UUID` | No | — |
| `correction_type` | `payment_correction_type (ENUM: REVERSAL, MISPOST)` | No | — |
| `amount` | `NUMERIC(10,2)` | Yes | — |
| `correct_target_student_id` | `UUID` | Yes | — |
| `correct_target_invoice_id` | `UUID` | Yes | — |
| `reason` | `TEXT` | No | — |
| `status` | `change_request_status (reuses the Wave-5 ENUM)` | No | 'PENDING_APPROVAL' |
| `requested_by` | `UUID` | No | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`payment_id → payments(id)` RESTRICT; `correct_target_student_id → students(id)` RESTRICT, nullable; `correct_target_invoice_id → invoices(id)` RESTRICT, nullable; `requested_by → users(id)` RESTRICT; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK.

## 6. Indexes

`(status, correction_type)` — the combined approvals inbox.

## 7. Relationships

On approval, a `REVERSAL` sets `payments.status = 'REVERSED'` and inserts a negative `credit_balance_entries` row (or, where the amount was never disbursed beyond a credit, simply restores dues) — never deletes the `payments` row; a `MISPOST` correction's approval moves the *allocation*, not the payment record itself (writing new `payment_allocations` rows against the corrected student/invoice and reversing the original ones), again never touching the immutable `payments` row.

## 8. Soft Delete Strategy

Standard; retained permanently as the disclosed correction history.

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

**Derived, via `payment_id → payments → students → institute_id`.** No direct institute column.

## 11. Notes for TypeORM Entity Design

This table's approval-commit logic is the most consequential transaction boundary in the Payment domain after recording itself — both correction types must, in one transaction, update `payments.status` (for `REVERSAL`) or write the new/reversed `payment_allocations` (for `MISPOST`), and in either case **never** issue a `DELETE` against `payments`, structurally guaranteed by that table having no delete path at all (§8).

---

## 12. Performance Considerations

**Very low volume** (reversals and mis-post corrections should both be rare exceptions in a well-functioning collection process). No special performance concerns.