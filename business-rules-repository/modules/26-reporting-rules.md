# 26 — Reporting Business Rules

## 1. Module Purpose
Govern **reporting, dashboards, and exports** — turning operational data into insight and documents. Reports read from **projections/read models** (eventual-consistency-aware) and must **never bypass the three-layer authorization** (scope + ownership) that protects every record — reporting is a classic data-exposure vector, so this module makes scope-safety and export governance explicit. It covers standard and ad-hoc reports, scheduled reports, large async exports (delivered via signed URLs, Doc 30), sensitive-data masking, cross-institute (org-only, never cross-deployment) reporting, and full audit of access and export.

## 2. Actors
- **Administrator / Coordinator** — runs and schedules reports within scope.
- **Teacher / Staff** — run reports limited to their ownership (own classes/sections/subjects).
- **Finance / HR roles** — financial/payroll reports per need-to-know.
- **Student / Guardian** — own data reports (e.g., a child's report card, fee statement).
- **System** — enforces scope/ownership in every report, masks sensitive data, runs exports async, and audits.

## 3. Use Cases

**Use Case ID:** UC-REP-01 — Run a Scoped Report
**Actors:** Administrator / Teacher, System
**Description:** Generate a report constrained to the requester's authorized scope and ownership.
**Preconditions:** Report defined; requester authorized.
**Main Flow:** 1) Requester selects a report and parameters. 2) The engine applies the **mandatory scope filter and ownership predicates** to the underlying query (AUTHZ-002/003), so the report can only contain authorized data. 3) Sensitive fields are masked per role. 4) Report renders with an as-of timestamp (projection freshness).
**Alternative Flow:** A1) Org admin runs a cross-institute report (still within one deployment, REP-006).
**Exception Flow:** E1) Requester lacks the report permission → denied. E2) Parameters reaching beyond scope → silently constrained to authorized scope, never widened.
**Post Conditions:** Authorized, masked report produced; access audited.
**Business Rules Applied:** REP-001, REP-002, REP-003, REP-004.

**Use Case ID:** UC-REP-02 — Large / Scheduled Export
**Actors:** Administrator, System
**Description:** Export a large dataset or schedule recurring reports.
**Preconditions:** Export permission; export governance satisfied.
**Main Flow:** 1) Requester triggers an export (or schedule). 2) System runs it asynchronously, applying scope/ownership/masking, and stores the output privately. 3) A short-lived signed URL (Doc 30) delivers it. 4) The export (who/what/when) is audited as a potential data-exfiltration event.
**Exception Flow:** E1) Bulk PII export beyond authority → blocked/approval-gated (REP-005). E2) Output too large/slow → batched/streamed.
**Post Conditions:** Export delivered securely; fully audited.
**Business Rules Applied:** REP-005, REP-007.

**Use Case ID:** UC-REP-03 — Data-Subject Report (Access Request)
**Actors:** Administrator / Guardian, System
**Description:** Produce a subject's own data for a right-to-access request.
**Preconditions:** Verified data-subject request.
**Main Flow:** 1) Authorized handler runs the subject-data report for the student/guardian/staff. 2) System compiles the subject's data across modules, scoped to that subject. 3) Delivered securely; the access request is audited.
**Exception Flow:** E1) Mixed data (other subjects) → only the requesting subject's data included.
**Post Conditions:** Subject-access report produced; audited.
**Business Rules Applied:** REP-002, REP-008.

## 4. Business Rules

**Rule ID:** REP-001
**Rule Name:** Configurable, Parameterized Report Definitions
**Description:** Reports are defined and parameterized configurably; outputs are consistent with source business rules.
**Priority:** High
**Category:** Configuration / correctness
**Preconditions:** Report use.
**Business Rule:** Report definitions (fields, filters, aggregations, formats) are configurable; aggregations honor the same rules as source modules (e.g., attendance % per ATT-009, results per RES, dues per FEE) so reports never contradict authoritative computations. Config-version awareness keeps historical reports faithful.
**System Action:** Resolve report definitions; compute consistently with source rules.
**Validation:** Definition valid; aggregations consistent with source semantics.
**Failure Behavior:** Block reports whose computation would diverge from source rules.
**Audit Requirement:** Log report definition changes.
**Example Scenario:** An attendance report uses the same instructional-day basis as ATT-009, not an ad-hoc count.
**Related Rules:** ATT-009, RES, FEE-007, CFG-004.

**Rule ID:** REP-002
**Rule Name:** Reports Never Bypass Authorization (Scope + Ownership)
**Description:** Every report applies the mandatory scope filter and ownership predicates; reporting can never expose data the requester could not access directly.
**Priority:** Critical
**Category:** Security (primary exposure vector)
**Preconditions:** Any report execution.
**Business Rule:** Report queries are subject to the same three-layer authorization as direct access (AUTHZ-001/002/003): institute/campus scope and role ownership constrain the data; a teacher's report shows only their students; an institute admin only their institute. Parameters can narrow but never widen beyond authorized scope.
**System Action:** Inject scope filter + ownership predicates into every report query.
**Validation:** Active scope/ownership applied; no unconstrained queries.
**Failure Behavior:** Constrain to authorized data; deny if no permission; never widen.
**Audit Requirement:** Log report access with applied scope.
**Example Scenario:** A class teacher's "class performance" report cannot include another class's students.
**Related Rules:** AUTHZ-002, AUTHZ-003, REP-005.

**Rule ID:** REP-003
**Rule Name:** Sensitive-Data Masking in Reports
**Description:** Sensitive fields are masked or excluded in reports per role need-to-know.
**Priority:** High
**Category:** Privacy (minors/financial/staff)
**Preconditions:** A report includes sensitive fields.
**Business Rule:** Sensitive data (minors' identifiers, medical notes, staff salary, financial specifics) is masked or omitted unless the requester's role has explicit need-to-know; the same masking as direct access (STU-005, STF-007) applies in reports and exports.
**System Action:** Apply role-based masking to report fields.
**Validation:** Field sensitivity + role need-to-know evaluated.
**Failure Behavior:** Mask/omit on insufficient authorization.
**Audit Requirement:** Sensitive-field inclusion recorded where configured.
**Example Scenario:** A general staff report shows names but masks national IDs and salaries.
**Related Rules:** STU-005, STF-007, REP-005.

**Rule ID:** REP-004
**Rule Name:** Projection Freshness & As-Of Transparency
**Description:** Reports read from projections and state their data-freshness (as-of) to avoid misleading consumers.
**Priority:** Medium
**Category:** Correctness / transparency
**Preconditions:** Reporting from read models.
**Business Rule:** Reports read from projections that may lag source writes slightly; each report states an as-of timestamp; decisions requiring strict consistency (e.g., exam eligibility, financial close) use confirmed source data, not potentially-stale projections.
**System Action:** Read projections; stamp as-of; route consistency-critical checks to source.
**Validation:** Freshness exposed; critical checks use source.
**Failure Behavior:** Flag stale/unavailable projections rather than show silently-wrong data.
**Audit Requirement:** As-of recorded on generated reports where material.
**Example Scenario:** A live dashboard notes "as of 2 minutes ago," while exam eligibility reads confirmed data.
**Related Rules:** RES-009, ATT-009, architecture (CQRS projections).

**Rule ID:** REP-005
**Rule Name:** Export Governance & Bulk-PII Controls
**Description:** Exports (especially bulk PII) are governed, possibly approval-gated, and audited as exfiltration-sensitive.
**Priority:** Critical
**Category:** Data protection
**Preconditions:** An export, especially of personal data.
**Business Rule:** Exports apply scope/ownership/masking; bulk personal-data exports may require elevated permission/approval; every export records who exported what, when, and the scope (data-exfiltration accountability). Exports of minors'/financial/staff data are tightly controlled.
**System Action:** Enforce scope/masking; gate bulk PII; audit thoroughly.
**Validation:** Export authority sufficient; approval where required.
**Failure Behavior:** Block over-authority bulk exports; never export unmasked sensitive data without need-to-know.
**Audit Requirement:** Log `REPORT_EXPORTED` with requester, scope, dataset, row count.
**Example Scenario:** Exporting the full student directory requires elevated permission and is fully logged.
**Related Rules:** REP-002, REP-003, Doc 30 (delivery), Compliance.

**Rule ID:** REP-006
**Rule Name:** Cross-Institute Reporting Is Org-Only and Deployment-Bound
**Description:** Multi-institute reports are an organization-admin capability and never cross client deployments.
**Priority:** High
**Category:** Isolation
**Preconditions:** A report spanning institutes.
**Business Rule:** Reports spanning multiple institutes require org-level authority and remain within the single client deployment; cross-deployment reporting is structurally impossible.
**System Action:** Permit cross-institute aggregation only for org admins; enforce deployment boundary.
**Validation:** Org authority for cross-institute; deployment scope absolute.
**Failure Behavior:** Deny cross-institute to non-org roles; cross-deployment impossible.
**Audit Requirement:** Log cross-institute report access.
**Example Scenario:** An org admin compares enrollment across the client's three institutes; an institute admin cannot.
**Related Rules:** AUTHZ-002, INST-006, CFG-011.

**Rule ID:** REP-007
**Rule Name:** Asynchronous, Performance-Safe Large Reports
**Description:** Heavy reports/exports run asynchronously to protect interactive performance.
**Priority:** Medium
**Category:** Performance
**Preconditions:** A large/expensive report.
**Business Rule:** Large reports/exports run as background jobs (batched/streamed), delivered when ready via signed URL (Doc 30); interactive latency is protected; jobs are cancellable and progress-tracked.
**System Action:** Queue heavy jobs; stream/batch; deliver async; track progress.
**Validation:** Job sizing per tier; idempotent generation.
**Failure Behavior:** Retry/resume failed jobs; never block interactive workloads.
**Audit Requirement:** Log report-job runs.
**Example Scenario:** A year-end financial export runs in the background and notifies when its secure download is ready.
**Related Rules:** RES-009, Doc 30, NOT-005.

**Rule ID:** REP-008
**Rule Name:** Report Access & Export Auditing
**Description:** Running and exporting reports is audited (who accessed/exported what), supporting data-governance and breach investigation.
**Priority:** High
**Category:** Accountability
**Preconditions:** Report run/export.
**Business Rule:** Report execution and especially exports are audited with requester, parameters, scope, and (for exports) dataset/row count; this trail supports breach investigation and demonstrates lawful access to minors'/financial data.
**System Action:** Audit report runs and exports.
**Validation:** Audit captured with sufficient detail.
**Failure Behavior:** No silent report access to sensitive datasets.
**Audit Requirement:** Log `REPORT_RUN` and `REPORT_EXPORTED`.
**Example Scenario:** An investigation can show exactly who exported a student list and when.
**Related Rules:** REP-005, AUD, Compliance.

## 5. Validation Rules
- Report definitions valid; aggregations consistent with source-module rules.
- Every report applies scope + ownership; parameters narrow, never widen.
- Sensitive fields masked per role; bulk PII exports governed.
- Cross-institute reporting org-only, deployment-bound.
- Large reports async; freshness (as-of) exposed; runs/exports audited.

## 6. State Machine
*(Report-generation lifecycle.)*

**State Name:** REQUESTED
**Description:** Report/export requested with parameters.
**Allowed Transitions:** → GENERATING; → DENIED (authz).
**Forbidden Transitions:** output before authorization.
**System Actions:** Authorize; apply scope/ownership; size the job.

**State Name:** GENERATING
**Description:** Running (sync small / async large).
**Allowed Transitions:** → READY; → FAILED; → CANCELLED.
**Forbidden Transitions:** unscoped queries.
**System Actions:** Query projections with scope/masking; batch/stream.

**State Name:** READY
**Description:** Output available (rendered or as a secure download).
**Allowed Transitions:** → DELIVERED; → EXPIRED.
**Forbidden Transitions:** unauthorized access to output.
**System Actions:** Render / issue signed URL; record as-of.

**State Name:** DELIVERED / EXPIRED
**Description:** Output accessed, or its link expired.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** indefinite open links.
**System Actions:** Expire links; retain access record.

**State Name:** FAILED / CANCELLED / ARCHIVED
**Description:** Job failed/cancelled, or historical record.
**Allowed Transitions:** to ARCHIVED.
**Forbidden Transitions:** edits.
**System Actions:** Retain run/audit record.

## 7. Status Definitions
`REQUESTED` · `GENERATING` · `READY` · `DELIVERED` · `EXPIRED` · `FAILED` · `CANCELLED` · `ARCHIVED`. Report types: `STANDARD` · `AD_HOC` · `SCHEDULED` · `EXPORT` · `SUBJECT_ACCESS`.

## 8. Workflow Rules
- Bulk-PII exports may require approval (REP-005) via the Workflow Engine.
- Scheduled reports run on configured cadences with the same scope/masking.
- Large reports run async with secure delivery (REP-007).
- Subject-access requests follow a verified, audited handling path (REP-008).

## 9. Permission Rules
- `report.run` — run reports within scope/ownership.
- `report.export` — export data (scoped; bulk PII elevated).
- `report.schedule.manage` — manage scheduled reports.
- `report.definition.manage` — define/edit report definitions.
- `report.cross_institute` — org-level cross-institute reporting.
- All constrained by REP-002 (scope/ownership) regardless of report permission.

## 10. Notification Rules
- Async report/export ready → notify the requester with a secure link (Doc 30).
- Scheduled report delivery → notify recipients.
- Bulk-PII export approvals → notify approver/requester.
- Export of sensitive data → optional security-channel notice per policy.

## 11. Audit Requirements
Mandatory: `REPORT_RUN` (requester, parameters, scope, as-of), `REPORT_EXPORTED` (dataset, row count, scope), report-definition changes, cross-institute access, subject-access fulfillment, bulk-PII approvals. Exports are exfiltration-sensitive and fully attributable.

## 12. Data Retention Rules
- Report definitions retained as configuration; generated outputs retained per policy then expired/purged.
- Export audit retained long-term (data-governance evidence).
- Generated files follow file-retention (Doc 30); signed links are short-lived.
- Subject-access fulfillment records retained per compliance.

## 13. Edge Cases
- **Teacher report limited to ownership:** cannot include non-owned classes (REP-002).
- **Parameter trying to widen scope:** silently constrained to authorized scope (REP-002).
- **Report over a config change:** version-aware so historical figures stay faithful (REP-001/CFG-004).
- **Report of archived/anonymized data:** anonymized subjects appear without PII; archived data read-only.
- **Stale projection:** as-of shown; critical checks use source (REP-004).
- **Bulk export of minors' data:** governed/approval-gated/audited (REP-005).
- **Cross-institute by a non-org role:** denied (REP-006).
- **Huge export:** async/streamed; secure delivery (REP-007).
- **Subject-access mixed data:** only the requesting subject included (REP-008/UC-REP-03).

## 14. Failure Scenarios
- **Unscoped/over-scoped query:** prevented; scope/ownership always applied (REP-002).
- **Unmasked sensitive export:** blocked without need-to-know (REP-003/REP-005).
- **Cross-deployment report:** structurally impossible (REP-006).
- **Projection unavailable:** flag rather than show wrong data (REP-004).
- **Large job overload:** async/backpressure protects interactive use (REP-007).

## 15. Exception Handling Rules
- Authorization (scope + ownership) is applied to every report and export, unconditionally.
- Sensitive data is masked/omitted absent need-to-know.
- Exports are governed and fully audited; freshness is transparent.
- Heavy jobs degrade gracefully (async), never blocking interactive workloads.

## 16. Compliance Considerations
- **Data protection:** reporting cannot become an authorization backdoor; exports of minors'/financial/staff data are governed and audited.
- **Right of access:** subject-data reports support data-subject access requests.
- **Accountability:** export auditing supports breach investigation and lawful-access demonstration.
- **Isolation:** cross-deployment reporting is impossible; cross-institute is org-bounded.

## 17. Future Considerations
- Self-service analytics with enforced row-level security.
- Privacy-preserving aggregate analytics (differential privacy) for cohort insights.
- Verifiable, watermarked official report documents.
- Real-time operational dashboards with consistency-aware indicators.
