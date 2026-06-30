# Role Permissions

## 1. Table Purpose

The many-to-many join between `roles` and `permissions` — a proper normalized table (not a JSONB array on `roles`) because "who has permission X" (`role-permission.api.md` #18) and "which permissions does role Y grant" both need to be efficient, indexed, set-oriented queries.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `role_id` | `UUID` | No | — |
| `permission_id` | `UUID` | No | — |

## 3. Primary Key

`id` (a surrogate key is used rather than a composite `(role_id, permission_id)` PK, purely for TypeORM ergonomics — every other table in this schema has a single-column UUID PK, and a composite-PK exception here would complicate the ORM mapping without a corresponding benefit, since the composite is still enforced as a unique constraint below).

## 4. Foreign Keys

`role_id → roles(id)` ON DELETE CASCADE (a join row is meaningless once its role is gone — the role itself is soft-deleted/archived in practice, so this cascade rarely fires, but it is the structurally correct behavior for a pure junction table per §0.1's "cascade only where a row is a true dependent"); `permission_id → permissions(id)` ON DELETE RESTRICT (permissions are never deleted, per `permissions`' own notes, so this path is theoretical but stated for completeness).

## 5. Unique Constraints

`(role_id, permission_id)` — a role cannot grant the same permission twice.

## 6. Indexes

`(permission_id, role_id)` — specifically ordered to make the "who has permission X" reverse lookup (`role-permission.api.md` #18) an index-only scan; `(role_id)` is covered by the unique constraint below and needs no separate index.

## 7. Relationships

The associative table between `roles` and `permissions`.

## 8. Soft Delete Strategy

Not soft-deleted; a permission is added to or removed from a role by inserting/deleting this join row outright (the *role's* edit-creates-a-version semantics, captured in `audit_log` as described in `roles`' notes, covers the historical record — the join table itself stays a clean current-state table).

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

**Derived, via `roles`.** This junction table has no institute column of its own; its isolation is entirely inherited from its parent `roles` row.

## 11. Notes for TypeORM Entity Design

Map as a plain entity with a `@ManyToOne` on each side (not TypeORM's implicit `@ManyToMany` join-table sugar), since the explicit table is needed for the composite unique constraint, the `permission_id`-first index, and to keep open the option of adding columns later (e.g. a future `granted_at`/`granted_by` if per-permission provenance within a role is ever needed) without a breaking schema change.

---

## 12. Performance Considerations

**Low-to-moderate volume, read on the authorization hot path.** The `(permission_id, role_id)` index ordering is deliberate — it makes 'who has permission X' (a relatively rare, admin-facing query) an index-only scan, while 'what does role Y grant' (the much more frequent path, resolved once per session and cached) uses the implicit `role_id` coverage from the composite unique constraint. Given the low row count (roles × their permission grants, typically under a few thousand rows even at large scale), this table is not a realistic performance bottleneck on its own — the bottleneck, if any, is in `memberships`.