# Fee Collections (Invoices)

## 1. Table Purpose

Immutable, idempotently-generated invoices (FEE-003/004/005) — backs `fee-collection.api.md` #7–9.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `student_id` | `UUID` | No | — |
| `invoice_number` | `VARCHAR(32)` | No | — |
| `period` | `VARCHAR(32)` | No | — |
| `structure_version_stamped` | `INTEGER` | No | — |
| `lines` | `JSONB` | No | — |
| `total_net` | `NUMERIC(10,2)` | No | — |
| `status` | `invoice_status (ENUM: ISSUED, PARTIALLY_PAID, PAID, ADJUSTED, VOIDED, ARCHIVED)` | No | 'ISSUED' |
| `issued_at` | `TIMESTAMPTZ` | No | now() |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`student_id → students(id)` RESTRICT (Wave 5).

## 5. Unique Constraints

`(student_id, period) WHERE deleted_at IS NULL` — the direct DB-level form of FEE-004's duplicate-invoice prevention; a second `POST /invoices/generate` for the same student/period is rejected at the database level, not merely caught by application logic, the moment it attempts the insert.

## 6. Indexes

`(student_id, status)`; `(period)`.

## 7. Relationships

Referenced by `credit_notes` (§5), `fines` (§6), `payment_allocations` (§9 — wait, §9 is `payment_corrections`; allocations are §8), and `scholarship_disbursements` (coverage type).

## 8. Soft Delete Strategy

**None — this table has no soft-delete path at all, by design.** An issued invoice is never deleted, soft or otherwise; `status = 'VOIDED'`/`'ARCHIVED'` are the only retirement states, and even those never remove the row, since FEE-003's immutability guarantee is meaningless if the row itself could simply disappear.

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

**Direct, via `student_id → students → institute_id`.** No direct `institute_id` column on this table itself — unlike `enrollments`, this was *not* denormalized, since invoice queries are overwhelmingly student-scoped (a parent paying their child's fees) rather than institute-wide-listing-scoped in the dominant access pattern; revisit this choice if institute-wide invoice reporting ever becomes a more central use case than it is today.

## 11. Notes for TypeORM Entity Design

Generation (`fee-collection.api.md` #9) is the second-most transaction-critical write in this schema after admission conversion (Wave 5) — resolve the effective `fee_structures` row, apply any active `discounts`/scholarship coverage as line-item reductions, compute `total_net`, generate the receipt-style `invoice_number` (a simple per-institute sequence, **not** routed through the Wave-2 ID-generation engine, since invoice numbers don't need the token-pattern flexibility receipts do — a plain `nextval()`-backed sequence per institute suffices here), and insert — all in one transaction, with the unique-constraint violation on retry being the *expected*, idempotent-success path, not an error to surface to the caller.

---

## 12. Performance Considerations

**Very high volume** (one invoice per student per billing period, accumulating every term/month across the institute's entire operating history — never deleted, per its no-soft-delete design). The `(student_id, period) WHERE deleted_at IS NULL` unique constraint is this table's most safety-critical guarantee (FEE-004's duplicate prevention) and must remain a genuine, well-maintained index even as row count grows into the hundreds of thousands or millions at a large multi-year deployment. **Strong partitioning candidate** (by `period` or by session) given this unbounded, append-only growth pattern.