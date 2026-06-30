# Grading (Grade Scales)

## 1. Table Purpose

Versioned, effective-dated grade scales (GRD-001…008) — backs `grading.api.md` #1–9. Append-only, exclusion-constraint enforced, per §0.1.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `name` | `VARCHAR(255)` | No | — |
| `scope_level` | `grade_scope_level (ENUM: INSTITUTE, CLASS, EXAM)` | No | — |
| `scope_ref_id` | `UUID` | Yes | — |
| `scheme` | `grading_scheme (ENUM: ABSOLUTE, RELATIVE)` | No | 'ABSOLUTE' |
| `bands` | `JSONB` | No | — |
| `gpa_rule` | `JSONB` | No | — |
| `special_grades` | `JSONB` | No | '[]' |
| `relative_config` | `JSONB` | Yes | — |
| `version` | `INTEGER` | No | — |
| `effective_from` | `TIMESTAMPTZ` | No | — |
| `effective_until` | `TIMESTAMPTZ` | Yes | — |
| `status` | `config_value_status (reuses the Wave-2 ENUM: SCHEDULED, PUBLISHED, SUPERSEDED, ARCHIVED)` | No | 'SCHEDULED' |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT; `workflow_instance_id → workflow_instances(id)` RESTRICT. **No FK on `scope_ref_id`** — same polymorphic trade-off as `sod_constraints`/`dynamic_forms`' JSONB references elsewhere, since it points to either `classes` or `exams` depending on `scope_level`.

## 5. Unique Constraints

Exactly Wave 2's `config_values` pattern, transplanted to this table's scope columns:
```sql
ALTER TABLE grade_scales ADD CONSTRAINT grade_scales_no_overlap
EXCLUDE USING gist (
  institute_id WITH =,
  scope_level WITH =,
  COALESCE(scope_ref_id, '00000000-0000-0000-0000-000000000000'::uuid) WITH =,
  tstzrange(effective_from, effective_until) WITH &&
) WHERE (status IN ('SCHEDULED', 'PUBLISHED') AND deleted_at IS NULL);
```
This is the direct DB-level enforcement of GRD-007's effective-dated resolution — two versions of the same scope can never have overlapping active periods, exactly mirroring §0.1's stated rationale.

## 6. Indexes

`(institute_id, scope_level, scope_ref_id, status)`.

## 7. Relationships

Read by `results` computation (which stamps the resolved row's `version` as `grade_ver`); the most-specific-wins resolution among `EXAM` > `CLASS` > `INSTITUTE` scope is an application-layer precedence query, the identical shape to Wave 2's `config_values` resolution, just with a fixed three-level scope instead of an arbitrary institute/campus/section chain.

## 8. Soft Delete Strategy

`deleted_at` present but essentially unused; `status = 'ARCHIVED'` is the disclosed retirement path, and — critically — **a scale already cited by any `results.grade_ver` is never hard-deleted**, blocked by trigger, for the same historical-reproducibility reason as `workflow_definitions`/`config_definitions`.

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

**Direct, mandatory, with nested scope.** `institute_id` is NOT NULL; `scope_level`/`scope_ref_id` further narrow to institute/class/exam, the same three-tier most-specific-wins shape as `config_values`' own scope chain, just fixed to exactly three levels instead of an arbitrary hierarchy.

## 11. Notes for TypeORM Entity Design

Map the `EXCLUDE` constraint via raw migration SQL, identical caveat to Wave 2's `config_values`. The "minimum cohort size" relative-grading fallback (GRD-005 — a cohort below the configured minimum automatically falls back to absolute grading, itself audited) is computed at result-computation time by the `ResultComputationService`, which queries the actual cohort size against `relative_config.minimumCohortSize` and records the fallback as an `audit_log` event when it occurs — no schema support beyond the JSONB config and the generic audit table is needed.

---

## 12. Performance Considerations

**Very low volume (a handful of grade scales per institute, versioned occasionally), but read on every single result computation** — every `results` row resolves and stamps a `grade_ver` from this table. Given the low row count and infrequent writes, the resolution query is cheap even uncached; the `EXCLUDE` constraint's GiST index adds the same modest, acceptable write-path overhead as `config_values`' and `fee_structures`' own.