# Exam Eligibility

## 1. Table Purpose

Per-student eligibility and admit-card gating (EXM-003) — backs `exam.api.md` #5–9.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `exam_id` | `UUID` | No | — |
| `student_id` | `UUID` | No | — |
| `eligible` | `BOOLEAN` | No | — |
| `reason` | `TEXT` | Yes | — |
| `override_reason` | `TEXT` | Yes | — |
| `overridden_by` | `UUID` | Yes | — |
| `admit_card_issued` | `BOOLEAN` | No | false |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`exam_id → exams(id)` RESTRICT; `student_id → students(id)` RESTRICT (Wave 5, a real FK); `overridden_by → users(id)` RESTRICT.

## 5. Unique Constraints

`(exam_id, student_id) WHERE deleted_at IS NULL` — eligibility is determined (and re-determined, overwriting in place) once per student per exam, **not** date-ranged or append-only, since `exam.api.md` #6 ("Determine Eligibility") is explicitly re-runnable and always reflects current data — there is no scenario requiring "what was this student's eligibility on date X" the way marks/results do.

## 6. Indexes

`(exam_id, eligible)` — the dominant "list eligible/ineligible students for this exam" query.

## 7. Relationships

Many-to-many between `exams` and `students`, with eligibility as the relationship's payload; admit cards generated against eligible rows are `documents` rows (§0.5), not stored here.

## 8. Soft Delete Strategy

Standard; in practice rows are simply re-determined (updated in place) rather than soft-deleted and recreated.

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

**Derived, via `exams → institute_id` and `students → institute_id`** (both expected to agree, application-verified).

## 11. Notes for TypeORM Entity Design

`admit_card_issued` flips `true` the moment the corresponding `documents` row (`document_type = 'ADMIT_CARD'`) is created — implement as one transaction (insert the document, update this flag), not two independent writes.

---

## 12. Performance Considerations

**High volume relative to `exams` itself** (one row per eligible student per exam — potentially thousands per institute per exam cycle), **written in a concentrated burst** when eligibility is (re-)determined for an entire cohort at once. The `(exam_id, eligible)` index supports the dominant triage-list query; this table's bulk-write pattern (one transaction inserting/updating an entire cohort's eligibility) should use a single batched statement, never a per-student loop.