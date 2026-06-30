# Section Restructure Requests

## 1. Table Purpose

Pending merge/split/close operations with a re-validatable transfer map (SEC-006), per §0.3 — backs `section.api.md` #7–9.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `operation` | `restructure_operation (ENUM: MERGE, SPLIT, CLOSE)` | No | — |
| `source_section_ids` | `UUID[]` | No | — |
| `target_section_ids` | `UUID[]` | No | — |
| `transfer_map` | `JSONB` | No | — |
| `effective_date` | `DATE` | No | — |
| `status` | `restructure_status (ENUM: PENDING_APPROVAL, APPROVED, REJECTED)` | No | 'PENDING_APPROVAL' |
| `requested_by` | `UUID` | No | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`requested_by → users(id)` RESTRICT; `workflow_instance_id → workflow_instances(id)` RESTRICT. **No FK array constraint on `source_section_ids`/`target_section_ids`** — Postgres cannot express "every element of this array is a valid foreign key" declaratively; validated at the application layer when the request is submitted and again at approval time.

## 5. Unique Constraints

None beyond the PK.

## 6. Indexes

`(status)`; GIN index on `source_section_ids` and on `target_section_ids` if "find pending restructures touching section X" becomes a common query (omitted by default — low volume, rare operation).

## 7. Relationships

References many `sections` (both as source and target) without typed FKs, for the array-of-FK reason above; one approved request ultimately mutates those `sections` rows' `status` and, transitively, Wave 5's `enrollments`.

## 8. Soft Delete Strategy

Standard; in practice rows are retained indefinitely as the historical record of *why* a section was merged/split/closed, which `section.api.md`'s own UI surfaces as part of section history.

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

**Derived, via the `source_section_ids`/`target_section_ids` arrays' parent sections** — no direct institute column, and notably **not enforceable via FK** at all (array-of-references), so the application layer must independently verify all referenced sections belong to the same institute before allowing a restructure to proceed.

## 11. Notes for TypeORM Entity Design

Approval execution (triggered when the linked `workflow_instances` row reaches `APPROVED`) must run the capacity re-check and the actual per-student transfers (writing to Wave 5's `enrollment_history`) inside a single transaction — a partially-applied restructure (some students moved, some not) is exactly the kind of inconsistent state SEC-006's "preserves history via transfers" promise depends on never happening.

---

## 12. Performance Considerations

**Very low volume (restructuring is a rare, deliberate administrative event), but each row's `transfer_map` JSONB can be large** (one entry per affected student). No indexing concerns at this table's realistic write frequency; the approval-commit transaction's cost is dominated by the per-student `enrollments` writes it triggers, not by this table itself.