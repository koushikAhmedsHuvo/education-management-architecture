# 26 — Reporting Use Cases

Transforms the Reporting business rules (`REP-001`…`REP-008`) into use cases. Insight and documents from operational data: configurable parameterized reports that **never bypass the three-layer authorization**, sensitive-data masking, projection-freshness transparency, governed exports with bulk-PII controls, org-only cross-institute reporting, async large reports, and full access/export auditing.

## 1. Primary Actors
Administrator / Coordinator (runs/schedules reports), Teacher/Staff (ownership-limited reports), Finance/HR roles (need-to-know reports), Student/Guardian (own-data reports).

## 2. Secondary Actors
System (scope/ownership enforcement, masking, async generation, audit), Read-model/projections, File module (export delivery via signed URL), Workflow Engine (bulk-PII approval), Audit service.

## 3. Goals
Run configurable parameterized reports consistent with source rules; apply scope + ownership to every report and export (no authorization backdoor); mask sensitive data per need-to-know; expose data freshness; govern exports (bulk-PII gated/audited); keep cross-institute reporting org-only and deployment-bound; run large reports asynchronously; audit all access/exports.

## 4. User Journeys
- **Run scoped:** a user selects a report → the engine injects scope + ownership → masks sensitive fields → renders with an as-of timestamp.
- **Export:** a large/PII export runs async → scope/ownership/masking applied → delivered via short-lived signed URL → fully audited.
- **Subject access:** a data-subject report compiles only that subject's data.
- **Govern:** bulk-PII exports route through approval; cross-institute reports require org authority.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-REP-001 | Run Scoped Report (scope + ownership) | Critical |
| Core | UC-REP-002 | Large / Scheduled Export (async, governed) | High |
| Core | UC-REP-003 | Data-Subject Access Report | High |
| Core | UC-REP-004 | Mask Sensitive Data in Reports | High |
| Core | UC-REP-005 | Expose Projection Freshness (as-of) | Medium |
| CRUD | UC-REP-006 | Manage Report Definitions | High |
| Admin | UC-REP-007 | Schedule Reports | Medium |
| Approval | UC-REP-008 | Approve Bulk-PII Export (SoD) | High |
| Search | UC-REP-009 | Search Reports | Low |
| Reporting | UC-REP-010 | Cross-Institute Summary (org-only) | Medium |
| Bulk | UC-REP-011 | Bulk Report Generation | Medium |
| Export | UC-REP-012 | Export Report (signed-URL delivery) | High |
| Workflow | UC-REP-013 | Export-Approval Workflow | Medium |
| Reporting | UC-REP-014 | Report Access & Export Audit | High |
| Exception | UC-REP-015 | Scope-Widening Parameter Constrained | High |
| Exception | UC-REP-016 | Cross-Deployment Report Blocked | Critical |

---

## 6. Detailed Specifications (high-value use cases)

### UC-REP-001 — Run Scoped Report (scope + ownership)
- **Module:** Reporting · **Priority:** Critical
- **Actors:** Administrator / Teacher / Staff (primary), System
- **Goal:** Generate a report constrained to the requester's authorized scope and ownership — reporting can never expose data the requester could not access directly.
- **Description:** The engine injects the mandatory scope filter and ownership predicates into every report query; parameters can narrow but never widen; sensitive fields are masked; output carries an as-of timestamp.
- **Business Rules Applied:** REP-001, REP-002, REP-003, REP-004.
- **Preconditions:** Report defined; requester holds the report permission.
- **Trigger:** Requester runs a report with parameters.
- **Main Success Scenario:**
  1. Requester selects a report and parameters.
  2. The engine injects scope filter + ownership predicates into the query (REP-002, AUTHZ-002/003).
  3. The engine applies role-based masking (REP-003) and computes consistently with source rules (REP-001).
  4. The report renders with an as-of timestamp (REP-004).
- **Alternative Flows:** A1) Org admin runs a cross-institute report (UC-REP-010, still one deployment).
- **Exception Flows:** E1) Parameters reaching beyond scope → silently constrained to authorized scope (UC-REP-015). E2) Missing report permission → denied.
- **Validation Rules:** Scope + ownership applied; parameters narrow only; masking per role (REP-002/003).
- **Permissions Required:** `report.run` (+ the data's own scope/ownership).
- **Notifications Triggered:** None routine.
- **Audit Events Generated:** `REPORT_RUN` (requester, parameters, scope, as-of).
- **Data Created/Updated/Deleted:** None (read).
- **Post Conditions:** Authorized, masked report produced; access audited.
- **Related Use Cases:** UC-REP-002, UC-REP-004, UC-AUTHZ-001.
- **Acceptance Criteria:**
  - Given a teacher, When they run a class report, Then it includes only their students (ownership).
  - Given a parameter reaching beyond scope, When run, Then results are constrained to authorized scope (never widened).
  - Given sensitive fields, When rendered, Then they are masked per role.
- **Edge Case Analysis:**
  - *Invalid Input:* invalid parameters rejected.
  - *Permission Failure:* no `report.run` → denied; out-of-scope data never returned.
  - *Concurrent Update:* report during data change → consistent projection snapshot (as-of shown).
  - *Duplicate Data:* N/A.
  - *System Failure:* projection unavailable → flag, not silently-wrong data.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* scoped/owned report; masked fields.
  - *Negative:* scope-widening; missing permission; out-of-scope leakage (must not occur).
  - *Boundary:* org-wide vs institute-scoped; as-of freshness edge.

### UC-REP-002 — Large / Scheduled Export (async, governed)
- **Module:** Reporting · **Priority:** High
- **Actors:** Administrator (primary), System, Approver
- **Goal:** Export a large dataset or schedule recurring reports, governed and delivered securely.
- **Description:** Exports run asynchronously with scope/ownership/masking; bulk-PII exports may require approval; output is stored privately and delivered via a short-lived signed URL; the export is audited as exfiltration-sensitive.
- **Business Rules Applied:** REP-005, REP-007, REP-008, FILE-005.
- **Preconditions:** Export permission; export governance satisfied.
- **Trigger:** Requester triggers an export or schedule.
- **Main Success Scenario:**
  1. Requester triggers an export (or schedule).
  2. The engine runs it asynchronously, applying scope/ownership/masking (REP-002/003).
  3. Bulk-PII exports route through approval (REP-005, UC-REP-008).
  4. Output is stored privately and delivered via a short-lived signed URL (FILE-005); the export is audited (REP-008).
- **Alternative Flows:** A1) Scheduled recurring export with the same controls.
- **Exception Flows:** E1) Bulk PII beyond authority → blocked/approval-gated (REP-005). E2) Output large/slow → batched/streamed (REP-007).
- **Validation Rules:** Scope/ownership/masking applied; bulk PII governed; signed-URL delivery; audited (REP-005/007/008).
- **Permissions Required:** `report.export` (elevated for bulk PII).
- **Notifications Triggered:** Export-ready (secure link); approval notices.
- **Audit Events Generated:** `REPORT_EXPORTED` (dataset, row count, scope).
- **Data Created:** Export file (private).
- **Data Updated:** None.
- **Data Deleted:** None (file retained per policy; link expires).
- **Post Conditions:** Export delivered securely; fully audited.
- **Related Use Cases:** UC-REP-008, UC-FILE-002, UC-REP-014.
- **Acceptance Criteria:**
  - Given a large export, When run, Then it executes asynchronously without blocking interactive use.
  - Given a bulk-PII export, When requested beyond authority, Then it is blocked or approval-gated.
  - Given an export, When delivered, Then it uses a short-lived signed URL and is audited.
- **Edge Case Analysis:**
  - *Invalid Input:* invalid export spec rejected.
  - *Permission Failure:* bulk PII without elevation → blocked.
  - *Concurrent Update:* export during data change → snapshot-consistent.
  - *Duplicate Data:* idempotent scheduled runs.
  - *System Failure:* resumable export job.
  - *Workflow Failure:* approval pends; no export until approved.
- **QA Coverage:**
  - *Positive:* async export; scheduled; secure delivery.
  - *Negative:* over-authority bulk PII; unmasked sensitive export.
  - *Boundary:* very large dataset; link-expiry edge.

### UC-REP-003 — Data-Subject Access Report
- **Module:** Reporting · **Priority:** High
- **Actors:** Administrator / Guardian (primary), System
- **Goal:** Compile a subject's own data for a right-to-access request, scoped to that subject only.
- **Description:** An authorized handler runs a subject-data report compiling the student's/guardian's/staff's data across modules, scoped to that subject; mixed data of others is excluded; the access is audited.
- **Business Rules Applied:** REP-002, REP-008, STU-005, P11.
- **Preconditions:** Verified data-subject request.
- **Trigger:** A subject-access request.
- **Main Success Scenario:**
  1. Authorized handler runs the subject-data report for the subject.
  2. The engine compiles the subject's data across modules, scoped to that subject only (REP-002).
  3. The report is delivered securely; the access request is audited (REP-008).
- **Alternative Flows:** A1) Guardian requests their child's data (children-only, GRD-N-004).
- **Exception Flows:** E1) Data referencing other subjects → only the requesting subject's data included.
- **Validation Rules:** Subject-scoped; mixed data excluded; audited (REP-002/008).
- **Permissions Required:** `report.subject_access` / guardian (own child).
- **Notifications Triggered:** Fulfillment notice.
- **Audit Events Generated:** `SUBJECT_ACCESS_FULFILLED`.
- **Data Created:** Subject-access report.
- **Data Updated:** None.
- **Data Deleted:** None.
- **Post Conditions:** Subject-access report produced; audited.
- **Related Use Cases:** UC-STU-015, UC-REP-014.
- **Acceptance Criteria:**
  - Given a verified request, When run, Then only the requesting subject's data is compiled.
  - Given a guardian request, When run, Then it covers only their child (children-only).
  - Given fulfillment, When complete, Then the access is audited.
- **Edge Case Analysis:**
  - *Invalid Input:* unverified request rejected.
  - *Permission Failure:* unauthorized handler → denied.
  - *Concurrent Update:* data change during compile → as-of consistent.
  - *Duplicate Data:* idempotent.
  - *System Failure:* resumable; complete or fail cleanly.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* subject-access compile; guardian-child.
  - *Negative:* mixed-data inclusion (must not occur); unverified request.
  - *Boundary:* subject with data across many modules.

---

## 7. Compact Specifications (routine use cases)

- **UC-REP-004 — Mask Sensitive Data in Reports** · *High* · Rules: REP-003, STU-005, STF-007. Mask/omit sensitive fields per role need-to-know in reports/exports. *Edge:* same masking as direct access. *QA:* masking per role; need-to-know unmask.
- **UC-REP-005 — Expose Projection Freshness (as-of)** · *Medium* · Rules: REP-004. State as-of time; consistency-critical checks use source. *Edge:* stale projection flagged. *QA:* as-of shown; critical-from-source.
- **UC-REP-006 — Manage Report Definitions** · *High* · Rules: REP-001, CFG-004. Configure fields/filters/aggregations (consistent with source rules, versioned). *Permissions:* `report.definition.manage`. *Edge:* aggregations match source semantics. *QA:* definition CRUD; consistency.
- **UC-REP-007 — Schedule Reports** · *Medium* · Rules: REP-007, REP-002. Schedule recurring reports (same scope/masking). *Edge:* scheduled runs scoped. *QA:* schedule; scope preserved.
- **UC-REP-008 — Approve Bulk-PII Export (SoD)** · *High* · Rules: REP-005, AUTHZ-009, WFL-004. Approver gates bulk-PII exports. *Edge:* SoD; minors'/financial data tightly gated. *QA:* approval gates; self-approval blocked.
- **UC-REP-009 — Search Reports** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-REP-010 — Cross-Institute Summary (org-only)** · *Medium* · Rules: REP-006, INST-006. Org-level multi-institute reporting; deployment-bound. *Edge:* institute admin denied; never cross-deployment. *QA:* org-only; cross-deployment blocked.
- **UC-REP-011 — Bulk Report Generation** · *Medium* · Rules: REP-007. Bulk-generate (e.g., per-class report cards) async. *Edge:* resumable; scoped per item. *QA:* bulk run; scope per item.
- **UC-REP-012 — Export Report (signed-URL delivery)** · *High* · Rules: REP-005, FILE-005/007. Deliver exports/generated docs via short-lived signed URL. *Edge:* immutable generated docs; link expiry. *QA:* signed-URL; expiry; immutability.
- **UC-REP-013 — Export-Approval Workflow** · *Medium* · Rules: WFL-002/004, REP-005. Version-pinned bulk-PII export approval. *QA:* pinning; SoD; escalation.
- **UC-REP-014 — Report Access & Export Audit** · *High* · Rules: REP-008, AUD. Audit report runs and exports (exfiltration accountability). *Edge:* who exported what/when. *QA:* audit completeness.
- **UC-REP-015 — Scope-Widening Parameter Constrained (Exception)** · *High* · Rules: REP-002. Parameters reaching beyond scope are constrained, never widened. *QA:* constrained to authorized; no leakage.
- **UC-REP-016 — Cross-Deployment Report Blocked (Exception)** · *Critical* · Rules: REP-006, CFG-011. Cross-deployment reporting structurally impossible. *QA:* blocked; within-deployment allowed.

## 8. Module-level QA & Edge Themes
- **Never bypass authorization (REP-002):** the single most important suite — every report/export applies scope + ownership; parameters narrow, never widen; no out-of-scope leakage.
- **Export governance (REP-005 / REP-008):** bulk PII gated/approved; every export audited as exfiltration-sensitive.
- **Masking (REP-003):** sensitive/minor/financial fields masked per need-to-know in reports and exports.
- **Freshness transparency (REP-004):** as-of shown; consistency-critical checks read source.
- **Isolation (REP-006):** cross-institute is org-only; cross-deployment impossible.
