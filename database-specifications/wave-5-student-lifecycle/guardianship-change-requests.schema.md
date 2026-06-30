# Guardianship Change Requests

## 1. Table Purpose

Governed, reasoned, dated guardianship changes (GRD-N-007) — backs `guardian.api.md` #14–16.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `student_id` | `UUID` | No | — |
| `change_type` | `guardianship_change_type (ENUM: ADD, REMOVE, REPLACE)` | No | — |
| `target_link_id` | `UUID` | Yes | — |
| `new_guardian_id` | `UUID` | Yes | — |
| `reason` | `TEXT` | No | — |
| `effective_date` | `DATE` | No | — |
| `status` | `change_request_status (ENUM: PENDING_APPROVAL, APPROVED, REJECTED)` | No | 'PENDING_APPROVAL' |
| `requested_by` | `UUID` | No | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`student_id → students(id)` RESTRICT; `target_link_id → student_guardian_links(id)` RESTRICT, nullable; `new_guardian_id → guardians(id)` RESTRICT, nullable; `requested_by → users(id)` RESTRICT; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK.

## 6. Indexes

`(student_id, status)`; `(status)` for the approval inbox.

## 7. Relationships

Approval, on commit, writes to `student_guardian_links` (revoking `target_link_id` and/or creating a new link to `new_guardian_id`) — never directly mutates those rows outside this governed flow.

## 8. Soft Delete Strategy

Standard; retained indefinitely as the disclosed historical record of guardianship changes, which `guardian.api.md` itself treats as worth surfacing in guardian-history views.

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

**Direct, via `student_id → students → institute_id`.** No direct column, but unlike most Wave 5 junction tables this one is queried institute-wide for the approvals inbox, so `(status)` alone (without an institute filter) is the index, relying on the application's own scope-filtering at the query-building layer rather than the index itself partitioning by institute.

## 11. Notes for TypeORM Entity Design

The approval-commit handler must perform the last-guardian-of-minor re-check, the `target_link_id`/`new_guardian_id` writes, and the resulting notification/access re-resolution all inside one transaction — partial application here (e.g., the old guardian revoked but the new one not yet linked) would transiently leave a minor with zero guardians, the exact state GRD-N-002 exists to prevent.

---

## 12. Performance Considerations

**Very low volume** (guardianship changes are rare, deliberate, governed events). No performance concerns.