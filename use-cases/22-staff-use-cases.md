# 22 — Staff Use Cases

Transforms the Staff business rules (`STF-001`…`STF-008`) into use cases. The base personnel entity (Teacher specializes it): unique employee identity, user-account linkage, dedup, status-driven access, institute/campus scoping with multi-assignment, reporting hierarchy, sensitive-data protection, and archive-not-delete with handover on separation.

## 1. Primary Actors
HR Administrator (manages staff records), Institute/Campus Administrator (scoped staff oversight), Staff Member (self-service profile).

## 2. Secondary Actors
System (identity, dedup, status→access, scoping, hierarchy), Auth/Authz modules (account linkage, access), Configuration Engine (custom fields), File module (documents), Audit service.

## 3. Goals
Maintain one record per employee with an immutable employee ID; link to a user account; prevent duplicates; drive access from employment status; scope to institutes/campuses with multi-assignment; model the reporting hierarchy (which feeds workflow approver resolution); protect sensitive staff data; archive/anonymize with handover, never hard-delete.

## 4. User Journeys
- **Onboard:** HR creates a staff record (employee ID, designation, scope) → links/invites a user account → status `PROBATION`/`ACTIVE` enables access.
- **Operate:** staff is scoped to institute(s)/campus(es), possibly multi-assigned; the reporting hierarchy determines approvers for leave/payroll.
- **Maintain:** profile updates (self/HR), sensitive data masked; reporting line and scope adjusted with audit.
- **Separate:** on departure, access is revoked, a handover is mandated, and the record is archived/anonymized per retention — never deleted.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core/CRUD | UC-STF-001 | Create Staff Record (dedup, employee ID) | High |
| Core | UC-STF-002 | Link / Invite User Account | High |
| Core | UC-STF-003 | Manage Employment Status (status→access) | High |
| Core | UC-STF-004 | Manage Scope & Multi-Assignment | High |
| Core | UC-STF-005 | Manage Reporting Hierarchy | High |
| CRUD | UC-STF-006 | View Staff Profile (masked, scoped) | High |
| CRUD | UC-STF-007 | Update Staff Profile (controlled) | Medium |
| Core | UC-STF-008 | Separate & Archive (handover, anonymize) | High |
| Admin | UC-STF-009 | Manage Staff Documents | Medium |
| Approval | UC-STF-010 | Approve Sensitive Change / Status | Medium |
| Search | UC-STF-011 | Search Staff (scoped) | Medium |
| Reporting | UC-STF-012 | Staff Directory & Headcount Report | Medium |
| Bulk | UC-STF-013 | Bulk Staff Update | Medium |
| Import | UC-STF-014 | Import Staff | Medium |
| Export | UC-STF-015 | Export Staff Data (governed, masked) | High |
| Workflow | UC-STF-016 | Separation Workflow | Medium |
| Exception | UC-STF-017 | Duplicate Staff Detected | High |
| Exception | UC-STF-018 | Hard-Delete Blocked | Critical |

---

## 6. Detailed Specifications (high-value use cases)

### UC-STF-001 — Create Staff Record (dedup, employee ID)
- **Module:** Staff · **Priority:** High
- **Actors:** HR Administrator (primary), System
- **Goal:** Create exactly one record per employee with a permanent employee ID.
- **Description:** Runs duplicate detection, assigns an immutable employee ID, captures core + custom fields, and sets institute/campus scope.
- **Business Rules Applied:** STF-001, STF-003, STF-005, STF-002.
- **Preconditions:** Authorized HR; required core fields available.
- **Trigger:** HR creates a staff record.
- **Main Success Scenario:**
  1. System runs duplicate-staff detection (STF-003).
  2. If no match, System creates the record with an immutable employee ID (STF-001) and validated fields.
  3. System sets institute/campus scope (STF-005).
  4. A user account is linked/invited (STF-002, UC-STF-002).
- **Alternative Flows:** A1) Match → link to existing record (UC-STF-017). A2) Rehire → new employment linked to history (HR-008).
- **Exception Flows:** E1) Duplicate → link/merge, not create. E2) Invalid core field → reject.
- **Validation Rules:** Dedup performed; immutable ID; scope valid; account linkage (STF-001/002/003/005).
- **Permissions Required:** `staff.manage`.
- **Notifications Triggered:** Account invitation (UC-STF-002).
- **Audit Events Generated:** `STAFF_CREATED` / `STAFF_LINKED`.
- **Data Created:** Staff record; employee ID.
- **Data Updated:** None.
- **Data Deleted:** None.
- **Post Conditions:** One canonical staff record with a permanent ID and scope.
- **Related Use Cases:** UC-STF-002, UC-STF-017, UC-TCH-001.
- **Acceptance Criteria:**
  - Given attributes matching no existing staff, When created, Then a record with an immutable employee ID is produced.
  - Given a match, When creation is attempted, Then the existing record is linked (no duplicate).
  - Given a rehire, When created, Then it is new employment linked to prior history.
- **Edge Case Analysis:**
  - *Invalid Input:* invalid core/custom field rejected.
  - *Permission Failure:* lacks `staff.manage` → 403.
  - *Concurrent Update:* two creations for one person → dedup yields one.
  - *Duplicate Data:* prevented by STF-003.
  - *System Failure:* atomic; ID never reused.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* new staff; rehire-linked.
  - *Negative:* duplicate; invalid field.
  - *Boundary:* fuzzy dedup threshold; ID uniqueness.

### UC-STF-003 — Manage Employment Status (status→access)
- **Module:** Staff · **Priority:** High
- **Actors:** HR Administrator (primary), System
- **Goal:** Drive system access from employment status transitions.
- **Description:** Status (probation/active/suspended/on-leave/separated) governs access; suspending/separating revokes access immediately (via Auth), reflecting STF-004.
- **Business Rules Applied:** STF-004, HR-002, AUTH-005.
- **Preconditions:** Staff record exists; authorized actor.
- **Trigger:** HR changes employment status.
- **Main Success Scenario:**
  1. HR sets a new status (e.g., probation→confirmed, or active→suspended).
  2. System applies status-driven access (STF-004): suspension/separation revokes access immediately (AUTH-005).
  3. Reporting/payroll/leave eligibility update accordingly.
- **Alternative Flows:** A1) Probation→confirmation via HR-002 (UC-HR).
- **Exception Flows:** E1) Separation → triggers handover + archive (UC-STF-008).
- **Validation Rules:** Valid status transition; access reflects status (STF-004).
- **Permissions Required:** `staff.manage` / `hr.manage`.
- **Notifications Triggered:** Status-change notice to the staff member/manager.
- **Audit Events Generated:** `STAFF_STATUS_CHANGED` (+ access revocation on suspend/separate).
- **Data Created:** None.
- **Data Updated:** Status; access; eligibility.
- **Data Deleted:** None.
- **Post Conditions:** Access consistent with status.
- **Related Use Cases:** UC-STF-008, UC-AUTH-013, UC-HR (confirmation).
- **Acceptance Criteria:**
  - Given a suspension, When applied, Then access is revoked immediately.
  - Given confirmation from probation, When applied, Then status and any gated capabilities update.
  - Given separation, When applied, Then handover and archival are triggered.
- **Edge Case Analysis:**
  - *Invalid Input:* invalid transition rejected.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* status change + active session → revocation wins (denylist).
  - *Duplicate Data:* idempotent same-status set.
  - *System Failure:* status/access atomic.
  - *Workflow Failure:* separation workflow pends handover.
- **QA Coverage:**
  - *Positive:* confirm; suspend; reinstate.
  - *Negative:* invalid transition; access after suspension.
  - *Boundary:* access exactly at suspension moment.

### UC-STF-008 — Separate & Archive (handover, anonymize)
- **Module:** Staff · **Priority:** High
- **Actors:** HR Administrator (primary), System, Manager (handover)
- **Goal:** Exit a staff member with mandatory handover, access revocation, and retention-aware archival.
- **Description:** Separation revokes access, mandates handover of owned responsibilities (classes/subjects), runs clearance, and archives/anonymizes per retention (Conflict C-08) — never hard-deletes.
- **Business Rules Applied:** STF-008, STF-004, TCH-006, HR-007, AUD-008 (Cross-Cutting P3/P4).
- **Preconditions:** Separation initiated; authorized actor.
- **Trigger:** Resignation/termination/end-of-contract.
- **Main Success Scenario:**
  1. HR initiates separation with an effective date.
  2. System revokes access (AUTH-005) and mandates handover of owned ownerships (TCH-006).
  3. Clearance and final settlement run (HR-007).
  4. System archives the record; anonymizes the erasable subset per retention; statutory records retained (C-08).
- **Alternative Flows:** A1) Garden-leave / notice period handling. A2) Separation workflow approval (UC-STF-016).
- **Exception Flows:** E1) Pending ownerships without handover → block completion until reassigned (TCH-006). E2) Hard-delete attempt → blocked (UC-STF-018).
- **Validation Rules:** Handover complete; clearance done; archive-not-delete; retention precedence (STF-008, TCH-006, HR-007, C-08).
- **Permissions Required:** `hr.separation.manage` (+ approval).
- **Notifications Triggered:** Separation/clearance notices to staff/manager/finance.
- **Audit Events Generated:** `STAFF_SEPARATED`, `HANDOVER_COMPLETED`, `STAFF_ARCHIVED/ANONYMIZED`.
- **Data Created:** Clearance/settlement records.
- **Data Updated:** Status→separated; access revoked; record archived.
- **Data Deleted:** Erasable personal data per retention (statutory retained).
- **Post Conditions:** Clean exit; responsibilities handed over; history preserved per law.
- **Related Use Cases:** UC-TCH-006, UC-HR-007, UC-STF-018.
- **Acceptance Criteria:**
  - Given separation, When processed, Then access is revoked and a handover of owned responsibilities is mandated.
  - Given pending ownerships, When separating, Then completion is blocked until reassigned.
  - Given retention rules, When archiving, Then statutory records are retained while erasable data is anonymized; no hard delete.
- **Edge Case Analysis:**
  - *Invalid Input:* separation without effective date rejected.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* separation during active assignment → handover enforced first.
  - *Duplicate Data:* idempotent re-separation.
  - *System Failure:* atomic; no orphaned ownership.
  - *Workflow Failure:* approval/clearance pends.
- **QA Coverage:**
  - *Positive:* clean separation with handover; settlement.
  - *Negative:* pending ownership block; hard-delete attempt.
  - *Boundary:* separation on a term boundary; last-day access.

---

## 7. Compact Specifications (routine use cases)

- **UC-STF-002 — Link / Invite User Account** · *High* · Rules: STF-002, AUTH-011. Link staff to a user account or send an invitation. *Audit:* `STAFF_ACCOUNT_LINKED`. *Edge:* one account per staff; invitation activation. *QA:* link; invite; duplicate-account prevention.
- **UC-STF-004 — Manage Scope & Multi-Assignment** · *High* · Rules: STF-005, AUTHZ-007. Assign to institute(s)/campus(es); multi-assignment scoped. *Audit:* scope change. *Edge:* access reflects scope; multi-campus. *QA:* scope set; multi-assignment isolation.
- **UC-STF-005 — Manage Reporting Hierarchy** · *High* · Rules: STF-006. Set the manager/reporting line (feeds workflow approver resolution, WFL-004). *Audit:* `REPORTING_LINE_CHANGED`. *Edge:* no cycles; vacancy handling. *QA:* hierarchy set; cycle blocked; approver resolution.
- **UC-STF-006 — View Staff Profile (masked, scoped)** · *High* · Rules: STF-007, AUTHZ-003. Scoped, masked view; sensitive data need-to-know. *Edge:* salary/national-ID masked unless authorized. *QA:* scope; masking.
- **UC-STF-007 — Update Staff Profile (controlled)** · *Medium* · Rules: STF-007. Self/HR updates; sensitive fields controlled/audited. *Audit:* `STAFF_UPDATED`. *Edge:* immutable ID; sensitive change may need approval. *QA:* controlled edit; immutable-ID protection.
- **UC-STF-009 — Manage Staff Documents** · *Medium* · Rules: STF-007, FILE-001/004. Upload/manage documents (private, scoped). *Edge:* sensitive docs access-controlled. *QA:* upload; scoped access.
- **UC-STF-010 — Approve Sensitive Change / Status** · *Medium* · Rules: AUTHZ-009, WFL-004. Approver gates sensitive changes. *Edge:* SoD; escalation. *QA:* approval; self-approval blocked.
- **UC-STF-011 — Search Staff (scoped)** · *Medium* · Rules: AUTHZ-002, REP-002. Scoped search. *QA:* scope respected.
- **UC-STF-012 — Staff Directory & Headcount Report** · *Medium* · Rules: REP-002/003. Directory/headcount (masked, scoped). *Edge:* sensitive data masked. *QA:* counts; masking.
- **UC-STF-013 — Bulk Staff Update** · *Medium* · Rules: STF-007. Batch update permitted fields. *Edge:* per-row validation; immutable fields excluded. *QA:* clean bulk; protection.
- **UC-STF-014 — Import Staff** · *Medium* · Rules: STF-001/003/005. Import with dedup/immutable IDs/scope. *Edge:* duplicates linked. *QA:* clean import; dedup.
- **UC-STF-015 — Export Staff Data (governed, masked)** · *High* · Rules: REP-005, STF-007. Governed, scoped, masked export; bulk PII gated. *Permissions:* `report.export` (elevated). *Edge:* salary/PII controlled. *QA:* scoped; gated; masked.
- **UC-STF-016 — Separation Workflow** · *Medium* · Rules: WFL-002/004, HR-007. Version-pinned separation/clearance. *QA:* pinning; SoD; clearance gating.
- **UC-STF-017 — Duplicate Staff Detected (Exception)** · *High* · Rules: STF-003. Match → link/merge, not create. *QA:* dedup link; no duplicate ID.
- **UC-STF-018 — Hard-Delete Blocked (Exception)** · *Critical* · Rules: STF-008. Staff records archived/anonymized, never hard-deleted. *QA:* delete blocked; archive path.

## 8. Module-level QA & Edge Themes
- **Status→access (STF-004):** suspension/separation revokes access immediately — a security-critical suite tied to AUTH-005.
- **Reporting hierarchy (STF-006):** correctness feeds workflow approver resolution (leave, payroll); cycles blocked, vacancies handled.
- **Sensitive-data protection (STF-007):** salary/national-ID masking in views, reports, exports.
- **Archive-not-delete + handover (STF-008 / TCH-006 / C-08):** separation always hands over ownership and preserves statutory history.
