# Class Session Instances

## 1. Table Purpose

The thin per-session instantiation of a class (SESS-004) — created by `06-academic-session-api.md` #5, read by everything in this wave that needs "this class, in this specific session."

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `class_id` | `UUID` | No | — |
| `session_id` | `UUID` | No | — |
| `status` | `class_instance_status (ENUM: ACTIVE, CLOSED)` | No | 'ACTIVE' |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`class_id → classes(id)` RESTRICT; `session_id → academic_sessions(id)` RESTRICT.

## 5. Unique Constraints

`(class_id, session_id) WHERE deleted_at IS NULL` — instantiation is idempotent per `06-academic-session-api.md` #5's own contract ("does not duplicate existing instances"); this constraint is exactly what makes a re-run of that endpoint safe to retry.

## 6. Indexes

`(session_id, status)`; `(class_id)`.

## 7. Relationships

One per (class, session); one instance has many `sections`.

## 8. Soft Delete Strategy

Standard; `status = 'CLOSED'` mirrors the parent `academic_sessions.status = 'CLOSED'` (set together, in the same transaction, when a session closes) rather than being independently managed.

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

**Derived, via `classes → institute_id` and `academic_sessions → institute_id`** (both parents agree, by construction — an instantiation can never cross institutes since both FKs originate from the same institute's own definition and session).

## 11. Notes for TypeORM Entity Design

Bulk-insert all of an institute's `ACTIVE_DEFINITION` classes as instances in a single statement when `06-academic-session-api.md` #5 runs (`INSERT ... SELECT ... ON CONFLICT (class_id, session_id) DO NOTHING`), not a per-class loop — this table existing as a separate, intentionally-minimal row is precisely what makes that bulk operation a single fast statement instead of N application-level round trips.

---

## 12. Performance Considerations

**Low volume but bulk-write bursty** — an entire institute's class roster is inserted in one batch every time a new session is instantiated (`06-academic-session-api.md` #5). The `ON CONFLICT (class_id, session_id) DO NOTHING` upsert pattern keeps re-runs cheap (a no-op for already-instantiated pairs) rather than requiring an expensive pre-check query.