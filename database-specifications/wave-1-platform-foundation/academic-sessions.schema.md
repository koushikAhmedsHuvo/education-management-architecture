# Academic Sessions

## 1. Table Purpose

The time dimension every academic and financial record attaches to (Doc 05) — backs `academic-session.api.md`. Deliberately supports **controlled overlap** (a current session running while a future one takes admissions, SESS-002) while guaranteeing **exactly one `is_current` session per institute** (SESS-003).

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `name` | `VARCHAR(64)` | No | — |
| `start_date` | `DATE` | No | — |
| `end_date` | `DATE` | Yes | — |
| `open_ended` | `BOOLEAN` | No | false |
| `admission_open` | `BOOLEAN` | No | false |
| `is_current` | `BOOLEAN` | No | false |
| `status` | `session_status_enum (ENUM: DRAFT, ACTIVE, CLOSED, ARCHIVED)` | No | 'DRAFT' |
| `structure_instantiated` | `BOOLEAN` | No | false |
| `closed_at` | `TIMESTAMPTZ` | Yes | — |
| `reopened_at` | `TIMESTAMPTZ` | Yes | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` ON DELETE RESTRICT.

## 5. Unique Constraints

`(institute_id, name) WHERE deleted_at IS NULL`; **`(institute_id) WHERE is_current = true AND deleted_at IS NULL`** — the precise, single-statement DB-level enforcement of SESS-003's "exactly one current session per institute," exactly mirroring the `campuses.is_default` pattern above; setting a new session current and demoting the previous one is therefore done as a single transaction that updates both rows together (the old current → `is_current = false`, the new one → `is_current = true`), which the partial unique index will reject if done out of order, giving the application a hard backstop against the race condition this invariant is most at risk from.

## 6. Indexes

`(institute_id, status)`; `(institute_id, start_date DESC)` — the default list sort (`academic-session.api.md` #1).

## 7. Relationships

Many sessions per institute; referenced in later waves by every session-scoped academic/financial record (`class_instances`, `enrollments`, `invoices`, etc.).

## 8. Soft Delete Strategy

Standard `deleted_at`, though `status = 'CLOSED'`/`'ARCHIVED'` is the disclosed lifecycle (SESS-005/007) — closing a session does **not** soft-delete it; it makes its dependent records read-only (an application-layer enforcement point checked at every write against session-scoped data in later waves, not expressible here since those tables don't exist in this file).

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

**Direct, mandatory.** `institute_id` is NOT NULL; every session belongs to exactly one institute, and the `(institute_id) WHERE is_current = true` partial unique index is itself a per-tenant invariant (one current session *per institute*, not one globally).

## 11. Notes for TypeORM Entity Design

The rollover/promotion **job** itself is tracked via the generic `async_jobs` table (§2, `job_type = 'SESSION_ROLLOVER'`) — no dedicated `session_rollover_jobs` table, per the cross-wave design note; **per-student** promotion outcomes (`STUDENT_PROMOTED`/`RETAINED`/`GRADUATED`) are recorded in Wave 5's `enrollment_history`, not here, since they're properties of a specific student's enrollment, not of the session row itself. The close-readiness checklist result (`unpublished_results`, `outstanding_dues`, `open_workflows` counts) is written as a single `audit_log` row (`action = 'SESSION_CLOSE_CHECK'`, `after_state` holding the checklist JSONB) at the moment of the close attempt, rather than a dedicated `session_lifecycle_events` table, per §0.6.

---

## 12. Performance Considerations

**Low volume (a handful of sessions per institute, accumulating slowly over years), but high *fan-out* significance** — nearly every Wave 4–7 table either directly or transitively scopes through a session. The `(institute_id, start_date DESC)` index supports the default list view cheaply; the real performance weight of this table is in how heavily other tables join through it, not in this table's own row count.