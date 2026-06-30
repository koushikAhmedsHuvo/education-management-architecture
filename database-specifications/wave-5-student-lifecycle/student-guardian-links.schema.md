# Student Guardian Links

## 1. Table Purpose

The (student, guardian) relationship with role-scoped capabilities (GRD-N-001/004/005) — backs `student.api.md` #11–14 and `guardian.api.md` #5–6.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `student_id` | `UUID` | No | — |
| `guardian_id` | `UUID` | No | — |
| `relationship` | `VARCHAR(64)` | No | — |
| `is_primary` | `BOOLEAN` | No | false |
| `is_financial_responsible` | `BOOLEAN` | No | false |
| `emergency_priority` | `SMALLINT` | Yes | — |
| `portal_access` | `BOOLEAN` | No | false |
| `status` | `link_status (ENUM: ACTIVE, REVOKED)` | No | 'ACTIVE' |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`student_id → students(id)` RESTRICT; `guardian_id → guardians(id)` RESTRICT.

## 5. Unique Constraints

`(student_id, guardian_id) WHERE status = 'ACTIVE' AND deleted_at IS NULL` — no duplicate active link; **`(student_id) WHERE is_primary = true AND status = 'ACTIVE' AND deleted_at IS NULL`** — exactly one active primary guardian-of-record per student, the precise DB-level form of GRD-N-001.

## 6. Indexes

`(student_id, status)`; `(guardian_id, status)` — both directions are queried equally often (a student's guardian list; a guardian's children list, the portal's own `7. Get My Children`).

## 7. Relationships

The associative table between `students` and `guardians`; `guardian_restrictions` and the consent/portal logic all key off this link, not off the (student, guardian) pair directly, since restrictions and roles are inherently per-relationship.

## 8. Soft Delete Strategy

Standard `deleted_at`, though `status = 'REVOKED'` (set via the governed `guardianship_change_requests` flow, §7) is the disclosed lifecycle actually exercised.

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

**Derived, via `students → institute_id`** (and transitively, `guardians → institute_id`, which the application verifies agree — a guardian profile genuinely should not link to a student at a different institute).

## 11. Notes for TypeORM Entity Design

Direct deletion/revocation of this row is **only ever reached through** `student.api.md` #14 (immediate, simple unlink) or the governed `guardianship_change_requests` flow (§7, for reasoned/dated changes) — never a bare `DELETE`/`UPDATE` from elsewhere in the codebase, since both paths carry distinct validation (the immediate-unlink path still checks the last-guardian-of-minor rule synchronously; the governed path checks it twice, at submission and again at approval, per §7's notes).

---

## 12. Performance Considerations

**High volume (roughly one row per student per active guardian, so comparable in scale to `students` itself, multiplied by ~1.5–2×).** Both directional indexes are genuinely load-bearing — `(student_id, status)` for every student-detail view's guardian panel, `(guardian_id, status)` for every portal login's 'my children' resolution — making this one of the highest-read-frequency junction tables in the entire schema.