# Config Definitions

## 1. Table Purpose

The closed registry of *what is configurable* (CFG-001) — no value may ever be set against an unregistered key. Backs `settings.api.md` #1–4.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `key` | `VARCHAR(128)` | No | — |
| `data_type` | `config_data_type (ENUM: STRING, NUMBER, BOOLEAN, ENUM, DATE, JSON)` | No | — |
| `allowed_values` | `JSONB` | Yes | — |
| `range_min` | `NUMERIC` | Yes | — |
| `range_max` | `NUMERIC` | Yes | — |
| `default_value` | `JSONB` | No | — |
| `allowed_scope_levels` | `TEXT[]` | No | — |
| `overridable` | `BOOLEAN` | No | true |
| `sensitive` | `BOOLEAN` | No | false |
| `immutable_after_use` | `BOOLEAN` | No | false |
| `high_impact` | `BOOLEAN` | No | false |
| `status` | `config_definition_status (ENUM: REGISTERED, DEPRECATED)` | No | 'REGISTERED' |

## 3. Primary Key

`id`.

## 4. Foreign Keys

None outbound (the root of this wave's hierarchy, exactly as `permissions` was the root of Wave 1's authorization model).

## 5. Unique Constraints

`(key)` — globally unique, and like `permissions`, **this table has no soft-delete path at all**; `deleted_at` is present for schema consistency but never set.

## 6. Indexes

`(status)`; GIN index on `allowed_scope_levels` only if "find all definitions overridable at scope X" becomes a real query pattern (omitted by default — small table, sequential scan is fine at registry scale).

## 7. Relationships

Referenced by `config_values`, `config_secrets`, and `config_type_templates`.

## 8. Soft Delete Strategy

None — `status = 'DEPRECATED'` is the sole retirement path (CFG-001/Doc 27 §6: "edits/hard-delete forbidden — consumers may depend on history").

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

**Deployment-global by design.** A configuration *definition* (the registry of what is configurable) is shared across every institute — the key `attendance.lockWindowHours` means the same thing everywhere; only its *value* (Wave 2's `config_values`) is institute-scoped. No `institute_id` column exists or should exist here.

## 11. Notes for TypeORM Entity Design

Seed the platform's own built-in definitions (lockout thresholds, etc. — anything Wave 1's `auth_policies` *doesn't* already own as its own dedicated table, see Wave 1 §9's note that auth policy deliberately bypasses this generic engine) from a versioned migration; `POST /config/definitions` exists for legitimate ongoing registration by later modules' own rollouts, not for ad hoc end-user use.

---

## 12. Performance Considerations

**Low volume (tens to low hundreds of registered keys), read on every config-resolution path, written almost never.** Like `permissions`, this is a strong candidate for full in-memory application caching loaded at startup; the deployment-global, rarely-changing nature of this table means querying Postgres fresh on every resolution would be pure overhead.