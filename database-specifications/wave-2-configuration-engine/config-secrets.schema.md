# Config Secrets

## 1. Table Purpose

Sensitive configuration values — integration secrets, key references — encrypted at rest, masked in every read, and structurally excluded from export/audit (CFG-009). Backs `settings.api.md` #16–17.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `definition_id` | `UUID` | No | — |
| `institute_id` | `UUID` | Yes | — |
| `value_encrypted` | `BYTEA` | No | — |
| `version` | `INTEGER` | No | 1 |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`definition_id → config_definitions(id)` RESTRICT; `institute_id → institutes(id)` RESTRICT.

## 5. Unique Constraints

`(definition_id, COALESCE(institute_id, '00000000-...'::uuid)) WHERE deleted_at IS NULL` — exactly one current secret per definition+scope.

## 6. Indexes

None beyond the unique constraint (small table, point lookups by `(definition_id, institute_id)` only).

## 7. Relationships

None inbound from elsewhere — by design, a leaf table no other table's FK or JSONB ever points into, which is precisely what makes the "never appears in an export" guarantee structural (§0.3).

## 8. Soft Delete Strategy

Standard `deleted_at`, though rotation (`PUT /config/sensitive/{key}`) **overwrites in place** (a true `UPDATE` of `value_encrypted`, bumping `version`) rather than inserting a new versioned row — deliberately, since retaining an old, possibly-compromised secret's ciphertext "for history" is a liability with no offsetting benefit, unlike every other versioned table in this schema where history is actively useful.

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

**Direct, nullable**, same scope shape as `config_values` (minus `campus_id`/`section_id`, since secrets are deliberately kept simpler — institute-level only, per the approved API).

## 11. Notes for TypeORM Entity Design

`value_encrypted` is `@Column({ type: 'bytea', select: false })` — never selected by default, mirroring `users.password_hash`. The `GET /config/sensitive/{key}` endpoint's response (`{ isSet, lastUpdatedAt, lastUpdatedBy }`) is satisfiable from `EXISTS(...)` plus `updated_at`/`updated_by` alone — the repository method backing it should never even attempt to fetch `value_encrypted`.

---

## 12. Performance Considerations

**Very low volume (a handful of secrets per institute), read rarely** (only on explicit unmask, never on routine resolution). No performance concerns at any realistic scale; the `select: false` default on `value_encrypted` ensures this table's one sensitive column never adds overhead to routine existence-check reads.