# Permissions

## 1. Table Purpose

The Permission Registry (AUTHZ-008) — the closed catalog of namespaced, code-enforced permission keys every role's grants must come from.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `key` | `VARCHAR(128)` | No | — |
| `category` | `VARCHAR(64)` | No | — |
| `description` | `TEXT` | No | — |
| `status` | `permission_status (ENUM: REGISTERED, DEPRECATED)` | No | 'REGISTERED' |

## 3. Primary Key

`id`.

## 4. Foreign Keys

None outbound.

## 5. Unique Constraints

`(key)` — **globally unique, not soft-delete-partial**, and in fact this table has **no delete path at all** (`role-permission.api.md` #13 is deprecate-only, never delete) — a permission key, once registered, must never be reissued with a different meaning, even after deprecation, since historical role/audit records reference it by key.

## 6. Indexes

`(category)`; `(status)`.

## 7. Relationships

Referenced by `role_permissions` (many-to-many with `roles`).

## 8. Soft Delete Strategy

**None — this table intentionally omits `deleted_at` from its effective lifecycle** (the column exists for schema consistency per §0.3 but is never set); `status = 'DEPRECATED'` is the only retirement path, by explicit business rule ("deprecate-not-delete").

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

**Deployment-global by design.** The Permission Registry is a single, deployment-wide closed catalog — a permission key like `institute.activate` means the same thing for every institute in the deployment. No institute scoping exists or should exist here.

## 11. Notes for TypeORM Entity Design

Seed this table from a versioned, code-reviewed seed migration (the permission catalog is effectively part of the application's deployed code, not user-editable data) — `role-permission.api.md` #12's "Register Permission" endpoint exists for legitimate platform-level registration, but in practice most entries originate from seed migrations alongside each module's own rollout.

---

## 12. Performance Considerations

**Extremely low volume (tens to low hundreds of rows), read very frequently, written almost never.** Every permission-resolution check ultimately validates against this table's keys; given its small, stable size, it is an ideal candidate for full in-memory caching at the application layer (loaded once at startup, invalidated only on the rare `Register`/`Deprecate` write) rather than a fresh query per check.