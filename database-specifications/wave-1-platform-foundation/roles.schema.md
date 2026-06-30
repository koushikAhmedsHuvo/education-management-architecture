# Roles

## 1. Table Purpose

Role definitions (Doc 02 §6 state machine: `ROLE_DRAFT → ROLE_ACTIVE → ROLE_DEPRECATED/ROLE_ARCHIVED`) — backs `role-permission.api.md` #1–6.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | Yes | — |
| `name` | `VARCHAR(128)` | No | — |
| `description` | `TEXT` | Yes | — |
| `type` | `role_type (ENUM: SYSTEM, CUSTOM)` | No | 'CUSTOM' |
| `scope_levels` | `TEXT[]` | No | — |
| `status` | `role_status (ENUM: ROLE_DRAFT, ROLE_ACTIVE, ROLE_DEPRECATED, ROLE_ARCHIVED)` | No | 'ROLE_DRAFT' |
| `cloned_from_role_id` | `UUID` | Yes | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT; `cloned_from_role_id → roles(id)` RESTRICT (self-referencing, nullable).

## 5. Unique Constraints

Two partial unique indexes (the standard "unique within scope, NULL-scope handled separately" pattern, also used by `auth_policies`): `(institute_id, name) WHERE institute_id IS NOT NULL AND deleted_at IS NULL`, and `(name) WHERE institute_id IS NULL AND deleted_at IS NULL`.

## 6. Indexes

`(institute_id, status)`; `(type)`.

## 7. Relationships

Many-to-many with `permissions` via `role_permissions`; one-to-many to `memberships`.

## 8. Soft Delete Strategy

Standard `deleted_at`, though `status = 'ROLE_ARCHIVED'` is the typical end-of-life path (Doc 02 §6) rather than deletion — both exist because archival is a disclosed lifecycle state visible in role-history UIs, while `deleted_at` remains reserved for true erroneous-row correction.

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

**Direct, nullable — same shape as `auth_policies`.** `institute_id IS NULL` rows are org-wide/system roles; `institute_id IS NOT NULL` rows are institute-specific custom roles. The dual partial-unique-index pattern (institute-scoped name uniqueness, separately org-wide name uniqueness) is what makes this nullable-scope model work correctly.

## 11. Notes for TypeORM Entity Design

`scope_levels` as a Postgres `text[]`, not a join table, since the set of valid scope levels is small and fixed (`institute`, `campus`) and is never queried "which roles apply at scope X" at a volume that would justify normalization. `workflow_instance_id` is declared `UUID NULL` with **no FK constraint yet** in this migration (added as a real FK in the Wave-3 migration once `workflow_instances` exists) — note this explicitly in the migration file comment so a future engineer doesn't mistake the omission for an oversight.

---

## 12. Performance Considerations

**Low-to-moderate volume (tens of roles per institute at most), read on every permission-resolution path via `memberships`.** The `(institute_id, status)` index supports the role-list screen; the real hot path for *authorization* itself runs through `memberships`/`role_permissions`, not this table directly, so `roles` itself is not a performance-critical table despite being conceptually central.