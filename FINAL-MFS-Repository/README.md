# Module Functional Specification Repository — Final, Implementation-Ready

> Configurable Enterprise Education ERP — one product serving school / college / university / madrasa / coaching / training, Bangladesh-first, multi-tenant. This repository is the controlling **functional baseline** for development, traceable end-to-end to the approved Architecture Blueprint, 248 Business Rules, and 512 Use Cases.

## What this repository contains

**A. The 30 Module Functional Specifications** (`module-functional-specifications/`) — each to a fixed 22-section template (Overview, Actors, Functional Requirements, Features, Screens, Screen Actions, Forms, Search & Filter, Table, Workflow, Permissions, Business Rule References, Use Case References, API Requirements, Database, Notifications, Audit, Reporting, Error Handling, Edge Cases, Acceptance Criteria, Future Enhancements).

**B. Eight cross-cutting analysis matrices** (`final-repo/`) synthesizing the 30 specs into the views an enterprise build needs:

| # | Document | Purpose |
|---|---|---|
| 00 | README (this file) | Index, metrics, how-to-use |
| 01 | Module Dependency Matrix | Inter-module dependencies (from §12 cross-citations); build order / fan-in |
| 02 | Screen Dependency Matrix | 416 screens + 7 shared cross-cutting archetypes to build once |
| 03 | Permission Matrix | 265 namespaced permissions; 24 SoD `.approve` authorities |
| 04 | Workflow Matrix | Which modules invoke the Workflow Engine and for what governed processes |
| 05 | API Coverage Matrix | 317 endpoint purposes by module and HTTP method |
| 06 | Business Rule Mapping | All 248 rules → owning + referencing modules (100% covered) |
| 07 | Use Case Mapping | All 512 use cases → implementing modules (100% covered) |
| 08 | Development Readiness Review | Gaps, overlaps, missing requirements, risks, build sequencing |

## Headline metrics

- **30 / 30** modules specified, **22 / 22** sections each
- **270** functional requirements
- **248 / 248** business rules cited (100%, no orphans)
- **512 / 512** use cases cited (100%, every module covers all its home use cases)
- **265** permissions · **317** API endpoint purposes · **416** screens · **24** SoD approval authorities
- **0** broken references across the repository

## The 30 modules

| Mod | Module | FRs | Perms | APIs |
|---|---|---|---|---|
| 01 | Authentication Module | 14 | 9 | 21 |
| 02 | Authorization Module | 14 | 9 | 10 |
| 03 | Institute Module | 11 | 10 | 10 |
| 04 | Campus Module | 9 | 10 | 10 |
| 05 | Academic Session Module | 10 | 10 | 11 |
| 06 | Admission Module | 10 | 8 | 13 |
| 07 | Student Module | 10 | 12 | 12 |
| 08 | Enrollment Module | 9 | 10 | 9 |
| 09 | Guardian Module | 9 | 10 | 10 |
| 10 | Attendance Module | 9 | 9 | 11 |
| 11 | Class Module | 8 | 10 | 9 |
| 12 | Section Module | 7 | 8 | 8 |
| 13 | Subject Module | 7 | 10 | 11 |
| 14 | Examination Module | 9 | 9 | 10 |
| 15 | Result Processing Module | 9 | 8 | 8 |
| 16 | Grading Module | 8 | 6 | 8 |
| 17 | Fee Management Module | 8 | 9 | 10 |
| 18 | Payment Module | 9 | 9 | 10 |
| 19 | Discount Module | 7 | 8 | 9 |
| 20 | Scholarship Module | 7 | 9 | 9 |
| 21 | Teacher Module | 7 | 7 | 10 |
| 22 | Staff Module | 8 | 10 | 11 |
| 23 | HR Module | 8 | 11 | 13 |
| 24 | Leave Management Module | 9 | 8 | 11 |
| 25 | Notification Module | 8 | 6 | 9 |
| 26 | Reporting Module | 8 | 8 | 10 |
| 27 | Configuration Engine Module | 11 | 9 | 13 |
| 28 | Workflow Engine Module | 11 | 8 | 12 |
| 29 | Audit Log Module | 8 | 7 | 9 |
| 30 | File Management Module | 8 | 8 | 10 |

## Conventions

- FR-IDs are **local per module** (FR-001…). §12 cites real business-rule IDs; §13 cites real use-case IDs — both validated programmatically (0 broken refs).
- Permissions are namespaced `module.entity.action` and enforced via the 3-layer model (RBAC role → scope → ownership; architecture D35).
- Architecture decisions are cited by ID (D-nn) where they constrain implementation; conflict resolutions by ID (C-01…C-10).
- ID prefix notes: Guardian uses `GRD-N` / Grading uses `GRD` (never collide); use cases carry `UC-` prefixes.

## How to use this repository

1. **Architects / leads:** start with the Readiness Review (08) and Module Dependency Matrix (01) for build sequencing and risk gating.
2. **Backend teams:** each module's §3 / §14 / §15 plus the API Coverage Matrix (05); produce OpenAPI contracts next (Readiness Review §4).
3. **Frontend teams:** each module's §5 / §6 / §7 / §9 plus the Screen Dependency Matrix (02); build the 7 shared archetypes first.
4. **QA:** each module's §19 / §20 / §21 (Error Handling / Edge Cases / Acceptance Criteria) as the test basis; Use Case Mapping (07) for traceability.
5. **Security / compliance:** Permission Matrix (03), Workflow Matrix (04), and the minors'-data / audit risks in Readiness Review §5.

## Outstanding decisions before code-complete (not blockers to kickoff)

- Product owner: re-evaluation request-intake portal (RES / C-02) — currently a future enhancement, no approved rule.
- Finance lead: confirm the refund decision (Fee 17) vs execution (Payment 18) split.
- Next artifacts to produce: API contracts (OpenAPI), data dictionary, NFR baseline, notification-template and report-definition catalogs, migration/seeding plan.
