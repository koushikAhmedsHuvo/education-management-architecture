# Exams

## 1. Table Purpose

Exam definitions — per-subject maxima/components/schedule, consistent with curriculum (EXM-001/002) — backs `exam.api.md` #1–4.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `session_id` | `UUID` | No | — |
| `name` | `VARCHAR(255)` | No | — |
| `type` | `VARCHAR(32)` | No | — |
| `class_ids` | `UUID[]` | No | — |
| `subjects` | `JSONB` | No | — |
| `schedule_start` | `DATE` | No | — |
| `schedule_end` | `DATE` | No | — |
| `mark_entry_window_from` | `TIMESTAMPTZ` | No | — |
| `mark_entry_window_to` | `TIMESTAMPTZ` | No | — |
| `status` | `exam_status (ENUM: DEFINED, SCHEDULED, RESCHEDULED, IN_PROGRESS, MARK_ENTRY, UNDER_VERIFICATION, LOCKED, CANCELLED, ARCHIVED)` | No | 'DEFINED' |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT; `session_id → academic_sessions(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK — an institute may run multiple same-named exams across different sessions or types.

## 6. Indexes

`(institute_id, session_id, status)`; GIN index on `class_ids` if "exams targeting class X" becomes a frequent independent query (typically resolved instead via `mark_sheets`/`exam_eligibility`, so omitted by default).

## 7. Relationships

One exam has many `exam_eligibility` rows, many `mark_sheets`, and (via those) many `results`.

## 8. Soft Delete Strategy

Standard `deleted_at`, though `status = 'ARCHIVED'`/`'CANCELLED'` is the disclosed lifecycle; a `LOCKED` exam (marks entered) is never hard-deleted, mirroring every other "consumers depend on this forever" rule in this schema.

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

**Direct, mandatory.** `institute_id` is NOT NULL; `session_id` further scopes to one academic session within that institute.

## 11. Notes for TypeORM Entity Design

**Edits are blocked entirely once any mark exists** (`exam.api.md` #4) — implement as an application-layer check (`EXISTS (SELECT 1 FROM marks m JOIN mark_sheets ms ON ms.id = m.mark_sheet_id WHERE ms.exam_id = $1)`), not a DB trigger, since the resulting error needs to name *which* subject/section blocks the edit, which a trigger's generic exception can't convey as usefully as a targeted application query.

---

## 12. Performance Considerations

**Low volume (a handful of exams per session per institute), read extremely frequently during mark-entry and results periods, written rarely.** The `subjects` JSONB is always read as a whole alongside the parent exam — no independent indexing of its internals is needed at this table's access pattern.