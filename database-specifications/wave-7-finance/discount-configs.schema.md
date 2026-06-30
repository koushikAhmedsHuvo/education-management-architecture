# Discount Configs

## 1. Table Purpose

Discount type catalog, stacking policy, and tiered approval authority (DSC-001/003/004/005) — backs `discount.api.md` #8–9.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `discount_types` | `JSONB` | No | — |
| `stacking_policy` | `JSONB` | No | — |
| `approval_tiers` | `JSONB` | No | — |
| `version` | `INTEGER` | No | 1 |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT.

## 5. Unique Constraints

`(institute_id) WHERE deleted_at IS NULL`.

## 6. Indexes

None beyond the unique constraint.

## 7. Relationships

Read by `discounts` (§15) at apply-time to resolve type definitions, stacking order, and the required approval tier for a given amount.

## 8. Soft Delete Strategy

Standard; updated in place, same shape as §3/§13.

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

**Direct, mandatory.** `institute_id` is NOT NULL.

## 11. Notes for TypeORM Entity Design

`discount_types[].id` values are referenced by `discounts.discount_type_id` (§15) as a JSONB-embedded reference, the same accepted trade-off as `dynamic_forms`' field references (Wave 2) — the type catalog is small and rarely changes, so re-validating membership at write time is cheap.

---

## 12. Performance Considerations

**Trivially low volume (one row per institute).** Cache at the application layer; read on every discount-application call.