# Results

## 1. Table Purpose

Provenance-stamped, deterministically computed results (RES-001…009) — backs `result.api.md` #1–8, #15–22. Append-only per §0.1.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `student_id` | `UUID` | No | — |
| `exam_id` | `UUID` | No | — |
| `version` | `INTEGER` | No | 1 |
| `subject_results` | `JSONB` | No | — |
| `marks_total` | `NUMERIC(8,2)` | Yes | — |
| `division` | `VARCHAR(32)` | Yes | — |
| `gpa` | `NUMERIC(4,2)` | Yes | — |
| `rank` | `INTEGER` | Yes | — |
| `rule_ver` | `INTEGER` | No | — |
| `config_ver` | `INTEGER` | No | — |
| `grade_ver` | `INTEGER` | No | — |
| `status` | `result_status (ENUM: DRAFT, CALCULATED, VERIFIED, PUBLISHED, REVISED, WITHHELD, INCOMPLETE, SUPERSEDED)` | No | 'CALCULATED' |
| `withhold_reason` | `VARCHAR(32)` | Yes | — |
| `withhold_reason_detail` | `TEXT` | Yes | — |
| `withhold_until` | `TIMESTAMPTZ` | Yes | — |
| `computed_at` | `TIMESTAMPTZ` | No | now() |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`student_id → students(id)` RESTRICT; `exam_id → exams(id)` RESTRICT.

## 5. Unique Constraints

**`(student_id, exam_id) WHERE status != 'SUPERSEDED' AND deleted_at IS NULL`** — exactly one *current* result per student per exam at a time, the precise DB-level form of "a revision supersedes, never duplicates."

## 6. Indexes

`(student_id, exam_id, version DESC)`; `(exam_id, status)` — review/publish queue; **`(rule_ver, config_ver, grade_ver)`** — directly supporting §0.2's "every result computed under grade scale version 3" forensic query.

## 7. Relationships

Many versions per (student, exam) over time (its own revision history, per §0.1); referenced by `result_revisions` and `grade_overrides`.

## 8. Soft Delete Strategy

Standard `deleted_at`, essentially unused — `status` (including the dedicated `SUPERSEDED` value) is the complete, queried lifecycle.

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

**Derived, via `student_id → students → institute_id` and `exam_id → exams → institute_id`** (both expected to agree). No direct institute column, though one could reasonably be denormalized here in a future optimization pass if results-reporting query performance ever warrants it, mirroring `enrollments`' own deliberate `institute_id` denormalization.

## 11. Notes for TypeORM Entity Design

A revision (`result.api.md` #12–14) executes as: `UPDATE results SET status = 'SUPERSEDED' WHERE id = $oldId; INSERT INTO results (..., version = $oldVersion + 1, rule_ver, config_ver, grade_ver) VALUES (...)` — the new row's provenance triple is **re-stamped**, not copied, since a revision may (rarely) be triggered by a grade-scale correction that genuinely changes `grade_ver` going forward; where the revision doesn't touch provenance-affecting inputs, the same triple values are simply carried forward, but always explicitly, never implicitly assumed unchanged.

---

## 12. Performance Considerations

**High volume, write-bursty around publish events specifically.** The `(rule_ver, config_ver, grade_ver)` index supports the (rare but important) forensic 'everything computed under version X' query; the `(student_id, exam_id, version DESC)` partial unique index is the schema's safety-critical guarantee against duplicate concurrent results. Bulk compute/publish (`result.api.md` #2/#7) processes an entire cohort in one async job — the per-student insert volume during a publish event can be substantial and should run as a true batch operation, never a per-student transaction loop, given the cohort sizes involved.