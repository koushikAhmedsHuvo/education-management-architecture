# Scholarship Disbursements

## 1. Table Purpose

The append-only disbursement ledger — coverage application and stipend payouts (SCH-004) — backs `scholarship.api.md` #10–11. Append-only per §0.3.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `award_id` | `UUID` | No | — |
| `type` | `disbursement_type (ENUM: COVERAGE, STIPEND)` | No | — |
| `amount` | `NUMERIC(10,2)` | No | — |
| `invoice_id` | `UUID` | Yes | — |
| `status` | `disbursement_status (ENUM: DISBURSED, PENDING_APPROVAL)` | No | 'DISBURSED' |
| `period` | `VARCHAR(32)` | Yes | — |
| `disbursed_at` | `TIMESTAMPTZ` | Yes | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`award_id → scholarship_awards(id)` RESTRICT; `invoice_id → invoices(id)` RESTRICT, nullable; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

None — many disbursements per award over time (each stipend period, each coverage application), by design.

## 6. Indexes

`(award_id, status)`.

## 7. Relationships

Many-to-one with `scholarship_awards`; `COVERAGE`-type rows write into the parent `invoice_id`'s `lines[].scholarshipApplied` field (the same JSONB-annotation pattern as `discounts`).

## 8. Soft Delete Strategy

Not soft-deleted — every disbursement is permanent ledger truth.

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

**Derived, via `award_id → scholarship_awards → scholarship_programs → institute_id`.** No direct institute column.

## 11. Notes for TypeORM Entity Design

This table, together with `credit_balance_entries` (§11), is the schema's second demonstration of the "balances are computed from an append-only ledger" principle — both deliberately avoid a mutable running-total column for the identical drift-prevention reason.

---

## 12. Performance Considerations

**Low-to-moderate volume, recurring per stipend period for active awards.** The fund-sufficiency atomic-guard pattern (mirroring `scholarship_programs.slots_filled`) carries the identical contention consideration during concentrated disbursement-run windows (e.g., a term-start stipend batch covering many awards from the same program's fund simultaneously).