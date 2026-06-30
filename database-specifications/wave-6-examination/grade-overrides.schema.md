# Grade Overrides

## 1. Table Purpose

Governed, transparent manual grade overrides (GRD-008) — backs `grading.api.md` #10.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `result_id` | `UUID` | No | — |
| `subject_id` | `UUID` | Yes | — |
| `computed_grade` | `VARCHAR(16)` | No | — |
| `override_grade` | `VARCHAR(16)` | No | — |
| `reason` | `TEXT` | No | — |
| `status` | `change_request_status (reuses the Wave-5 ENUM)` | No | 'PENDING_APPROVAL' |
| `requested_by` | `UUID` | No | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`result_id → results(id)` RESTRICT; `subject_id → subjects(id)` RESTRICT, nullable; `requested_by → users(id)` RESTRICT; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK.

## 6. Indexes

`(status)`.

## 7. Relationships

Many-to-one with `results`; on approval, the override is applied as a visible annotation on the `results.subject_results` JSONB (or the top-level `division`/`gpa`, for a non-subject-scoped override) — the **computed** value is never erased, only layered over, per GRD-008's "computed value preserved" — implemented the same way as `mark_moderations`' transparent-layer pattern (§0.4/§5), not by overwriting `results` in place.

## 8. Soft Delete Strategy

Standard; retained indefinitely as the disclosed override history.

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

**Derived, via `result_id → results → ... → institute_id`.** No direct institute column.

## 11. Notes for TypeORM Entity Design

Approval-commit updates the relevant element of `results.subject_results` (or the top-level grade field) to carry **both** `computedGrade` and `overrideGrade` going forward — the read path (`result.api.md` #9, the marksheet view) displays the override as the operative grade while still surfacing the computed value alongside it, consistent with GRD-008's transparency requirement.

---

## 12. Performance Considerations

**Very low volume** (overrides should be rare, governed exceptions). No performance concerns.