# 13 — Subject Use Cases

Transforms the Subject business rules (`SUB-001`…`SUB-007`) into use cases. Subjects map to classes/streams, decompose into components, confer marks ownership to subject-teachers, and support electives with prerequisites.

## 1. Primary Actors
Academic Administrator / Curriculum Coordinator (defines subjects & curriculum), Subject-Teacher (owns marks for assigned subjects).

## 2. Secondary Actors
System (identity/codes, curriculum mapping, component aggregation, ownership wiring, history-preserving retirement), Examination/Grading modules (consume components/weightage), Configuration Engine, Audit service.

## 3. Goals
Define subjects with stable codes; map them to classes/streams; decompose into components with aggregation; assign subject-teachers who own the subject's marks; classify type/credit/weightage; manage electives with selection rules and prerequisites; retire subjects without losing history.

## 4. User Journeys
- **Define curriculum:** coordinator creates subjects (code, type, credit) → maps them to classes/streams → splits into components (theory/practical) with weightage.
- **Staff marks ownership:** assign a subject-teacher per class/section → they own mark entry for that subject.
- **Electives:** define elective groups with selection rules and prerequisites → students choose within rules.
- **Evolve:** retire an outdated subject → it stops applying to new sessions while historical results remain intact.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core/CRUD | UC-SUB-001 | Define Subject (code, type, credit) | High |
| Core | UC-SUB-002 | Map Subject to Class / Stream (curriculum) | High |
| Core | UC-SUB-003 | Define Subject Components & Aggregation | High |
| Core | UC-SUB-004 | Assign Subject-Teacher (marks ownership) | High |
| Core | UC-SUB-005 | Manage Electives (selection & prerequisites) | High |
| CRUD | UC-SUB-006 | View Subject(s) | Medium |
| CRUD | UC-SUB-007 | Update Subject | Medium |
| Core | UC-SUB-008 | Retire Subject (history-preserving) | Medium |
| Admin | UC-SUB-009 | Set Type, Credit & Weightage | Medium |
| Search | UC-SUB-010 | Search Subjects | Low |
| Reporting | UC-SUB-011 | Curriculum Map & Subject Report | Medium |
| Bulk/Import | UC-SUB-012 | Import / Bulk-Define Subjects | Medium |
| Export | UC-SUB-013 | Export Curriculum | Low |
| Workflow | UC-SUB-014 | Elective Selection Workflow | Medium |
| Exception | UC-SUB-015 | Prerequisite-Not-Met Blocked | High |
| Exception | UC-SUB-016 | Component Weightage Sum Invalid | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-SUB-003 — Define Subject Components & Aggregation
- **Module:** Subject · **Priority:** High
- **Actors:** Curriculum Coordinator (primary), System
- **Goal:** Decompose a subject into components (e.g., theory/practical/internal) whose weightages aggregate correctly into the subject result.
- **Description:** Defines components with marks and weightages; the system validates the aggregation is coherent (weightages sum as required) so downstream grading/results compute consistently.
- **Business Rules Applied:** SUB-003, SUB-005, GRD-002 (consumer consistency), CFG-002.
- **Preconditions:** Subject defined; actor authorized.
- **Trigger:** Coordinator defines/edits components.
- **Main Success Scenario:**
  1. Coordinator adds components (name, max marks, weightage, pass criteria).
  2. System validates the weightage/aggregation rule (e.g., sums to 100% or the configured total).
  3. The aggregation definition is saved and consumed by exam/grading.
- **Alternative Flows:** A1) Component-level pass requirement (must pass practical separately).
- **Exception Flows:** E1) Weightages inconsistent with the required total → blocked (UC-SUB-016).
- **Validation Rules:** Component weightages valid and coherent; aggregation rule consistent with grading (SUB-003/005).
- **Permissions Required:** `curriculum.manage`.
- **Notifications Triggered:** None routine.
- **Audit Events Generated:** `SUBJECT_COMPONENTS_DEFINED`.
- **Data Created:** Component definitions.
- **Data Updated:** Subject aggregation config.
- **Data Deleted:** None.
- **Post Conditions:** Coherent component/aggregation drives exam & result computation.
- **Related Use Cases:** UC-EXM (components), UC-RES (aggregation), UC-GRD.
- **Acceptance Criteria:**
  - Given components whose weightages satisfy the required total, When saved, Then the aggregation is accepted.
  - Given inconsistent weightages, When saved, Then it is blocked with the discrepancy.
  - Given a component-level pass rule, When defined, Then results enforce it.
- **Edge Case Analysis:**
  - *Invalid Input:* negative/over-100 weightage rejected.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* two edits → versioned; last coherent set wins.
  - *Duplicate Data:* duplicate component name in subject rejected.
  - *System Failure:* atomic; partial component set not saved.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* theory+practical sums correctly; component pass rule.
  - *Negative:* weightage sum invalid; duplicate component.
  - *Boundary:* single 100% component; many small components summing exactly.

### UC-SUB-004 — Assign Subject-Teacher (marks ownership)
- **Module:** Subject · **Priority:** High
- **Actors:** Academic Coordinator (primary), System
- **Goal:** Give a teacher ownership of mark entry for a subject in a specific class/section.
- **Description:** Assigns a subject-teacher; the assignment confers ownership of that subject's marks (entry/edit within window) for the section, wired into authorization ownership predicates.
- **Business Rules Applied:** SUB-004, AUTHZ-003, TCH-004, EXM (mark entry).
- **Preconditions:** Subject mapped to the class; teacher exists; actor authorized.
- **Trigger:** Coordinator assigns a subject-teacher.
- **Main Success Scenario:**
  1. Coordinator assigns teacher T to subject S for section X.
  2. System records the assignment, conferring marks ownership for S in X (SUB-004).
  3. T can enter/edit marks for S in X within the marking window; others cannot.
- **Alternative Flows:** A1) Co-teachers/multiple sections handled as distinct assignments.
- **Exception Flows:** E1) Unassignment ends live marks ownership immediately; historical attribution preserved (C-05). E2) Assigning a teacher outside scope → blocked.
- **Validation Rules:** Subject mapped to the class; teacher in scope; ownership wired (SUB-004, AUTHZ-003).
- **Permissions Required:** `curriculum.manage` / `academic.assignment.manage`.
- **Notifications Triggered:** `SUBJECT_TEACHER_ASSIGNED` to the teacher.
- **Audit Events Generated:** `SUBJECT_TEACHER_ASSIGNED/CHANGED`.
- **Data Created:** Assignment.
- **Data Updated:** Ownership predicates for marks.
- **Data Deleted:** None.
- **Post Conditions:** Teacher owns the subject's marks for the section; enforced by authz.
- **Related Use Cases:** UC-SEC-004, UC-EXM, UC-TCH.
- **Acceptance Criteria:**
  - Given an assignment, When made, Then the teacher can enter marks for that subject/section and others cannot.
  - Given an unassignment, When made, Then live marks ownership ends immediately while past entries remain attributed.
  - Given an out-of-scope teacher, When assigned, Then it is blocked.
- **Edge Case Analysis:**
  - *Invalid Input:* assigning to an unmapped subject rejected.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* reassign during mark entry → ownership transfers cleanly; in-flight entry by old owner stops.
  - *Duplicate Data:* duplicate assignment idempotent.
  - *System Failure:* atomic; ownership consistent.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* assign; multi-section; co-teacher.
  - *Negative:* out-of-scope; unmapped subject; post-unassign entry blocked.
  - *Boundary:* ownership transfer exactly at reassignment.

### UC-SUB-005 — Manage Electives (selection & prerequisites)
- **Module:** Subject · **Priority:** High
- **Actors:** Curriculum Coordinator (config), Student/Guardian (selection), System
- **Goal:** Offer elective groups with selection rules and prerequisites, and let students choose within the rules.
- **Description:** Defines elective groups (choose N of M), prerequisites, and capacity; students select within rules; prerequisite and capacity violations are blocked.
- **Business Rules Applied:** SUB-006, SUB-002, ENR-003 (capacity analogue).
- **Preconditions:** Electives configured; student eligible/enrolled.
- **Trigger:** Coordinator configures electives; student selects.
- **Main Success Scenario:**
  1. Coordinator defines an elective group (choose N of M), prerequisites, and per-elective capacity.
  2. Student selects electives within the rules.
  3. System validates count, prerequisites, and capacity, then records the selection.
- **Alternative Flows:** A1) Selection routed through an approval/advisor workflow (UC-SUB-014).
- **Exception Flows:** E1) Prerequisite not met → block (UC-SUB-015). E2) Elective full → block/waitlist. E3) Wrong count (not N) → block.
- **Validation Rules:** Exactly N chosen; prerequisites satisfied; capacity available (SUB-006).
- **Permissions Required:** Student/guardian (own selection); `curriculum.manage` (config).
- **Notifications Triggered:** `ELECTIVE_SELECTED/CONFIRMED`.
- **Audit Events Generated:** `ELECTIVE_SELECTION_RECORDED`.
- **Data Created:** Elective selection.
- **Data Updated:** Elective capacity counts.
- **Data Deleted:** None.
- **Post Conditions:** Valid elective selection recorded; curriculum personalized.
- **Related Use Cases:** UC-SUB-002, UC-SUB-015.
- **Acceptance Criteria:**
  - Given an elective group requiring N choices, When a student selects exactly N meeting prerequisites with capacity, Then the selection is recorded.
  - Given an unmet prerequisite, When selected, Then it is blocked.
  - Given a full elective, When selected, Then it is blocked or waitlisted.
- **Edge Case Analysis:**
  - *Invalid Input:* selecting more/fewer than N rejected.
  - *Permission Failure:* selecting for another student → denied.
  - *Concurrent Update:* two students for the last elective seat → one succeeds (capacity).
  - *Duplicate Data:* duplicate selection idempotent.
  - *System Failure:* atomic; capacity consistent.
  - *Workflow Failure:* advisor approval pends.
- **QA Coverage:**
  - *Positive:* valid N-of-M with prerequisites/capacity.
  - *Negative:* prerequisite unmet; full elective; wrong count.
  - *Boundary:* last seat under concurrency; exactly meeting prerequisite.

---

## 7. Compact Specifications (routine use cases)

- **UC-SUB-001 — Define Subject (code, type, credit)** · *High* · Rules: SUB-001, SUB-005. Create subject with stable code, type, credit. *Permissions:* `curriculum.manage`. *Audit:* `SUBJECT_DEFINED`. *Edge:* code unique/stable; type valid. *QA:* define; duplicate code; invalid type.
- **UC-SUB-002 — Map Subject to Class / Stream (curriculum)** · *High* · Rules: SUB-002. Map subject to class(es)/stream(s) as core/elective. *Audit:* `CURRICULUM_MAPPED`. *Edge:* stream-specific mapping; unmapped subject not teachable. *QA:* mapping; stream specificity.
- **UC-SUB-006 — View Subject(s)** · *Medium* · Rules: AUTHZ-002. Scoped read. *QA:* scope respected.
- **UC-SUB-007 — Update Subject** · *Medium* · Rules: SUB-001/005. Edit details; code stable; edits don't alter historical results. *Audit:* `SUBJECT_UPDATED`. *Edge:* history fidelity. *QA:* edit forward; history kept.
- **UC-SUB-008 — Retire Subject (history-preserving)** · *Medium* · Rules: SUB-007. Retire from future mapping; historical results preserved. *Edge:* cannot delete with historical results. *QA:* retire blocks future; history kept.
- **UC-SUB-009 — Set Type, Credit & Weightage** · *Medium* · Rules: SUB-005. Configure type/credit/weightage feeding grading. *Edge:* weightage consistent with grading. *QA:* config consistency.
- **UC-SUB-010 — Search Subjects** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-SUB-011 — Curriculum Map & Subject Report** · *Medium* · Rules: REP-002. Curriculum mapping/subject coverage report. *Edge:* scoped; as-of. *QA:* mapping accuracy.
- **UC-SUB-012 — Import / Bulk-Define Subjects** · *Medium* · Rules: SUB-001/002/003. Import subjects + mappings + components (validated). *Edge:* invalid weightage/duplicate code rejected. *QA:* clean import; invalid rows.
- **UC-SUB-013 — Export Curriculum** · *Low* · Rules: REP-005. Export curriculum map. *QA:* scoped export.
- **UC-SUB-014 — Elective Selection Workflow** · *Medium* · Rules: WFL-002/004, SUB-006. Version-pinned advisor approval for selections. *QA:* pinning; approval; escalation.
- **UC-SUB-015 — Prerequisite-Not-Met Blocked (Exception)** · *High* · Rules: SUB-006. Selection blocked when prerequisites unmet. *QA:* blocked; met allowed.
- **UC-SUB-016 — Component Weightage Sum Invalid (Exception)** · *High* · Rules: SUB-003. Incoherent weightage sum blocked. *QA:* invalid sum blocked; coherent allowed.

## 8. Module-level QA & Edge Themes
- **Aggregation coherence (SUB-003):** component weightages must aggregate correctly — the upstream guarantee for grading/results consistency.
- **Marks ownership (SUB-004 / C-05):** subject-teacher assignment confers marks ownership; unassignment ends live ownership immediately, history preserved.
- **Electives integrity (SUB-006):** count, prerequisite, and capacity rules — a rich negative/boundary suite.
- **History-preserving retirement (SUB-007):** retiring a subject never erases historical results.
