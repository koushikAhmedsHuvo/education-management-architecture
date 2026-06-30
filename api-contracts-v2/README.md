# Enterprise Education ERP — API Contract Repository

Production-ready API contract documentation for an Enterprise Education ERP: a modular-monolith, multi-institute (tenant-isolated), configuration-driven system deployed separately per client. Backend: NestJS + PostgreSQL + TypeORM. Frontend: Next.js App Router + RTK Query.

This repository is grounded strictly in the already-approved Architecture Blueprint, Business Rules Catalog, Use Case Specifications, Module Functional Specifications, and UI Screen Specifications. No business logic was changed, no module was introduced, and no workflow was redesigned in producing it — every endpoint cites the specific approved rule(s) it implements. The one explicitly flagged exception is `wave-7-finance/accounting.api.md`, which proposes a conventional double-entry ledger beyond the approved Business Rules Catalog (see that file's header note).

**Start here:** [`00-api-conventions.md`](./00-api-conventions.md) — every file in this repository inherits its conventions (versioning, the response envelope, HTTP status mapping, pagination/filtering, soft delete, optimistic concurrency, async/bulk job handling, governed exports) rather than restating them.

## Repository Structure

```
/api-contracts
├── 00-api-conventions.md                    Shared conventions — read first
│
├── wave-1-platform-foundation/               (100 endpoints)
│   ├── auth.api.md                           21  — Authentication, MFA, sessions, policy
│   ├── user.api.md                            14  — User account lifecycle
│   ├── role-permission.api.md                 23  — Roles, Permission Registry, Membership, SoD
│   ├── institute.api.md                        17  — Institute (primary tenant scope)
│   ├── campus.api.md                           12  — Campus (secondary scope)
│   └── academic-session.api.md                 13  — Academic Session, rollover, promotion
│
├── wave-2-configuration-engine/              (33 endpoints)
│   ├── settings.api.md                         20  — Definition registry, resolution, versioning, rollback
│   ├── custom-fields.api.md                     3  — Dynamic custom-field schemas
│   ├── dynamic-forms.api.md                     6  — Form composition over custom fields
│   └── id-generation.api.md                     4  — Student/Employee ID format & generation
│
├── wave-3-workflow-engine/                   (24 endpoints)
│   ├── workflow-definition.api.md               5  — FSM definitions, versioning
│   ├── workflow-instance.api.md                15  — Initiate, approve, delegate, recover
│   └── approval-flow.api.md                     4  — SLA reporting, definition-change governance
│
├── wave-4-academic-structure/                (43 endpoints)
│   ├── class.api.md                            15  — Class structure, promotion path
│   ├── section.api.md                          12  — Section (enrollment leaf), restructure
│   ├── subject.api.md                          12  — Subject identity, components, electives
│   └── curriculum.api.md                        4  — Subject↔Class/Stream mapping
│
├── wave-5-student-lifecycle/                 (91 endpoints)
│   ├── admission.api.md                        27  — Application, evaluation, decision, conversion
│   ├── student.api.md                          25  — Masked PII, official-field governance, archive
│   ├── enrollment.api.md                       20  — Placement, transfer, withdrawal, roll number
│   └── guardian.api.md                         19  — Children-only scope, custody, consent
│
├── wave-6-examination/                       (62 endpoints)
│   ├── exam.api.md                             12  — Definition, eligibility, admit cards
│   ├── marks.api.md                             9  — Entry, lock, moderation, correction
│   ├── result.api.md                           19  — Compute, publish, revise, withhold
│   ├── grading.api.md                          15  — Grade scales, GPA, overrides
│   └── transcript.api.md                        7  — Marksheet view + cumulative transcript
│
└── wave-7-finance/                           (96 endpoints)
    ├── fee-structure.api.md                     9  — Fee structures, heads, late-fine policy
    ├── fee-collection.api.md                   15  — Invoicing, dues, fines, refund decision
    ├── payment.api.md                          28  — Recording, allocation, reconciliation, reversal
    ├── discount.api.md                         15  — Discounts, waivers, stacking
    ├── scholarship.api.md                      19  — Funded programs, selection, disbursement
    └── accounting.api.md ⚠                     10  — Beyond approved repo — see file header
```

**Total: 32 files · 449 endpoints**, every one specifying Method, URL, Description, Authentication (role-based), Request DTO (JSON Schema), Response DTO (JSON Schema), Validation Rules, Error Codes, Business Rules Applied, and Pagination.

## How to read a file

Each `*.api.md` file is self-contained:

1. **API Overview** — Purpose and Module Context (which approved Business Rules Catalog doc + UI Screen Spec it implements).
2. **Endpoints** — every endpoint in the file, fully specified per the structure above.
3. **Standards** — the file's RESTful/versioning/soft-delete/audit/multi-tenant conventions, consistent across all 32 files (see `00-api-conventions.md` for the full detail each one points back to).

## Cross-file references

Several endpoints are intentionally **not duplicated** across files where two screens in the approved UI specs surface the same underlying operation:

- `wave-1-platform-foundation/institute.api.md` #14 (Apply Type Template) is the same action referenced by Configuration Engine's `SCR-CFG-09`.
- `wave-4-academic-structure/class.api.md` defers per-session instantiation to `academic-session.api.md` #5.
- `wave-5-student-lifecycle/enrollment.api.md` defers promotion to `academic-session.api.md` #7/#13 (driven by session rollover).
- `wave-6-examination/transcript.api.md` reproduces — never recomputes — the provenance already stamped by `result.api.md` #2.
- `wave-7-finance/fee-collection.api.md` #18–20 (Refund) **decides**; `wave-7-finance/payment.api.md` #20 (Reverse Payment) **executes** — one decision, no duplicated logic.

## Splits from the original flat contract

This structure reorganizes (and in four cases, extends) an earlier flat 23-file delivery covering the same 7 waves. Files that combined multiple UI-spec concerns were split along the same lines this repository's requested structure implies (e.g., Configuration Engine → `settings` + `custom-fields`; Workflow Engine → `workflow-definition` + `workflow-instance` + `approval-flow`; Examination → `exam` + `marks`; Result Processing → `result` + `transcript`; Fee Management → `fee-structure` + `fee-collection`). `dynamic-forms.api.md`, `id-generation.api.md`, and the cumulative-transcript endpoints in `transcript.api.md` are newly authored, each with an explicit grounding note in its own Module Context explaining the approved rule(s) it extends. `accounting.api.md` is newly authored and explicitly flagged as beyond the approved repo.
