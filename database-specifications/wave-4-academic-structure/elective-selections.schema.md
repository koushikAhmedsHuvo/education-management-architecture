# Elective Selections

## 1. Table Purpose

Student elective choices within configured group rules (SUB-006) — backs `subject.api.md` #11–12.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `subject_id` | `UUID` | No | — |
| `session_id` | `UUID` | No | — |
| `student_id` | `UUID` | No | — |
| `status` | `elective_selection_status (ENUM: RECORDED, WAITLISTED)` | No | 'RECORDED' |
| `selected_at` | `TIMESTAMPTZ` | No | now() |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`subject_id → subjects(id)` RESTRICT; `session_id → academic_sessions(id)` RESTRICT. `student_id` has no FK yet (see above).

## 5. Unique Constraints

`(student_id, session_id, subject_id) WHERE deleted_at IS NULL` — a student selects a given elective once per session.

## 6. Indexes

`(subject_id, session_id)` — `subject.api.md` #12's admin view of who selected a given elective; `(student_id, session_id)` — a student's own selection set (will become fully effective once the FK lands in Wave 5, but the index is created now since the column and its query pattern both already exist).

## 7. Relationships

Will become a proper many-to-one to Wave 5's `students` once that table exists; already a proper many-to-one to `subjects` and `academic_sessions`.

## 8. Soft Delete Strategy

Standard; a withdrawn selection is soft-deleted rather than hard-removed, since `subject.api.md` #11's selection-window and group-capacity logic benefits from seeing the full historical attempt record, not just current selections.

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

**Derived, via `subjects → institute_id` and (once FK-complete) `students → institute_id`.** Both parents are expected to agree; the application verifies this since the `student_id` FK doesn't exist until Wave 5.

## 11. Notes for TypeORM Entity Design

This table is the clearest concrete example, in this entire wave, of why the forward-reference pattern from Wave 1 exists at all — `elective_selections` is unambiguously *this* wave's table (it's about subjects and their elective rules, not about students), yet it cannot be fully foreign-keyed until Wave 5 exists; building it now with the column present and the FK deferred is strictly better than either delaying this whole table to Wave 5 (wrong ownership) or omitting the column entirely (losing the index and the eventual one-line completion).

---

## 12. Performance Considerations

**Low-to-moderate volume, concentrated in narrow selection-window bursts** (most students select electives within a short administrative window each session, not steadily over time) — expect spiky write load during those windows specifically. The `(subject_id, session_id)` index supports the admin capacity-monitoring view that becomes most heavily used precisely during those same bursts.