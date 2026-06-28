# 12 — Section Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`SEC-001…006`), and Use Case Repository (`UC-SEC-001…014`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Manage the section — the enrollment leaf: name uniqueness within class+campus+shift, capacity as the operative enrollment limit (the hard cap), class-teacher (homeroom) ownership, shift as a first-class attribute, and history-preserving merge/split/close via transfers.

**Business Goal.** Provide the concrete unit students enroll into with an authoritative capacity, a single accountable class-teacher, and safe restructuring that never loses placement history.

**Scope.** Section CRUD (capacity, shift); name uniqueness within class+campus+shift; capacity as operative enrollment limit (hard cap consumed by Enrollment); class-teacher assignment conferring ownership; shift attribute; merge/split/close preserving history via transfers; capacity-reduction protection.

**Out of Scope.** Class structure definition (Class module). Enrollment record mechanics (Enrollment module — consumes section capacity). Attendance/marks (respective modules — use class-teacher/subject-teacher ownership). Inter-campus moves (Campus module).

---

## 2. Actors

**Primary Actors.** Academic Administrator / Coordinator (creates/restructures sections, assigns class-teacher), System (capacity enforcement, uniqueness, history-preserving transfers).

**Secondary Actors.** Class module (parent), Enrollment module (capacity consumer), Teacher module (class-teacher), Attendance/Result (ownership consumers), Workflow Engine (closure/restructure approval), Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Create section | Create a section under a class with capacity and shift. | Critical | Class |
| FR-002 | Name uniqueness | Unique within class + campus + shift. | High | Class, Campus |
| FR-003 | Capacity as enrollment limit | Section capacity is the operative hard cap. | Critical | Enrollment |
| FR-004 | Class-teacher ownership | Assigning a class-teacher confers homeroom ownership (one per section). | High | Teacher (TCH-007) |
| FR-005 | Shift attribute | Shift is a first-class section attribute. | High | FR-002 |
| FR-006 | Merge/split/close | Restructure with history preserved via transfers. | High | Enrollment, Workflow |
| FR-007 | Capacity-reduction protection | Cannot reduce capacity below current enrolled. | High | Enrollment |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Enrollment leaf | Concrete unit students join. | Operational placement. |
| Hard-cap capacity | Authoritative enrollment limit. | Class-size integrity. |
| Class-teacher ownership | One accountable homeroom teacher. | Clear responsibility. |
| Shift as attribute | Morning/day/evening sections. | Multi-shift operations. |
| History-preserving restructure | Merge/split/close via transfers. | No lost placement history. |
| Capacity guard | No reduction below enrolled. | Data integrity. |

---

## 5. Screens

Section List; Section Detail; Create Section; Edit Section (capacity/details); Assign/Change Class-Teacher; Merge/Split/Close Section; Shift Management; Section Fill & Capacity Report; Bulk-Create/Import Sections; Export Section List; Restructure Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Section List | Create, Open, Search/Filter, Export | Bulk Create/Import |
| Section Detail | Edit, Assign Class-Teacher, Merge/Split/Close, View Fill | — |
| Create/Edit Section | Set Capacity/Shift, Save | — |
| Assign Class-Teacher | Select Teacher, Confirm (one per section) | — |
| Merge/Split/Close | Select Targets, Map Transfers, Validate, Submit (→ approval) | — |
| Fill & Capacity Report | Run, Filter, Export | Export |

---

## 7. Forms

**Create Section** — `class` (select, required), `name` (text, required), `capacity` (number, required ≥1), `shift` (select, required), `campus` (select, required). Validation: name unique within class+campus+shift (SEC-002); section is enrollment leaf (SEC-001); capacity is operative limit (SEC-003).

**Assign Class-Teacher** — `teacher` (search-select, required, qualified staff). Validation: exactly one class-teacher per section (SEC-004/TCH-007); confers ownership.

**Edit Capacity** — `capacity` (number). Validation: not below current enrolled count (SEC-003 / UC-SEC-013).

**Merge/Split/Close** — `operation` (merge/split/close), `targetSection(s)` (required), `transferMap` (student→target). Validation: history preserved via transfers (SEC-006); targets have capacity; routed to approval.

**Shift** — `shift` (morning/day/evening/custom). Validation: first-class attribute affecting uniqueness (SEC-005).

---

## 8. Search & Filter Requirements

**Sections:** by class, campus, shift, name, class-teacher, fill status (open/full), status (active/closed). Sorting: class/name/fill. Pagination: server-side, 25 default. Scope-bound (institute/campus).

---

## 9. Table Requirements

**Section table:** Class, Section, Shift, Campus, Capacity, Enrolled, Fill %, Class-Teacher, Status. Sorting on Class/Fill. Filtering as above. Export (governed). Bulk: create/import.

---

## 10. Workflow Requirements

**Trigger events:** create, capacity change, class-teacher assign/change, merge/split/close. **Status changes:** Section `ACTIVE → CLOSED` (via restructure). **Approvals:** closure/merge/split via Workflow Engine. **Notifications:** class-teacher assignment, restructure (to affected guardians via Enrollment transfers — custody-aware). **Audit:** create/capacity/teacher/restructure changes (before/after, immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| View section(s) | `section.view` |
| Create section | `section.create` |
| Update section/capacity | `section.update` |
| Assign class-teacher | `section.classteacher.assign` |
| Merge/split/close | `section.restructure` |
| Approve closure/restructure | `section.restructure.approve` |
| Import/export | `section.import`, `section.export` |

Section management is institute/campus-scoped; restructures are governed.

---

## 12. Business Rule References

SEC-001 (section is the enrollment leaf), SEC-002 (name uniqueness within class+campus+shift), SEC-003 (capacity is operative enrollment limit), SEC-004 (class-teacher assignment confers ownership), SEC-005 (shift is first-class attribute), SEC-006 (merge/split/close preserves history via transfers). Cross-cutting: CLS-003 (class contains sections), ENR-003 (capacity consumer), TCH-007 (class-teacher singularity), ENR-005 (transfer history), C-01 (section hard cap wins at enrollment), WFL-004, AUD-001.

## 13. Use Case References

UC-SEC-001 (Create — capacity, shift), UC-SEC-002 (View), UC-SEC-003 (Update capacity/details), UC-SEC-004 (Assign/Change Class-Teacher), UC-SEC-005 (Merge/Split/Close — history-preserving), UC-SEC-006 (Manage Shift), UC-SEC-007 (Approve Closure/Restructure), UC-SEC-008 (Search), UC-SEC-009 (Fill & Capacity Report), UC-SEC-010 (Bulk-Create/Import), UC-SEC-011 (Export), UC-SEC-012 (Restructure Workflow), UC-SEC-013 (Capacity-Reduction Below Enrolled Blocked), UC-SEC-014 (Duplicate Section Name Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Create section | POST | Academic Admin |
| Get / list sections | GET | Authorized roles |
| Update section (capacity/details) | PUT | Academic Admin |
| Assign / change class-teacher | POST | Academic Admin |
| Merge / split / close section | POST | Academic Admin (→ approval) |
| Manage shift | PUT | Academic Admin |
| Section fill & capacity report | GET | Admin |
| Bulk-create / import / export sections | POST/GET | Admin |

Capacity is the authoritative enrollment hard cap (SEC-003); reduction below enrolled is rejected (UC-SEC-013).

---

## 15. Database Requirements

**Entities:** `Section` (classId, campusId, name, capacity, shift, classTeacherId, status), `SectionRestructure` (operation, source/target, transfer map), `ClassTeacherAssignment` (sectionId, teacherId). **Relationships:** Class 1—* Section; Section 1—* Enrollment; Section 1—1 ClassTeacher. **Indexes:** unique(Section.classId, campusId, shift, name) (SEC-002), unique(Section.classTeacherId per section) (SEC-004), index(Section.classId, status). Capacity check is the enrollment hard cap (SEC-003); restructure preserves history via Enrollment transfers (SEC-006).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| In-App | Class-teacher assignment; restructure pending approvals. |
| Email | Section closure/restructure affecting students (via Enrollment, custody-aware). |
| SMS/Push | Optional restructure notice to guardians. |

---

## 17. Audit Requirements

Log: section create/update, capacity changes (before/after, with enrolled-count guard), class-teacher assignment/change, merge/split/close (with transfer map). Record who/when/before/after. Restructures are first-class audit events (history via transfers). Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Section fill/capacity, Class-teacher coverage, Restructure history, Shift distribution. **Exports:** governed section list. **Dashboards:** section fill rates, capacity alerts, unassigned class-teacher.

---

## 19. Error Handling

**Validation:** duplicate section name (within class+campus+shift), capacity below enrolled, second class-teacher → specific errors (UC-SEC-013/014). **Permission:** out-of-scope section → 403. **Workflow:** restructure pending approval → clear state. **System:** Enrollment count service unavailable → block capacity reduction (fail safe).

---

## 20. Edge Cases

**Concurrent updates:** two capacity edits → versioned; reduction re-checks enrolled. **Duplicate data:** duplicate name → blocked. **Partial failures:** bulk-create partial → per-row report. **Rollback:** restructure reversed → students remain in source, history accurate. **Capacity race:** capacity reduction vs new enrollment → serialized; never below enrolled.

---

## 21. Acceptance Criteria

**Functional.** Section names are unique within class+campus+shift; section capacity is the authoritative enrollment hard cap and cannot be reduced below current enrolled; each section has exactly one class-teacher conferring ownership; shift is a first-class attribute; merge/split/close preserves placement history via transfers.

**Business.** Students enroll into well-defined, capacity-controlled sections with clear homeroom accountability; restructuring is safe and history-preserving; class sizes and shift operations are accurate and governable.

---

## 22. Future Enhancements

Auto-balancing across sections; room/timetable binding per section; co-class-teacher/assistant roles; capacity-forecast planning; drag-and-drop section restructuring with live transfer preview; shift-based timetable templates.
