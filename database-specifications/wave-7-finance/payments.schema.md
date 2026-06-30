# Payments

## 1. Table Purpose

The central money table — gapless, immutable receipts (PAY-001), idempotent recording (PAY-008) — backs `payment.api.md` #1–9.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `student_id` | `UUID` | No | — |
| `receipt_number` | `VARCHAR(32)` | No | — |
| `amount` | `NUMERIC(10,2)` | No | — |
| `method` | `payment_method (ENUM: CASH, CHEQUE, BANK_TRANSFER, GATEWAY)` | No | — |
| `reference` | `VARCHAR(255)` | Yes | — |
| `status` | `payment_status (ENUM: RECORDED, PENDING_CLEARANCE, CLEARED, BOUNCED, REVERSED, ARCHIVED)` | No | 'RECORDED' |
| `idempotency_key` | `VARCHAR(128)` | No | — |
| `recorded_by` | `UUID` | No | — |
| `recorded_at` | `TIMESTAMPTZ` | No | now() |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`student_id → students(id)` RESTRICT; `recorded_by → users(id)` RESTRICT.

## 5. Unique Constraints

`(receipt_number)` — globally unique, never reused, not soft-delete-partial (PAY-001's gapless immutability applies regardless of row lifecycle); `(idempotency_key)` — globally unique, the direct DB-level backstop behind PAY-008's "no double-credit."

## 6. Indexes

`(student_id, status)`; `(receipt_number)` — covered by the unique constraint; `(status) WHERE status = 'PENDING_CLEARANCE'` — the clearance-sweep's scan path.

## 7. Relationships

One payment has many `payment_allocations` (its split across invoices); referenced by `payment_corrections` (reversal/mis-post) and `payment_reconciliation_discrepancies`.

## 8. Soft Delete Strategy

**None — like `invoices`, this table has no soft-delete path.** A payment is reversed (`status = 'REVERSED'`, via `payment_corrections`, §9), never deleted — PAY-007 is explicit that "delete payment" is never offered anywhere in the system, and this schema makes that structurally true by simply never including `deleted_at` in this table's operative lifecycle (the column exists for cross-schema consistency but a `BEFORE DELETE` trigger blocks hard deletion unconditionally, with no exception path at all — the one table in this entire schema where even the rare "true erroneous-row correction" carve-out other tables get does not apply).

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

**Direct, via `student_id → students → institute_id`.** No direct institute column — the same student-scoped-access reasoning as `invoices`, its closest sibling table.

## 11. Notes for TypeORM Entity Design

Recording a payment is a single transaction: resolve the receipt number via `id_generation_sequences`' atomic UPSERT (§0.1), insert this row, compute and insert the `payment_allocations` rows (§9... actually allocations are the next table, renumber reference), and — if the allocated amount exceeds outstanding dues — insert a `credit_balance_entries` row for the excess (§11) — all atomically, so a payment is never recorded with a dangling, unallocated remainder.

---

## 12. Performance Considerations

**Very high volume, the single most write-contended table in the Finance domain** — every fee collection across the institute's entire history, never deleted. The `(receipt_number)` and `(idempotency_key)` unique indexes are this schema's two most safety-critical financial-integrity guarantees overall and must be monitored for bloat/performance as row count grows into the millions at scale. **Strong partitioning candidate** (by `recorded_at`), mirroring `invoices`' own recommendation, given the identical unbounded-growth, never-deleted access pattern.