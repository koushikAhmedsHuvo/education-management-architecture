# Workflow Definitions

## 1. Table Purpose

Declarative, versioned finite-state-machine definitions (WFL-001/003) — backs `workflow-definition.api.md`.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `key` | `VARCHAR(128)` | No | — |
| `version` | `INTEGER` | No | — |
| `name` | `VARCHAR(255)` | No | — |
| `consuming_module` | `VARCHAR(64)` | No | — |
| `states` | `JSONB` | No | — |
| `transitions` | `JSONB` | No | — |
| `approver_rules` | `JSONB` | No | — |
| `sod_rules` | `JSONB` | No | — |
| `routing` | `JSONB` | No | — |
| `timeout_policy` | `JSONB` | No | — |
| `return_bound` | `SMALLINT` | No | — |
| `status` | `workflow_definition_status (ENUM: DRAFT, PENDING_APPROVAL, PUBLISHED, SUPERSEDED, ARCHIVED)` | No | 'DRAFT' |
| `governing_instance_id` | `UUID` | Yes | — |
| `published_by` | `UUID` | Yes | — |
| `published_at` | `TIMESTAMPTZ` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`governing_instance_id → workflow_instances(id)` RESTRICT, nullable, **added via a separate `ALTER TABLE` after both tables exist** (see TypeORM Notes — `workflow_definitions` and `workflow_instances` reference each other, so the migration cannot declare both FKs inline in `CREATE TABLE`); `published_by → users(id)` RESTRICT.

## 5. Unique Constraints

`(key, version)`; **`(key) WHERE status = 'PUBLISHED' AND deleted_at IS NULL`** — at most one published version per key at a time, the row every new instance pins to (this single partial index is the entire DB-level mechanism behind WFL-002's version-pinning, per §0.1).

## 6. Indexes

`(key, status)`; `(consuming_module)`.

## 7. Relationships

One definition-version has many `workflow_instances` (all pinned to it forever); optionally one `workflow_instances` row governing its own publication.

## 8. Soft Delete Strategy

None operative — like `config_definitions`/`permissions` before it, a published version must remain resolvable for as long as any instance is pinned to it, so hard deletion is blocked by trigger and `status = 'ARCHIVED'` is the sole retirement path.

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

**Deployment-global by design.** A workflow definition (the FSM shape — states, transitions, approver-resolution rules) is shared product logic, not institute data; `consuming_module` identifies which domain declared it, not which institute owns it. Institute-specific *behavior* comes from how approver-resolution rules reference scoped roles/memberships at runtime, not from this table being partitioned per institute.

## 11. Notes for TypeORM Entity Design

**Migration sequencing:** create `workflow_definitions` and `workflow_instances` in the same migration, both initially *without* their mutual FK (`workflow_instances.workflow_definition_id` can be declared inline since `workflow_definitions` exists first if ordered correctly — only `workflow_definitions.governing_instance_id` needs the deferred `ALTER TABLE ADD CONSTRAINT` once `workflow_instances` also exists). **Bootstrap seed:** this table's very first row, inserted by a seed migration, is the platform's own `"workflow-definition-change-approval"` definition (§0.4) — every subsequent `governing_instance_id` across every other definition's high-impact changes is an instance of this one bootstrap row.

---

## 12. Performance Considerations

**Very low volume (tens of distinct workflow keys, a handful of versions each), read once per instance-initiation and cached thereafter for the instance's lifetime** (since the instance pins to a specific version row permanently). No performance concerns; this table's significance is structural, not volumetric.