# Result Revisions

## 1. Table Purpose

Governed, version-pinned post-publish revision requests (RES-006) — backs `result.api.md` #12–14.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `result_id` | `UUID` | No | — |
| `reason` | `TEXT` | No | — |
| `re_evaluation_context` | `JSONB` | Yes | — |
| `direction` | `VARCHAR(16)` | No | — |
| `status` | `change_request_status (reuses the Wave-5 ENUM)` | No | 'PENDING_APPROVAL' |
| `requested_by` | `UUID` | No | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`result_id → results(id)` RESTRICT; `requested_by → users(id)` RESTRICT; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK.

## 6. Indexes

`(status)`.

## 7. Relationships

On approval, this is the row whose commit handler performs the `results` append-only revision transaction described in §7's TypeORM notes.

## 8. Soft Delete Strategy

Standard; retained indefinitely as the disclosed revision-request history.

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

This table is intentionally **separate** from `mark_corrections` (§6) despite superficial similarity, because the two operate at different layers — a mark correction changes an input; a result revision changes (or re-derives) an output — and, per §6's own notes, an approved mark correction on an already-published result's underlying data is precisely what *creates* a row here, rather than the two being the same governed action wearing two names.

---

## 12. Performance Considerations

**Very low volume** (post-publish revisions should be rare exceptions). No performance concerns.