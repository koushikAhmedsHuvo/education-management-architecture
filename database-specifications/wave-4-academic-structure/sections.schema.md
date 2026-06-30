# Sections

## 1. Table Purpose

The enrollment leaf (SEC-001) — the concrete, session-bound, capacity-enforcing unit students actually sit in. Backs `section.api.md` #1–5, #10–12.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `class_session_instance_id` | `UUID` | No | — |
| `campus_id` | `UUID` | No | — |
| `name` | `VARCHAR(64)` | No | — |
| `shift` | `VARCHAR(32)` | No | — |
| `capacity` | `INTEGER` | No | — |
| `class_teacher_id` | `UUID` | Yes | — |
| `status` | `section_status (ENUM: ACTIVE, INACTIVE, MERGED, CLOSED)` | No | 'ACTIVE' |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`class_session_instance_id → class_session_instances(id)` RESTRICT; `campus_id → campuses(id)` RESTRICT; `class_teacher_id → users(id)` RESTRICT (nullable — a section may briefly have no class-teacher).

## 5. Unique Constraints

`(class_session_instance_id, campus_id, shift, name) WHERE deleted_at IS NULL` — the precise DB-level form of SEC-002 ("unique within class+campus+shift" — shift participates, since `class_session_instance_id` already pins class+session, this constraint adds the remaining campus/shift/name dimensions).

## 6. Indexes

`(class_session_instance_id, status)`; `(campus_id, shift)`; `(class_teacher_id)` for "this teacher's sections" lookups.

## 7. Relationships

One `class_session_instance` has many sections; one section has many `subject_teacher_assignments`, many `section_class_teacher_assignments` (its history), and — in Wave 5 — many `enrollments`.

## 8. Soft Delete Strategy

Standard `deleted_at`, though `status ∈ {INACTIVE, MERGED, CLOSED}` is the disclosed lifecycle SEC-006's restructure flow actually produces; `deleted_at` reserved for erroneous-row correction.

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

**Derived, via `class_session_instance_id → classes → institute_id`**, with `campus_id` as an additional, independently-verified scope dimension (a section's campus must be a member of its parent class's `campus_applicability` array, application-checked).

## 11. Notes for TypeORM Entity Design

`class_teacher_id` is written **only** by the same transaction that inserts a new `section_class_teacher_assignments` row and closes the prior one (§4) — never updated independently, or the denormalized pointer and the date-ranged history can drift. The "merge/split/close" endpoints (`section.api.md` #7) never write to this table directly except via the governed `section_restructure_requests` flow (§5) reaching approval — a section's `status` change is always a *consequence* of an approved restructure request, never a standalone write.

---

## 12. Performance Considerations

**Moderate volume (the real per-session, per-class roster — potentially hundreds of sections across a large multi-campus institute), read extremely frequently** (every enrollment check, every attendance/mark-entry session resolves through a section). The `(class_session_instance_id, status)` index is the dominant scope-resolution path; `(campus_id, shift)` supports the management-screen filter view.