# MFA Recovery Codes

## 1. Table Purpose

Single-use recovery codes issued on MFA enrollment confirmation (AUTH-007 edge case — "never lock out a user permanently").

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `user_id` | `UUID` | No | — |
| `code_hash` | `VARCHAR(255)` | No | — |
| `used_at` | `TIMESTAMPTZ` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`user_id → users(id)` ON DELETE RESTRICT.

## 5. Unique Constraints

`(code_hash)` globally unique.

## 6. Indexes

`(user_id, used_at)` — "how many unused codes remain" is checked on every regenerate (`auth.api.md` #16).

## 7. Relationships

Many codes per user (a batch issued together, typically 8–10).

## 8. Soft Delete Strategy

Not soft-deleted; `used_at` is the lifecycle marker. Regenerating (`16. Regenerate Recovery Codes`) invalidates the old batch by **deleting** those rows outright (recovery codes are not meaningfully "soft-deletable" — an old, superseded batch has zero retained value, unlike most data in this schema) and inserting a fresh batch.

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

Bulk-insert the new batch and bulk-delete the old one inside a single transaction in `RegenerateRecoveryCodesHandler`, never a per-row loop.

---

## 12. Performance Considerations

**Low volume, bursty writes.** A batch of 8–10 rows inserted together on enrollment/regeneration, then read individually (and updated once, atomically) on use. The `(user_id, used_at)` index supports the 'how many unused codes remain' check cheaply even with this bursty pattern.