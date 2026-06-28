# 04 — Campus Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`CAMP-001…007`), and Use Case Repository (`UC-CAMP-001…015`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Manage campuses/branches within an institute — the second scope level: campus CRUD, default-campus guarantee, within-institute code uniqueness, campus-level config override, inter-campus transfer, and dependency-safe deactivation/archival.

**Business Goal.** Let a single institute operate multiple physical branches with correct per-campus scoping, configuration, and student movement, while preventing data loss or orphaned records.

**Scope.** Campus CRUD; belongs-to-exactly-one-institute; guaranteed default campus; code uniqueness within institute; campus scope filtering; campus-level config override; inter-campus student transfer (single + bulk); deactivate/archive only with no active dependents; blocked cross-institute reparenting.

**Out of Scope.** Institute lifecycle (Institute module). Student records themselves (Student module — this module moves their campus association via transfer). Class/section structure (Academic modules). Config mechanics (Configuration Engine).

---

## 2. Actors

**Primary Actors.** Institute Administrator (manages campuses), Campus Administrator (manages own campus), Registrar (executes transfers), System (default-campus guarantee, scope enforcement, dependency checks).

**Secondary Actors.** Institute module (parent scope), Student/Enrollment modules (transfer subjects), Configuration Engine (overrides), Workflow Engine (closure/transfer approval), Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Create campus | Create a campus under exactly one institute with a unique code within that institute. | Critical | Institute |
| FR-002 | Default campus guarantee | Every institute always has exactly one default campus. | High | Institute activation |
| FR-003 | Campus scope filtering | Scope data access to campus where applicable (below institute). | Critical | Authorization |
| FR-004 | Campus config override | Allow campus-level overrides (most-specific-wins below institute). | High | Configuration Engine |
| FR-005 | Inter-campus transfer | Move a student between campuses within the institute as a governed transfer (history preserved). | High | Student/Enrollment, Workflow |
| FR-006 | Bulk transfer | Transfer a cohort between campuses with per-record validation. | Medium | FR-005 |
| FR-007 | Dependency-safe deactivation | Block deactivation/archival while active dependents (students/staff/classes) exist. | High | Student/Class/Staff |
| FR-008 | No cross-institute reparenting | A campus cannot move to another institute. | High | Institute scope |
| FR-009 | Last-campus protection | Prevent deactivating the institute's only/last active campus. | High | FR-002 |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Campus lifecycle | Create → active → deactivate/archive (dependency-checked). | Branch management without orphans. |
| Default campus | Always-present default. | No "campus-less" records. |
| Campus overrides | Branch-specific configuration. | Local flexibility within institute policy. |
| Inter-campus transfer | Governed student movement, history-preserving. | Accurate multi-branch operations. |
| Bulk transfer | Cohort moves with validation. | Efficient reorganizations. |
| Dependency guard | Block unsafe deactivation. | Data integrity; no lost associations. |

---

## 5. Screens

Campus List; Campus Detail; Campus Create; Campus Edit; Campus Config Overrides; Inter-Campus Transfer (single); Bulk Student Transfer; Deactivate/Archive Campus (confirm + dependency report); Campus Roster & Branch Dashboard; Campus Import; Campus Export; Transfer Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Campus List | Create, Open, Search/Filter, Export | Bulk Import |
| Campus Detail | Edit, Set as Default, Deactivate/Archive, Manage Overrides, View Roster | — |
| Campus Create/Edit | Save, Cancel | — |
| Config Overrides | Add Override, Edit, Reset to Inherited, Save (versioned) | — |
| Inter-Campus Transfer | Select Student, Target Campus, Reason, Submit (→ approval) | — |
| Bulk Transfer | Select Cohort, Target Campus, Validate, Submit, Download Errors | Bulk Transfer |
| Deactivate/Archive | Run Dependency Check, Confirm with Reason, Submit (→ approval) | — |
| Branch Dashboard | Filter, Export | Export |

---

## 7. Forms

**Create Campus** — `name` (text, required), `code` (text, required, unique within institute), `address`/`contact` (text), `isDefault` (toggle, default false). Validation: code unique within institute (CAMP-003); belongs to exactly one institute (CAMP-001); first campus auto-set default (CAMP-002).

**Config Override** — `definitionKey` (select), `value` (typed), `scope` = this campus. Validation: registered/typed/scope-allowed/versioned (CFG-001/002/003/004); campus override beats institute (CAMP-005).

**Inter-Campus Transfer** — `student` (search-select, required), `targetCampus` (select, required, same institute), `effectiveDate` (date, required), `reason` (text, required). Validation: target ≠ source; same institute (no reparenting — CAMP-006); routed to approval; history preserved.

**Deactivate/Archive** — `reason` (text, required). Validation: no active dependents (CAMP-007); not the last active campus (CAMP-002).

---

## 8. Search & Filter Requirements

**Campuses:** by name, code, status (active/inactive/archived), default flag. **Transfers:** by student, source/target campus, status, date. Sorting: name/code/status. Pagination: server-side, 25 default. Campus admins see only their campus; institute admins see all campuses in the institute.

---

## 9. Table Requirements

**Campus table:** Name, Code, Default?, Status, #Students, #Staff, Created. Sorting on Name/Code/Status. Filtering as above. Export (governed). **Transfer table:** Student, From, To, Effective Date, Status, Approver. Bulk transfer supported from cohort selection.

---

## 10. Workflow Requirements

**Trigger events:** create, set-default, override publish, transfer request, deactivate/archive request. **Status changes:** Campus `ACTIVE → INACTIVE → ARCHIVED`; Transfer `REQUESTED → APPROVED/REJECTED → COMPLETED`. **Approvals:** campus closure (deactivate/archive) and transfers via Workflow Engine. **Notifications:** transfer requested/approved/completed (to guardians of affected students — custody-aware), campus deactivation. **Audit:** lifecycle + transfers with reason/actor (immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| View campus(es) | `campus.view` |
| Create campus | `campus.create` |
| Update campus | `campus.update` |
| Set default campus | `campus.default.set` |
| Deactivate/archive | `campus.deactivate` |
| Manage config overrides | `campus.config.manage` |
| Transfer student(s) | `campus.transfer.execute` |
| Approve closure/transfer | `campus.transfer.approve` |
| Import/Export | `campus.import`, `campus.export` |

All campus capabilities are scope-bound below the institute (CAMP-004).

---

## 12. Business Rule References

CAMP-001 (belongs to exactly one institute), CAMP-002 (default campus guarantee), CAMP-003 (code uniqueness within institute), CAMP-004 (campus scope filtering), CAMP-005 (campus-level config override), CAMP-006 (inter-campus movement is a transfer), CAMP-007 (deactivate/archive requires no active dependents). Cross-cutting: INST-006 (institute scope parent), CFG-003 (most-specific-wins), AUTHZ-002/003, WFL-004, AUD-001, NOT-002 (custody-aware transfer notices).

## 13. Use Case References

UC-CAMP-001 (Create), UC-CAMP-002 (View), UC-CAMP-003 (Update), UC-CAMP-004 (Deactivate/Archive), UC-CAMP-005 (Inter-Campus Transfer), UC-CAMP-006 (Approve Closure/Transfer), UC-CAMP-007 (Manage Config Overrides), UC-CAMP-008 (Search), UC-CAMP-009 (Roster & Branch Dashboard), UC-CAMP-010 (Bulk Transfer), UC-CAMP-011/012 (Import/Export), UC-CAMP-013 (Transfer Approval Workflow), UC-CAMP-014 (Last-Campus Deactivation Blocked), UC-CAMP-015 (Cross-Institute Reparenting Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Create campus | POST | Institute Admin |
| List / get campus(es) | GET | Admin |
| Update campus | PUT | Admin |
| Set default campus | POST | Institute Admin |
| Deactivate / archive (dependency-checked) | POST | Admin (→ approval) |
| Get/set campus config overrides | GET/PUT | Campus/Institute Admin |
| Transfer student (single) | POST | Registrar (→ approval) |
| Bulk transfer | POST | Registrar (→ approval) |
| Campus roster / dashboard | GET | Admin |
| Import / export campuses | POST/GET | Admin |

---

## 15. Database Requirements

**Entities:** `Campus` (instituteId, code, status, isDefault), `CampusConfigOverride` (via Config), `CampusTransfer` (studentId, fromCampus, toCampus, effectiveDate, status, history). **Relationships:** Institute 1—* Campus; Campus 1—* Student (association); Campus referenced by scoped entities. **Indexes:** unique(Campus.code, instituteId), index(Campus.instituteId, status), partial-unique(isDefault true per institute), index(CampusTransfer.studentId), index(CampusTransfer.status). Reparenting (changing instititeId) is disallowed at the data layer (CAMP-006).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Transfer approved/completed to financial-responsible/custody-permitted guardian; campus deactivation notice. |
| In-App | Transfer requests/approvals; dependency-block warnings. |
| SMS/Push | Optional transfer confirmation to guardians. |

Transfer notifications are custody-aware (NOT-002) — restricted parties excluded.

---

## 17. Audit Requirements

Log: campus create/update, set-default changes, config override publishes (before/after, version), transfers (student, from/to, approver, reason), deactivation/archival (with dependency-check result). Record who/when/before/after. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Campus roster, Branch dashboard (headcounts, capacity), Transfer history, Config-override drift. **Exports:** campus list/roster (governed). **Dashboards:** institute-level multi-campus overview.

---

## 19. Error Handling

**Validation:** duplicate code within institute, transfer to different institute, deactivation with dependents or last-campus → specific errors (UC-CAMP-014/015). **Permission:** campus admin acting on another campus → 403/not-found. **Workflow:** transfer/closure pending approval → clear state. **System:** dependency-check service unavailable → block deactivation (fail safe).

---

## 20. Edge Cases

**Concurrent updates:** two transfers for one student → serialized; one completes, other re-validates. **Duplicate data:** duplicate campus code → blocked. **Partial failures:** bulk transfer partial → per-record report, completed rows stand, failed rows reported. **Rollback:** transfer approval reversed before completion → student remains at source. **Default race:** deactivating default while setting a new default → new default must be set first (guarantee preserved).

---

## 21. Acceptance Criteria

**Functional.** A campus belongs to exactly one institute and cannot be reparented; campus code is unique within its institute; every institute always has exactly one default campus; campus-level config overrides win over institute defaults; student movement between campuses is a governed, history-preserving transfer; a campus with active dependents (or the last active campus) cannot be deactivated.

**Business.** Multi-branch institutes operate with correct per-campus scoping and configuration; student transfers are accurate, governed, and auditable; no record is ever orphaned by a campus change.

---

## 22. Future Enhancements

Campus capacity/seat planning; geo/mapping and catchment areas; per-campus calendar/timing overrides; campus-to-campus resource sharing (staff/rooms); transfer-reason analytics; self-service guardian-initiated transfer requests (governed).
