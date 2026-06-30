# Campuses

## 1. Table Purpose

The secondary, optional scope boundary nested under institute (Doc 04) — backs `campus.api.md`.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `name` | `VARCHAR(255)` | No | — |
| `code` | `VARCHAR(32)` | No | — |
| `is_default` | `BOOLEAN` | No | false |
| `status` | `campus_status (ENUM: ACTIVE, INACTIVE, ARCHIVED)` | No | 'ACTIVE' |
| `address` | `TEXT` | Yes | — |
| `contact` | `VARCHAR(255)` | Yes | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` ON DELETE RESTRICT — a campus can never be reparented to a different institute (CAMP-001 "no reparenting"), which this single immutable FK (never updated after insert, by service-layer convention, not a DB trigger) directly supports.

## 5. Unique Constraints

`(institute_id, code) WHERE deleted_at IS NULL` — campus codes are unique **per institute**, so two different institutes may each legitimately have a `'MAIN'` campus (CAMP-003); **`(institute_id) WHERE is_default = true AND deleted_at IS NULL`** — at most one default campus per institute, the direct DB-level enforcement of CAMP-002's default-campus guarantee (the *complementary* half — "at least one active campus must always exist" — cannot be expressed as a single-table constraint, since it's a property of the *set* of rows for an institute, and remains an application-layer check performed at the moment of deactivation, per `campus.api.md` #9's `409 LAST_ACTIVE_CAMPUS`).

## 6. Indexes

`(institute_id, status)`; `(institute_id, is_default)` — covered by the partial unique index below, no separate index needed.

## 7. Relationships

Many campuses per institute; referenced by `memberships` (campus-scoped grants) and, in Wave 5, by `enrollments`/student placement.

## 8. Soft Delete Strategy

Standard `deleted_at`, though as with institutes, `status = 'INACTIVE'`/`'ARCHIVED'` is the disclosed lifecycle (CAMP-007) rather than `deleted_at` itself.

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

**Direct, mandatory, immutable.** `institute_id` is NOT NULL and — per Business Rules CAMP-001 — never updated after insert (no reparenting); this FK is therefore one of the strongest, most reliable isolation boundaries in the schema, since the relationship genuinely cannot drift.

## 11. Notes for TypeORM Entity Design

Per the cross-wave design note in the API contract (`05-campus-api.md`'s own header), **no `campus_transfers` table is created in this wave** — section/campus movement history is recorded in Wave 5's `enrollment_history` table once a student (the actual subject of a transfer) exists; `campuses` itself only needs to exist as a stable FK target here.

---

## 12. Performance Considerations

**Low volume (a handful of campuses per institute at most), read frequently as a scope-resolution dimension.** No special performance concerns beyond the existing indexes; campus counts are small enough at any realistic institute size that this table will never need partitioning or special caching beyond ordinary connection-pool-level query caching.