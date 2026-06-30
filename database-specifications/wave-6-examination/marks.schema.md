# Marks

## 1. Table Purpose

Individual student marks or special outcomes (EXM-004/005/006) — backs `marks.api.md` #1–6.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `mark_sheet_id` | `UUID` | No | — |
| `student_id` | `UUID` | No | — |
| `mark` | `NUMERIC(6,2)` | Yes | — |
| `special_outcome` | `mark_special_outcome (ENUM: AB, EX, MAL)` | Yes | — |
| `entered_by` | `UUID` | No | — |
| `entered_at` | `TIMESTAMPTZ` | No | now() |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`mark_sheet_id → mark_sheets(id)` ON DELETE CASCADE — a mark is a true dependent of its sheet, the same reasoning as `refresh_tokens → sessions` and `workflow_tasks → workflow_instances`; `student_id → students(id)` RESTRICT; `entered_by → users(id)` RESTRICT.

## 5. Unique Constraints

`(mark_sheet_id, student_id) WHERE deleted_at IS NULL` — one mark row per student per sheet.

## 6. Indexes

`(mark_sheet_id)` — covered by the unique constraint below; `(student_id)` for a student's own mark history across sheets.

## 7. Relationships

Many marks per sheet (its full roster); read by `results` computation (joined against `mark_moderations` for the effective value, per §0.4) and by `mark_corrections` (one correction request targets one specific mark row).

## 8. Soft Delete Strategy

Not soft-deleted — once a sheet locks, its marks are immutable through any normal path (EXM-007); the only way a mark's value ever changes post-lock is via an approved `mark_corrections` row, which **updates this row in place** (the correction's own table, §6, is what retains the "before" value for history — `marks` itself doesn't need a parallel append-only shape, since corrections are rare, individually governed events, unlike `enrollments`' routine transfers).

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

**Derived, via `mark_sheets → exams → institute_id`.** No direct institute column.

## 11. Notes for TypeORM Entity Design

Entry is **idempotent and batched** (`marks.api.md` #2's `idempotencyKey`) — implement as a bulk `INSERT ... ON CONFLICT (mark_sheet_id, student_id) DO UPDATE SET mark = EXCLUDED.mark, special_outcome = EXCLUDED.special_outcome WHERE mark_sheets.status = 'DRAFT'` (joined/guarded against the parent sheet's lock state in the same statement), never a per-student loop of separate `save()` calls, both for performance on large rosters and for the atomicity a "save the whole grid at once" UX genuinely needs.

---

## 12. Performance Considerations

**The highest-volume table in the Examination domain** (every student × every subject × every exam, accumulating permanently — never deleted). Mark entry's idempotent, batched `INSERT ... ON CONFLICT ... DO UPDATE` pattern (per the TypeORM Notes) is essential at this volume — a per-student save loop would be unacceptably slow for a roster-sized grid submitted at once. This table is a strong long-term partitioning candidate (by exam/session) once multi-year mark history accumulates.