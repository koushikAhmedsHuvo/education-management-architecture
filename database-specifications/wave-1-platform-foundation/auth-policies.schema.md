# Auth Policies

## 1. Table Purpose

The configurable password/lockout/MFA/token-TTL policy (AUTH-004/007/008/009's "configurable within secure minimums") — backs `auth.api.md` #18–19.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | Yes | — |
| `min_length` | `SMALLINT` | No | 10 |
| `complexity_classes` | `TEXT[]` | No | '{}' |
| `breach_check_enabled` | `BOOLEAN` | No | true |
| `max_failed_attempts` | `SMALLINT` | No | 5 |
| `lockout_cooldown_minutes` | `SMALLINT` | No | 15 |
| `mfa_required_roles` | `UUID[]` | No | '{}' |
| `access_token_ttl_minutes` | `SMALLINT` | No | 15 |
| `refresh_token_ttl_days` | `SMALLINT` | No | 30 |
| `recovery_token_ttl_minutes` | `SMALLINT` | No | 60 |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` ON DELETE RESTRICT.

## 5. Unique Constraints

`(institute_id) WHERE deleted_at IS NULL`, with `institute_id IS NULL` also constrained to **at most one row** via a separate partial unique index `UNIQUE ((institute_id IS NULL)) WHERE institute_id IS NULL AND deleted_at IS NULL` (a standard Postgres idiom for "at most one NULL-keyed row").

## 6. Indexes

None beyond the unique constraint below (tiny table, point lookups only).

## 7. Relationships

Read by every authentication flow; not referenced by foreign key from elsewhere (a leaf configuration table).

## 8. Soft Delete Strategy

Standard; in practice never deleted (always exactly one default row, optionally overridden per institute).

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

**Direct, nullable — the canonical 'institute override of a deployment default' shape.** Exactly one `institute_id IS NULL` row holds the deployment default; an institute-specific row (when present) overrides it. This is the simplest, smallest-scale instance of the most-specific-wins pattern this schema uses extensively (Wave 2's `config_values`, Wave 6's `grade_scales`, Wave 7's `fee_structures`), here without the full effective-dated versioning those larger tables need.

## 11. Notes for TypeORM Entity Design

Resolve "effective policy for this institute" as `COALESCE` of the institute-specific row over the `NULL`-keyed default row — implement as a small repository method (`getEffectivePolicy(instituteId)`), not a view, to keep it simple and explicit; this is intentionally **not** routed through the generic Configuration Engine (Wave 2) even though it resembles scoped configuration, because the approved `auth.api.md` models it as its own dedicated resource, not `/config/*` — the schema mirrors that API decision rather than "fixing" it.

---

## 12. Performance Considerations

**Trivially low volume — at most one row per institute plus one default.** Read on every authentication attempt (to resolve lockout thresholds, MFA requirements), so it should be cached in application memory or Redis rather than queried fresh on every login; this is the one table in Wave 1 where an application-level cache is more important than any database index.