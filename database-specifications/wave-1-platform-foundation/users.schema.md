# Users

## 1. Table Purpose

The core identity and authentication-state entity (Doc 01 §6 state machine) — shared by `auth.api.md` (login/session mechanics) and `user.api.md` (account lifecycle administration). One row per person per **deployment**, never per institute (§0.1).

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `email` | `CITEXT` | No | — |
| `display_name` | `VARCHAR(255)` | No | — |
| `password_hash` | `VARCHAR(255)` | Yes | — |
| `status` | `user_status (ENUM: INVITED, ACTIVE, LOCKED, SUSPENDED, DEACTIVATED, ANONYMIZED)` | No | 'INVITED' |
| `mfa_enabled` | `BOOLEAN` | No | false |
| `failed_login_count` | `SMALLINT` | No | 0 |
| `locked_until` | `TIMESTAMPTZ` | Yes | — |
| `last_login_at` | `TIMESTAMPTZ` | Yes | — |
| `password_changed_at` | `TIMESTAMPTZ` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

None outbound (the root identity table).

## 5. Unique Constraints

`(email) WHERE deleted_at IS NULL` — partial unique, per §0.4 (a deactivated/anonymized user's email becomes reusable).

## 6. Indexes

`(status)` partial `WHERE deleted_at IS NULL` (admin list-screen filter, `user.api.md` #1); `(locked_until)` partial `WHERE locked_until IS NOT NULL` (for the lockout-expiry sweep).

## 7. Relationships

Referenced by `memberships` (a user's scoped roles), `sessions`/`refresh_tokens`/`mfa_*`/`auth_tokens` (a user's own auth state), and as `created_by`/`updated_by`/`actor_user_id` from essentially every other table in the system.

## 8. Soft Delete Strategy

Standard `deleted_at`, but in practice a user's **lifecycle status** (`DEACTIVATED`/`ANONYMIZED`) is the operative state most queries filter on, not `deleted_at` directly — `deleted_at` is reserved for the rare case of an erroneously-created row being struck entirely, while `ANONYMIZED` (a distinct, retained status) is how Doc 06's STU-007-style "anonymize, never hard-delete" pattern is realized for staff/user identities specifically.

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

**Deployment-global, deliberately not institute-scoped.** A `users` row represents one identity for the entire deployment, since the same person can hold a Teacher membership at Institute A and an Institute Admin membership at Institute B simultaneously (an explicit Doc 02 edge case). Tenant isolation for *what a user can see and do* is enforced entirely through the `memberships` table, never by partitioning this table itself — any attempt to add `institute_id` here would be a modeling error that breaks the documented multi-institute identity-sharing requirement.

## 11. Notes for TypeORM Entity Design

Use the `citext` Postgres extension (`CREATE EXTENSION IF NOT EXISTS citext;` in the initial migration) and map `email` as `@Column({ type: 'citext' })` — do **not** approximate case-insensitivity with `LOWER(email)` application-side comparisons, which silently breaks the partial unique index's guarantee. `password_hash` should never be selected by default (`@Column({ select: false })`) so a stray `find()` never accidentally serializes it.

---

## 12. Performance Considerations

**High read volume, low write volume.** Every authenticated request resolves the caller's identity from this table (typically via a cached JWT claim, not a fresh row read, after initial login) — the `(email)` partial unique index is the hot path for login specifically. `password_hash` is marked non-selectable by default specifically to keep routine `find()` calls from pulling an unnecessary, sensitive 60+ byte column on every read.