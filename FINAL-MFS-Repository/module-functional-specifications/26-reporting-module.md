# 26 — Reporting Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint (D42 report defs as config), Business Rules Catalog (`REP-001…008`), and Use Case Repository (`UC-REP-001…016`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Provide reporting and export that **never bypass authorization**: configurable parameterized report definitions, scope + ownership injected on every run, sensitive-data masking, projection freshness/as-of transparency, export governance with bulk-PII controls, org-only deployment-bound cross-institute reporting, async large reports, and full access/export auditing.

**Business Goal.** Let users see exactly the data they are entitled to — no more — through reusable reports, with bulk-PII exports governed and every access/export audited, so reporting is a window, never a back door.

**Scope.** Report definitions (configurable, parameterized); authorization-preserving execution (scope + ownership injected; parameters narrow, never widen); sensitive-data masking; projection freshness/as-of; export governance + bulk-PII SoD; cross-institute (org-only, deployment-bound); async large reports; access/export auditing; signed-URL export delivery.

**Out of Scope.** Source data ownership (each domain module owns it; Reporting reads via authorized projections). Authorization rules themselves (Authorization module — Reporting enforces them). File storage of exports (File module — signed-URL delivery). Notification (Notification module).

---

## 2. Actors

**Primary Actors.** Report Consumer (runs scoped reports), Report Administrator (manages definitions/schedules), Export Approver (bulk-PII — SoD), Data-Protection Officer (subject-access), System (scope/ownership injection, async generation).

**Secondary Actors.** Authorization (scope/ownership predicates), all domain modules (data sources/projections), File module (export delivery), Workflow Engine (export approval), Configuration Engine (report defs), Audit.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Parameterized report definitions | Define configurable, parameterized reports. | High | Configuration Engine (D42) |
| FR-002 | Authorization-preserving execution | Inject scope + ownership; parameters narrow, never widen. | Critical | Authorization (AUTHZ-002/003) |
| FR-003 | Sensitive-data masking | Mask sensitive fields by role need-to-know. | Critical | Authorization |
| FR-004 | Freshness / as-of transparency | Expose projection freshness/as-of time. | Medium | Projections |
| FR-005 | Export governance + bulk-PII SoD | Govern exports; bulk-PII requires approval (SoD). | Critical | Workflow, AUTHZ-009 |
| FR-006 | Cross-institute org-only, deployment-bound | Cross-institute only at org level; never cross-deployment. | High | Institute isolation |
| FR-007 | Async large reports | Generate large reports asynchronously, performance-safe. | High | File, queue |
| FR-008 | Access & export auditing | Audit every report access and export. | High | Audit |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Parameterized reports | Reusable definitions. | Efficiency. |
| Authorization-preserving | Scope + ownership injected. | No data leakage. |
| Masking | Need-to-know fields. | Privacy. |
| As-of transparency | Freshness exposed. | Trust in numbers. |
| Export governance | Bulk-PII SoD. | Controlled disclosure. |
| Org-only cross-institute | Deployment-bound. | Tenant isolation. |
| Async large reports | Non-blocking. | Scale. |
| Access/export audit | Full trail. | Accountability. |

---

## 5. Screens

Report Catalog; Run Report (parameterized, scoped); Report Viewer (with as-of); Export (signed-URL); Bulk-PII Export Approval; Data-Subject Access Report; Report Definition Management; Schedule Reports; Cross-Institute Summary (org-only); Report Search; Report Access & Export Audit; Bulk Report Generation; Export-Approval.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Report Catalog | Open, Run, Schedule, Search | — |
| Run Report | Set Parameters (narrowing), Run, View As-Of | — |
| Report Viewer | Filter, Drill (within scope), Export | — |
| Export | Export (signed-URL), Request Bulk-PII (→ approval) | — |
| Bulk-PII Approval | Approve, Reject (SoD) | — |
| Definition Management | Create/Edit/Version Definition | — |
| Cross-Institute Summary | Run (org-only), View | — |
| Access/Export Audit | Run, Filter, Export | Export |

---

## 7. Forms

**Run Report** — `report` (select), `parameters` (typed, within entitlement). Validation: scope + ownership injected server-side (REP-002/AUTHZ-002/003); parameters may only narrow, never widen (REP-002 / UC-REP-015); sensitive fields masked (REP-003).

**Report Definition** — `name`, `dataSource/projection`, `parameters`, `columns`, `maskingRules`. Validation: config-defined (REP-001/D42); declares masking and required entitlement; versioned (CFG-004).

**Export** — `format`, `scope`. Validation: governed; bulk-PII triggers SoD approval (REP-005 / UC-REP-008); delivered via signed URL (FILE-005); access/export audited (REP-008).

**Cross-Institute Summary** — `institutes` (within deployment/org). Validation: org-level only, deployment-bound (REP-006 / UC-REP-016 cross-deployment blocked).

---

## 8. Search & Filter Requirements

**Reports:** by name, category, schedule, last-run, owner. Sorting: name/last-run. Pagination: server-side, 25 default. Catalog scoped to entitlement; results inherit scope/ownership injection.

---

## 9. Table Requirements

**Report results:** columns per definition with masking applied; as-of timestamp shown. **Catalog:** Report, Category, Parameters, Schedule, Last Run. Sorting on Name/Last-Run. Filtering as above. Export (governed, signed-URL; bulk-PII gated). Bulk: scheduled generation.

---

## 10. Workflow Requirements

**Trigger events:** run, schedule, export, bulk-PII export request, async generation, subject-access. **Status changes:** report job `QUEUED → RUNNING → READY/FAILED`; export `REQUESTED → APPROVED → DELIVERED`. **Approvals:** bulk-PII export via Workflow Engine (SoD). **Notifications:** report ready, export approved/delivered (via signed URL). **Audit:** every access and export (who/what/when/scope — REP-008), bulk-PII approvals (immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Run report (scoped) | `report.run` |
| Manage definitions | `report.definition.manage` |
| Schedule reports | `report.schedule.manage` |
| Export | `report.export` |
| Approve bulk-PII export | `report.bulk_pii.approve` (SoD) |
| Cross-institute (org) | `report.cross_institute.view` |
| Subject-access report | `report.subject_access` |
| Access/export audit view | `report.audit.view` |

Every run injects scope + ownership (REP-002); parameters cannot widen entitlement; bulk-PII requires SoD.

---

## 12. Business Rule References

REP-001 (configurable parameterized report definitions), REP-002 (reports never bypass authorization — scope + ownership), REP-003 (sensitive-data masking), REP-004 (projection freshness & as-of transparency), REP-005 (export governance & bulk-PII controls), REP-006 (cross-institute org-only & deployment-bound), REP-007 (asynchronous performance-safe large reports), REP-008 (report access & export auditing). Cross-cutting: AUTHZ-002/003 (scope/ownership), CFG-004/D42 (report defs as config), FILE-005 (signed-URL delivery), WFL-002/004 (export approval), AUTHZ-009 (SoD), AUD-001/008, STU-005 (masking source).

## 13. Use Case References

UC-REP-001 (Run Scoped Report), UC-REP-002 (Large/Scheduled Export — async), UC-REP-003 (Data-Subject Access Report), UC-REP-004 (Mask Sensitive Data), UC-REP-005 (Expose Projection Freshness), UC-REP-006 (Manage Report Definitions), UC-REP-007 (Schedule Reports), UC-REP-008 (Approve Bulk-PII Export — SoD), UC-REP-009 (Search), UC-REP-010 (Cross-Institute Summary — org-only), UC-REP-011 (Bulk Report Generation), UC-REP-012 (Export — signed-URL), UC-REP-013 (Export-Approval Workflow), UC-REP-014 (Report Access & Export Audit), UC-REP-015 (Scope-Widening Parameter Constrained), UC-REP-016 (Cross-Deployment Report Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| List report catalog (entitled) | GET | Consumer |
| Run report (scope/ownership injected) | POST | Consumer |
| Get report result (with as-of) | GET | Consumer |
| Export report (signed-URL) | POST | Consumer |
| Request bulk-PII export (approval) | POST | Consumer (→ approval) |
| Manage / version report definition | POST/PUT | Admin |
| Schedule report | POST | Admin |
| Cross-institute summary (org-only) | GET | Org Admin |
| Subject-access report | POST | DPO |
| Report access & export audit | GET | Admin |

Scope + ownership are injected server-side and parameters can only narrow (REP-002); bulk-PII export requires SoD approval (REP-005); large reports run async (REP-007).

---

## 15. Database Requirements

**Entities:** `ReportDefinition` (source/projection, parameters, columns, masking, version), `ReportJob` (status, as-of, requester), `ReportExport` (format, approval, signed-URL ref), `ExportApproval` (bulk-PII, approver), `ReportAccessLog`. **Relationships:** Definition 1—* Job; Job 1—* Export. **Indexes:** index(ReportJob.status, requesterId), index(ReportExport.status), index(ReportAccessLog.reportId, ts). Reads go through authorized projections with scope/ownership predicates (REP-002); access/export logged (REP-008). Reporting holds no authoritative domain data — only definitions, jobs, logs.

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Report ready; export approved/delivered (signed-URL link). |
| In-App | Bulk-PII export approvals; scheduled-report completion. |
| SMS/Push | Optional report-ready alert. |

Export links are short-lived signed URLs (FILE-005); routed per Notification rules.

---

## 17. Audit Requirements

Log: every report access (report, parameters, scope, requester), every export (format, volume, bulk-PII flag), bulk-PII approvals (approver), definition/schedule changes, cross-institute access, subject-access fulfilment. Record who/what/when/scope. Access and export are first-class audit events (REP-008); no raw sensitive values stored (AUD-004). Immutable via outbox.

---

## 18. Reporting Requirements

**Reports (self):** Report-usage/access, Export volume + bulk-PII frequency, Definition inventory, Cross-institute access log, Slow/async-report performance. **Exports:** governed (own audit). **Dashboards:** reporting governance (access patterns, export approvals, masking coverage, async queue).

---

## 19. Error Handling

**Validation:** scope-widening parameter, cross-deployment request → constrained/blocked (UC-REP-015/016). **Permission:** export beyond entitlement / bulk-PII without approval → blocked. **Workflow:** bulk-PII pending → not delivered until approved. **System:** large report → async (never blocks); projection stale → as-of disclosed (REP-004), not hidden.

---

## 20. Edge Cases

**Concurrent updates:** definition change during scheduled run → run pins the version. **Duplicate data:** duplicate export request idempotent. **Partial failures:** bulk generation partial → per-report status. **Rollback:** failed export → no partial signed-URL leak. **Scope race:** entitlement reduced mid-run → result reflects scope at run start; next run reflects new scope (never broader than current entitlement).

---

## 21. Acceptance Criteria

**Functional.** Reports are parameterized definitions; every run injects scope + ownership and parameters can only narrow (never widen) entitlement; sensitive fields are masked by role; projection freshness/as-of is exposed; exports are governed and bulk-PII requires SoD approval delivered via signed URL; cross-institute reporting is org-only and never crosses deployments; large reports run asynchronously; every access and export is audited.

**Business.** Users see exactly what they're entitled to and nothing more; reporting cannot become a privilege-escalation or data-exfiltration path; bulk disclosure is controlled and fully accountable.

---

## 22. Future Enhancements

Self-service report builder (entitlement-bounded); natural-language query over authorized projections; scheduled-report distribution lists; differential-privacy aggregates; report-level data-retention; anomaly detection on export patterns; embedded analytics dashboards.
