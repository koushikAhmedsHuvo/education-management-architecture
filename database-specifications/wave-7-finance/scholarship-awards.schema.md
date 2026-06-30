# Scholarship Awards

## 1. Table Purpose

Individual scholarship selections/awards with conflict-of-interest controls (SCH-002/003) — backs `scholarship.api.md` #5–9.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `program_id` | `UUID` | No | — |
| `student_id` | `UUID` | No | — |
| `eligibility_score` | `NUMERIC(6,2)` | Yes | — |
| `status` | `scholarship_award_status (ENUM: SHORTLISTED, PENDING_APPROVAL, AWARDED, ACTIVE, REVOKED, COMPLETED)` | No | 'SHORTLISTED' |
| `selected_by` | `UUID` | Yes | — |
| `coi_acknowledged` | `BOOLEAN` | No | false |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`program_id → scholarship_programs(id)` RESTRICT; `student_id → students(id)` RESTRICT; `selected_by → users(id)` RESTRICT, nullable; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

`(program_id, student_id) WHERE status NOT IN ('REVOKED') AND deleted_at IS NULL` — a student holds at most one non-revoked award per program at a time (a revoked-then-reawarded student gets a fresh row, the same "no resurrection, new row instead" pattern as Wave 1's `memberships`).

## 6. Indexes

`(program_id, status)`; `(student_id)`.

## 7. Relationships

Many-to-one with `scholarship_programs`; has many `scholarship_disbursements` and `scholarship_award_reviews`.

## 8. Soft Delete Strategy

Standard; `status = 'REVOKED'`/`'COMPLETED'` is the disclosed end state.

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

**Derived, via `program_id → scholarship_programs → institute_id`.** No direct institute column.

## 11. Notes for TypeORM Entity Design

`coi_acknowledged` must be explicitly set `true` by the selecting committee member before submission proceeds — modeled as a plain boolean here, with the actual COI-declaration detail (who declared what conflict) living in `audit_log` rather than a dedicated table, since COI declarations are infrequent, low-volume events better served by the generic audit trail than a bespoke table.

---

## 12. Performance Considerations

**Low-to-moderate volume (bounded by each program's `slots`, summed across all programs at an institute).** No special performance concerns beyond the slot-consumption contention already noted on the parent `scholarship_programs` table.