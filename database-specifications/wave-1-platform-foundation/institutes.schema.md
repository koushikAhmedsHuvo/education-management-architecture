# Institutes

## 1. Table Purpose

The primary multi-tenant scope boundary (Doc 03 §6 state machine: `DRAFT → ACTIVE → SUSPENDED/ARCHIVED`) — backs `institute.api.md`.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `name` | `VARCHAR(255)` | No | — |
| `code` | `VARCHAR(32)` | No | — |
| `institution_type` | `VARCHAR(32)` | No | — |
| `status` | `institute_status (ENUM: DRAFT, ACTIVE, SUSPENDED, ARCHIVED)` | No | 'DRAFT' |
| `address` | `TEXT` | Yes | — |
| `contact` | `VARCHAR(255)` | Yes | — |
| `locale` | `VARCHAR(16)` | No | 'en' |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

None outbound (the top of the scope hierarchy within a deployment).

## 5. Unique Constraints

`(code) WHERE deleted_at IS NULL` — partial unique per §0.4 (a hard-deleted empty-draft institute's code, the one true-delete case in this schema, frees up naturally since the row is gone entirely; an archived institute's code does **not** free up under this constraint as written, since `ARCHIVED` institutes are soft-deleted-equivalent in status but **not** `deleted_at`-set — this is intentional: INST-008 treats archive as permanent closure, not "available for reuse," so the partial index correctly continues to protect an archived institute's code from collision even though `deleted_at IS NULL` still holds for it).

## 6. Indexes

`(status)`; `(institution_type)`.

## 7. Relationships

Referenced by `campuses`, `academic_sessions`, `memberships`, `roles`, `auth_policies`, `sod_constraints`, and effectively every institute-scoped table in every later wave.

## 8. Soft Delete Strategy

Standard `deleted_at`, exercised only by the single narrow hard-delete path (`institute.api.md` #13, an actual `DELETE`, not a `deleted_at` set) for an empty `DRAFT` institute — there is otherwise no soft-delete path for institutes at all; `SUSPENDED`/`ARCHIVED` status values are the real lifecycle, per INST-007/008's explicit "no data altered or deleted on suspend; archive preserves all data immutably."

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

**The tenant root — not itself isolated by anything, since it *is* the isolation boundary every other institute-scoped table's `institute_id` ultimately references.** One deployment may host a handful to a few dozen institutes; this table has no parent scope to inherit from.

## 11. Notes for TypeORM Entity Design

`institution_type` seeding its Configuration Engine template (INST-002) is a Wave 2 concern (the template itself lives in `02-configuration-engine.md`'s `config_values`, scoped by `institute_id` once the institute is created) — this table only stores the chosen type, not the template content. The setup-checklist (`institute.api.md` #4) is **not** a stored table — it's a computed/derived response assembled by querying `campuses`, `academic_sessions`, and `memberships` for existence under this `institute_id`, exactly as flagged in the cross-wave design note: do not create an `institute_setup_checklist` table.

---

## 12. Performance Considerations

**Extremely low volume, read very frequently (every request needs to resolve the active institute's basic metadata/status).** Cache aggressively at the application layer; this table will never be a write bottleneck (institute creation is a rare, deliberate administrative act) and should never need a query-plan optimization beyond its existing indexes at any realistic deployment scale.