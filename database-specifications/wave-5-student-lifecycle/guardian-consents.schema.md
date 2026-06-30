# Guardian Consents

## 1. Table Purpose

Scoped, withdrawable parental consent records (GRD-N-008) — backs `guardian.api.md` #12–13.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `student_guardian_link_id` | `UUID` | No | — |
| `consent_type` | `VARCHAR(64)` | No | — |
| `status` | `consent_status (ENUM: GRANTED, REVOKED)` | No | — |
| `recorded_at` | `TIMESTAMPTZ` | No | now() |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`student_guardian_link_id → student_guardian_links(id)` RESTRICT.

## 5. Unique Constraints

None — like `config_values`, this table is append-only by convention (a grant or revoke is a new row, never an update), so "current status" is always the most recent row per `(student_guardian_link_id, consent_type)`, not a single mutable flag.

## 6. Indexes

`(student_guardian_link_id, consent_type, recorded_at DESC)` — "current consent status for this type" resolves as the latest row per `(link, type)`.

## 7. Relationships

Many-to-one with `student_guardian_links`.

## 8. Soft Delete Strategy

Not soft-deleted — every grant/revoke is permanently retained as the actual consent history a future dispute or audit needs.

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

**Derived, via `student_guardian_links → students → institute_id`.** No direct institute column.

## 11. Notes for TypeORM Entity Design

Resolve "current consent" via `SELECT status FROM guardian_consents WHERE student_guardian_link_id = $1 AND consent_type = $2 ORDER BY recorded_at DESC LIMIT 1` — a small, cheap query given this table's low write volume; no materialized "current consent" column is needed on `student_guardian_links` itself.

---

## 12. Performance Considerations

**Low volume, append-only, infrequent writes.** The `(student_guardian_link_id, consent_type, recorded_at DESC)` index keeps 'current consent' resolution cheap (`LIMIT 1` off a small per-link, per-type row count) even as historical entries accumulate indefinitely.