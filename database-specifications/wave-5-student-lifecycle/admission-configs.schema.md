# Admission Configs

## 1. Table Purpose

Capacity, evaluation criteria, and fee-gating configuration per (level, session) (ADM-004/006) — backs `admission.api.md` #23–24. A dedicated table rather than routing through Wave 2's generic `config_values`, because this is a single compound, admission-specific structure (capacity + criteria + fee rule together), not a simple typed key-value pair — the same reasoning that kept `auth_policies` (Wave 1) out of the generic engine.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `level_class_id` | `UUID` | No | — |
| `session_id` | `UUID` | No | — |
| `capacity` | `INTEGER` | No | — |
| `criteria` | `JSONB` | No | — |
| `fee_gating` | `JSONB` | No | — |
| `version` | `INTEGER` | No | 1 |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`level_class_id → classes(id)` RESTRICT; `session_id → academic_sessions(id)` RESTRICT.

## 5. Unique Constraints

`(level_class_id, session_id) WHERE deleted_at IS NULL`.

## 6. Indexes

None beyond the unique constraint (small table, point lookups only).

## 7. Relationships

Read by `admission_applications`' capacity/criteria/fee checks at evaluation, decision, and conversion time; `evaluation_rubric_version` on that table pins to this row's `version` at the moment of evaluation.

## 8. Soft Delete Strategy

Standard; in practice never deleted, only re-versioned (a new `version` value, same row updated in place — unlike `config_values`, this table does **not** need date-ranged history, since the API contract describes no scenario where a *past* admission cycle's criteria need re-resolution after the fact the way a fee structure or grade scale does).

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

**Direct, via `level_class_id`/`session_id`, both of which trace back to one institute.** No direct `institute_id` column, but the composite unique constraint effectively scopes correctly since both parent FKs originate from the same institute by construction.

## 11. Notes for TypeORM Entity Design

Capacity enforcement reads `COUNT(*) FROM enrollments WHERE section_id IN (sections under this level_class_id/session) AND period_to IS NULL` (i.e., the Wave-4 `SectionCapacityGuard`'s same query), **not** a counter column on this table — keeping capacity-checking as a live count against `enrollments` (the actual source of truth) rather than a denormalized counter avoids exactly the kind of drift risk a cached count would introduce.

---

## 12. Performance Considerations

**Very low volume (one row per level per session, a few dozen at most per institute).** Read on every application-evaluation and conversion-capacity check, but cheap given the small row count — a natural candidate for application-level caching per (level, session) given how rarely these rows actually change relative to how often they're read.