# Section Class-Teacher Assignments

## 1. Table Purpose

The date-ranged ownership history behind `sections.class_teacher_id` (SEC-004), per §0.2.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `section_id` | `UUID` | No | — |
| `teacher_id` | `UUID` | No | — |
| `valid_from` | `TIMESTAMPTZ` | No | now() |
| `valid_to` | `TIMESTAMPTZ` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`section_id → sections(id)` RESTRICT; `teacher_id → users(id)` RESTRICT.

## 5. Unique Constraints

`(section_id) WHERE valid_to IS NULL AND deleted_at IS NULL` — at most one *current* assignment per section, the precise DB-level guarantee behind "exactly one class-teacher per section" (SEC-004).

## 6. Indexes

`(section_id, valid_from)` — the "who owned this section over time" query Attendance (a future wave) will run.

## 7. Relationships

Many historical rows per section; `sections.class_teacher_id` always equals the `teacher_id` of this table's current (`valid_to IS NULL`) row for that section.

## 8. Soft Delete Strategy

Not soft-deleted; `valid_to` is the complete lifecycle marker — reassignment is a single transaction that sets the prior row's `valid_to = now()` and inserts a new row, never an update-in-place of the teacher on an existing row (which would destroy the "immediately ends prior ownership, history preserved" guarantee SEC-004 names explicitly).

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

**Derived, via `sections`.** No direct institute column.

## 11. Notes for TypeORM Entity Design

Implement reassignment as one transaction: `UPDATE section_class_teacher_assignments SET valid_to = now() WHERE section_id = $1 AND valid_to IS NULL; INSERT INTO section_class_teacher_assignments (section_id, teacher_id) VALUES ($1, $2); UPDATE sections SET class_teacher_id = $2 WHERE id = $1;` — all three statements in one transaction, never split across separate service calls, since a crash between them would leave `sections.class_teacher_id` and this table's current row inconsistent.

---

## 12. Performance Considerations

**Low write volume (class-teacher reassignment is an infrequent administrative event), but this table's read pattern will become significant once a future Attendance wave is built** — the `(section_id, valid_from)` index is sized and ordered specifically to support that future 'who owned this section on date X' query efficiently, even though no current wave exercises it yet.