# Refresh Tokens

## 1. Table Purpose

The rotation chain backing AUTH-002 (rotating refresh tokens with reuse detection) and AUTH-003 (the concurrent-refresh grace window) — never stores a usable token, only its hash.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `session_id` | `UUID` | No | — |
| `token_hash` | `VARCHAR(255)` | No | — |
| `status` | `refresh_token_status (ENUM: CURRENT, ROTATED, REVOKED)` | No | 'CURRENT' |
| `issued_at` | `TIMESTAMPTZ` | No | now() |
| `rotated_at` | `TIMESTAMPTZ` | Yes | — |
| `expires_at` | `TIMESTAMPTZ` | No | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`session_id → sessions(id)` ON DELETE CASCADE (a token has no meaning once its session row is gone — this is the one deliberate `CASCADE` in Wave 1, justified because `refresh_tokens` is a pure dependent/child of `sessions`, never referenced from elsewhere).

## 5. Unique Constraints

`(token_hash)` globally unique (not soft-delete-partial — a hash collision must never be possible regardless of row state).

## 6. Indexes

`(token_hash)` — the hot lookup path on every refresh call; `(session_id, status)`.

## 7. Relationships

Many `refresh_tokens` per `session` (the chain), exactly one of which is `CURRENT` at any time — enforced as below.

## 8. Soft Delete Strategy

Not soft-deleted; lifecycle is fully captured by `status`. Rows are retained (not purged) for the duration needed to detect reuse of a stale token presented after rotation, then garbage-collected by a retention job past `expires_at`.

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

**Deployment-global, via `sessions → users`.** No direct institute scoping; isolation is purely ownership-based (a token belongs to exactly one session, one user).

## 11. Notes for TypeORM Entity Design

Never log or return `token_hash` in any API response (it already isn't, per `auth.api.md`, but worth stating at the schema level too — mark `@Column({ select: false })`). The grace-window check (AUTH-003) reads the *immediately preceding* `ROTATED` row by `rotated_at DESC LIMIT 1` — keep that index-friendly via `(session_id, rotated_at DESC)`.

---

## 12. Performance Considerations

**Very high write volume** — every access-token refresh (typically every 15 minutes per active session, per `auth_policies.access_token_ttl_minutes`) inserts a new row here. The `(token_hash)` unique index is the single hottest lookup in the Authentication domain, hit on literally every refresh call; keep this index as lean as possible (no included columns beyond what `token_hash` itself needs). Old `ROTATED`/`REVOKED` rows past the grace window have zero ongoing value and should be purged aggressively — this table will otherwise grow faster than almost any other in the schema.