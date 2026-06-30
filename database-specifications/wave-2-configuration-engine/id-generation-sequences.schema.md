# ID Generation Sequences

## 1. Table Purpose

The live, atomically-incremented counter state behind every generated Student/Employee ID — backs `id-generation.api.md` #4, and is the one table in this wave whose entire reason for existing is **safe concurrency**, not configuration shape.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `format_id` | `UUID` | No | — |
| `partition_key` | `VARCHAR(255)` | No | — |
| `last_sequence` | `INTEGER` | No | 0 |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`format_id → id_generation_formats(id)` RESTRICT.

## 5. Unique Constraints

`(format_id, partition_key)` — exactly one running counter per format+partition (e.g., one counter for `'2026'` under a per-session-reset Student ID format, a fresh counter automatically starting at `'2027'` simply by virtue of being a new, not-yet-existing `partition_key` row).

## 6. Indexes

None beyond the unique constraint.

## 7. Relationships

Many-to-one with `id_generation_formats`.

## 8. Soft Delete Strategy

Not applicable — these rows are pure counter state with no independent business meaning to retain or display; they are never soft-deleted, only incremented, for the lifetime of the format.

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

**Derived, via `id_generation_formats → institute_id`.** No direct institute column; isolation is inherited from the parent format row.

## 11. Notes for TypeORM Entity Design

Implement generation as a single raw `queryRunner.query(...)` call executing the UPSERT above inside the transaction that also creates the Student/Employee row (Wave 5/HR), never as two separate ORM calls (`findOne` then `save`) — that pattern reintroduces exactly the race condition the UPSERT exists to prevent. The returned `last_sequence`, combined with the format's `tokens`, is formatted into the final identifier string entirely in application code (e.g. zero-padding per the `SEQUENCE` token's configured width) — this table stores only the integer, never the formatted string, so a later token-format change never requires rewriting historical sequence rows.

---

## 12. Performance Considerations

**Low row count (one row per active partition, e.g. per session-year), but this table's single row is the most write-contended row in its vicinity during peak admission/onboarding periods** — every Student/Employee/Payment-Receipt ID generation for that partition serializes through the same `UPDATE ... RETURNING` row. The atomic `UPSERT` pattern keeps this correct under concurrency, but it is worth monitoring for lock contention specifically during high-volume bulk-admission-conversion windows (Wave 5), where many ID-generation calls may target the same partition near-simultaneously.