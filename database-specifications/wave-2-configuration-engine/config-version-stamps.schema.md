# Config Version Stamps

## 1. Table Purpose

A trigger-maintained, O(1) cache-invalidation signal per scope (CFG-008) — backs `settings.api.md` #11.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | Yes | — |
| `campus_id` | `UUID` | Yes | — |
| `config_version` | `BIGINT` | No | 1 |
| `last_bumped_at` | `TIMESTAMPTZ` | No | now() |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT; `campus_id → campuses(id)` RESTRICT.

## 5. Unique Constraints

`(COALESCE(institute_id, '00000000-...'::uuid), COALESCE(campus_id, '00000000-...'::uuid))` — the same nil-UUID idiom as §2's exclusion constraint, here as a plain unique index since no range overlap is involved, just "exactly one stamp row per scope."

## 6. Indexes

None beyond the unique constraint (tiny table, point lookups only — by design, this table must stay small and hot in the buffer cache).

## 7. Relationships

Conceptually downstream of `config_values` (bumped by it), referenced by nothing else.

## 8. Soft Delete Strategy

Not applicable — rows are never deleted, only their counter incremented; a scope that never receives a config write simply never gets a stamp row (clients treat a missing row as "version 0 / never configured").

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

**Direct, nullable — mirrors `config_values`' own scope columns**, since its entire purpose is signaling change within that same scope hierarchy.

## 11. Notes for TypeORM Entity Design

Install the trigger via a raw-SQL migration; do **not** model the bump as an application-layer `afterInsert` TypeORM subscriber, since that would run in the same transaction but outside the database's own atomicity guarantee in the face of bulk/raw inserts (`settings.api.md` #20's bulk-set path) that might bypass the entity-level subscriber entirely.

---

## 12. Performance Considerations

**Extremely low row count (one row per scope that has ever received a config write), but very high read frequency** — this table exists purely to make cache-validity checks O(1), so its own query pattern is about as cheap as a database read can be: a single-row point lookup by primary key equivalent. The trigger-based bump on every `config_values` insert adds negligible overhead to that already-infrequent write path.