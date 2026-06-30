# Sessions

## 1. Table Purpose

Active login sessions/devices (Doc 01 AUTH-005/006) — backs `auth.api.md` #5–7 and `user.api.md` #13–14 (admin session management).

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `user_id` | `UUID` | No | — |
| `family_id` | `UUID` | No | — |
| `device_label` | `VARCHAR(255)` | Yes | — |
| `ip_address` | `INET` | Yes | — |
| `user_agent` | `TEXT` | Yes | — |
| `status` | `session_status (ENUM: ACTIVE, REVOKED)` | No | 'ACTIVE' |
| `revoked_reason` | `VARCHAR(64)` | Yes | — |
| `last_active_at` | `TIMESTAMPTZ` | No | now() |
| `revoked_at` | `TIMESTAMPTZ` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`user_id → users(id)` ON DELETE RESTRICT (a user is anonymized/deactivated, not deleted, while sessions exist — see §3 notes).

## 5. Unique Constraints

None beyond the PK.

## 6. Indexes

`(user_id, status)` — the dominant "list my/this user's active sessions" query; `(family_id)`.

## 7. Relationships

One user has many sessions; one session has many `refresh_tokens` (its rotation history).

## 8. Soft Delete Strategy

Not soft-deleted — sessions are **revoked** (`status = 'REVOKED'`, `revoked_at` set), which is the real-world equivalent and is queried directly rather than via `deleted_at`/`IS NULL` filtering, since "show revoked sessions" is itself a legitimate admin view (session history), unlike most soft-deleted data.

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

**Deployment-global, via `users`.** A session belongs to one user, not one institute — a user's session persists across scope-switches between institutes they hold membership in. No `institute_id` column exists or is needed here.

## 11. Notes for TypeORM Entity Design

Do not soft-delete-filter this entity by `deleted_at` in default queries; filter by `status = 'ACTIVE'` instead. A scheduled job should periodically hard-delete (or archive) long-revoked session rows past a retention window, since session volume grows unboundedly otherwise — this table is the one Wave-1 table most likely to need partitioning (by `created_at`, monthly) at scale; design the migration to make that an additive change later, not a rewrite.

---

## 12. Performance Considerations

**High write volume (every login, every refresh touches `last_active_at`), short-to-medium row lifetime.** The `(user_id, status)` index is the dominant query (a user's own active-sessions list, and admin session management). This table is the schema's strongest partitioning candidate after `audit_log` — partition by `created_at` and archive/purge long-`REVOKED` rows past a retention window, since session volume grows unboundedly with login frequency and most rows have no value beyond a few months.