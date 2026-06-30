# Guardian Restrictions

## 1. Table Purpose

Custody/contact restrictions on a specific student-guardian relationship, enforced system-wide and fail-closed (GRD-N-006) — backs `student.api.md` #13 / `guardian.api.md` #9–11 (the same underlying write, two API entry points, per the cross-reference already noted in the API contract).

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `student_guardian_link_id` | `UUID` | No | — |
| `type` | `restriction_type (ENUM: NO_ACCESS, NO_CONTACT, SUPERVISED)` | No | — |
| `reason` | `TEXT` | No | — |
| `document_ref` | `VARCHAR(255)` | Yes | — |
| `status` | `restriction_status (ENUM: ACTIVE, REMOVED)` | No | 'ACTIVE' |
| `applied_by` | `UUID` | No | — |
| `applied_at` | `TIMESTAMPTZ` | No | now() |
| `removed_by` | `UUID` | Yes | — |
| `removed_at` | `TIMESTAMPTZ` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`student_guardian_link_id → student_guardian_links(id)` RESTRICT; `applied_by → users(id)` RESTRICT; `removed_by → users(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK — multiple restriction types can legitimately stack on one link (e.g., `NO_CONTACT` and `SUPERVISED` simultaneously).

## 6. Indexes

**`(student_guardian_link_id) WHERE status = 'ACTIVE' AND deleted_at IS NULL`** — the hot path: every portal-access check (`guardian.api.md` #7), every document download (`student.api.md` #18), and every notification-routing decision queries this index to fail closed before granting access.

## 7. Relationships

Many-to-one with `student_guardian_links`; read (not joined via FK, for the same "checked at the point of access" reason as `workflow_delegations`) by every guardian-facing read path across the system.

## 8. Soft Delete Strategy

Not soft-deleted; `status = 'REMOVED'` plus `removed_at`/`removed_by` is the complete, disclosed lifecycle — removal is itself a governed, audited action per the API contract, never a casual toggle, so the full record (both application and removal) is permanently retained.

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

**Derived, via `student_guardian_links → students → institute_id`.** No direct institute column.

## 11. Notes for TypeORM Entity Design

Build a single shared `GuardianAccessGuard` service every portal/document/notification code path calls, rather than each module independently querying this table — the fail-closed guarantee GRD-N-006 demands is only as strong as its weakest, most-likely-to-be-forgotten call site, so centralizing the check in one reusable guard is a correctness requirement here, not just a DRY nicety.

---

## 12. Performance Considerations

**Low volume relative to `student_guardian_links`** (restrictions are the exception, not the norm), **but this table's single partial index is queried on an extremely hot path** — every portal access, every document download, every notification dispatch across the entire system checks it. Despite low row count, treat this index's query latency as a first-class performance concern precisely because of *how often*, not how much data, it's consulted — a centralized, well-cached `GuardianAccessGuard` (per the TypeORM Notes) matters more for overall system performance here than any schema-level optimization could.