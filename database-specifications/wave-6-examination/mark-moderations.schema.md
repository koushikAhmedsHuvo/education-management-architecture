# Mark Moderations

## 1. Table Purpose

Bounded, transparent grace/moderation as a separate layer over raw marks (EXM-008) — backs `marks.api.md` #13... (the moderation endpoint within `exam.api.md`/`marks.api.md`'s combined surface).

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `mark_sheet_id` | `UUID` | No | — |
| `rule` | `moderation_rule (ENUM: GRACE_TO_PASS, UNIFORM_SCALING)` | No | — |
| `bound` | `NUMERIC(6,2)` | No | — |
| `adjustments` | `JSONB` | No | — |
| `status` | `moderation_status (ENUM: APPLIED, PENDING_APPROVAL)` | No | 'APPLIED' |
| `applied_by` | `UUID` | No | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`mark_sheet_id → mark_sheets(id)` RESTRICT; `applied_by → users(id)` RESTRICT; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK — multiple moderation rounds against the same sheet are permitted (e.g., grace applied, then a separate uniform-scaling pass), each its own row.

## 6. Indexes

`(mark_sheet_id, status)`.

## 7. Relationships

Many-to-one with `mark_sheets`; read by result computation to resolve each student's effective mark (raw `marks.mark`, adjusted by the latest applicable row here, if any) — joined, never merged into `marks` itself, per §0.4.

## 8. Soft Delete Strategy

Standard `deleted_at`; a moderation round is essentially never deleted in practice, since it's part of the transparent, disclosed record EXM-008 requires.

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

Result computation's "effective mark" resolution is `COALESCE((adjustments JSONB lookup for this student in the latest APPLIED moderation row for this sheet), marks.mark)` — implement as a small `EffectiveMarkResolver` service method, not inline SQL repeated at every call site, since both `results` computation and any "preview the moderated grid" UI need the identical resolution logic.

---

## 12. Performance Considerations

**Low volume (moderation is an occasional, deliberate administrative action, not a routine one per mark sheet).** No special performance concerns; the `adjustments` JSONB array size is bounded by the sheet's own roster size, kept manageable by the same volume ceiling that bounds `marks` itself.