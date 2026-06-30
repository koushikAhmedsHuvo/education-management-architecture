# Memberships

## 1. Table Purpose

The (user, scope, role) assignment — Doc 02 §6 state machine (`ACTIVE → SUSPENDED/REVOKED`) — the single table that actually grants a user access, and therefore the most frequently queried table in the entire schema (every authenticated request resolves effective permissions through it).

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `user_id` | `UUID` | No | — |
| `institute_id` | `UUID` | No | — |
| `campus_id` | `UUID` | Yes | — |
| `role_id` | `UUID` | No | — |
| `status` | `membership_status (ENUM: ACTIVE, SUSPENDED, REVOKED, PENDING_APPROVAL)` | No | 'ACTIVE' |
| `granted_by` | `UUID` | No | — |
| `granted_at` | `TIMESTAMPTZ` | No | now() |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`user_id → users(id)` RESTRICT; `institute_id → institutes(id)` RESTRICT; `campus_id → campuses(id)` RESTRICT; `role_id → roles(id)` RESTRICT; `granted_by → users(id)` RESTRICT.

## 5. Unique Constraints

`(user_id, institute_id, campus_id, role_id) WHERE deleted_at IS NULL AND status != 'REVOKED'` — prevents a duplicate active grant of the identical (user, scope, role) combination; deliberately allows the combination to be re-granted after a prior grant was `REVOKED` (a new row, not a resurrected old one — Doc 02 §6: "`REVOKED → ACTIVE` forbidden; a re-grant creates a new membership").

## 6. Indexes

**`(user_id, institute_id, campus_id, status)`** — the critical hot-path index, hit on essentially every permission-resolution call (AUTHZ-001/002); `(role_id, status)` for `role-permission.api.md` #7's membership list and #6's deprecation-impact count; `(institute_id, status)`.

## 7. Relationships

The central many-to-many-with-attributes join between `users`, `institutes`/`campuses` (scope), and `roles`.

## 8. Soft Delete Strategy

Standard `deleted_at`, but as with `sessions`/`roles`, `status = 'REVOKED'` (not `deleted_at`) is the operative end-of-life marker users and admins actually see and query against — `deleted_at` again reserved for true erroneous-row correction.

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

**Direct, mandatory — the schema's actual tenant-isolation mechanism.** Every row carries a NOT NULL `institute_id`; this is the table application-layer scope-filtering (AUTHZ-002) is built around, and the one place in the entire schema where 'multi-institute isolation' is not a design choice but the table's literal reason for existing.

## 11. Notes for TypeORM Entity Design

This is the table to `EXPLAIN ANALYZE` first if Wave-1 performance work is ever needed — keep the composite index above as a genuine `(user_id, institute_id, campus_id, status)` B-tree, not an index TypeORM happens to generate incidentally from decorator order; consider a Redis-backed effective-permissions cache in front of this table from day one (the Architecture Blueprint's stated direction) rather than treating that as a later optimization, given how hot this path is.

---

## 12. Performance Considerations

**The single most performance-critical table in the entire schema** — every authenticated request resolves effective permissions through it. The `(user_id, institute_id, campus_id, status)` composite index must be a genuine, dedicated B-tree, not an incidental side effect of decorator ordering. **Strongly recommend a Redis-backed effective-permissions cache in front of this table from initial launch**, keyed by `(user_id, institute_id)` and invalidated the instant a membership's `version` changes (per the Audit Fields note on `roles`), rather than treating caching as a later optimization — at any meaningful user count, hitting Postgres for this lookup on every single request is the schema's most likely early bottleneck.