# Discounts

## 1. Table Purpose

Applied discounts/waivers — transparent, capped, governed (DSC-001…007) — backs `discount.api.md` #1–7.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `student_id` | `UUID` | No | — |
| `invoice_id` | `UUID` | No | — |
| `fee_head_id` | `UUID` | Yes | — |
| `classification` | `discount_classification (ENUM: DISCOUNT, WAIVER)` | No | — |
| `discount_type_id` | `UUID` | Yes | — |
| `type` | `discount_value_type (ENUM: PERCENTAGE, FIXED)` | No | — |
| `amount` | `NUMERIC(10,2)` | No | — |
| `reason` | `TEXT` | Yes | — |
| `status` | `discount_status (ENUM: ELIGIBLE, PENDING_APPROVAL, APPROVED, APPLIED, REVERSED, EXPIRED, REJECTED)` | No | 'APPLIED' |
| `expiry_condition` | `JSONB` | Yes | — |
| `applied_by` | `UUID` | No | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`student_id → students(id)` RESTRICT; `invoice_id → invoices(id)` RESTRICT; `fee_head_id → fee_heads(id)` RESTRICT, nullable; `applied_by → users(id)` RESTRICT; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK — multiple discounts can legitimately stack on one invoice (subject to `discount_configs.stacking_policy`'s combined cap, application-enforced).

## 6. Indexes

`(invoice_id, status)` — the dominant "what's applied to this invoice" query, also driving `invoices.lines[].discountApplied` resolution; `(student_id, status)`.

## 7. Relationships

Many-to-one with `invoices`; read by Scholarship's coverage logic for the SCH-006 interaction check.

## 8. Soft Delete Strategy

Standard `deleted_at`, though `status = 'REVERSED'`/`'EXPIRED'`/`'REJECTED'` are the disclosed lifecycle states actually exercised; a reversal (`discount.api.md` #7) is a status transition with a `reason`, never a row deletion, since DSC-007 requires the reversal itself to be "transparent and audited."

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

**Direct, via `student_id → students → institute_id` and `invoice_id → invoices → ... → institute_id`** (both expected to agree). No direct institute column.

## 11. Notes for TypeORM Entity Design

Net-floors-at-zero (DSC-004) and stacking-cap enforcement both require summing sibling `discounts` rows against the parent invoice's gross and remain application-layer, computed by the same `DiscountApplicationValidator` that resolves `discount_configs`.

---

## 12. Performance Considerations

**Moderate volume (a meaningful fraction of invoices carry at least one discount at many institutes), read on every invoice-detail view and every dues calculation.** The `(invoice_id, status)` index is the dominant access path, since invoice net-due resolution joins through this table on essentially every dues/collection screen. The `BEFORE INSERT`/`UPDATE` trigger enforcing the non-waivable gate adds a small, acceptable per-write cost (one indexed lookup into `fee_heads`) given how infrequently discounts are applied relative to how often invoices themselves are read.