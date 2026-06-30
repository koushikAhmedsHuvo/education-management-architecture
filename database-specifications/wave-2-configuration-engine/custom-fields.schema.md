# Custom Fields

## 1. Table Purpose

The stable-identity registry of individually-addressable dynamic custom fields (CFG-010) — backs `custom-fields.api.md`.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `entity` | `VARCHAR(64)` | No | — |
| `institute_id` | `UUID` | Yes | — |
| `key` | `VARCHAR(64)` | No | — |
| `label` | `VARCHAR(255)` | No | — |
| `data_type` | `config_data_type (reuses the ENUM from §1)` | No | — |
| `options` | `JSONB` | Yes | — |
| `required` | `BOOLEAN` | No | false |
| `validation` | `TEXT` | Yes | — |
| `status` | `config_definition_status (reuses the ENUM from §1)` | No | 'REGISTERED' |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT.

## 5. Unique Constraints

Dual partial indexes, the same pattern as Wave 1's `roles`: `(entity, institute_id, key) WHERE institute_id IS NOT NULL AND deleted_at IS NULL`, and `(entity, key) WHERE institute_id IS NULL AND deleted_at IS NULL`.

## 6. Indexes

`(entity, institute_id, status)` — the dominant "fields available for this entity in this institute" query, which also covers org-wide fields via a second pass with `institute_id IS NULL` (handled in the repository method as a `UNION` or `OR`, not a single index trick, since "org-wide plus institute-specific" is inherently a two-source merge).

## 7. Relationships

Referenced *by id*, inside JSONB (not a typed FK — see `dynamic_forms` below), from form compositions; captured *values* of these fields live in each subject entity's own `dynamic_fields JSONB` column in later waves (e.g. Student in Wave 5), not in this table.

## 8. Soft Delete Strategy

None operative — `status = 'DEPRECATED'` is the retirement path, identical rationale to `config_definitions` and `permissions`: a captured historical value (e.g., a student's `dynamic_fields.bloodGroup` set last year) must remain meaningful even after the field definition is deprecated.

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

**Direct, nullable.** `institute_id IS NULL` rows are org-wide custom fields; institute-specific rows are the exception explicitly carved out by CFG-010's own worked example (an institute adding its own 'blood group' field).

## 11. Notes for TypeORM Entity Design

`data_type`/`options` deliberately reuse `config_definitions`' shapes rather than inventing a parallel type system — both ultimately feed the same generic "render and validate a typed field" client-side logic referenced in `dynamic-forms.api.md`'s Module Context (the Architecture's shared form-engine decision).

---

## 12. Performance Considerations

**Low-to-moderate volume (a few dozen custom fields per entity type at most), read whenever a dynamic form or a subject-entity's dynamic-field editor renders.** The `(entity, institute_id, status)` index supports this cheaply; given the low row count, this table is not expected to need caching beyond ordinary connection-level reuse.