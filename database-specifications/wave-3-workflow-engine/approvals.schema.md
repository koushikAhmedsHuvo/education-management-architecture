# Approvals (Delegation of Authority)

## 1. Table Purpose

Bounded, audited delegation of approval authority (WFL-006) — backs `workflow-instance.api.md` #12–14.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `delegator_id` | `UUID` | No | — |
| `delegate_id` | `UUID` | No | — |
| `institute_id` | `UUID` | Yes | — |
| `period_from` | `TIMESTAMPTZ` | No | — |
| `period_to` | `TIMESTAMPTZ` | No | — |
| `chain_depth` | `SMALLINT` | No | 1 |
| `status` | `delegation_status (ENUM: ACTIVE, EXPIRED, REVOKED)` | No | 'ACTIVE' |
| `revoked_at` | `TIMESTAMPTZ` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`delegator_id → users(id)` RESTRICT; `delegate_id → users(id)` RESTRICT; `institute_id → institutes(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK — a delegator may set up multiple concurrent delegations to different delegates for different scopes, and the rules don't forbid overlapping periods to different delegates (unlike `config_values`, there's no "exactly one wins" semantic here to protect with an exclusion constraint).

## 6. Indexes

**`(delegate_id, status, period_from, period_to)`** — the resolution query every task-inbox lookup runs ("is this user covered by an active delegation from someone in this task's `resolved_approver_ids`"); `(delegator_id, status)` for "my granted delegations."

## 7. Relationships

Read (not joined via FK) by the task-resolution query in §3; references only `users`.

## 8. Soft Delete Strategy

Standard `deleted_at`, though `status = 'EXPIRED'`/`'REVOKED'` is the real, queried lifecycle — a scheduled sweep flips `ACTIVE → EXPIRED` the moment `period_to` passes (mirroring `config_values`' activation-sweep pattern from Wave 2, here running in the opposite direction).

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

**Direct, nullable.** `institute_id` scopes a delegation to one institute's approval authority where set; NULL would represent a deployment-wide delegation (uncommon in practice, but structurally permitted).

## 11. Notes for TypeORM Entity Design

The "is user X covered by an active delegation" resolution used by the My Tasks query is: `EXISTS (SELECT 1 FROM workflow_delegations WHERE delegate_id = $userId AND status = 'ACTIVE' AND now() BETWEEN period_from AND period_to AND delegator_id = ANY($task.resolved_approver_ids))` — implement as a repository-level subquery or a small SQL view (`active_delegations_now`) rather than materializing it, since delegation volume is low enough that a live `now()`-bounded query is cheap and always exactly correct (a materialized/cached version would need its own invalidation logic for no real benefit at this scale).

---

## 12. Performance Considerations

**Low volume (a handful of active delegations at any time, across the whole deployment), but read on every single task-resolution check** (the 'is this user covered by a delegation' subquery runs for every My Tasks inbox load). Given the low row count, a live, uncached query is appropriately cheap here — caching would add complexity disproportionate to the actual query cost at this table's realistic scale.