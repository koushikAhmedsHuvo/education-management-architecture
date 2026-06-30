# Settings (Config Values)

## 1. Table Purpose

Scoped, versioned, effective-dated values resolved by most-specific-wins (CFG-003) — the engine's central table. Backs `settings.api.md` #5–10, #20.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `definition_id` | `UUID` | No | — |
| `institute_id` | `UUID` | Yes | — |
| `campus_id` | `UUID` | Yes | — |
| `section_id` | `UUID` | Yes | — |
| `value` | `JSONB` | No | — |
| `version` | `INTEGER` | No | — |
| `effective_from` | `TIMESTAMPTZ` | No | — |
| `effective_until` | `TIMESTAMPTZ` | Yes | — |
| `status` | `config_value_status (ENUM: SCHEDULED, PUBLISHED, SUPERSEDED, ARCHIVED)` | No | 'SCHEDULED' |
| `published_by` | `UUID` | Yes | — |
| `published_at` | `TIMESTAMPTZ` | Yes | — |
| `rolled_back_to_version` | `INTEGER` | Yes | — |
| `rollback_reason` | `TEXT` | Yes | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`definition_id → config_definitions(id)` ON DELETE RESTRICT; `institute_id → institutes(id)` RESTRICT; `campus_id → campuses(id)` RESTRICT; `published_by → users(id)` RESTRICT.

## 5. Unique Constraints

```sql
-- requires: CREATE EXTENSION IF NOT EXISTS btree_gist;
ALTER TABLE config_values ADD CONSTRAINT config_values_no_overlap
EXCLUDE USING gist (
  definition_id WITH =,
  COALESCE(institute_id, '00000000-0000-0000-0000-000000000000'::uuid) WITH =,
  COALESCE(campus_id,    '00000000-0000-0000-0000-000000000000'::uuid) WITH =,
  COALESCE(section_id,   '00000000-0000-0000-0000-000000000000'::uuid) WITH =,
  tstzrange(effective_from, effective_until) WITH &&
) WHERE (status IN ('SCHEDULED', 'PUBLISHED') AND deleted_at IS NULL);
```
This is the direct DB-level enforcement of CFG-005 — two versions of the same (definition, scope) can never have overlapping `[effective_from, effective_until)` ranges, checked atomically by Postgres itself against concurrent inserts, not by an application-side "check then insert" race. The `COALESCE` to a nil UUID is the standard idiom for making NULL-scope rows (org-wide values) collide with each other correctly, since plain `=` treats `NULL = NULL` as unknown rather than true.

## 6. Indexes

`(definition_id, institute_id, campus_id, section_id, status)` — the dominant resolution-query index; `(definition_id, effective_from)` for version-history listing (`settings.api.md` #9, sorted `version DESC`); partial index `(status) WHERE status = 'SCHEDULED'` to make the activation sweep job (§0.5 below) cheap.

## 7. Relationships

Many `config_values` rows per `config_definitions` row (its full history-by-scope); read by every other wave's resolution logic.

## 8. Soft Delete Strategy

`deleted_at` is present but essentially unused in normal operation — a published value is **superseded** (`status = 'SUPERSEDED'`, `effective_until` set), never deleted; `deleted_at` is reserved for true erroneous-row correction by a platform operator, which itself requires re-validating the exclusion constraint isn't violated by the now-uncovered gap.

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

**Direct, nullable, hierarchical.** `institute_id`/`campus_id`/`section_id` together form the most-specific-wins scope chain (CFG-003); a NULL at any level means 'applies at the broader scope.' This is the schema's primary example of *configurable* tenant isolation — a value can be deliberately deployment-wide, institute-wide, or drilled down to one campus or section.

## 11. Notes for TypeORM Entity Design

A small scheduled job (`ConfigActivationSweepJob`, every minute or via `pg_cron`) promotes `SCHEDULED → PUBLISHED` the moment `effective_from` arrives and simultaneously sets the previous `PUBLISHED` row's `effective_until`/`status → SUPERSEDED` — this is CFG-005's "activate on schedule without manual intervention" requirement, and is exactly why `config_values` cannot be a purely passive table; something must tick it forward. Map the `EXCLUDE` constraint via a raw migration (`queryRunner.query(...)`), since TypeORM's schema-sync/decorators have no first-class support for exclusion constraints. Resolution itself (`GET /config/resolve`) is a repository method that queries this table ordered `section_id NOT NULL DESC, campus_id NOT NULL DESC, institute_id NOT NULL DESC LIMIT 1` filtered to `status = 'PUBLISHED' AND effective_from <= now AND (effective_until IS NULL OR effective_until > now)`, falling back to `config_definitions.default_value` if no row matches — never a JOIN-and-COALESCE one-liner, since the most-specific-wins precedence needs explicit ordering, not implicit NULL-coalescing.

---

## 12. Performance Considerations

**Potentially high volume at scale** (every overridden setting × every scope it's overridden at × every historical version, since the table is append-only). The `EXCLUDE USING gist` constraint adds modest write-path overhead per insert (a GiST index check) in exchange for the non-overlap guarantee — acceptable given config writes are infrequent relative to reads. **Resolution reads should never hit this table directly on the hot path** — `config_version_stamps` exists specifically so the application can cache resolved values and only re-resolve when the relevant scope's stamp changes, making this table's own query volume far lower than its row count might suggest.