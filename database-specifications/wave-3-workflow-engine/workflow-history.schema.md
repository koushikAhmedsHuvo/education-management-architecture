# Workflow History (Tasks)

## 1. Table Purpose

Individual step-occurrences within an instance — the approver-inbox unit and, by virtue of being append-only, the full decision lineage (§0.2). Backs `workflow-instance.api.md` #9–11, #17.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `instance_id` | `UUID` | No | — |
| `step_key` | `VARCHAR(64)` | No | — |
| `resolved_approver_ids` | `UUID[]` | No | — |
| `status` | `workflow_task_status (ENUM: PENDING, DECIDED, ESCALATED, REASSIGNED, EXPIRED)` | No | 'PENDING' |
| `decision` | `workflow_task_decision (ENUM: APPROVE, REJECT, RETURN)` | Yes | — |
| `decided_by` | `UUID` | Yes | — |
| `decided_at` | `TIMESTAMPTZ` | Yes | — |
| `acting_for` | `UUID` | Yes | — |
| `reason` | `TEXT` | Yes | — |
| `sla_due_at` | `TIMESTAMPTZ` | Yes | — |
| `idempotency_key` | `VARCHAR(128)` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`instance_id → workflow_instances(id)` ON DELETE CASCADE — the one deliberate `CASCADE` in this wave, for the same reason `refresh_tokens → sessions` was in Wave 1: a task is a true, meaningless-without-its-parent dependent of its instance; `decided_by → users(id)` RESTRICT; `acting_for → users(id)` RESTRICT.

## 5. Unique Constraints

`(idempotency_key) WHERE idempotency_key IS NOT NULL` — global uniqueness, the direct DB-level backstop behind WFL-010's "a double-submitted click decides exactly once."

## 6. Indexes

**GIN index on `resolved_approver_ids`** (`USING gin (resolved_approver_ids)`) — makes "tasks resolved to user X" (the My Tasks inbox) an index-supported containment query (`WHERE resolved_approver_ids @> ARRAY[$userId]`), not a sequential scan; `(instance_id, created_at)` — the lineage-ordering query (§0.2); `(status, sla_due_at)` — the inbox-sort-by-SLA and escalation-sweep path.

## 7. Relationships

Many tasks per instance (its full history, per §0.2); read by `workflow_delegations` resolution logic (see §4) without a direct FK, since delegation coverage is computed at query time, not stored per-task.

## 8. Soft Delete Strategy

Not soft-deleted; `status` is the complete lifecycle marker. A `RETURN` or reassignment does not delete or soft-delete the prior task row — it leaves it `DECIDED`/`REASSIGNED` and **inserts a new row** for the next occurrence of that step, which is what makes this table double as the lineage table.

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

**Derived, via `workflow_instances → institute_id`.** No direct institute column; a task's scope is entirely inherited from its parent instance.

## 11. Notes for TypeORM Entity Design

The decide-handler's SQL should be a single statement combining the SoD check, the idempotency check, and the status transition (`UPDATE workflow_tasks SET status='DECIDED', decision=$1, decided_by=$2, decided_at=now() WHERE id=$3 AND status='PENDING' AND decided_by IS NULL AND $2 != (SELECT initiated_by FROM workflow_instances WHERE id=instance_id) RETURNING id`) — if zero rows return, the service layer distinguishes "already decided" (idempotent replay) from "SoD violation" by a follow-up read, rather than racing two separate `SELECT`-then-`UPDATE` calls.

---

## 12. Performance Considerations

**The highest-volume table in the Workflow Engine** (potentially several tasks per instance, across every return/escalation/reassignment occurrence, per its append-only lineage design). The **GIN index on `resolved_approver_ids`** is the single most important index in this wave — every approver's 'My Tasks' inbox query depends on it being a genuine index-supported containment scan, not a sequential scan over a growing table. Monitor this index's size and bloat specifically, since `UUID[]` GIN indexes can grow faster than a comparable scalar B-tree.