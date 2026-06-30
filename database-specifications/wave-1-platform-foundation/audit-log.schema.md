# Audit Log

## 1. Table Purpose

The single immutable, append-only audit trail backing every domain's mandated audit events across the entire system (Doc 01 §11 `LOGIN_SUCCEEDED`/`LOGIN_FAILED`/`ACCOUNT_LOCKED`/etc.; Doc 02 §11 `MEMBERSHIP_GRANTED`/`PRIVILEGE_ESCALATION_BLOCKED`/etc.; Doc 03–05 §11 institute/campus/session lifecycle events; and every later wave's equivalent). Backs `auth.api.md` #20–21 (Login & Security Events) directly, and is the generic substrate every other module's "audit trail" / "decision lineage" view reads from.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `entity_type` | `VARCHAR(64)` | No | — |
| `entity_id` | `UUID` | No | — |
| `action` | `VARCHAR(64)` | No | — |
| `actor_user_id` | `UUID` | Yes | — |
| `institute_id` | `UUID` | Yes | — |
| `campus_id` | `UUID` | Yes | — |
| `before_state` | `JSONB` | Yes | — |
| `after_state` | `JSONB` | Yes | — |
| `ip_address` | `INET` | Yes | — |
| `correlation_id` | `UUID` | Yes | — |
| `created_at` | `TIMESTAMPTZ` | No | now() |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`actor_user_id → users(id)` (no action on delete — see Notes); `institute_id → institutes(id)`; `campus_id → campuses(id)`. None are enforced with `ON DELETE CASCADE`; audit rows must survive their subject's eventual archival.

## 5. Unique Constraints

None — this is a pure event log; duplicates are valid (e.g., two failed logins one second apart).

## 6. Indexes

`(entity_type, entity_id, created_at DESC)` — the dominant "show me this record's history" query; `(actor_user_id, created_at DESC)`; `(institute_id, created_at DESC)` — partial `WHERE institute_id IS NOT NULL`; `(action, created_at DESC)` for the security-events screen's `eventType` filter; GIN index on `after_state` only if ad hoc JSONB querying of payloads becomes a real need (omitted by default to control write-path index overhead on a high-volume insert-only table).

## 7. Relationships

Polymorphic by convention (`entity_type` + `entity_id`), not a typed FK to each possible table — this is the one deliberate denormalization in the schema, justified because a generic audit table referencing dozens of possible entity tables with typed FKs is unworkable, and the alternative (a table per entity type) is exactly the duplication §0.6 avoids.

## 8. Soft Delete Strategy

**None — rows in `audit_log` are never soft-deleted, never updated, and never hard-deleted** except by an explicit, separately-governed data-retention purge job (outside this schema's scope), since the table's entire purpose is immutability.

## 9. Audit Fields

| Column | Type | Nullable | Default Value |
|---|---|---|---|
| `created_at` | `TIMESTAMPTZ` | No | `now()` |
| `created_by` | `UUID` | Yes | — |
| `updated_at` | `TIMESTAMPTZ` | No | `now()` |
| `updated_by` | `UUID` | Yes | — |
| `deleted_at` | `TIMESTAMPTZ` | Yes | — |
| `deleted_by` | `UUID` | Yes | — |

*Exception:* this table intentionally omits `updated_at`, `updated_by`, `deleted_at`, and `deleted_by` from its operative columns — see Soft Delete Strategy. The columns are present in the base entity for schema consistency but are never populated, since an audit-log row is never updated or deleted by any application code path (enforced by revoking `UPDATE`/`DELETE` grants at the database-role level).

## 10. Multi-Institute Isolation Strategy

**Polymorphic/contextual.** `institute_id`/`campus_id` are nullable scope columns populated from the source event, not a structural tenant boundary on the table itself — a single deployment-wide `audit_log` intentionally spans every institute, since cross-institute audit queries (e.g., a platform-level security review) are a legitimate, expected access pattern. Every *read* path used by an institute-scoped screen (`auth.api.md`'s Login & Security Events) filters `WHERE institute_id = :callerInstitute` at the application layer; the table itself does not enforce isolation structurally.

## 11. Notes for TypeORM Entity Design

Model as a write-only entity (no `@UpdateDateColumn`/`@DeleteDateColumn`); writes go through a single `AuditLogService.record(...)` call from a domain-event listener (per the Architecture's outbox-pattern note), never directly from feature code, so every module's audit write is structurally identical. Use `@Index()` decorators matching the composite indexes above; do **not** map `before_state`/`after_state` as typed classes — keep them `Record<string, unknown>` (`jsonb` column type) since their shape is genuinely heterogeneous across event types.

---

## 12. Performance Considerations

**Write-heavy, append-only, read-occasional.** Expect this to be the single highest-insert-volume table in the schema (every login, every state change, every governed decision across all seven waves writes here). The `(entity_type, entity_id, created_at DESC)` index is the dominant read path and must stay lean — avoid adding speculative secondary indexes that would slow the insert-only write path for marginal read benefit. **Strong partitioning candidate**: partition by `created_at` (monthly or quarterly range partitions) once volume grows past tens of millions of rows, since old partitions can be detached to cold storage without touching the hot insert path at all.