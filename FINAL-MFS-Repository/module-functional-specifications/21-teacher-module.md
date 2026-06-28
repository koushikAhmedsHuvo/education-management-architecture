# 21 — Teacher Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`TCH-001…007`), and Use Case Repository (`UC-TCH-001…017`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Manage teachers as specialized staff: qualification-aware subject assignment, workload limits, assignment-conferred ownership that re-resolves immediately on change, temporary attributed substitution, mandatory handover, and class-teacher (homeroom) singularity.

**Business Goal.** Ensure qualified teachers own the subjects/sections they teach, within workload limits, with clean ownership transitions so attendance and marks always attribute correctly and nothing is orphaned.

**Scope.** Teacher designation (specializes Staff); qualification-aware subject assignment; workload limits; assignment-conferred ownership (re-resolving on change, C-05); temporary attributed substitution; mandatory handover on departure/reassignment; one class-teacher per section.

**Out of Scope.** Base staff record (Staff module). Subject/curriculum definitions (Subject module — Teacher assigns against them). Marks entry (Examination module — consumes ownership). Leave/substitution scheduling (Leave module coordinates; Teacher provides substitute mechanics).

---

## 2. Actors

**Primary Actors.** Academic Administrator / Coordinator (assigns teachers), Teacher (owns assignments), HR Administrator (base staff record), System (qualification checks, workload enforcement, ownership wiring, substitution attribution).

**Secondary Actors.** Staff module (base record), Subject/Section modules (assignment targets), Authorization (ownership predicates), Leave module (substitution coordination), Workflow Engine (over-workload/handover approval), Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Designate teacher | Mark a staff member as a teacher (specialization). | Medium | Staff (STF-001) |
| FR-002 | Qualification-aware assignment | Assign subjects only to qualified teachers. | High | Subject (SUB-004) |
| FR-003 | Workload limits | Enforce workload limits on assignment. | High | FR-002 |
| FR-004 | Ownership re-resolution | Assignment confers ownership; change ends live ownership immediately (history preserved). | High | Authorization, C-05 |
| FR-005 | Substitution (attributed) | Temporary, time-boxed, attributed ownership transfer that reverts. | High | Leave (LEV-008) |
| FR-006 | Mandatory handover | Handover owned responsibilities on departure/reassignment. | High | Staff (STF-008) |
| FR-007 | Class-teacher singularity | Exactly one class-teacher per section. | High | Section (SEC-004) |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Teacher specialization | Teacher is-a staff. | Reuse + role-specific. |
| Qualified assignment | Competence-gated. | Quality assurance. |
| Workload limits | Capacity-gated. | Fair distribution. |
| Ownership re-resolution | Clean transitions. | Correct attribution. |
| Attributed substitution | Time-boxed acting-for. | Coverage without confusion. |
| Mandatory handover | No orphaned ownership. | Continuity. |
| Homeroom singularity | One class-teacher. | Clear accountability. |

---

## 5. Screens

Teacher List; Teacher Detail; Designate Teacher; Assign Subject (qualification + workload); Assign Class-Teacher; Change Assignment; Assign Substitute; Mandatory Handover; Qualifications; Workload & Coverage Report; Bulk Assignment; Import/Export Assignments; Assignment/Handover Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Teacher List | Designate, Open, Search/Filter, Export | Bulk Assign |
| Teacher Detail | Assign Subject, Assign Class-Teacher, Change Assignment, Handover, Manage Qualifications | — |
| Assign Subject | Validate Qualification/Workload, Confirm | Bulk Assign |
| Assign Class-Teacher | Confirm (one per section) | — |
| Change Assignment | Reassign, Confirm (ownership re-resolves) | — |
| Assign Substitute | Set Period, Confirm (attributed) | — |
| Handover | Map Responsibilities, Confirm | — |
| Workload Report | Run, Filter, Export | Export |

---

## 7. Forms

**Designate Teacher** — `staff` (search-select, required). Validation: is-a staff (TCH-001/STF-001); inherits staff rules.

**Assign Subject** — `subject`/`section` (required), `teacher` (qualified). Validation: qualified (TCH-002); workload ≤ limit (TCH-003 / UC-TCH-016); confers marks ownership (TCH-004/SUB-004); over-workload → governed exception (UC-TCH-009).

**Assign Class-Teacher** — `section`, `teacher`. Validation: one class-teacher per section (TCH-007/SEC-004 / UC-TCH-017).

**Change Assignment** — `newTeacher`. Validation: live ownership ends immediately; history attributed (TCH-004/C-05); handover on in-progress work (TCH-006).

**Assign Substitute** — `originalTeacher`, `substitute` (qualified), `period` (date range). Validation: time-boxed, attributed acting-for, reverts (TCH-005/LEV-008).

**Handover** — `responsibilities` map, `recipient`. Validation: complete before separation/reassignment (TCH-006).

---

## 8. Search & Filter Requirements

**Teachers/Assignments:** by name, subject, section, qualification, workload, class-teacher, substitute. Sorting: name/workload. Pagination: server-side, 25 default. Scope-bound.

---

## 9. Table Requirements

**Teacher table:** Name, Qualifications, Subjects/Sections, Workload, Class-Teacher Of, Status. **Workload table:** Teacher, Assigned Load, Limit, Coverage. Sorting on Name/Workload. Filtering as above. Export (governed). Bulk: assign.

---

## 10. Workflow Requirements

**Trigger events:** designate, assign subject/class-teacher, change assignment, substitute, handover. **Status changes:** assignment `ACTIVE → CHANGED/ENDED`; substitute `ACTIVE → REVERTED`. **Approvals:** over-workload exceptions and handover via Workflow Engine. **Notifications:** assignment, substitute, handover (to teachers). **Audit:** assignments, ownership transfers, substitutions, handovers (immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Designate teacher | `teacher.designate` |
| Assign subject | `academic.assignment.manage` |
| Assign class-teacher | `section.classteacher.assign` |
| Change assignment | `academic.assignment.manage` |
| Assign substitute | `academic.assignment.manage` |
| Manage qualifications | `teacher.qualification.manage` |
| Approve over-workload exception | `teacher.workload.approve` |
| Import/export | `teacher.import`, `teacher.export` |

Assignment confers ownership (TCH-004); changes re-resolve ownership immediately (C-05).

---

## 12. Business Rule References

TCH-001 (teacher specializes staff), TCH-002 (qualification-aware subject assignment), TCH-003 (workload limits), TCH-004 (assignment confers ownership, re-resolves on change), TCH-005 (substitute — temporary attributed ownership transfer), TCH-006 (mandatory handover on departure/reassignment), TCH-007 (class-teacher singularity). Cross-cutting: STF-001/008 (staff base/handover), SUB-004 (marks ownership), SEC-004 (class-teacher), LEV-008 (substitution coordination), C-05 (ownership re-resolution), WFL-004, AUD-001.

## 13. Use Case References

UC-TCH-001 (Designate), UC-TCH-002 (Assign Subject — qualification + workload), UC-TCH-003 (Assign Class-Teacher — singularity), UC-TCH-004 (Change Assignment — ownership re-resolves), UC-TCH-005 (Assign Substitute — attributed), UC-TCH-006 (Mandatory Handover), UC-TCH-007 (View Assignments/Workload), UC-TCH-008 (Manage Qualifications), UC-TCH-009 (Approve Over-Workload Exception), UC-TCH-010 (Search), UC-TCH-011 (Workload & Coverage Report), UC-TCH-012 (Bulk Assignment), UC-TCH-013 (Import/Export), UC-TCH-014 (Assignment/Handover Workflow), UC-TCH-015 (Unqualified Assignment Blocked), UC-TCH-016 (Workload Limit Exceeded), UC-TCH-017 (Second Class-Teacher Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Designate teacher | POST | Academic Admin |
| Assign subject (qualification/workload-checked) | POST | Coordinator |
| Assign / change class-teacher | POST | Coordinator |
| Change assignment (ownership re-resolves) | POST | Coordinator |
| Assign substitute (attributed) | POST | Coordinator |
| Mandatory handover | POST | Coordinator (→ approval) |
| Manage qualifications | POST/PUT | Academic Admin |
| Approve over-workload exception | POST | Approver |
| Workload & coverage report | GET | Admin |
| Bulk assign / import / export | POST/GET | Admin |

Assignment validates qualification + workload (TCH-002/003); reassignment ends live ownership immediately (TCH-004/C-05).

---

## 15. Database Requirements

**Entities:** `Teacher` (staffId specialization), `Qualification`, `SubjectAssignment` (teacherId, subjectId, sectionId, ownership), `ClassTeacherAssignment` (one per section), `Substitution` (original, substitute, period, attributed), `Handover` (responsibilities map). **Relationships:** Teacher is-a Staff; Teacher 1—* SubjectAssignment; Section 1—1 ClassTeacher. **Indexes:** index(SubjectAssignment.teacherId, sectionId), unique(ClassTeacherAssignment.sectionId), index(Substitution.period). Workload computed/enforced (TCH-003); ownership re-resolves on change (C-05).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Subject/class-teacher assignment; substitute; handover. |
| In-App | Assignment changes; over-workload approvals; handover tasks. |
| SMS/Push | Optional assignment alerts. |

---

## 17. Audit Requirements

Log: designation, subject/class-teacher assignments, assignment changes (ownership transfer, history attribution), substitutions (acting-for), handovers. Record who/when/before/after. Ownership-conferring/transferring actions are first-class audit events. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Workload distribution, Coverage gaps, Qualification matrix, Substitution log, Class-teacher coverage. **Exports:** governed assignment export. **Dashboards:** academic staffing (over/under-load, unassigned subjects, coverage).

---

## 19. Error Handling

**Validation:** unqualified assignment, workload exceeded, second class-teacher → specific errors (UC-TCH-015/016/017). **Permission:** unauthorized assignment → 403. **Workflow:** over-workload/handover pending → clear state. **System:** reassignment with in-progress marks → handover required (TCH-006); substitute revert reliable.

---

## 20. Edge Cases

**Concurrent updates:** parallel assignments over workload → serialized; limit enforced. **Duplicate data:** duplicate assignment idempotent. **Partial failures:** bulk assign partial → per-row report. **Rollback:** assignment change reversed → prior ownership restored, history intact. **Ownership race:** reassignment during mark entry → prior teacher's entry stops at transfer; substitute auto-reverts at window end.

---

## 21. Acceptance Criteria

**Functional.** Teachers are specialized staff; subjects assign only to qualified teachers within workload limits; assignment confers ownership and reassignment ends live ownership immediately while preserving historical attribution; substitutes get time-boxed attributed ownership that reverts; departure/reassignment mandates handover; each section has exactly one class-teacher.

**Business.** Qualified teachers own what they teach within fair limits; attendance and marks always attribute correctly across changes; no class or subject is ever orphaned by a staffing change.

---

## 22. Future Enhancements

Auto subject-teacher suggestion by qualification/workload; timetable-aware load balancing; substitute pools and auto-coverage from leave; qualification-expiry tracking; co-teaching/assistant roles; workload analytics and forecasting.
