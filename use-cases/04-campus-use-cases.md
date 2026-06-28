# 04 — Campus Use Cases

Transforms the Campus business rules (`CAMP-001`…`CAMP-007`) into use cases. Campus is the secondary scope; optional, with a guaranteed default; the enrollment-scope boundary beneath institute.

## 1. Primary Actors
Institute Administrator (creates/manages campuses), Campus Administrator (manages one campus).

## 2. Secondary Actors
System (uniqueness, default-campus invariant, scope cascade, transfer history), Workflow Engine (closure/transfer approvals), Configuration Engine (campus overrides), Audit Service.

## 3. Goals
Model branches within an institute; keep single-location institutes campus-invisible via a default; isolate branch staff/data by campus scope; move students/sections as audited transfers preserving period-accurate history; never strand students or violate the default-campus invariant.

## 4. User Journeys
- **Add a branch:** institute admin creates a campus (name, unique code within institute, address, shift options) → it becomes an enrollment/scope target.
- **Operate a branch:** campus admin sees only their campus; config overrides resolve campus-first.
- **Restructure/close:** transfer students between campuses (audited, dated) → close/archive only after dependents handled and never the last active campus.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core/CRUD | UC-CAMP-001 | Create Campus | High |
| CRUD | UC-CAMP-002 | View Campus(es) | Medium |
| CRUD | UC-CAMP-003 | Update Campus | Medium |
| Core | UC-CAMP-004 | Deactivate / Archive Campus | High |
| Core | UC-CAMP-005 | Inter-Campus Student Transfer | High |
| Approval | UC-CAMP-006 | Approve Campus Closure / Transfer | Medium |
| Config | UC-CAMP-007 | Manage Campus Config Overrides | Medium |
| Search | UC-CAMP-008 | Search Campuses | Low |
| Reporting | UC-CAMP-009 | Campus Roster & Branch Dashboard | Medium |
| Bulk | UC-CAMP-010 | Bulk Student Transfer | Medium |
| Import | UC-CAMP-011 | Import Campuses | Low |
| Export | UC-CAMP-012 | Export Campus List | Low |
| Workflow | UC-CAMP-013 | Transfer Approval Workflow | Medium |
| Exception | UC-CAMP-014 | Last-Campus Deactivation Blocked | High |
| Exception | UC-CAMP-015 | Cross-Institute Reparenting Blocked | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-CAMP-001 — Create Campus
- **Module:** Campus · **Priority:** High
- **Actors:** Institute Administrator (primary), System
- **Goal:** Add a branch to an institute as a scope/enrollment target.
- **Description:** Creates a campus bound to exactly one institute, with a code unique within that institute; the first institute setup auto-creates a default campus.
- **Business Rules Applied:** CAMP-001, CAMP-002, CAMP-003.
- **Preconditions:** Institute `ACTIVE`/in-setup; actor holds `institute.campus.manage`.
- **Trigger:** Admin submits a new campus.
- **Main Success Scenario:**
  1. Admin enters name, unique code (within institute), address, contact, shift options.
  2. System binds the campus to the institute and enforces per-institute code uniqueness.
  3. Campus is created `ACTIVE` and becomes a scope/enrollment target.
- **Alternative Flows:** A1) First institute setup auto-creates the default campus (CAMP-002).
- **Exception Flows:** E1) Duplicate code within the institute → reject. E2) Attempt to bind to two institutes / reparent → reject (CAMP-001).
- **Validation Rules:** Code unique within institute, immutable after creation; `institute_id` valid and accessible (CAMP-001/003).
- **Permissions Required:** `institute.campus.manage`.
- **Notifications Triggered:** `CAMPUS_CREATED` to institute admins.
- **Audit Events Generated:** `CAMPUS_CREATED` (institute, code); `DEFAULT_CAMPUS_CREATED` (auto).
- **Data Created:** Campus record.
- **Data Updated:** None.
- **Data Deleted:** None.
- **Post Conditions:** Campus `ACTIVE`, available for sections/students/staff.
- **Related Use Cases:** UC-CAMP-004, UC-CAMP-005.
- **Acceptance Criteria:**
  - Given a unique code within the institute, When a campus is created, Then it is bound to that institute and the code is locked.
  - Given a duplicate code within the same institute, When creation is attempted, Then it is rejected (but the same code may exist in another institute).
- **Edge Case Analysis:**
  - *Invalid Input:* malformed code/address rejected.
  - *Permission Failure:* non-authorized → 403.
  - *Concurrent Update:* same code created twice → one wins (uniqueness).
  - *Duplicate Data:* per-institute duplicate blocked; cross-institute same code allowed.
  - *System Failure:* creation transactional; no partial campus.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* create campus; default-campus auto-creation.
  - *Negative:* duplicate within institute; reparent attempt.
  - *Boundary:* "MAIN" in two institutes (allowed) vs twice in one (blocked).

### UC-CAMP-004 — Deactivate / Archive Campus
- **Module:** Campus · **Priority:** High
- **Actors:** Institute Administrator (primary), System, Approver
- **Goal:** Take a campus out of operation safely, preserving history.
- **Description:** Deactivate/archive only after active sections/students are transferred; the last active campus of an active institute cannot be deactivated.
- **Business Rules Applied:** CAMP-007, CAMP-002, CAMP-006.
- **Preconditions:** Actor holds `institute.campus.manage`; dependents handled.
- **Trigger:** Admin requests deactivation/archival.
- **Main Success Scenario:**
  1. Admin requests deactivation/archival.
  2. System checks for active sections/enrollments/assignments at the campus.
  3. If clear (or after transfers), and not the last active campus, the campus → `INACTIVE`/`ARCHIVED`; data preserved.
- **Alternative Flows:** A1) Closure may require approval (UC-CAMP-006).
- **Exception Flows:** E1) Active dependents → block; require transfers (UC-CAMP-005/010). E2) Last active campus of an active institute → block (UC-CAMP-014).
- **Validation Rules:** Zero active dependents (or handled); ≥1 active campus must remain for an active institute (CAMP-002/007).
- **Permissions Required:** `institute.campus.manage` (+ approval).
- **Notifications Triggered:** `CAMPUS_DEACTIVATED/ARCHIVED` to admins and affected staff.
- **Audit Events Generated:** `CAMPUS_DEACTIVATED/ARCHIVED`.
- **Data Created:** None.
- **Data Updated:** Campus state → `INACTIVE`/`ARCHIVED`.
- **Data Deleted:** None.
- **Post Conditions:** Campus closed; data read-only; default-campus invariant intact.
- **Related Use Cases:** UC-CAMP-005, UC-CAMP-014.
- **Acceptance Criteria:**
  - Given a campus with no active dependents and a sibling active campus, When archival is requested, Then it is archived and data preserved.
  - Given active students at the campus, When deactivation is attempted, Then it is blocked until they are transferred.
  - Given the only active campus, When deactivation is attempted on an active institute, Then it is blocked.
- **Edge Case Analysis:**
  - *Invalid Input:* archival without handling dependents rejected.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* transfer + deactivate race → serialized; deactivate blocked while transfers pending.
  - *Duplicate Data:* N/A.
  - *System Failure:* dependent check fails → block (no partial archive).
  - *Workflow Failure:* approval unresolved → pending, escalates.
- **QA Coverage:**
  - *Positive:* archive cleared campus.
  - *Negative:* active dependents; last-campus deactivation.
  - *Boundary:* deactivate immediately after the last transfer completes.

### UC-CAMP-005 — Inter-Campus Student Transfer
- **Module:** Campus · **Priority:** High
- **Actors:** Institute Administrator (primary), System, Guardian (notified)
- **Goal:** Move a student between campuses within an institute, preserving period-accurate history.
- **Description:** A dated close-and-open transfer; prior-period attendance/marks stay attributed to the old campus.
- **Business Rules Applied:** CAMP-006, CAMP-001, ENR-005.
- **Preconditions:** Active enrollment; target campus active, same institute, with capacity.
- **Trigger:** Admin initiates a transfer with an effective date.
- **Main Success Scenario:**
  1. Admin selects the student and target campus + effective date.
  2. System validates same-institute target, active, with capacity.
  3. System closes the current placement as-of the date and opens the new one.
  4. Historical records remain attributed to the prior campus for the prior period.
- **Alternative Flows:** A1) Bulk transfer of many students (UC-CAMP-010). A2) Transfer may require approval (UC-CAMP-013).
- **Exception Flows:** E1) Target full → block. E2) Cross-institute "transfer" → rejected (that is withdrawal + new admission).
- **Validation Rules:** Target active, same institute, capacity available; effective date valid (CAMP-006/001).
- **Permissions Required:** `institute.campus.manage`/`enrollment.manage`.
- **Notifications Triggered:** `CAMPUS_TRANSFER` to guardians and receiving campus admin.
- **Audit Events Generated:** `CAMPUS_TRANSFER` (from/to, subject, effective date, actor).
- **Data Created:** Transfer event; new placement.
- **Data Updated:** Current campus; prior placement closed as-of date.
- **Data Deleted:** None.
- **Post Conditions:** Student at the new campus from the effective date; history accurate.
- **Related Use Cases:** UC-CAMP-010, UC-ENR (enrollment transfer).
- **Acceptance Criteria:**
  - Given a valid same-institute target with capacity, When a transfer is executed, Then the student moves from the effective date and prior-period records stay with the old campus.
  - Given a cross-institute target, When a "transfer" is attempted, Then it is rejected.
- **Edge Case Analysis:**
  - *Invalid Input:* invalid effective date (future-of-bounds) rejected.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* two transfers of the same student → serialized; latest wins, both audited.
  - *Duplicate Data:* duplicate transfer to the same campus same date → no-op.
  - *System Failure:* transactional; no half-transfer.
  - *Workflow Failure:* approval unresolved → transfer pending.
- **QA Coverage:**
  - *Positive:* mid-year transfer; period attribution correct.
  - *Negative:* full target; cross-institute attempt; inactive target.
  - *Boundary:* transfer effective on a term boundary; capacity exactly at limit.

---

## 7. Compact Specifications (routine use cases)

- **UC-CAMP-002 — View Campus(es)** · *Medium* · Rules: CAMP-004. Campus admin sees only their campus; institute admin sees all. *QA:* scope isolation.
- **UC-CAMP-003 — Update Campus** · *Medium* · Rules: CAMP-003. Edit mutable fields; code immutable. *Audit:* `CAMPUS_UPDATED`. *QA:* mutable edit; code edit rejected.
- **UC-CAMP-006 — Approve Campus Closure / Transfer** · *Medium* · Rules: WFL-004. Approver gates closure/transfer. *Edge:* SoD/escalation. *QA:* approval gates; timeout escalates.
- **UC-CAMP-007 — Manage Campus Config Overrides** · *Medium* · Rules: CAMP-005, CFG-004. Campus-overridable settings only. *Edge:* non-overridable rejected, inherits institute. *QA:* override applies; disallowed rejected.
- **UC-CAMP-008 — Search Campuses** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-CAMP-009 — Campus Roster & Branch Dashboard** · *Medium* · Rules: REP-002, CAMP-004. Branch-scoped reporting. *Edge:* campus admin cannot see sibling campuses. *QA:* roster scoped; sibling data excluded.
- **UC-CAMP-010 — Bulk Student Transfer** · *Medium* · Rules: CAMP-006, ENR-005. Transfer many students (e.g., campus merge) with dated history. *Validation:* capacity at target; per-student attribution. *Edge:* partial-failure report; capacity overflow handled. *QA:* clean bulk; capacity overflow; period attribution.
- **UC-CAMP-011 — Import Campuses** · *Low* · Rules: CAMP-001/003. Import campuses (validated, unique codes). *QA:* valid import; duplicate code rejected.
- **UC-CAMP-012 — Export Campus List** · *Low* · Rules: REP-005. Scoped export. *QA:* scoped export.
- **UC-CAMP-013 — Transfer Approval Workflow** · *Medium* · Rules: WFL-002/004. Version-pinned transfer approval. *QA:* version-pinning; SoD; escalation.
- **UC-CAMP-014 — Last-Campus Deactivation Blocked (Exception)** · *High* · Rules: CAMP-002. Cannot deactivate the only active campus of an active institute. *QA:* blocked; allowed only after institute deactivation.
- **UC-CAMP-015 — Cross-Institute Reparenting Blocked (Exception)** · *High* · Rules: CAMP-001. "Move campus to another institute" rejected. *QA:* reparent rejected; correct path is recreate + transfer.

## 8. Module-level QA & Edge Themes
- **Default-campus invariant:** an active institute must always retain ≥1 active campus — a key negative suite.
- **Period-accurate transfers:** assert historical attendance/marks remain attributed to the pre-transfer campus.
- **Scope isolation:** campus admins must never see sibling-campus data (frequent authz test target).
