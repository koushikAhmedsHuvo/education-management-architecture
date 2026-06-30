# Classes

## 1. Table Purpose

The timeless class/level definition (CLS-001/002/004/005/006/007) — backs `class.api.md` #1–10, #12–15.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `label` | `VARCHAR(128)` | No | — |
| `code` | `VARCHAR(32)` | Yes | — |
| `ordering` | `SMALLINT` | No | — |
| `campus_applicability` | `UUID[]` | No | — |
| `streams` | `TEXT[]` | No | '{}' |
| `aggregate_capacity` | `INTEGER` | Yes | — |
| `next_class_id` | `UUID` | Yes | — |
| `is_terminal` | `BOOLEAN` | No | false |
| `status` | `class_status (ENUM: DRAFT, ACTIVE_DEFINITION, DEPRECATED_DEFINITION)` | No | 'DRAFT' |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT; `next_class_id → classes(id)` RESTRICT, self-referencing, nullable; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

`(institute_id, ordering) WHERE deleted_at IS NULL` — CLS-002's "ordering already used" guard; `(institute_id, code) WHERE code IS NOT NULL AND deleted_at IS NULL`.

## 6. Indexes

`(institute_id, status)`; `(next_class_id)` for promotion-graph traversal.

## 7. Relationships

One class has many `class_session_instances` (one per session it's ever been active in); referenced by `curriculum_mappings` and, via `class_session_instances`, by `sections`.

## 8. Soft Delete Strategy

Standard `deleted_at`, though `status = 'DEPRECATED_DEFINITION'` (CLS-001's "never hard-deleted, historical instances preserved") is the disclosed retirement path actually exercised by `class.api.md` #10's Retire endpoint — `deleted_at` itself is reserved for true erroneous-row correction.

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

**Direct, mandatory.** `institute_id` is NOT NULL; the class structure definition is permanently and exclusively owned by one institute, with `(institute_id, ordering)` and `(institute_id, code)` both partial-unique to that scope.

## 11. Notes for TypeORM Entity Design

`campus_applicability` and `streams` as native Postgres arrays (`uuid[]`, `text[]`) rather than join tables — both are small, rarely-queried-independently sets scoped tightly to one class, exactly the case where Wave 1's `roles.scope_levels` precedent (array over join table) applies again. `aggregate_capacity` must never be read by any capacity-enforcement code path — only `sections.capacity` is authoritative; consider a code-review lint or a clearly-named getter (`getAdvisoryCapacity()`, never `getCapacityLimit()`) to keep this distinction visible in application code, not just in this document.

---

## 12. Performance Considerations

**Very low volume (a few dozen class levels per institute at most), read constantly as a structural reference but written rarely** (defining/editing a class is an infrequent administrative act). No special performance concerns; the promotion-graph traversal (`next_class_id` chasing) is bounded by the small row count and never needs a recursive-CTE optimization at this table's realistic scale.