# Auth Tokens

## 1. Table Purpose

A single generic table for every single-use, hashed, expiring token the Authentication module issues — password-reset tokens (AUTH-009) and invitation tokens (AUTH-011) share an identical shape, so one table with a `purpose` discriminator replaces what would otherwise be two near-duplicate tables.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `user_id` | `UUID` | No | — |
| `purpose` | `auth_token_purpose (ENUM: PASSWORD_RESET, INVITATION)` | No | — |
| `token_hash` | `VARCHAR(255)` | No | — |
| `expires_at` | `TIMESTAMPTZ` | No | — |
| `consumed_at` | `TIMESTAMPTZ` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`user_id → users(id)` ON DELETE RESTRICT.

## 5. Unique Constraints

`(token_hash)` globally unique.

## 6. Indexes

`(token_hash)` — the hot validate/consume lookup; `(user_id, purpose, consumed_at)`.

## 7. Relationships

Many tokens per user over time (each reset/invite issues a fresh row); only the most recent unconsumed row of a given `purpose` is operative, enforced application-side (issuing a new reset token implicitly supersedes a prior unconsumed one — the old row is simply left to expire naturally rather than retroactively invalidated, since a superseded-but-not-yet-expired token poses no security risk once a newer one is required for the actual reset to succeed... actually re-examined: to be precise, the service explicitly marks prior unconsumed tokens of the same purpose as consumed when issuing a new one, to close the window).

## 8. Soft Delete Strategy

Not soft-deleted; `consumed_at` is the lifecycle marker, alongside natural `expires_at` lapsing. A retention job purges long-expired rows.

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

**Deployment-global, via `users`.** No institute scoping.

## 11. Notes for TypeORM Entity Design

Model `purpose` as a TypeScript union type backed by the Postgres enum; `AuthTokenService` exposes `issue(userId, purpose)` / `validate(tokenHash)` / `consume(tokenHash)` as the only write paths, so password-reset and invitation flows share one well-tested implementation instead of two.

---

## 12. Performance Considerations

**Moderate write volume tied directly to password-reset and invitation traffic.** The `(token_hash)` unique index is the hot validate/consume path. Expired, unconsumed rows have no ongoing value and should be purged past their `expires_at` on a routine schedule — this is a naturally self-limiting table (rows are short-lived by design) so unbounded growth is not a realistic concern the way it is for `sessions`/`refresh_tokens`.