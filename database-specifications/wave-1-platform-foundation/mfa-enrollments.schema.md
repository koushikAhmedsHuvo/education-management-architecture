# MFA Enrollments

## 1. Table Purpose

TOTP enrollment state per user (AUTH-007) — backs `auth.api.md` #13–15.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `user_id` | `UUID` | No | — |
| `secret_encrypted` | `BYTEA` | No | — |
| `status` | `mfa_status (ENUM: PENDING, ACTIVE, DISABLED)` | No | 'PENDING' |
| `enrolled_at` | `TIMESTAMPTZ` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`user_id → users(id)` ON DELETE RESTRICT.

## 5. Unique Constraints

`(user_id) WHERE status != 'DISABLED' AND deleted_at IS NULL` — a user has at most one *active or pending* enrollment at a time, matching the API's `409 MFA_ALREADY_ENROLLED` guard; a `DISABLED` row is retained for history and a fresh `PENDING` row may then be created.

## 6. Indexes

None beyond the unique constraint below (table is small, point-lookup only).

## 7. Relationships

One-to-one(-ish, per the constraint above) with `users`.

## 8. Soft Delete Strategy

Standard; in practice `status = 'DISABLED'` is the operative "off" state (mirrors `sessions`' pattern of status-driven, not deletion-driven, lifecycle).

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

**Deployment-global, via `users`.** No institute scoping — MFA enrollment is a property of the identity, not any one institute's relationship to it.

## 11. Notes for TypeORM Entity Design

`secret_encrypted` maps to a `Buffer`/`bytea` column; encryption/decryption happens in a dedicated `MfaSecretCipher` service, never inline in the entity or repository.

---

## 12. Performance Considerations

**Low volume, low write frequency.** One row (or a small handful, across `DISABLED` history) per MFA-enrolled user. No special performance concerns; the partial unique index keeps point lookups trivially fast at any realistic scale.