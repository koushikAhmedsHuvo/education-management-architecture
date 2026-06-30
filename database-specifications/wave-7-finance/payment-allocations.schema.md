# Payment Allocations

## 1. Table Purpose

Deterministic allocation of a payment across invoices (PAY-002) — backs `payment.api.md` #2, #10.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `payment_id` | `UUID` | No | — |
| `invoice_id` | `UUID` | No | — |
| `allocated_amount` | `NUMERIC(10,2)` | No | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`payment_id → payments(id)` ON DELETE CASCADE — a true dependent of its payment, the same reasoning as `payment_allocations`' siblings throughout this schema (`refresh_tokens`, `workflow_tasks`, `marks`); `invoice_id → invoices(id)` RESTRICT.

## 5. Unique Constraints

`(payment_id, invoice_id) WHERE deleted_at IS NULL` — one allocation row per (payment, invoice) pair; a re-allocation (§9's own re-allocate endpoint) updates this row's `allocated_amount` in place rather than inserting a duplicate.

## 6. Indexes

`(invoice_id)` — "how much has this invoice received" rolls up from here; `(payment_id)` — covered implicitly by typical query patterns, an explicit index added since this is the hot path for `payment.api.md` #10's allocation view.

## 7. Relationships

Many allocations per payment (its split); many allocations per invoice (its payment history) — together, `SUM(allocated_amount) GROUP BY invoice_id` is exactly what drives `invoices.status` transitions (`ISSUED → PARTIALLY_PAID → PAID`).

## 8. Soft Delete Strategy

Standard, though re-allocation (an audited, governed action per the API) updates in place rather than soft-deleting and re-inserting, to keep the "current allocation state" query simple (no need to filter out superseded rows).

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

`invoices.status` is **not** denormalized/cached from this table via a trigger — it's recomputed at read time (or updated transactionally alongside the allocation write, in the same transaction, by the service layer) rather than via a database trigger, since the status transition also depends on `credit_notes` (§5) and `fines` (§6), making a single-table trigger insufficient anyway; centralizing the computation in one `InvoiceStatusResolver` service, called from every write path that could affect it, is more maintainable than a partial trigger plus partial application logic.

---

## 12. Performance Considerations

**Volume scales with `payments` directly** (typically 1–3 allocation rows per payment, for split payments across multiple invoices). The `(invoice_id)` index is the dues/aging-report's dominant aggregation path (`SUM(allocated_amount) GROUP BY invoice_id`) and should be monitored alongside `invoices`' own growth, since both tables grow together.