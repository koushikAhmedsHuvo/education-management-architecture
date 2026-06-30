# Subject Teacher Assignments

## 1. Table Purpose

Section-scoped subject-teacher assignment conferring marks ownership (SUB-004) — backs `subject.api.md` #8.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `section_id` | `UUID` | No | — |
| `subject_id` | `UUID` | No | — |
| `teacher_id` | `UUID` | No | — |
| `assigned_at` | `TIMESTAMPTZ` | No | now() |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`section_id → sections(id)` RESTRICT; `subject_id → subjects(id)` RESTRICT; `teacher_id → users(id)` RESTRICT.

## 5. Unique Constraints

`(section_id, subject_id) WHERE deleted_at IS NULL` — one teacher per subject per section at a time.

## 6. Indexes

`(teacher_id)` — "this teacher's assignments" lookups; `(subject_id)`.

## 7. Relationships

Three-way join between `sections`, `subjects`, and `users` (teachers); read directly by Wave 6's mark-entry ownership check ("only the assigned subject-teacher may enter marks for this section/subject," EXM-005).

## 8. Soft Delete Strategy

Standard `deleted_at`; reassignment **updates `teacher_id` in place** on the existing row (deliberately *not* the date-ranged §4 pattern) — the API contract describes no "prior ownership ends, history preserved" requirement for this assignment the way SEC-004 explicitly does for class-teachers, and Wave 6's marks themselves independently stamp `entered_by` on every mark row, which is where historical attribution actually needs to live; adding a parallel date-ranged history here would be exactly the kind of unjustified extra table this brief's "no unnecessary tables" rule warns against.

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

**Derived, via `sections → institute_id`.** No direct institute column.

## 11. Notes for TypeORM Entity Design

The service-layer `SubjectTeacherAssignmentValidator` should perform the curriculum-mapping existence check, the qualification check, and the workload-limit check together, in that order, so the API's three distinct error codes (`409 SUBJECT_NOT_MAPPED_TO_CLASS`, `422 TEACHER_NOT_QUALIFIED`, `422 WORKLOAD_LIMIT_EXCEEDED`) remain individually distinguishable to the caller — a single DB trigger could only ever surface one generic failure.

---

## 12. Performance Considerations

**Moderate volume (one row per section×subject combination, potentially hundreds per institute), read on the mark-entry ownership check for every single mark-entry write in Wave 6** — this table's read performance directly gates Wave 6's hottest write path, so its `(section_id, subject_id)` unique index doubling as the lookup index is load-bearing well beyond this wave's own scope.