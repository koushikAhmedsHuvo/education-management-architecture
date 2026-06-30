# Guardians

## 1. Table Purpose

The guardian profile (GRD-N-001…008) — optionally linked to a `users` row for portal access — backs `guardian.api.md` #1–4.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `user_id` | `UUID` | Yes | — |
| `institute_id` | `UUID` | No | — |
| `name` | `VARCHAR(255)` | No | — |
| `contact` | `VARCHAR(32)` | Yes | — |
| `national_id_encrypted` | `BYTEA` | Yes | — |
| `national_id_hash` | `VARCHAR(64)` | Yes | — |
| `status` | `guardian_status (ENUM: ACTIVE, ARCHIVED)` | No | 'ACTIVE' |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`user_id → users(id)` RESTRICT, nullable; `institute_id → institutes(id)` RESTRICT.

## 5. Unique Constraints

`(user_id) WHERE user_id IS NOT NULL AND deleted_at IS NULL` — a `users` row backs at most one guardian profile per institute.

## 6. Indexes

`(institute_id, status)`; `(national_id_hash) WHERE national_id_hash IS NOT NULL` — dedup matching (GRD-N-003), same pattern as `students`; `(user_id) WHERE user_id IS NOT NULL`.

## 7. Relationships

Referenced by `student_guardian_links` (many students per guardian — "one guardian, many students," GRD-N-003), `guardian_restrictions`, `guardian_consents`.

## 8. Soft Delete Strategy

Standard, though `status = 'ARCHIVED'` is the typical end state rather than `deleted_at`.

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

**Direct, mandatory.** `institute_id` is NOT NULL — deliberately *not* deployment-global like `users`, since a guardian's profile here is fundamentally 'whose parent/guardian are you at this institute,' per the table's own design note; the same real person with children at two institutes would have two separate guardian profile rows.

## 11. Notes for TypeORM Entity Design

**Portal activation is a distinct, later event from guardian-profile creation** — a guardian profile is very often created during admission (as pure contact data, `user_id IS NULL`) long before anyone grants them `portalAccess` on a specific student link (§5); only at that point does the activation flow create the backing `users` row and set `guardians.user_id`. This two-stage design is exactly why `guardians` is its own table rather than guardians simply *being* `users` rows from the start — most guardians, in practice, never need portal access at all.

---

## 12. Performance Considerations

**Moderate-to-high volume (roughly proportional to student count, often 1.5–2× given siblings share guardians), read on every portal-access and student-detail view.** The `national_id_hash` trigram-adjacent dedup index carries the identical write-path trade-off as `students`' own, for the identical reason (sibling-reuse dedup matching, GRD-N-003).