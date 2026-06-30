# ID Generation Formats

## 1. Table Purpose

The Student/Employee ID pattern configuration (STU-001, CFG-007 immutable-after-use) — backs `id-generation.api.md` #1–3.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `target` | `id_generation_target (ENUM: STUDENT, EMPLOYEE)` | No | — |
| `institute_id` | `UUID` | Yes | — |
| `tokens` | `JSONB` | No | — |
| `uniqueness_scope` | `id_uniqueness_scope (ENUM: DEPLOYMENT, INSTITUTE, CAMPUS)` | No | — |
| `reset_policy` | `id_reset_policy (ENUM: PER_SESSION, PER_CAMPUS, NEVER)` | No | — |
| `locked` | `BOOLEAN` | No | false |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT.

## 5. Unique Constraints

`(target, COALESCE(institute_id, '00000000-...'::uuid)) WHERE deleted_at IS NULL`.

## 6. Indexes

None beyond the unique constraint (tiny table).

## 7. Relationships

Referenced by `id_generation_sequences` (the live counter state per format).

## 8. Soft Delete Strategy

Standard, though in practice a `locked = true` row is essentially permanent for the institute's lifetime — replacing it requires the same exceptional, audited path any other immutable-after-use definition would (Wave 1's analogous note on `auth_policies`' minimums, here applied to "supersede forward only").

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

**Direct, nullable.** `institute_id IS NULL` is the deployment default format; an institute-specific override is the exception.

## 11. Notes for TypeORM Entity Design

`PUT /config/id-formats/{target}` must check `locked = false` before allowing any write — the check itself is application-layer (since "is locked" combined with "is this institute's row vs. the deployment default" needs the same scope-resolution logic as `config_values`), but the trigger in §9 is what makes the underlying fact (`locked`) trustworthy regardless of whether every code path remembers to check it.

---

## 12. Performance Considerations

**Trivially low volume (one row per target per institute at most), read once per ID-generation call** (cached or re-fetched cheaply given the table's tiny size). No performance concerns.