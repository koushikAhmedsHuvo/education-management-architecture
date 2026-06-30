# Scholarship Award Reviews

## 1. Table Purpose

Condition-based renewal/revocation checkpoints with due process (SCH-007/008) — backs `scholarship.api.md` #14–15.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `award_id` | `UUID` | No | — |
| `review_type` | `award_review_type (ENUM: RENEWAL, REVOCATION)` | No | — |
| `checkpoint_basis` | `JSONB` | No | — |
| `decision` | `award_review_decision (ENUM: RENEWED, REVOKED, PROBATION)` | Yes | — |
| `reason` | `TEXT` | No | — |
| `reviewed_by` | `UUID` | Yes | — |
| `reviewed_at` | `TIMESTAMPTZ` | Yes | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`award_id → scholarship_awards(id)` RESTRICT; `reviewed_by → users(id)` RESTRICT, nullable; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK — an award is reviewed repeatedly over its lifetime (each renewal checkpoint, and potentially a revocation review later).

## 6. Indexes

`(award_id, review_type)`.

## 7. Relationships

Many-to-one with `scholarship_awards`; on `decision = 'REVOKED'`, this is what flips the parent award's `status`.

## 8. Soft Delete Strategy

Standard; retained permanently as the disclosed due-process record.

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

**Derived, via `award_id → scholarship_awards → scholarship_programs → institute_id`.** No direct institute column.

## 11. Notes for TypeORM Entity Design

**Revocation is forward-looking only** — approving a `REVOCATION` review stops *future* `scholarship_disbursements` (by setting the award `status = 'REVOKED'`, which the disbursement-creation path checks before allowing a new row) but **never** retroactively touches already-`DISBURSED` rows in §18, structurally guaranteed by that table's append-only, no-update nature — SCH-008's "no silent clawback" is, like several other guarantees in this schema, true by the absence of any mechanism to violate it, not by an explicit check that could be bypassed.

---

## 12. Performance Considerations

**Very low volume** (renewal/revocation checkpoints occur on a fixed, typically termly or yearly, schedule per award — not a high-frequency event). No performance concerns.