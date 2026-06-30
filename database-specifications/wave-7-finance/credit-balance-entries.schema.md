# Credit Balance Entries

## 1. Table Purpose

The append-only overpayment/credit ledger (PAY-006), per §0.3 — backs `payment.api.md` #18–19.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `student_id` | `UUID` | No | — |
| `amount` | `NUMERIC(10,2)` | No | — |
| `source` | `credit_source (ENUM: OVERPAYMENT, REFUND_OFFSET, APPLIED_TO_INVOICE)` | No | — |
| `reference_id` | `UUID` | No | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`student_id → students(id)` RESTRICT. **No FK on `reference_id`** — it points to one of two different tables depending on `source`, the same accepted polymorphic trade-off as `audit_log`/`sod_constraints` elsewhere in this schema.

## 5. Unique Constraints

None — many entries per student over time, by design (this is a ledger, not a single-row state).

## 6. Indexes

`(student_id, created_at)` — both the balance-computation query and the history view read this in order.

## 7. Relationships

Conceptually downstream of `payments` (a source of credit) and `invoices` (a destination for applied credit).

## 8. Soft Delete Strategy

Not soft-deleted — every ledger entry is permanently retained; "current balance" is a query (`SUM(amount)`), never a stored, mutable figure, per §0.3.

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

`GET /payments/credit-balances` (`payment.api.md` #18) is `SELECT student_id, SUM(amount) AS balance FROM credit_balance_entries GROUP BY student_id HAVING SUM(amount) > 0` — a cheap aggregate at this table's expected volume; no materialized view needed at this scale, though one is a natural future optimization if credit-balance lookups ever become a genuine hot path.

---

## 12. Performance Considerations

**Low-to-moderate volume** (overpayments are the exception relative to exact-amount payments). The `(student_id, created_at)` index keeps the `SUM(amount)` balance-resolution query cheap even as entries accumulate over a student's entire enrollment history; revisit with a materialized balance column only if this aggregate ever becomes a measurably hot path at very large student counts.