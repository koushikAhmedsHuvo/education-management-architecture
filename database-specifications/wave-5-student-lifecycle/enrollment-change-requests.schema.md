# Enrollment Change Requests

## 1. Table Purpose

Governed inter-section transfer and withdrawal requests (ENR-005/006), unified per §0.5 — backs `enrollment.api.md` #7–10, #12.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `enrollment_id` | `UUID` | No | — |
| `request_type` | `enrollment_request_type (ENUM: TRANSFER, WITHDRAWAL)` | No | — |
| `target_section_id` | `UUID` | Yes | — |
| `is_cross_institution_exit` | `BOOLEAN` | No | false |
| `clearance_checklist` | `JSONB` | Yes | — |
| `reason` | `TEXT` | No | — |
| `effective_date` | `DATE` | No | — |
| `status` | `change_request_status (reuses the ENUM from §8: PENDING_APPROVAL, APPROVED, REJECTED)` | No | 'PENDING_APPROVAL' |
| `requested_by` | `UUID` | No | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`enrollment_id → enrollments(id)` RESTRICT; `target_section_id → sections(id)` RESTRICT, nullable; `requested_by → users(id)` RESTRICT; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK — a student could in principle have a withdrawal request submitted, rejected, and a transfer request submitted afterward; the rules don't forbid sequential requests.

## 6. Indexes

`(status, request_type)` — the combined approvals inbox (`enrollment.api.md` #8); `(enrollment_id)`.

## 7. Relationships

On approval, mutates `enrollments` (closing the current row and, for `TRANSFER`, opening a new one per §11's append-only pattern; for `WITHDRAWAL`, simply setting `status = 'WITHDRAWN'` with no successor row) and, for a cross-institution `WITHDRAWAL`, generates a `documents` row (`document_type = 'TRANSFER_CERTIFICATE'`).

## 8. Soft Delete Strategy

Standard; retained indefinitely as the disclosed historical record `enrollment.api.md`'s own screens surface.

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

**Derived, via `enrollment_id → enrollments → institute_id`.** No direct institute column.

## 11. Notes for TypeORM Entity Design

Approval-commit logic for `TRANSFER` is structurally identical to a direct `enrollments` transfer (§11's TypeORM notes) — both close the current row and open a new one in one transaction — the only difference is that this table's approval gate runs first. For `WITHDRAWAL`, approval-commit sets the current `enrollments` row's `status = 'WITHDRAWN'`, `period_to = effective_date`, and, where `is_cross_institution_exit = true`, triggers transfer-certificate generation as part of the same transaction's follow-up work (enqueued, not synchronous, since document generation may be slow).

---

## 12. Performance Considerations

**Low-to-moderate volume (transfers and withdrawals are real but infrequent relative to total enrollment count).** No special performance concerns beyond the standard approvals-inbox index pattern shared with every other governed-request table in this schema.