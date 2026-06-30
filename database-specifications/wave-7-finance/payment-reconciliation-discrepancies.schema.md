# Payment Reconciliation Discrepancies

## 1. Table Purpose

Flagged mismatches between recorded payments and bank/gateway settlement data (PAY-005) — backs `payment.api.md` #16–17. References the importing `async_jobs` row directly, per §0.5.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `import_job_id` | `UUID` | No | — |
| `type` | `discrepancy_type (ENUM: MISSING, EXTRA, AMOUNT_MISMATCH)` | No | — |
| `recorded_payment_id` | `UUID` | Yes | — |
| `settlement_ref` | `VARCHAR(255)` | Yes | — |
| `recorded_amount` | `NUMERIC(10,2)` | Yes | — |
| `settled_amount` | `NUMERIC(10,2)` | Yes | — |
| `status` | `discrepancy_status (ENUM: OPEN, RESOLVED)` | No | 'OPEN' |
| `resolution` | `VARCHAR(32)` | Yes | — |
| `resolution_reason` | `TEXT` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`import_job_id → async_jobs(id)` RESTRICT; `recorded_payment_id → payments(id)` RESTRICT, nullable (NULL for an `EXTRA` settlement entry with no matching recorded payment at all).

## 5. Unique Constraints

None beyond the PK.

## 6. Indexes

`(import_job_id, status)`; `(status)` for the cross-batch discrepancy inbox.

## 7. Relationships

Many discrepancies per import job; optionally one to one `payments` row.

## 8. Soft Delete Strategy

Standard; retained as the permanent reconciliation audit trail.

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

**Derived, via `import_job_id → async_jobs → institute_id`** (and, where set, `recorded_payment_id → payments → students → institute_id`).

## 11. Notes for TypeORM Entity Design

Auto-matching (`payment.api.md` #15) only ever resolves *exact* reference/amount matches directly — anything ambiguous always lands here as a row, never silently force-matched, exactly as that endpoint's own notes specify; this table's very existence for an ambiguous case **is** the enforcement of that rule, not a separate constraint.

---

## 12. Performance Considerations

**Low volume relative to `payments`** (discrepancies should be a small fraction of total transaction volume in a healthy reconciliation process). No special performance concerns.