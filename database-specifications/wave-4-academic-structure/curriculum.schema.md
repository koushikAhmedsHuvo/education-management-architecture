# Curriculum Mappings

## 1. Table Purpose

Subject ↔ Class/Stream offerings (SUB-002) — backs `curriculum.api.md`.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `class_id` | `UUID` | No | — |
| `subject_id` | `UUID` | No | — |
| `stream` | `VARCHAR(64)` | Yes | — |
| `mandatory` | `BOOLEAN` | No | true |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`class_id → classes(id)` RESTRICT; `subject_id → subjects(id)` RESTRICT.

## 5. Unique Constraints

`(class_id, subject_id, COALESCE(stream, '')) WHERE deleted_at IS NULL` — a subject maps to a class once per stream (or once overall, where `stream IS NULL`).

## 6. Indexes

`(subject_id)` — `curriculum.api.md` #1's "which classes is this subject mapped to"; `(class_id)` — the reverse, `curriculum.api.md` #4's report.

## 7. Relationships

The associative table between `classes` and `subjects`; `stream` must be one of the parent class's own `classes.streams` array values where non-null (application-layer check, the same array-membership trade-off as `sections.campus_id` against `classes.campus_applicability`).

## 8. Soft Delete Strategy

Standard; removing a mapping is a real soft-delete here (unlike `role_permissions`, which used hard insert/delete) because curriculum mapping changes are infrequent, deliberate, structural edits worth retaining a trace of via `deleted_at` rather than relying solely on `audit_log`.

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

**Derived, via `classes → institute_id`** (and transitively via `subjects → institute_id`, both of which the application verifies agree at write time, since a cross-institute mapping would be a data-integrity error no FK alone can prevent).

## 11. Notes for TypeORM Entity Design

This table is read on the hot path of `subject_teacher_assignments`' own write validation (§8) and, in a later wave, exam definition — index it accordingly; no GIN/JSONB needed here at all, a clean normalized join table fully suffices, in deliberate contrast to `subjects.components` just above.

---

## 12. Performance Considerations

**Low-to-moderate volume (classes × subjects × streams, realistically a few hundred rows per institute), read on the hot path of subject-teacher assignment and (in a later wave) exam-definition validation.** Both directional indexes — `(subject_id)` and `(class_id)` — are genuinely needed given how frequently this table is queried from both directions; this is one of the few Wave 4 tables where a normalized join table's dual-index cost is clearly justified by real query volume.