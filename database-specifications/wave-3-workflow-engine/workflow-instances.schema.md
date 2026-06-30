# Workflow Instances

## 1. Table Purpose

Running process instances — version-pinned, stateful, scoped (WFL-002/003) — backs `workflow-instance.api.md` #6–8, #15–20.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `workflow_definition_id` | `UUID` | No | — |
| `current_state` | `VARCHAR(64)` | No | — |
| `lifecycle_status` | `workflow_lifecycle_status (ENUM: ACTIVE, APPROVED, REJECTED, WITHDRAWN, CANCELLED, ORPHANED)` | No | 'ACTIVE' |
| `institute_id` | `UUID` | Yes | — |
| `context` | `JSONB` | No | — |
| `initiated_by` | `UUID` | No | — |
| `initiated_at` | `TIMESTAMPTZ` | No | now() |
| `sla_due_at` | `TIMESTAMPTZ` | Yes | — |
| `stuck_since` | `TIMESTAMPTZ` | Yes | — |
| `stuck_reason` | `VARCHAR(32)` | Yes | — |
| `return_count` | `SMALLINT` | No | 0 |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`workflow_definition_id → workflow_definitions(id)` RESTRICT; `institute_id → institutes(id)` RESTRICT; `initiated_by → users(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK — an institute/workflow combination may have arbitrarily many concurrent instances.

## 6. Indexes

`(initiated_by, lifecycle_status)`; `(institute_id, lifecycle_status)`; `(workflow_definition_id)`; `(lifecycle_status, sla_due_at)` — the orphan/escalation sweep's primary scan path.

## 7. Relationships

Many instances per definition-version; one instance has many `workflow_tasks` (its full lineage).

## 8. Soft Delete Strategy

Standard `deleted_at`, essentially unused — `lifecycle_status` is the real, queried lifecycle marker, identical rationale to `sessions`/`memberships`/`academic_sessions` in Wave 1.

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

**Direct, nullable.** `institute_id` scopes most instances (a fee-waiver approval, an institute-suspend request); a small minority of platform-level instances (e.g., governing a workflow-definition's own change, per Wave 3's self-hosting design) may have it NULL.

## 11. Notes for TypeORM Entity Design

Resolve "my tasks" (the API's `workflow-instance.api.md` #9) by joining `workflow_tasks` rather than querying this table directly — `workflow_instances` itself answers "what's the state of process X," not "what's in my inbox," which is `workflow_tasks`' job. See `workflow_definitions`' notes for this table's role in the mutual-FK migration sequencing.

---

## 12. Performance Considerations

**Moderate-to-high volume at scale** (every governed action across every wave creates one row) **but individually low-touch** — each instance is read/written a handful of times over its lifetime, not repeatedly polled. The `(lifecycle_status, sla_due_at)` index is the orphan/escalation sweep's critical path and should be monitored as overall instance volume grows; this is a reasonable future partitioning candidate (by `initiated_at`) once historical instance volume becomes large, mirroring `audit_log`'s own partitioning recommendation.