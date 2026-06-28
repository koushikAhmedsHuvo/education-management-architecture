# 13 — Subject Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`SUB-001…007`), and Use Case Repository (`UC-SUB-001…016`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Define subjects and the curriculum: subject identity/code, curriculum mapping to class/stream, components and aggregation (the structure exams and results consume), subject-teacher marks ownership, type/credit/weightage, elective selection with prerequisites, and history-preserving retirement.

**Business Goal.** Provide an authoritative curriculum and assessment structure so examinations, results, and grading compute correctly, and so teachers own marks for the subjects they teach.

**Scope.** Subject definition (identity, code, type, credit, weightage); curriculum mapping (subject ↔ class/stream); components and aggregation rules; subject-teacher assignment conferring marks ownership; electives (selection rules, prerequisites); retirement preserving history.

**Out of Scope.** Exam definition/maxima (Examination module — consumes components). Result computation (Result module). Grade scales (Grading module). Teacher records (Teacher module — assignment links here). Section structure (Section module).

---

## 2. Actors

**Primary Actors.** Academic Administrator / Curriculum Coordinator (defines subjects/curriculum), Academic Coordinator (assigns subject-teachers), Student/Guardian (elective selection), System (component-sum validation, prerequisite checks, ownership wiring).

**Secondary Actors.** Class module (curriculum target), Teacher module (subject-teacher), Examination/Result/Grading (consumers), Enrollment (elective context), Workflow Engine (elective selection), Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Define subject | Define subject with identity/code, type, credit, weightage. | Critical | Configuration Engine |
| FR-002 | Curriculum mapping | Map subjects to class/stream. | High | Class module |
| FR-003 | Components & aggregation | Define components (theory/practical/CA) and aggregation rules. | High | Examination/Result |
| FR-004 | Subject-teacher ownership | Assigning a subject-teacher confers marks ownership. | High | Teacher (TCH-002/004) |
| FR-005 | Type, credit & weightage | Configure subject type, credit, weightage. | Medium | Grading (GPA) |
| FR-006 | Electives & prerequisites | Manage elective selection rules and prerequisites. | Medium | Enrollment |
| FR-007 | Retire (history-preserving) | Retire subjects without losing historical curriculum/marks. | High | Retention/Audit |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Subject identity | Coded, authoritative subjects. | Curriculum backbone. |
| Curriculum mapping | Subject ↔ class/stream. | Correct subject offering. |
| Components & aggregation | Theory/practical/CA structure. | Drives exam/result computation. |
| Subject-teacher ownership | Marks ownership via assignment. | Accountability; access control. |
| Type/credit/weightage | GPA/weighting inputs. | Correct grading. |
| Electives & prerequisites | Choice with gating. | Higher-secondary/university models. |
| History-preserving retirement | Retire without data loss. | Integrity across years. |

---

## 5. Screens

Subject List; Subject Detail; Define Subject; Edit Subject; Curriculum Mapping (subject ↔ class/stream); Components & Aggregation; Assign Subject-Teacher; Electives (selection & prerequisites); Type/Credit/Weightage; Retire Subject; Curriculum Map & Subject Report; Import/Bulk-Define Subjects; Export Curriculum; Elective Selection Workflow.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Subject List | Define, Open, Search/Filter, Export | Bulk Define/Import |
| Subject Detail | Edit, Map Curriculum, Define Components, Assign Teacher, Retire | — |
| Curriculum Mapping | Map to Class/Stream, Remove Mapping, Save | Bulk Map |
| Components | Add/Edit Component, Set Weightage, Validate Sum | — |
| Assign Subject-Teacher | Select Teacher (qualified), Confirm (ownership) | Bulk Assign |
| Electives | Set Selection Rules, Prerequisites, Save | — |
| Retire | Confirm with Reason | — |
| Curriculum Report | Run, Filter, Export | Export |

---

## 7. Forms

**Define Subject** — `name` (text, required), `code` (text, required, unique), `type` (select: core/elective/practical), `credit` (number), `weightage` (number). Validation: code unique (SUB-001); type/credit valid (SUB-005).

**Curriculum Mapping** — `subject` (required), `class/stream` (multi-select, required). Validation: mapping consistent with class/stream (SUB-002).

**Components & Aggregation** — components list (`name`, `maxMarks`, `weightage`, `passMark`), aggregation rule. Validation: component weightages sum to the configured total (SUB-003 / UC-SUB-016); pass rules defined.

**Assign Subject-Teacher** — `teacher` (search-select, qualified), `section` (required). Validation: qualified (TCH-002); confers marks ownership (SUB-004); workload respected (TCH-003).

**Electives** — `selectionRules` (min/max, group), `prerequisites` (subject list). Validation: prerequisite met before selection (SUB-006 / UC-SUB-015).

**Retire** — `reason` (text, required). Validation: history preserved; no hard delete (SUB-007).

---

## 8. Search & Filter Requirements

**Subjects:** by name, code, type, class/stream, teacher, status (active/retired). Sorting: name/code/type. Pagination: server-side, 25 default. Scope-bound to institute.

---

## 9. Table Requirements

**Subject table:** Code, Name, Type, Credit, Mapped Classes/Streams, Components, Status. Sorting on Code/Name/Type. Filtering as above. Export (governed — curriculum). Bulk: define/import, map, assign.

---

## 10. Workflow Requirements

**Trigger events:** define, curriculum map, component change, subject-teacher assign/change, elective selection, retire. **Status changes:** Subject `ACTIVE → RETIRED`. **Approvals:** elective selection (student/guardian → approval where configured); structural changes governed. **Notifications:** subject-teacher assignment, elective confirmation. **Audit:** definition/mapping/component/teacher/retirement changes (before/after, immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| View subject(s) | `subject.view` |
| Define subject | `subject.create` |
| Update subject | `subject.update` |
| Map curriculum | `subject.curriculum.manage` |
| Define components | `subject.component.manage` |
| Assign subject-teacher | `subject.teacher.assign` |
| Manage electives | `subject.elective.manage` |
| Retire subject | `subject.retire` |
| Import/export | `subject.import`, `subject.export` |

Subject/curriculum management is institute-scoped; teacher assignment confers marks ownership (SUB-004).

---

## 12. Business Rule References

SUB-001 (subject identity & code), SUB-002 (curriculum mapping), SUB-003 (components & aggregation), SUB-004 (subject-teacher confers marks ownership), SUB-005 (type, credit & weightage), SUB-006 (elective selection & prerequisites), SUB-007 (retirement preserves history). Cross-cutting: CLS-006 (streams), TCH-002/003/004 (qualification/workload/ownership), EXM-002 (exam maxima/components consume SUB-003), RES-002 (layered pass/fail), GRD-004 (GPA from credit/weightage), WFL-004, AUD-001.

## 13. Use Case References

UC-SUB-001 (Define — code, type, credit), UC-SUB-002 (Map to Class/Stream), UC-SUB-003 (Components & Aggregation), UC-SUB-004 (Assign Subject-Teacher), UC-SUB-005 (Manage Electives), UC-SUB-006 (View), UC-SUB-007 (Update), UC-SUB-008 (Retire), UC-SUB-009 (Type/Credit/Weightage), UC-SUB-010 (Search), UC-SUB-011 (Curriculum Map & Report), UC-SUB-012 (Import/Bulk-Define), UC-SUB-013 (Export Curriculum), UC-SUB-014 (Elective Selection Workflow), UC-SUB-015 (Prerequisite-Not-Met Blocked), UC-SUB-016 (Component Weightage Sum Invalid).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Define subject | POST | Curriculum Admin |
| Get / list subjects | GET | Authorized roles |
| Update subject | PUT | Curriculum Admin |
| Map subject ↔ class/stream | POST/DELETE | Curriculum Admin |
| Define components & aggregation | POST/PUT | Curriculum Admin |
| Assign / change subject-teacher | POST | Academic Coordinator |
| Manage electives & prerequisites | POST/PUT | Curriculum Admin |
| Submit elective selection | POST | Student/Guardian (→ approval) |
| Retire subject | POST | Curriculum Admin |
| Curriculum map & subject report | GET | Admin |
| Import / export subjects/curriculum | POST/GET | Admin |

Component weightages are validated to sum correctly at write (SUB-003); subject-teacher assignment wires marks ownership (SUB-004).

---

## 15. Database Requirements

**Entities:** `Subject` (code, name, type, credit, weightage, status), `CurriculumMap` (subjectId, classId/streamId), `SubjectComponent` (name, maxMarks, weightage, passMark), `SubjectTeacherAssignment` (subjectId, sectionId, teacherId), `ElectiveRule` (selection, prerequisites), `ElectiveSelection` (studentId, subjectId). **Relationships:** Subject *—* Class/Stream (via map); Subject 1—* Component; Subject 1—* TeacherAssignment; Subject 1—* ElectiveSelection. **Indexes:** unique(Subject.code), index(CurriculumMap.classId), index(SubjectTeacherAssignment.teacherId, sectionId), index(ElectiveSelection.studentId). Component-weightage-sum constraint validated on write (SUB-003); retire-not-delete (SUB-007).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| In-App | Subject-teacher assignment; elective selection status; curriculum changes. |
| Email | Elective confirmation to student/guardian; major curriculum changes. |
| SMS/Push | Optional elective-deadline reminders. |

---

## 17. Audit Requirements

Log: subject define/update, curriculum mapping changes, component/weightage changes (before/after, sum-check result), subject-teacher assignment/change (ownership), elective selections, retirement. Record who/when/before/after. Ownership-conferring assignments are first-class audit events. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Curriculum map (subject × class/stream), Component/weightage structure, Subject-teacher coverage, Elective uptake, Credit distribution. **Exports:** governed curriculum export. **Dashboards:** curriculum health (unmapped subjects, unassigned subject-teachers, invalid component sums).

---

## 19. Error Handling

**Validation:** duplicate code, component weightage sum invalid, prerequisite not met → specific errors (UC-SUB-015/016). **Permission:** unqualified subject-teacher assignment → blocked (TCH-002); out-of-scope → 403. **Workflow:** elective selection pending approval → clear state. **System:** Exam/Result dependency unavailable → define allowed, computation deferred.

---

## 20. Edge Cases

**Concurrent updates:** two component edits → versioned; sum re-validated. **Duplicate data:** duplicate subject code → blocked. **Partial failures:** bulk-define/map partial → per-row report. **Rollback:** retirement reversed → subject restored, history intact. **Ownership race:** subject-teacher reassignment mid-marking → live ownership ends immediately, history attributed (TCH-004/C-05).

---

## 21. Acceptance Criteria

**Functional.** Subjects have unique codes and configurable type/credit/weightage; curriculum maps subjects to class/stream; component weightages must sum correctly; assigning a subject-teacher confers marks ownership; electives enforce selection rules and prerequisites; subjects are retired (history-preserving), never hard-deleted.

**Business.** The curriculum and assessment structure is authoritative so exams, results, and grading compute correctly; teachers own marks for what they teach; elective choice is gated and fair; historical curriculum and marks remain intact across years.

---

## 22. Future Enhancements

Visual curriculum/prerequisite-graph designer; cross-class shared-subject templates; learning-outcome/competency mapping; credit-transfer rules; auto subject-teacher suggestion by qualification/workload; syllabus/material attachment per subject; elective capacity and balloting.
