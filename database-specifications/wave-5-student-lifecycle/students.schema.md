# Students

## 1. Table Purpose

The core student identity (STU-001…008) — backs `student.api.md` #1–10, #20–23.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `student_id` | `VARCHAR(32)` | No | — |
| `first_name` | `VARCHAR(128)` | No | — |
| `last_name` | `VARCHAR(128)` | No | — |
| `dob` | `DATE` | No | — |
| `gender` | `VARCHAR(16)` | Yes | — |
| `national_id_encrypted` | `BYTEA` | Yes | — |
| `national_id_hash` | `VARCHAR(64)` | Yes | — |
| `contact` | `VARCHAR(32)` | Yes | — |
| `dynamic_fields` | `JSONB` | No | '{}' |
| `status` | `student_status (ENUM: PRE_ENROLLED, ACTIVE, WITHDRAWN, GRADUATED, ARCHIVED, ANONYMIZED)` | No | 'PRE_ENROLLED' |
| `converted_from_application_id` | `UUID` | Yes | — |
| `workflow_instance_id` | `UUID` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT; `converted_from_application_id → admission_applications(id)` RESTRICT, nullable; `workflow_instance_id → workflow_instances(id)` RESTRICT.

## 5. Unique Constraints

`(student_id) WHERE deleted_at IS NULL` — though in practice `student_id` is **never** reused even after deletion (STU-001: "never reused"), so this is effectively a global unique constraint with the partial form kept only for schema consistency with the rest of this document, not because reuse is actually intended.

## 6. Indexes

`(institute_id, status)`; `(national_id_hash) WHERE national_id_hash IS NOT NULL` — the dedup-match lookup; for fuzzy name+DOB dedup matching (STU-003's "name + DOB + guardian" heuristic), a **trigram index** (`CREATE EXTENSION IF NOT EXISTS pg_trgm; CREATE INDEX ON students USING gin ((first_name || ' ' || last_name) gin_trgm_ops);`) supports similarity-based candidate matching far better than exact-match indexing alone.

## 7. Relationships

Referenced by `student_guardian_links`, `enrollments`, `elective_selections` (Wave 4, now FK-complete), `documents` (polymorphically), and — in later waves — exam eligibility, results, and fee invoices.

## 8. Soft Delete Strategy

Standard `deleted_at`, though `status ∈ {WITHDRAWN, GRADUATED, ARCHIVED, ANONYMIZED}` (STU-007's disclosed lifecycle) is overwhelmingly the operative marker; a hard-delete attempt is blocked by trigger at the data layer regardless of caller intent, identical rationale to every other "must remain resolvable forever" table in this schema.

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

**Direct, mandatory.** `institute_id` is NOT NULL — unlike `users`, a student's identity genuinely is institute-scoped (a student transferring institutes becomes, conceptually, a new admission/conversion at the destination, not a re-scoped row), so this table's isolation model deliberately differs from `users`' deployment-global one.

## 11. Notes for TypeORM Entity Design

`national_id_encrypted`/`national_id_hash` are `@Column({ select: false })` by default, identical rationale to `users.password_hash`; the unmask endpoint (`student.api.md` #3) is the only code path that ever explicitly selects `national_id_encrypted` and decrypts it, and every such call must emit the `audit_log` event itself (§0.1's masking-is-access-control framing means the *audit* of access matters more here than any DB-level mechanism could capture).

---

## 12. Performance Considerations

**The largest-volume entity table in the schema at most deployments** (every enrolled student, every year, accumulating — never hard-deleted). The trigram index on `(first_name || ' ' || last_name)` adds meaningful write-path overhead per insert/update in exchange for dedup-matching capability — acceptable given student creation/editing is orders of magnitude less frequent than, say, payment recording. `national_id_encrypted`/`national_id_hash` are both `select: false` by default specifically to keep this table's by-far-most-common read pattern (ordinary list/detail views) from paying for columns it never needs.