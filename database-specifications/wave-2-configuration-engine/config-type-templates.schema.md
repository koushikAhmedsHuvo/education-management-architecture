# Config Type Templates

## 1. Table Purpose

The institution-type seed catalog (CFG-011) — closes the forward reference left open in `01-platform-core.md` §15's notes on `institutes.institution_type` ("seeding its Configuration Engine template is a Wave 2 concern"). Read by `institute.api.md` #14 (`Apply Type Template`, Wave 1) to bulk-seed a new institute's initial `config_values`.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institution_type` | `VARCHAR(32)` | No | — |
| `definition_id` | `UUID` | No | — |
| `default_value` | `JSONB` | No | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`definition_id → config_definitions(id)` RESTRICT.

## 5. Unique Constraints

`(institution_type, definition_id) WHERE deleted_at IS NULL`.

## 6. Indexes

`(institution_type)` — the dominant "give me everything to seed for type X" query.

## 7. Relationships

Read (never written to by end users) at institute-creation/apply-template time; the seeded rows it produces are ordinary `config_values` rows scoped to the new institute, with no ongoing link back to the template row — CFG-011 explicitly notes seeded values "are then editable," i.e. independent from this point forward, which is exactly why no FK from `config_values` back to `config_type_templates` exists.

## 8. Soft Delete Strategy

Standard; a template entry is removed (soft-deleted) when a definition's recommended default for a given institution type changes going forward — this never retroactively touches institutes already seeded from it.

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

**Deployment-global.** A template entry (`institution_type` → recommended `config_values` defaults) is not institute-scoped at all — it's product-level seed data describing what *any* institute of a given type should start with.

## 11. Notes for TypeORM Entity Design

Populate via versioned seed migrations alongside each module's rollout (the same pattern as Wave 1's `permissions` seed data) — this is effectively curated product data, not end-user-editable through any exposed endpoint in the approved API contract.

---

## 12. Performance Considerations

**Trivially low volume (a few dozen rows per institution type at most), read exactly once per institute creation** (the apply-template operation). No performance concerns whatsoever.