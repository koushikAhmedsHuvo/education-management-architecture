# 22 — Staff Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`STF-001…008`), and Use Case Repository (`UC-STF-001…018`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Maintain the base personnel record (Teacher specializes it): unique employee identity, user-account linkage, duplicate detection, employment-status-driven access, institute/campus scoping with multi-assignment, the reporting hierarchy (which feeds workflow approver resolution), sensitive-data protection, and archive-not-delete with handover on separation.

**Business Goal.** Hold one accurate, access-controlled record per employee whose status governs system access and whose reporting line drives every approval that routes through management.

**Scope.** Staff CRUD (immutable employee ID, dedup); user-account linkage/invitation; employment-status lifecycle → access; institute/campus scope + multi-assignment; reporting hierarchy; sensitive-data masking; separation + archival/anonymization with mandatory handover.

**Out of Scope.** Teacher specialization specifics (Teacher module). Payroll/salary/department structure (HR module). Leave (Leave module). Authentication mechanics (Authentication — Staff drives status→access via revocation). Config of custom fields (Configuration Engine).

---

## 2. Actors

**Primary Actors.** HR Administrator (manages staff), Institute/Campus Administrator (scoped oversight), Staff Member (self-service profile), System (dedup, status→access, scoping, hierarchy).

**Secondary Actors.** Authentication/Authorization (account linkage, access, scope), Configuration Engine (custom fields), File module (documents), Workflow Engine (separation/sensitive-change approval), Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Unique employee identity (dedup) | Create one record per employee with an immutable employee ID. | Critical | Config (ID scheme) |
| FR-002 | User-account linkage | Link/invite a user account for staff. | High | Authentication |
| FR-003 | Status-driven access | Employment status governs access; suspend/separate revokes immediately. | Critical | Authentication (AUTH-005) |
| FR-004 | Scope & multi-assignment | Scope to institute(s)/campus(es) with multi-assignment. | High | Authorization (AUTHZ-007) |
| FR-005 | Reporting hierarchy | Model the manager/reporting line (feeds approver resolution). | High | Workflow (WFL-004) |
| FR-006 | Sensitive-data protection | Mask sensitive staff data (salary/national ID) by need-to-know. | Critical | Authorization, Reporting |
| FR-007 | Separation & archival | Separate with handover, revoke access, archive/anonymize per retention. | High | Teacher (handover), Retention |
| FR-008 | Archive-not-delete | Never hard-delete staff records. | High | Audit/Retention |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Staff master record | One immutable record per employee. | Single source of truth. |
| Account linkage | Invite-based activation. | Secure onboarding. |
| Status→access | Status governs access. | Security control. |
| Scope & multi-assignment | Institute/campus scoping. | Multi-branch staffing. |
| Reporting hierarchy | Manager line. | Drives approvals. |
| Sensitive-data masking | Need-to-know. | Privacy. |
| Separation + handover | Governed exit. | No orphaned ownership. |
| Archive-not-delete | Retention-aware. | Compliance. |

---

## 5. Screens

Staff List; Staff Detail (masked); Create Staff; Link/Invite Account; Employment Status; Scope & Multi-Assignment; Reporting Hierarchy; Staff Profile Edit (controlled); Staff Documents; Separation & Archive; Staff Directory & Headcount Report; Bulk Staff Update; Staff Import; Staff Export; Sensitive-Change/Separation Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Staff List | Create, Open, Search/Filter, Export | Bulk Update, Import |
| Staff Detail | Edit, Link Account, Change Status, Manage Scope, Set Manager, Separate | — |
| Employment Status | Confirm/Suspend/Reinstate (status→access) | — |
| Scope & Assignment | Add/Remove Scope, Multi-Assign | — |
| Reporting Hierarchy | Set Manager, Validate (no cycle) | — |
| Documents | Upload, Download (signed URL), Delete (governed) | — |
| Separation & Archive | Run Handover/Clearance, Archive/Anonymize | — |
| Directory Report | Run, Filter, Export | Export |

---

## 7. Forms

**Create Staff** — core: `name` (text, required), `designation` (select), `employeeId` (auto, immutable), `joinDate` (date). Dynamic: config fields. Validation: dedup (STF-003); immutable ID (STF-001); scope set (STF-005); account linkage path (STF-002).

**Employment Status** — `status` (probation/active/suspended/on-leave/separated). Validation: valid transition; suspend/separate revokes access immediately (STF-004/AUTH-005).

**Reporting Hierarchy** — `manager` (search-select). Validation: no cycle; vacancy handling (STF-006); feeds WFL-004 approver resolution.

**Profile Edit** — official vs self-service fields; sensitive fields masked. Validation: controlled edit; sensitive changes → approval (STF-007).

**Separation** — `reason` (text, required), `effectiveDate` (date). Validation: handover of owned responsibilities complete (STF-008/TCH-006); clearance; archive-not-delete (STF-008/C-08).

---

## 8. Search & Filter Requirements

**Staff:** by name, employee ID, designation, status, scope/campus, manager, role. Sorting: name/ID/status. Pagination: server-side, 25 default. Scope-bound; sensitive fields masked.

---

## 9. Table Requirements

**Staff table:** Employee ID, Name, Designation, Status, Scope, Manager, Account. Sensitive columns masked (STF-007). Sorting on Name/ID/Status. Filtering as above. Export (governed, masked). Bulk: update, import.

---

## 10. Workflow Requirements

**Trigger events:** create, account link, status change, scope change, hierarchy change, separation. **Status changes:** `PROBATION → ACTIVE → SUSPENDED ⇄ ACTIVE → SEPARATED → ARCHIVED`. **Approvals:** sensitive changes and separation via Workflow Engine. **Notifications:** account invite, status change, separation. **Audit:** all status/scope/hierarchy/separation changes (before/after, immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| View staff (masked) | `staff.view` |
| Create staff | `staff.manage` |
| Link account | `staff.account.link` |
| Manage status | `staff.status.manage` |
| Manage scope/assignment | `staff.scope.manage` |
| Manage reporting hierarchy | `staff.hierarchy.manage` |
| Separate & archive | `hr.separation.manage` |
| Approve sensitive change/separation | `staff.change.approve` |
| Import/export | `staff.import`, `staff.export` |

Status→access is security-critical; sensitive fields need-to-know.

---

## 12. Business Rule References

STF-001 (unique employee ID), STF-002 (staff–user account linkage), STF-003 (duplicate-staff detection), STF-004 (employment-status lifecycle drives access), STF-005 (institute/campus scoping & multi-assignment), STF-006 (reporting hierarchy), STF-007 (sensitive staff-data protection), STF-008 (archive/anonymize, never hard-delete; handover on separation). Cross-cutting: AUTH-005/011 (revocation/invite), AUTHZ-007 (scope authority), TCH-006 (handover), WFL-004 (approver resolution), AUD-008/C-08 (retention), CFG-010, AUD-001.

## 13. Use Case References

UC-STF-001 (Create — dedup, employee ID), UC-STF-002 (Link/Invite Account), UC-STF-003 (Manage Status — status→access), UC-STF-004 (Scope & Multi-Assignment), UC-STF-005 (Reporting Hierarchy), UC-STF-006 (View — masked, scoped), UC-STF-007 (Update — controlled), UC-STF-008 (Separate & Archive), UC-STF-009 (Documents), UC-STF-010 (Approve Sensitive Change/Status), UC-STF-011 (Search), UC-STF-012 (Directory & Headcount), UC-STF-013 (Bulk Update), UC-STF-014 (Import), UC-STF-015 (Export — governed, masked), UC-STF-016 (Separation Workflow), UC-STF-017 (Duplicate Staff Detected), UC-STF-018 (Hard-Delete Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Create staff (dedup) | POST | HR Admin |
| Get / list staff (masked, scoped) | GET | Authorized roles |
| Link / invite user account | POST | HR Admin |
| Change employment status | POST | HR Admin |
| Manage scope / multi-assignment | PUT | HR Admin |
| Set reporting manager | PUT | HR Admin |
| Update staff profile (controlled) | PUT | HR Admin / Staff |
| Separate & archive (handover) | POST | HR Admin (→ approval) |
| Manage documents | POST | HR Admin |
| Directory & headcount report | GET | Admin |
| Import / export staff | POST/GET | Admin |

Suspension/separation revokes access immediately (STF-004/AUTH-005); records are archived, never deleted (STF-008).

---

## 15. Database Requirements

**Entities:** `Staff` (immutable employeeId, core, status), `StaffAccountLink` (userId), `StaffScope` (institute/campus, multi), `ReportingLine` (staffId, managerId), `StaffDocument`, `StaffSeparation` (handover, clearance), `StaffDynamicField`. **Relationships:** Staff 1—1 Account; Staff *—* Scope; Staff *—1 Manager (ReportingLine); Staff 1—* Document. **Indexes:** unique(Staff.employeeId), dedup index(name, dob, nationalId), index(Staff.status, scope), index(ReportingLine.managerId). Status→access at data/auth layer (STF-004); archive-not-delete (STF-008); hierarchy cycle check (STF-006).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Account invitation; status change; separation/clearance. |
| In-App | Sensitive-change approvals; scope/hierarchy updates. |
| SMS/Push | Optional status alerts. |

Security/separation notices are mandatory-category (NOT-003).

---

## 17. Audit Requirements

Log: create, account link, status changes (+ access revocation), scope/hierarchy changes, profile changes (before/after), document add/delete, separation (handover, clearance, archive/anonymize). Record who/when/before/after. Status→access and separation are first-class audit events. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Staff directory, Headcount (by designation/scope/status), Reporting-hierarchy/org chart, Separation/attrition. **Exports:** governed masked staff export. **Dashboards:** workforce overview (headcount, vacancies, pending separations).

---

## 19. Error Handling

**Validation:** duplicate staff, hard-delete attempt, hierarchy cycle → specific errors (UC-STF-017/018). **Permission:** unmask without need-to-know → masked; out-of-scope → 403/not-found. **Workflow:** sensitive change/separation pending → clear state. **System:** account-service unavailable → create allowed, linkage deferred; separation with pending ownership → blocked until handover (TCH-006).

---

## 20. Edge Cases

**Concurrent updates:** two profile edits → versioned; sensitive change pends approval. **Duplicate data:** dedup match → link, not duplicate. **Partial failures:** bulk update partial → per-row report. **Rollback:** separation reversed pre-archive → reinstated, history intact. **Status race:** status change + active session → revocation wins (denylist).

---

## 21. Acceptance Criteria

**Functional.** Each employee has a unique immutable ID with duplicates prevented; employment status governs access (suspend/separate revokes immediately); staff are scoped with multi-assignment; the reporting hierarchy is cycle-free and feeds approver resolution; sensitive data is masked by need-to-know; separation mandates handover and archives/anonymizes (never hard-deletes).

**Business.** One accurate, access-controlled record per employee; access always reflects employment status; the reporting line reliably drives approvals; separations are clean, handover-complete, and retention-compliant.

---

## 22. Future Enhancements

Org-chart visual editor; self-service profile with verification; configurable onboarding checklists; skills/competency registry; multi-manager/matrix reporting; staff-photo/ID-card integration; contract/visa expiry tracking.
