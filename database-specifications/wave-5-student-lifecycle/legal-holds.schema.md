# Legal Holds

## 1. Table Purpose

Generic, polymorphic legal-hold markers blocking erasure (STU-007), per §0.3 — reusable by any future wave needing the same protection (e.g., staff records).

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `entity_type` | `VARCHAR(64)` | No | — |
| `entity_id` | `UUID` | No | — |
| `reason` | `TEXT` | No | — |
| `applied_by` | `UUID` | No | — |
| `applied_at` | `TIMESTAMPTZ` | No | now() |
| `lifted_at` | `TIMESTAMPTZ` | Yes | — |
| `lifted_by` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`applied_by → users(id)` RESTRICT; `lifted_by → users(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK — though application logic typically prevents placing a second concurrent hold on an already-held entity, this isn't enforced as a DB constraint since a legitimate scenario (two separate legal matters both requiring a hold on the same record) isn't actually forbidden by any business rule.

## 6. Indexes

`(entity_type, entity_id) WHERE lifted_at IS NULL` — the exact predicate `students`' deletion-blocking trigger (§1) queries on every anonymize/archive attempt.

## 7. Relationships

Polymorphic, read by `students`' (and later, other entities') hard-delete-blocking trigger.

## 8. Soft Delete Strategy

Not soft-deleted; `lifted_at` is the complete lifecycle marker — a hold is lifted, never deleted, preserving the full record of when legal protection applied.

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

**Polymorphic/contextual**, same shape as `documents` — no institute column; resolved via the referenced entity.

## 11. Notes for TypeORM Entity Design

Placing or lifting a hold should itself be a narrowly-permissioned, audited action (`audit_log` event `LEGAL_HOLD_APPLIED`/`LIFTED`) — this table is intentionally small and rarely written, exactly matching how infrequently legal holds should occur in practice.

---

## 12. Performance Considerations

**Extremely low volume (legal holds should be rare events by their very nature) and low write frequency.** No performance concerns whatsoever; this table exists for correctness and compliance, not scale.