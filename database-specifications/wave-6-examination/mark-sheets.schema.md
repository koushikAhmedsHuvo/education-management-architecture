# Mark Sheets

## 1. Table Purpose

The lock-state grouping for a (exam, subject, section, component) mark-entry sheet (EXM-007/009) — the unit Submit & Lock actually operates on.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `exam_id` | `UUID` | No | — |
| `subject_id` | `UUID` | No | — |
| `section_id` | `UUID` | No | — |
| `component_key` | `VARCHAR(64)` | Yes | — |
| `status` | `mark_sheet_status (ENUM: DRAFT, LOCKED)` | No | 'DRAFT' |
| `locked_by` | `UUID` | Yes | — |
| `locked_at` | `TIMESTAMPTZ` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`exam_id → exams(id)` RESTRICT; `subject_id → subjects(id)` RESTRICT; `section_id → sections(id)` RESTRICT (Wave 4); `locked_by → users(id)` RESTRICT.

## 5. Unique Constraints

`(exam_id, subject_id, section_id, component_key) WHERE deleted_at IS NULL` — NULL-`component_key` handled via the same `COALESCE` idiom used throughout this schema where a nullable scope column participates in uniqueness.

## 6. Indexes

`(exam_id, status)` — the mark-entry-progress screen's dominant query.

## 7. Relationships

One sheet has many `marks` rows (one per eligible student); referenced by `mark_moderations` and indirectly by `mark_corrections` (via `marks.mark_sheet_id`).

## 8. Soft Delete Strategy

Not soft-deleted; `status` is the complete lifecycle marker, and a `LOCKED` sheet is — by EXM-007's own definition — never reversible back to `DRAFT` through any normal write path (only through the fully separate, governed `mark_corrections` flow, §6).

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

**Derived, via `exams → institute_id`.** No direct institute column.

## 11. Notes for TypeORM Entity Design

Ownership-scoped mark entry (EXM-005: only the assigned subject-teacher) is enforced by joining `04-academic-structure.md`'s `subject_teacher_assignments` (`section_id`, `subject_id`) against the caller's identity on every write to this sheet's `marks` — implement as a single repository-level guard reused by both the entry and the lock handlers, not duplicated logic in each.

---

## 12. Performance Considerations

**Low-to-moderate volume (exams × subjects × sections, potentially a few hundred per institute per exam cycle), read on the mark-entry-progress dashboard repeatedly throughout the entry window.** The `(exam_id, status)` index keeps that dashboard's aggregate query cheap even as the underlying `marks` row count (the real volume driver) grows large.