# Payment Method Configs

## 1. Table Purpose

Enabled payment methods and the deterministic allocation rule (PAY-002/003) — backs `payment.api.md` #24–25.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `enabled_methods` | `TEXT[]` | No | — |
| `allocation_rule` | `allocation_rule (ENUM: OLDEST_FIRST, SPECIFIC_INVOICE, HEAD_PRIORITY)` | No | 'OLDEST_FIRST' |
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

Read by `payments.api.md` #2/#3's allocation-preview and actual-allocation logic.

## 8. Soft Delete Strategy

Standard; updated in place like `late_fine_policies`.

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

Identical small-config-table shape to `late_fine_policies`/`auth_policies` — kept consistent deliberately rather than varying the pattern per module.

---

## 12. Performance Considerations

**Trivially low volume (one row per institute).** Cache at the application layer; read on every payment-recording call.