# Subjects

## 1. Table Purpose

Subject identity, components, credit/weightage, and elective configuration (SUB-001/003/005/006/007) — backs `subject.api.md` #1–4, #7, #9–13, #15–16.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `name` | `VARCHAR(255)` | No | — |
| `code` | `VARCHAR(32)` | No | — |
| `type` | `subject_type (ENUM: CORE, ELECTIVE, PRACTICAL)` | No | — |
| `credit` | `NUMERIC(4,2)` | Yes | — |
| `weightage` | `NUMERIC(5,2)` | Yes | — |
| `components` | `JSONB` | No | '[]' |
| `elective_selection_rules` | `JSONB` | Yes | — |
| `elective_prerequisite_ids` | `UUID[]` | No | '{}' |
| `status` | `subject_status (ENUM: DRAFT, ACTIVE, DEPRECATED)` | No | 'DRAFT' |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT. **No FK on `elective_prerequisite_ids`** — same accepted array-of-references trade-off as elsewhere in this schema, validated at write time.

## 5. Unique Constraints

`(institute_id, code) WHERE deleted_at IS NULL` — SUB-001's immutable, unique code.

## 6. Indexes

`(institute_id, status)`; `(institute_id, type)`.

## 7. Relationships

Referenced by `curriculum_mappings`, `subject_teacher_assignments`, `elective_selections`.

## 8. Soft Delete Strategy

Standard `deleted_at`, though `status = 'DEPRECATED'` (SUB-007's "never hard-deleted, deprecated/unmapped forward only") is the actually-exercised retirement path — a subject with any assessment history (Wave 6) must remain resolvable forever, so hard deletion of a `DEPRECATED` subject is blocked the same way `permissions`/`config_definitions`/`workflow_definitions` are.

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

**Direct, mandatory.** `institute_id` is NOT NULL; `(institute_id, code)` partial-unique enforces SUB-001's immutable, scoped code uniqueness.

## 11. Notes for TypeORM Entity Design

`components` as JSONB is the correct call here (not a child table) because components are always read/written as a complete, small, ordered set together with their parent subject — there is no independent "list all components across subjects" query the API ever needs, unlike `role_permissions` or `curriculum_mappings`, which genuinely needed normalized join tables because *their* cross-cutting queries ("who has permission X," "which classes teach subject Y") are real and frequent.

---

## 12. Performance Considerations

**Low volume (a few dozen to low hundreds of subjects per institute), read on nearly every curriculum, exam-definition, and mark-entry path, written rarely.** The `components`/`electives` JSONB columns are always read as a whole alongside their parent row — no independent indexing of their internals is warranted given the access pattern.