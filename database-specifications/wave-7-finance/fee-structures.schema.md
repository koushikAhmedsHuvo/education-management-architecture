# Fee Structures

## 1. Table Purpose

Versioned, effective-dated fee structures (FEE-001/006) — backs `fee-structure.api.md` #1–4. Append-only with an exclusion constraint, per §0.2.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `campus_id` | `UUID` | Yes | — |
| `class_id` | `UUID` | Yes | — |
| `category` | `VARCHAR(64)` | No | — |
| `heads` | `JSONB` | No | — |
| `version` | `INTEGER` | No | — |
| `effective_from` | `TIMESTAMPTZ` | No | — |
| `effective_until` | `TIMESTAMPTZ` | Yes | — |
| `status` | `config_value_status (reuses the Wave-2 ENUM)` | No | 'SCHEDULED' |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT; `campus_id → campuses(id)` RESTRICT; `class_id → classes(id)` RESTRICT.

## 5. Unique Constraints

```sql
ALTER TABLE fee_structures ADD CONSTRAINT fee_structures_no_overlap
EXCLUDE USING gist (
  institute_id WITH =,
  COALESCE(campus_id, '00000000-0000-0000-0000-000000000000'::uuid) WITH =,
  COALESCE(class_id,  '00000000-0000-0000-0000-000000000000'::uuid) WITH =,
  category WITH =,
  tstzrange(effective_from, effective_until) WITH &&
) WHERE (status IN ('SCHEDULED', 'PUBLISHED') AND deleted_at IS NULL);
```
The third application of this exact mechanism in the schema (§0.2) — direct DB-level enforcement of "no overlapping effective periods" for the same scope+category.

## 6. Indexes

`(institute_id, category, status)`.

## 7. Relationships

Read by `invoices` generation (§5), which stamps the resolved row's `version` onto `invoices.structure_version_stamped`.

## 8. Soft Delete Strategy

`deleted_at` present, essentially unused; a structure never edited in place once published — only superseded by a new version, identical lifecycle to `config_values`/`grade_scales`.

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

**Direct, mandatory, with nested scope** (`institute_id` + optional `campus_id`/`class_id`), the same shape as `grade_scales`.

## 11. Notes for TypeORM Entity Design

Resolution at invoice-generation time follows the identical most-specific-wins precedence as `config_values`: `class_id` match beats `campus_id`-only beats institute-wide, ordered explicitly in the repository query, never relying on implicit NULL-coalescing.

---

## 12. Performance Considerations

**Low volume (structures change infrequently — typically once or twice per session per scope), but the resolution query runs on every single invoice generation.** The `EXCLUDE` constraint's GiST overhead is well within acceptable bounds given how rarely structures are actually written compared to how often invoices (which read them) are generated.