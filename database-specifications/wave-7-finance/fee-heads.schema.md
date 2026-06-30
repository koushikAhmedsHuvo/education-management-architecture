# Fee Heads

## 1. Table Purpose

Fee head/category catalog with the authoritative non-waivable classification (FEE-002) — the source `discounts` (§16) reads to block a waiver on a protected head (the C-03 boundary already established in the API contract).

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `name` | `VARCHAR(255)` | No | — |
| `type` | `VARCHAR(64)` | No | — |
| `non_waivable` | `BOOLEAN` | No | false |
| `schedule` | `VARCHAR(32)` | No | — |
| `status` | `fee_head_status (ENUM: ACTIVE, INACTIVE)` | No | 'ACTIVE' |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK — the rules don't require head names to be unique, only the heads themselves to be individually addressable.

## 6. Indexes

`(institute_id, status)`.

## 7. Relationships

Referenced by id from inside `fee_structures.heads` and `invoices.lines` (both JSONB), and directly by `fines.invoice_id`'s parent invoice lines.

## 8. Soft Delete Strategy

Standard `deleted_at`; flipping `non_waivable` forward never retroactively unwinds waivers already applied under the old classification (an explicit, stated forward-only effect).

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

This table is the single source of truth `discounts.api.md`'s waiver-application validator queries before allowing a waiver — never duplicate `non_waivable` as a cached flag elsewhere.

---

## 12. Performance Considerations

**Low volume (a few dozen heads per institute), read on every invoice-generation and discount-application call, written rarely.** No special performance concerns; this small, stable table is an easy application-cache candidate given how frequently `non_waivable` specifically is consulted.