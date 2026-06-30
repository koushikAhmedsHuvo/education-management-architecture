# Late Fine Policies

## 1. Table Purpose

Configured late-fine amount/grace-period/cap (FEE-007) — backs `fee-structure.api.md` #16–17. A simple versioned-in-place config, like `auth_policies` (Wave 1), not date-ranged — the rules describe no historical re-resolution need for this policy.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `amount` | `NUMERIC(10,2)` | No | — |
| `grace_period_days` | `SMALLINT` | No | — |
| `cap` | `NUMERIC(10,2)` | No | — |
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

Read by `fines` (§6) at application time.

## 8. Soft Delete Strategy

Standard; never deleted in practice, only updated in place with `version` incrementing.

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

Identical shape and rationale to Wave 1's `auth_policies` — kept as its own dedicated table rather than routed through the generic Configuration Engine, for the same reason (a small, compound, institute-scoped policy object the approved API models as its own resource).

---

## 12. Performance Considerations

**Trivially low volume (one row per institute).** Cache at the application layer; read on every fine-application check during the dues-aging sweep.