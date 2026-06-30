# Exam Schedules

## 1. Table Purpose

**This is not a separate table.** Per the already-approved API Contract (`wave-6-examination/exam.api.md` #3, "Define Exam"), exam scheduling is modeled as a single `{ start, end }` date range scoped to the whole exam, not a granular per-subject/per-section schedule with individual dates, times, and rooms. The approved business rules (Doc 14, EXM-001/002) likewise describe only an exam-level schedule, never a separately-tracked scheduling entity. Introducing a dedicated `exam_schedules` table here would either (a) duplicate the `schedule_start`/`schedule_end` columns already on `exams` for no benefit, or (b) silently expand scope beyond what the approved API and business rules actually specify — both of which this specification avoids per the standing "do not introduce new entities unless already defined in approved specifications" instruction.

**Where this data actually lives:** the `schedule_start` and `schedule_end` columns on the `exams` table — see [`exams.schema.md`](./exams.schema.md) §2, and the `CHECK (schedule_end >= schedule_start)` constraint documented there in §6/§12. If a future wave's approved specification introduces genuinely granular per-subject exam scheduling (distinct dates, times, and rooms per subject within one exam), that would warrant a real `exam_schedules` child table at that time — keyed `(exam_id, subject_id, scheduled_date, start_time, room)` — but no such requirement exists in the currently approved Module Functional Specifications, UI Screen Specifications, or API Contract, so no such table is speculatively created here.

## 2–9. (Not applicable)

No columns, keys, constraints, indexes, relationships, soft-delete strategy, or audit fields exist for this file, since it does not back a physical table. See `exams.schema.md` for the actual `schedule_start`/`schedule_end` column specifications.

## 10. Multi-Institute Isolation Strategy

Not applicable — covered by `exams.schema.md` §10, since exam scheduling data lives on that table.

## 11. Notes for TypeORM Entity Design

Do not create an `ExamSchedule` entity. Map `scheduleStart`/`scheduleEnd` as plain columns on the `Exam` entity itself, exactly as the approved API contract's `exam.api.md` DTOs already model them.

## 12. Performance Considerations

Not applicable — no independent table, no independent query path. Schedule-range queries (e.g., "exams scheduled between date X and Y") run directly against `exams.schedule_start`/`schedule_end`; add a B-tree index on `(institute_id, schedule_start)` to `exams.schema.md` if that specific query pattern becomes a measurably frequent access path in practice (not currently called out as such by the approved API contract).
