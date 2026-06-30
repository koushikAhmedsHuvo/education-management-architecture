# Dynamic Forms

## 1. Table Purpose

Versioned, ordered compositions of `custom_fields` into structured forms (e.g., an admission application) — backs `dynamic-forms.api.md`.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `entity` | `VARCHAR(64)` | No | — |
| `institute_id` | `UUID` | Yes | — |
| `name` | `VARCHAR(255)` | No | — |
| `version` | `INTEGER` | No | — |
| `sections` | `JSONB` | No | — |
| `status` | `config_value_status (reuses the ENUM from §2: SCHEDULED/PUBLISHED/SUPERSEDED/ARCHIVED — a dynamic form is a draft/publish lifecycle, not effective-dated, so only the DRAFT-equivalent semantics of that enum are exercised; SCHEDULED is simply unused here)` | No | 'SCHEDULED' |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT. **No FK from `sections[].fields[].customFieldId` to `custom_fields.id`** — it is a UUID value nested inside JSONB, validated at write time by the service layer (`dynamic-forms.api.md` #3/#4's `422 UNREGISTERED_FIELD_SCHEMA`), the same accepted JSONB-reference trade-off as Wave 1's `sod_constraints.duty_a/duty_b`.

## 5. Unique Constraints

`(entity, institute_id) WHERE status = 'PUBLISHED' AND deleted_at IS NULL` — exactly one published form per entity+institute at a time, mirroring `custom_field_schemas`'-style single-current-version semantics rather than `config_values`' date-ranged history (forms don't need effective-dating per the approved API).

## 6. Indexes

`(entity, institute_id, status)`; GIN index on `sections` only if server-side "which forms reference field X" queries become necessary (omitted by default — that lookup is rare enough to do at the application layer by deserializing).

## 7. Relationships

Conceptually downstream of `custom_fields` (referenced by id within JSONB); submissions validated against a specific `dynamic_forms.version` are captured wherever the consuming module stores them (e.g., Admission's `application.field_values` JSONB in a later wave), stamped with `formVersion` for the same "historical values resolve against their version" guarantee CFG-010 requires.

## 8. Soft Delete Strategy

Standard `deleted_at`; in practice `status = 'ARCHIVED'` is the disclosed retirement path, `deleted_at` reserved for true erroneous-row correction.

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

**Direct, nullable**, identical shape to `custom_fields` — org-wide vs. institute-specific form compositions.

## 11. Notes for TypeORM Entity Design

Map `sections` as `Record<string, unknown>` JSONB, validated against a Zod/class-validator schema in the service layer on every write — never trust the column type alone to guarantee shape. `dynamic-forms.api.md` #6 (`Validate Form Submission`) reads this row, resolves each `customFieldId` against `custom_fields`, and validates the submitted values against each field's `data_type`/`options`/`required` — a pure read-and-compute operation needing no additional table.

---

## 12. Performance Considerations

**Low volume (a handful of forms per entity per institute), read on every form-render and every submission-validation call.** The JSONB `sections` payload is read as a whole on every access (never partially queried), so no GIN index is warranted at this table's expected scale; revisit only if 'which forms reference field X' becomes a genuinely frequent server-side query.