# 12 — Section Use Cases

Transforms the Section business rules (`SEC-001`…`SEC-006`) into use cases. The section is the enrollment leaf: it carries the operative capacity limit, confers class-teacher ownership, and treats shift as first-class.

## 1. Primary Actors
Academic Administrator / Coordinator (creates and manages sections), Institute Administrator (oversight).

## 2. Secondary Actors
System (uniqueness, capacity enforcement, ownership wiring, history-preserving merge/split), Teacher module (class-teacher ownership), Enrollment module (capacity consumer), Audit service.

## 3. Goals
Model sections within a class+campus+shift; enforce the operative section capacity (the hard cap at enrollment); assign a class-teacher who owns the section; treat shift as a defining attribute; restructure (merge/split/close) without losing history.

## 4. User Journeys
- **Create:** coordinator adds a section under a class (name unique within class+campus+shift, capacity, shift) → it becomes an enrollment target.
- **Staff it:** assign a class-teacher → they gain ownership of the section's students for attendance and pastoral actions.
- **Restructure:** merge two under-filled sections or split an over-large one → students moved via audited transfers, prior-period history preserved.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core/CRUD | UC-SEC-001 | Create Section (capacity, shift) | High |
| CRUD | UC-SEC-002 | View Section(s) | Medium |
| CRUD | UC-SEC-003 | Update Section (capacity/details) | High |
| Core | UC-SEC-004 | Assign / Change Class-Teacher (ownership) | High |
| Core | UC-SEC-005 | Merge / Split / Close Section (history-preserving) | High |
| Admin | UC-SEC-006 | Manage Shift Attribute | Medium |
| Approval | UC-SEC-007 | Approve Section Closure / Restructure | Low |
| Search | UC-SEC-008 | Search Sections | Low |
| Reporting | UC-SEC-009 | Section Fill & Capacity Report | Medium |
| Bulk/Import | UC-SEC-010 | Bulk-Create / Import Sections | Medium |
| Export | UC-SEC-011 | Export Section List | Low |
| Workflow | UC-SEC-012 | Restructure Workflow | Low |
| Exception | UC-SEC-013 | Capacity-Reduction Below Enrolled Blocked | High |
| Exception | UC-SEC-014 | Duplicate Section Name Blocked | Medium |

---

## 6. Detailed Specifications (high-value use cases)

### UC-SEC-001 — Create Section (capacity, shift)
- **Module:** Section · **Priority:** High
- **Actors:** Academic Coordinator (primary), System
- **Goal:** Create an enrollment-target section with an operative capacity and shift.
- **Description:** Creates a section under a class, with a name unique within class+campus+shift, a capacity that becomes the hard enrollment limit, and a shift attribute.
- **Business Rules Applied:** SEC-001, SEC-002, SEC-003, SEC-005.
- **Preconditions:** Parent class exists; actor holds `academic.structure.manage`.
- **Trigger:** Coordinator creates a section.
- **Main Success Scenario:**
  1. Coordinator enters name, campus, shift, and capacity.
  2. System enforces name uniqueness within class+campus+shift (SEC-002).
  3. System records capacity as the operative enrollment limit (SEC-003).
  4. The section becomes an enrollment target.
- **Alternative Flows:** A1) Multiple shifts → distinct sections may share a name across shifts.
- **Exception Flows:** E1) Duplicate name within class+campus+shift → reject (UC-SEC-014).
- **Validation Rules:** Unique within class+campus+shift; capacity ≥ 0/positive; shift valid (SEC-002/003/005).
- **Permissions Required:** `academic.structure.manage`.
- **Notifications Triggered:** None routine.
- **Audit Events Generated:** `SECTION_CREATED` (class, campus, shift, capacity).
- **Data Created:** Section.
- **Data Updated:** None.
- **Data Deleted:** None.
- **Post Conditions:** Section exists as an enrollment leaf with an operative capacity.
- **Related Use Cases:** UC-SEC-003, UC-ENR-001.
- **Acceptance Criteria:**
  - Given a unique name within class+campus+shift, When created, Then the section exists with its capacity as the enrollment limit.
  - Given a duplicate name within the same class+campus+shift, When created, Then it is rejected.
  - Given the same name in a different shift, When created, Then it is allowed.
- **Edge Case Analysis:**
  - *Invalid Input:* negative capacity/invalid shift rejected.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* same name created twice → one wins (uniqueness).
  - *Duplicate Data:* per class+campus+shift duplicate blocked.
  - *System Failure:* atomic creation.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* create; multi-shift same name.
  - *Negative:* duplicate; negative capacity.
  - *Boundary:* capacity 0 (no enrollment) vs 1.

### UC-SEC-003 — Update Section (capacity/details)
- **Module:** Section · **Priority:** High
- **Actors:** Academic Coordinator (primary), System
- **Goal:** Adjust section attributes, with capacity changes guarded against under-cutting current enrollment.
- **Description:** Edits name/shift/capacity; capacity may increase freely but cannot be reduced below the count of currently active enrollments.
- **Business Rules Applied:** SEC-003, SEC-002, ENR-003.
- **Preconditions:** Section exists; actor authorized.
- **Trigger:** Coordinator edits the section.
- **Main Success Scenario:**
  1. Coordinator changes capacity or details.
  2. System validates a capacity reduction is ≥ current active enrollment (SEC-003/ENR-003).
  3. System applies the change; name uniqueness re-validated if changed.
- **Alternative Flows:** A1) Capacity increase → immediately frees seats for enrollment/waitlist.
- **Exception Flows:** E1) Reducing capacity below active enrollment → blocked (UC-SEC-013).
- **Validation Rules:** New capacity ≥ active enrollment; uniqueness preserved (SEC-002/003).
- **Permissions Required:** `academic.structure.manage`.
- **Notifications Triggered:** None routine; capacity increase may trigger waitlist promotion (ADM-005).
- **Audit Events Generated:** `SECTION_UPDATED` (capacity change recorded).
- **Data Created:** None.
- **Data Updated:** Section attributes/capacity.
- **Data Deleted:** None.
- **Post Conditions:** Section updated consistently with current enrollment.
- **Related Use Cases:** UC-SEC-013, UC-ENR-001, UC-ADM-006.
- **Acceptance Criteria:**
  - Given a section with N active enrollments, When capacity is set ≥ N, Then it is accepted.
  - Given the same section, When capacity is set < N, Then it is blocked.
  - Given a capacity increase, When applied, Then freed seats become available (waitlist may promote).
- **Edge Case Analysis:**
  - *Invalid Input:* non-numeric capacity rejected.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* capacity edit + new enrollment racing → consistent final state (no over-cap).
  - *Duplicate Data:* N/A.
  - *System Failure:* atomic update.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* increase; reduce to exactly N.
  - *Negative:* reduce below N.
  - *Boundary:* capacity == active enrollment; increase by one frees exactly one seat.

### UC-SEC-005 — Merge / Split / Close Section (history-preserving)
- **Module:** Section · **Priority:** High
- **Actors:** Academic Administrator (primary), System, Approver
- **Goal:** Restructure sections while preserving each student's period-accurate history.
- **Description:** Merge under-filled sections, split an over-large one, or close a section — always moving students via audited transfers so prior-period attendance/marks stay attributed to the original section.
- **Business Rules Applied:** SEC-006, ENR-005, SEC-003.
- **Preconditions:** Sections exist; target capacity available; actor authorized; approval if configured.
- **Trigger:** Admin initiates merge/split/close.
- **Main Success Scenario:**
  1. Admin selects the restructure (merge A+B, split A→A1/A2, or close A).
  2. System validates target capacity (hard cap) for the receiving section(s).
  3. System moves students via dated transfers (ENR-005), preserving prior-period attribution.
  4. The closed/merged source section is archived (history retained).
- **Alternative Flows:** A1) Restructure requires approval (UC-SEC-007).
- **Exception Flows:** E1) Target capacity insufficient → block until capacity/another target. E2) Closing a section with active students without a target → blocked.
- **Validation Rules:** Target capacity sufficient; transfers dated and history-preserving; source archived not deleted (SEC-006/ENR-005).
- **Permissions Required:** `academic.structure.manage` (+ approval).
- **Notifications Triggered:** `SECTION_RESTRUCTURED` to affected teachers/guardians.
- **Audit Events Generated:** `SECTION_MERGED/SPLIT/CLOSED`, per-student `SECTION_TRANSFER`.
- **Data Created:** New section(s) (split); transfer events.
- **Data Updated:** Enrollments' current section; source archived.
- **Data Deleted:** None.
- **Post Conditions:** Students re-placed; prior-period history intact; sources archived.
- **Related Use Cases:** UC-ENR-004, UC-SEC-013.
- **Acceptance Criteria:**
  - Given a merge, When executed, Then students move to the target within its hard cap and prior-period records stay with the source.
  - Given a split, When executed, Then students are distributed within target capacities with preserved history.
  - Given insufficient target capacity, When restructuring, Then it is blocked.
- **Edge Case Analysis:**
  - *Invalid Input:* split distribution exceeding capacities rejected.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* restructure + enrollment racing → serialized; capacity honored.
  - *Duplicate Data:* re-running is idempotent.
  - *System Failure:* atomic/recoverable; no half-moved cohort.
  - *Workflow Failure:* approval pends.
- **QA Coverage:**
  - *Positive:* merge; split; close-with-target.
  - *Negative:* capacity overflow; close without target.
  - *Boundary:* merge filling target exactly to cap.

---

## 7. Compact Specifications (routine use cases)

- **UC-SEC-002 — View Section(s)** · *Medium* · Rules: AUTHZ-002/003, SEC-004. Scoped read; class-teacher sees own section. *QA:* scope/ownership.
- **UC-SEC-004 — Assign / Change Class-Teacher (ownership)** · *High* · Rules: SEC-004, TCH-004, AUTHZ-003. Assign class-teacher → ownership of section students; unassign ends live ownership immediately (C-05). *Audit:* `CLASS_TEACHER_ASSIGNED/CHANGED`. *Edge:* live ownership ends immediately on change; historical attribution preserved. *QA:* ownership granted; immediate end on change.
- **UC-SEC-006 — Manage Shift Attribute** · *Medium* · Rules: SEC-005. Set/change shift (morning/day/evening). *Edge:* shift change affects timing/attendance defaults. *QA:* shift set; timing resolution.
- **UC-SEC-007 — Approve Section Closure / Restructure** · *Low* · Rules: WFL-004. Approver gates restructure. *QA:* approval gates; SoD.
- **UC-SEC-008 — Search Sections** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-SEC-009 — Section Fill & Capacity Report** · *Medium* · Rules: REP-002, SEC-003. Fill levels/vacancies. *Edge:* counts consistent with hard cap. *QA:* counts match; capacity accuracy.
- **UC-SEC-010 — Bulk-Create / Import Sections** · *Medium* · Rules: SEC-001/002/003. Bulk create with uniqueness/capacity validation. *Edge:* duplicate names rejected per row. *QA:* clean bulk; duplicates.
- **UC-SEC-011 — Export Section List** · *Low* · Rules: REP-005. Scoped export. *QA:* scoped.
- **UC-SEC-012 — Restructure Workflow** · *Low* · Rules: WFL-002/004. Version-pinned restructure approval. *QA:* pinning; SoD.
- **UC-SEC-013 — Capacity-Reduction Below Enrolled Blocked (Exception)** · *High* · Rules: SEC-003. Cannot set capacity below active enrollment. *QA:* blocked; reduce-to-N allowed.
- **UC-SEC-014 — Duplicate Section Name Blocked (Exception)** · *Medium* · Rules: SEC-002. Unique within class+campus+shift. *QA:* duplicate blocked; cross-shift allowed.

## 8. Module-level QA & Edge Themes
- **Operative capacity (SEC-003 / C-01):** the section cap is the authoritative enrollment limit; capacity-reduction guard and concurrent-enrollment races are headline suites.
- **Ownership wiring (SEC-004 / C-05):** class-teacher assignment grants ownership; unassignment ends live ownership immediately, historical attribution preserved.
- **History-preserving restructure (SEC-006):** merge/split/close must retain prior-period attribution via dated transfers.
