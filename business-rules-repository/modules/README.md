# Business Rules Catalog

The functional source of truth for the Enterprise Education ERP — the catalog developers implement against, QA tests against, and product evolves from. It does **not** redesign the architecture (locked in Blueprint Parts A–H); it specifies the *business logic* that runs on it.

## Conventions

**Every rule is:** Clear · Testable · Auditable · Versionable · Enterprise-grade.

**Rule ID prefixes (stable, never reused):**

| Domain | Prefix | Domain | Prefix |
|---|---|---|---|
| Authentication | `AUTH` | Class | `CLS` |
| Authorization | `AUTHZ` | Section | `SEC` |
| Institute | `INST` | Subject | `SUB` |
| Campus | `CAMP` | Examination | `EXM` |
| Academic Session | `SESS` | Result Processing | `RES` |
| Student | `STU` | Grading | `GRD` |
| Admission | `ADM` | Fee Management | `FEE` |
| Enrollment | `ENR` | Payment | `PAY` |
| Guardian | `GRD-N` | Discount | `DSC` |
| Attendance | `ATT` | Scholarship | `SCH` |
| Teacher | `TCH` | Leave | `LEV` |
| Staff | `STF` | HR / Payroll | `HR` |
| Notification | `NOT` | Reporting | `REP` |
| Configuration Engine | `CFG` | Workflow Engine | `WFL` |
| Audit | `AUD` | File Management | `FILE` |

**Each rule carries:** ID · Name · Description · Priority (Critical/High/Medium/Low) · Category · Preconditions · Business Rule · System Action · Validation · Failure Behavior · Audit Requirement · Example Scenario · Related Rules.

**Each module document carries 17 sections:** Module Purpose · Actors · Use Cases · Business Rules · Validation Rules · State Machine · Status Definitions · Workflow Rules · Permission Rules · Notification Rules · Audit Requirements · Data Retention Rules · Edge Cases · Failure Scenarios · Exception Handling Rules · Compliance Considerations · Future Considerations.

## Cross-cutting principles (inherited by every module)

1. **Per-client deployment isolation.** All rules execute within one client's isolated deployment; cross-client data access is structurally impossible.
2. **Multi-institute scoping.** Every business object is scoped (institute, often campus and session). Access requires permission **and** scope **and** ownership (three-layer, D35).
3. **Configuration-driven.** Where a rule says "configurable," the value resolves through the Configuration Engine (most-specific-wins) and is version-stamped — never hard-coded.
4. **Audit everything that matters.** Every state change emits an immutable audit event (actor, action, before/after, scope, timestamp) via the outbox (D13/D17).
5. **Minors' data is sensitive.** Student and guardian data receive heightened protection; rules reflect GDPR-aligned handling and child-data caution.
6. **Archive, never hard-delete** operational records; deletion is retention-driven anonymization with export-before-purge.

## Phases

- **Phase 1 — Core Platform (this delivery):** 01 Authentication · 02 Authorization · 03 Institute · 04 Campus · 05 Academic Session
- Phase 2 — Student Lifecycle · Phase 3 — Academic · Phase 4 — Examination & Results · Phase 5 — Finance · Phase 6 — HR · Phase 7 — Configuration Engine · Phase 8 — Workflow Engine · Phase 9 — Reporting/Notification/Audit

## Catalog Status — COMPLETE

All 9 phases delivered. **30 / 30 modules · 248 business rules · 87 use cases · all 17 sections each.**

| # | Module | Rules | # | Module | Rules |
|---|---|---|---|---|---|
| 01 | Authentication | 11 | 16 | Grading | 8 |
| 02 | Authorization | 9 | 17 | Fee Management | 8 |
| 03 | Institute | 8 | 18 | Payment | 9 |
| 04 | Campus | 7 | 19 | Discount | 7 |
| 05 | Academic Session | 8 | 20 | Scholarship | 8 |
| 06 | Student | 8 | 21 | Teacher | 7 |
| 07 | Admission | 8 | 22 | Staff | 8 |
| 08 | Enrollment | 8 | 23 | HR | 8 |
| 09 | Guardian | 8 | 24 | Leave Management | 9 |
| 10 | Attendance | 9 | 25 | Notification | 8 |
| 11 | Class | 7 | 26 | Reporting | 8 |
| 12 | Section | 6 | 27 | Configuration Engine | 11 |
| 13 | Subject | 7 | 28 | Workflow Engine | 11 |
| 14 | Examination | 9 | 29 | Audit Log | 8 |
| 15 | Result Processing | 9 | 30 | File Management | 8 |

**The two engines (27 Configuration, 28 Workflow) are the substrate**: every "configurable/version-stamped" promise resolves through the Configuration Engine; every "approval/escalation/SoD" promise resolves through the Workflow Engine. **Audit (29)** is the immutable backbone every module emits to; **File Management (30)** and **Notification (25)** close the loop on minors'-data protection (private signed-URL storage, custody-aware routing). The catalog is internally cross-referenced and consistent.
