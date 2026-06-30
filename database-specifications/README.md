# Enterprise Education ERP — Database Specification Repository

A complete, production-ready PostgreSQL database specification for an Enterprise Education ERP: a modular-monolith, multi-institute (tenant-isolated), configuration-driven system, deployed separately per client. Backend: NestJS + TypeORM. Database: PostgreSQL 16+.

This repository is grounded strictly in the already-approved Architecture Blueprint, Business Rules Catalog, Use Case Specifications, Module Functional Specifications, UI Screen Specifications, Design System, and API Contract Specifications (Waves 1–7). No business logic was changed, no module was redesigned, and no entity was introduced beyond what those approved artifacts already define — every table cites the specific rule(s) and API endpoint(s) it backs.

**83 tables, 85 specification files** (two files — `exam-schedules.schema.md` and `transcripts.schema.md` — document design decisions rather than separate tables; see their own Table Purpose sections for why).

## Repository Structure

```
/database-specifications
├── wave-1-platform-foundation/        (17 tables) — Authentication, Authorization, Institute, Campus, Academic Session
├── wave-2-configuration-engine/        (9 tables) — Definition registry, scoped/versioned values, custom fields, dynamic forms, ID generation
├── wave-3-workflow-engine/             (4 tables) — FSM definitions, instances, history, delegation
├── wave-4-academic-structure/          (9 tables) — Class, section, subject, curriculum
├── wave-5-student-lifecycle/          (12 tables) — Admission, student, guardian, enrollment, documents, legal holds
├── wave-6-examination/                (10 tables, 2 explainer files) — Exam, marks, results, grading
└── wave-7-finance/                    (22 tables) — Fee, payment, discount, scholarship, accounting (⚠ partially beyond approved repo)
```

## How each file is organized

Every `*.schema.md` file follows the identical 12-section structure:

1. Table Purpose
2. Columns (Column Name · Data Type · Nullable · Default Value)
3. Primary Key
4. Foreign Keys
5. Unique Constraints
6. Indexes
7. Relationships
8. Soft Delete Strategy
9. Audit Fields (mandatory: `created_at`, `created_by`, `updated_at`, `updated_by`, `deleted_at`, `deleted_by`)
10. Multi-Institute Isolation Strategy
11. Notes for TypeORM Entity Design
12. Performance Considerations

## Cross-cutting conventions (apply to every table)

- **UUID primary keys**, generated as UUIDv7 (time-ordered, index-friendly) — see `wave-1-platform-foundation/users.schema.md` for the canonical statement of this convention; every other file inherits it without restating it.
- **Soft delete everywhere**, via `deleted_at`/`deleted_by`, with partial unique indexes (`WHERE deleted_at IS NULL`) so business identifiers free up on deletion rather than being permanently locked. A small number of tables (`payments`, `invoices`, `permissions`, `config_definitions`, `workflow_definitions`) deliberately have **no operative soft-delete path at all** — each says so explicitly in its own §8, because consumers depend on those rows remaining resolvable forever (an issued payment receipt, a permission key referenced by historical role grants).
- **JSONB** is used deliberately and narrowly — for genuinely variable-shape data (audit payloads, job results, the Configuration Engine's own typed values) and for small, always-read-as-a-whole nested structures (a subject's components, an exam's per-subject maxima). Anywhere a cross-cutting, independently-queried relationship exists (role↔permission, class↔subject, journal entry↔account), a normalized join table is used instead — each file's own notes explain which choice was made and why.
- **3NF minimum**, with a small number of explicitly-justified denormalizations (`enrollments.institute_id`, `invoices`' lack of one) called out individually in their own Performance Considerations sections, never left implicit.

## Three platform-wide tables, built once, reused everywhere

`audit_log` and `async_jobs` (Wave 1), and `documents` and `legal_holds` (Wave 5), are generic, polymorphic tables built exactly once, at the wave where they were first genuinely needed, and reused unchanged by every subsequent wave — rather than each module inventing its own audit table, job-tracking table, or file-storage table. `exam-schedules.schema.md` and `transcripts.schema.md` (Wave 6) are explicit, documented examples of this same discipline applied at the file level: rather than create redundant tables for concepts the approved API contract already satisfies through `exams`' own columns and the shared `documents` table respectively, those two files document *why* no new table exists and *where* the data actually lives.

## A pattern reused four times across the schema

`config_values` (Wave 2), `grade_scales` (Wave 6), and `fee_structures` (Wave 7) all use the identical **append-only row + PostgreSQL `EXCLUDE USING gist` constraint** pattern to guarantee non-overlapping effective-dated versions at the database level, not just in application code. `enrollments` (Wave 5), `results` (Wave 6), `credit_balance_entries`, and `scholarship_disbursements` (Wave 7) use the related **append-only ledger** principle — balances and placement history are computed from immutable rows, never stored as a mutable running total that could drift. Each occurrence is noted explicitly in its file as a reuse of an established pattern, not a coincidence.

## ⚠ One flagged exception

`wave-7-finance/accounting.schema.md`, `journal-entries.schema.md`, and `journal-entry-lines.schema.md` implement a standard double-entry general-ledger model that is **not** part of the approved Business Rules Catalog — there is no corresponding rule document for general accounting in the approved specifications. These three files are explicitly flagged in their own Table Purpose / Isolation sections and should be reconciled with the architecture team before implementation, exactly as the API Contract's own `accounting.api.md` flags the same gap.

## Forward references, all closed

Several tables in earlier waves declare a column for a relationship that doesn't yet exist as a real foreign key until a later wave is built (e.g., `roles.workflow_instance_id` in Wave 1, resolved once `workflow_instances` exists in Wave 3; `elective_selections.student_id` in Wave 4, resolved once `students` exists in Wave 5). Every such forward reference is explicitly noted in both the originating table's Foreign Keys section and closed in the target wave — none are left dangling.
