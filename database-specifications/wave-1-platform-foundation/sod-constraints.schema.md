# SoD Constraints

## 1. Table Purpose

Configured separation-of-duties constraints between two "duties" (permissions or roles) — AUTHZ-009 — backs `role-permission.api.md` #14–16.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | Yes | — |
| `duty_a` | `VARCHAR(128)` | No | — |
| `duty_a_type` | `sod_duty_type (ENUM: PERMISSION, ROLE)` | No | — |
| `duty_b` | `VARCHAR(128)` | No | — |
| `duty_b_type` | `sod_duty_type` | No | — |
| `constraint_type` | `sod_constraint_type (ENUM: MUTUALLY_EXCLUSIVE, REQUESTER_NOT_APPROVER)` | No | — |
| `status` | `sod_status (ENUM: ACTIVE, DISABLED)` | No | 'ACTIVE' |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT. **Deliberately no FK on `duty_a`/`duty_b`** to either `permissions.id` or `roles.id` — a single column referencing either of two different tables depending on a discriminator is exactly the polymorphic-association problem `audit_log` already accepted for a different reason; here it's avoidable by simply storing the permission **key** (a stable, human-readable string already unique per `permissions`) rather than a typed FK, and validating the reference at the application layer on write.

## 5. Unique Constraints

None beyond the PK — `role-permission.api.md` #15 already allows the application to reject a *contradicting* new constraint, but two constraints over the same duty pair with different `constraint_type` values are not strictly disallowed by the rules as written, so no DB-level uniqueness is imposed beyond what the service layer's contradiction check already covers.

## 6. Indexes

`(institute_id, status)`; `(duty_a, duty_a_type)` and `(duty_b, duty_b_type)` — both directions are checked when validating a candidate membership grant.

## 7. Relationships

Read by the `memberships`-grant write path (#13's application-layer SoD check) and by `examination`/`finance` waves' own approval flows later, which reuse this same table rather than each defining their own SoD config (a cross-wave reuse decision worth flagging now: **this table is intentionally generic enough that Waves 3–7's "requester ≠ approver" checks all read from it**, rather than each wave inventing a parallel SoD table).

## 8. Soft Delete Strategy

Standard; `status = 'DISABLED'` is the typical retirement path (a disabled constraint is retained for audit/history, distinct from a corrected/erroneous row's true deletion).

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

**Direct, nullable.** `institute_id IS NULL` rows are deployment-wide constraints (the common case — most SoD rules, like 'requester ≠ approver,' apply universally); institute-specific overrides are the exception.

## 11. Notes for TypeORM Entity Design

Because `duty_a`/`duty_b` aren't typed FKs, add a repository-level integrity check (not a migration-time one) that runs periodically or on write to confirm every referenced permission key/role id still exists — this is the one place in the schema where referential integrity is intentionally traded for avoiding a worse polymorphic-FK problem, so compensating application-level validation is part of the design, not an oversight.

---

## 12. Performance Considerations

**Very low volume (a handful to a few dozen rules), read on every governed-action write path across all seven waves.** Given how frequently this table is consulted (every membership grant, every workflow approval routing decision across the entire system references it conceptually), it is small enough to fully cache in application memory and refresh only on the rare write — querying Postgres fresh for every SoD check would be unnecessary overhead at this table's realistic scale.