# 21 — Teacher Use Cases

Transforms the Teacher business rules (`TCH-001`…`TCH-007`) into use cases. Teacher specializes Staff: qualification-aware subject assignment, workload limits, assignment-conferred ownership (re-resolving on change), temporary substitution, mandatory handover, and class-teacher singularity.

## 1. Primary Actors
Academic Administrator / Coordinator (assigns teachers), Teacher (owns assigned classes/subjects), HR Administrator (staff base record).

## 2. Secondary Actors
System (qualification checks, workload enforcement, ownership wiring, substitution attribution, handover), Subject/Section modules (assignment targets), Authorization (ownership predicates), Audit service.

## 3. Goals
Treat teachers as specialized staff; assign subjects respecting qualifications; enforce workload limits; confer ownership through assignment that re-resolves immediately on change; support temporary, attributed substitution; mandate handover on departure/reassignment; ensure one class-teacher (homeroom) per section.

## 4. User Journeys
- **Assign:** coordinator assigns a qualified teacher to a subject/section within workload limits → teacher gains ownership (attendance/marks).
- **Cover:** for an absence, a substitute is assigned temporarily; their actions are attributed; original ownership resumes after.
- **Change/depart:** on reassignment/departure, ownership re-resolves immediately and a handover is mandated so nothing is orphaned.
- **Homeroom:** exactly one class-teacher per section is maintained.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-TCH-001 | Designate Teacher (specialize staff) | Medium |
| Core | UC-TCH-002 | Assign Subject (qualification + workload) | High |
| Core | UC-TCH-003 | Assign Class-Teacher (homeroom singularity) | High |
| Core | UC-TCH-004 | Change Assignment (ownership re-resolves) | High |
| Core | UC-TCH-005 | Assign Substitute (temporary, attributed) | High |
| Core | UC-TCH-006 | Mandatory Handover (departure/reassignment) | High |
| CRUD | UC-TCH-007 | View Teacher Assignments / Workload | Medium |
| Admin | UC-TCH-008 | Manage Qualifications | Medium |
| Approval | UC-TCH-009 | Approve Over-Workload Exception | Medium |
| Search | UC-TCH-010 | Search Teachers / Assignments | Low |
| Reporting | UC-TCH-011 | Workload & Coverage Report | Medium |
| Bulk | UC-TCH-012 | Bulk Assignment | Medium |
| Import/Export | UC-TCH-013 | Import / Export Assignments | Low |
| Workflow | UC-TCH-014 | Assignment / Handover Workflow | Medium |
| Exception | UC-TCH-015 | Unqualified Assignment Blocked | High |
| Exception | UC-TCH-016 | Workload Limit Exceeded | High |
| Exception | UC-TCH-017 | Second Class-Teacher Blocked | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-TCH-002 — Assign Subject (qualification + workload)
- **Module:** Teacher · **Priority:** High
- **Actors:** Academic Coordinator (primary), System
- **Goal:** Assign a qualified teacher to a subject/section within workload limits, conferring marks ownership.
- **Description:** Validates the teacher's qualification for the subject and that the assignment keeps them within workload limits, then confers ownership (wired into authorization).
- **Business Rules Applied:** TCH-002, TCH-003, TCH-004, SUB-004.
- **Preconditions:** Teacher (staff) exists; subject mapped to the class; coordinator authorized.
- **Trigger:** Coordinator assigns a teacher to a subject/section.
- **Main Success Scenario:**
  1. Coordinator selects teacher, subject, section.
  2. System validates qualification for the subject (TCH-002).
  3. System validates the resulting workload ≤ limit (TCH-003).
  4. System records the assignment, conferring marks ownership (TCH-004/SUB-004).
- **Alternative Flows:** A1) Over-workload with governed exception (UC-TCH-009).
- **Exception Flows:** E1) Unqualified → blocked (UC-TCH-015). E2) Workload exceeded → blocked (UC-TCH-016).
- **Validation Rules:** Qualified; within workload; subject mapped; ownership wired (TCH-002/003/004).
- **Permissions Required:** `academic.assignment.manage`.
- **Notifications Triggered:** `SUBJECT_ASSIGNED` to the teacher.
- **Audit Events Generated:** `TEACHER_ASSIGNED` (subject, section).
- **Data Created:** Assignment.
- **Data Updated:** Ownership predicates; workload total.
- **Data Deleted:** None.
- **Post Conditions:** Teacher owns the subject's marks for the section within workload.
- **Related Use Cases:** UC-SUB-004, UC-TCH-004, UC-TCH-009.
- **Acceptance Criteria:**
  - Given a qualified teacher within workload, When assigned, Then they own the subject's marks for that section.
  - Given an unqualified teacher, When assigned, Then it is blocked.
  - Given an assignment exceeding workload, When attempted, Then it is blocked (unless a governed exception is approved).
- **Edge Case Analysis:**
  - *Invalid Input:* assign to unmapped subject rejected.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* parallel assignments pushing over workload → serialized; limit enforced.
  - *Duplicate Data:* duplicate assignment idempotent.
  - *System Failure:* atomic; workload consistent.
  - *Workflow Failure:* over-workload exception pends approval.
- **QA Coverage:**
  - *Positive:* qualified within-workload assignment.
  - *Negative:* unqualified; over-workload; unmapped subject.
  - *Boundary:* workload exactly at limit; qualification edge.

### UC-TCH-004 — Change Assignment (ownership re-resolves)
- **Module:** Teacher · **Priority:** High
- **Actors:** Academic Coordinator (primary), System
- **Goal:** Reassign a subject/section so ownership transfers cleanly and immediately.
- **Description:** On assignment change, live ownership ends immediately for the prior teacher and is conferred on the new one; historical attribution of past actions is preserved (Conflict C-05).
- **Business Rules Applied:** TCH-004, TCH-006, AUTHZ-003, SUB-004.
- **Preconditions:** An existing assignment; authorized coordinator.
- **Trigger:** Coordinator changes the assignment.
- **Main Success Scenario:**
  1. Coordinator reassigns the subject/section to a new teacher.
  2. System ends the prior teacher's live ownership immediately (TCH-004/C-05).
  3. System confers ownership on the new teacher; past entries remain attributed to the prior teacher.
  4. A handover is prompted/mandated where in-progress work exists (TCH-006).
- **Alternative Flows:** A1) Temporary change → substitute path (UC-TCH-005).
- **Exception Flows:** E1) Reassignment with in-progress unsubmitted marks → handover required (TCH-006).
- **Validation Rules:** Live ownership ends immediately; historical attribution preserved; handover on in-progress work (TCH-004/006, C-05).
- **Permissions Required:** `academic.assignment.manage`.
- **Notifications Triggered:** `ASSIGNMENT_CHANGED` to both teachers.
- **Audit Events Generated:** `ASSIGNMENT_CHANGED`, `OWNERSHIP_TRANSFERRED`.
- **Data Created:** New assignment.
- **Data Updated:** Ownership predicates (immediate); prior assignment closed.
- **Data Deleted:** None.
- **Post Conditions:** New teacher owns it; prior teacher's live access ended; history intact.
- **Related Use Cases:** UC-TCH-005, UC-TCH-006, UC-SEC-004.
- **Acceptance Criteria:**
  - Given a reassignment, When made, Then the prior teacher's live ownership ends immediately and the new teacher gains it.
  - Given past mark entries, When reassigned, Then those remain attributed to the prior teacher.
  - Given in-progress work, When reassigning, Then a handover is required.
- **Edge Case Analysis:**
  - *Invalid Input:* reassign to unqualified teacher blocked (TCH-002).
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* reassignment during mark entry → prior teacher's entry stops at the transfer point.
  - *Duplicate Data:* idempotent.
  - *System Failure:* atomic ownership transfer.
  - *Workflow Failure:* handover pends.
- **QA Coverage:**
  - *Positive:* clean reassignment; ownership transfer.
  - *Negative:* post-transfer entry by prior teacher blocked; unqualified target.
  - *Boundary:* transfer exactly during an open mark sheet.

### UC-TCH-005 — Assign Substitute (temporary, attributed)
- **Module:** Teacher · **Priority:** High
- **Actors:** Academic Coordinator (primary), System
- **Goal:** Cover an absence with a temporary substitute whose actions are attributed and whose ownership reverts.
- **Description:** A substitute is granted temporary, time-boxed ownership for coverage; their attendance/mark actions are attributed as acting-for; original ownership reverts at the end.
- **Business Rules Applied:** TCH-005, TCH-004, LEV-008, AUTHZ-003.
- **Preconditions:** Original teacher unavailable (e.g., on leave); substitute qualified.
- **Trigger:** Coordinator (or leave coverage) assigns a substitute.
- **Main Success Scenario:**
  1. Coordinator assigns a substitute for a period/date range.
  2. System grants temporary, time-boxed ownership (TCH-005).
  3. Substitute's actions are attributed as acting-for the original (audit).
  4. At the end of the period, ownership reverts to the original teacher.
- **Alternative Flows:** A1) Substitution auto-coordinated from approved leave (LEV-008).
- **Exception Flows:** E1) Substitute unqualified → blocked. E2) Overlap beyond the window → auto-revert.
- **Validation Rules:** Time-boxed; qualified substitute; attributed; reverts (TCH-005, LEV-008).
- **Permissions Required:** `academic.assignment.manage`.
- **Notifications Triggered:** `SUBSTITUTE_ASSIGNED` to both teachers.
- **Audit Events Generated:** `SUBSTITUTE_ASSIGNED/REVERTED` (acting-for attribution).
- **Data Created:** Temporary assignment.
- **Data Updated:** Ownership (temporary); reverts on expiry.
- **Data Deleted:** None.
- **Post Conditions:** Coverage provided; actions attributed; ownership reverts.
- **Related Use Cases:** UC-LEV (coverage), UC-TCH-004.
- **Acceptance Criteria:**
  - Given a substitute assignment, When active, Then the substitute can act and their actions are attributed as acting-for the original.
  - Given the window ends, When reached, Then ownership reverts automatically.
  - Given an unqualified substitute, When assigned, Then it is blocked.
- **Edge Case Analysis:**
  - *Invalid Input:* invalid date range rejected.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* substitute + original both acting → window governs authority.
  - *Duplicate Data:* idempotent.
  - *System Failure:* revert reliable (no stuck temporary ownership).
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* substitute coverage; auto-revert; leave-coordinated.
  - *Negative:* unqualified; overlap beyond window.
  - *Boundary:* revert exactly at window end.

---

## 7. Compact Specifications (routine use cases)

- **UC-TCH-001 — Designate Teacher (specialize staff)** · *Medium* · Rules: TCH-001, STF-001. Mark a staff member as a teacher (specialization). *Edge:* teacher is-a staff; inherits staff rules. *QA:* designate; inheritance.
- **UC-TCH-003 — Assign Class-Teacher (homeroom singularity)** · *High* · Rules: TCH-007, SEC-004. Assign exactly one class-teacher per section. *Audit:* `CLASS_TEACHER_ASSIGNED`. *Edge:* second class-teacher blocked (UC-TCH-017). *QA:* assign; singularity.
- **UC-TCH-006 — Mandatory Handover (departure/reassignment)** · *High* · Rules: TCH-006, STF-008. Mandate handover of owned responsibilities on departure/reassignment. *Audit:* `HANDOVER_COMPLETED`. *Edge:* blocks separation/reassignment completion until handed over. *QA:* handover required; no orphaned ownership.
- **UC-TCH-007 — View Teacher Assignments / Workload** · *Medium* · Rules: AUTHZ-003, TCH-003. Scoped view of assignments/workload. *QA:* scope; workload accuracy.
- **UC-TCH-008 — Manage Qualifications** · *Medium* · Rules: TCH-002. Maintain teacher qualifications (drives assignment eligibility). *Audit:* `QUALIFICATION_UPDATED`. *Edge:* qualification change affects future assignments. *QA:* qualification CRUD; assignment effect.
- **UC-TCH-009 — Approve Over-Workload Exception** · *Medium* · Rules: TCH-003, WFL-004. Approver authorizes a bounded over-workload exception. *Edge:* SoD; bounded; audited. *QA:* approval; bound; self-approval blocked.
- **UC-TCH-010 — Search Teachers / Assignments** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-TCH-011 — Workload & Coverage Report** · *Medium* · Rules: REP-002, TCH-003. Workload distribution, coverage gaps. *Edge:* scoped; identifies over/under-load. *QA:* workload accuracy; coverage gaps.
- **UC-TCH-012 — Bulk Assignment** · *Medium* · Rules: TCH-002/003/004. Bulk assign (qualification/workload enforced per row). *Edge:* partial-failure report; limits per teacher. *QA:* clean bulk; qualification/workload.
- **UC-TCH-013 — Import / Export Assignments** · *Low* · Rules: CFG-002, REP-005. Import/export assignments (validated). *QA:* valid import; scoped export.
- **UC-TCH-014 — Assignment / Handover Workflow** · *Medium* · Rules: WFL-002/004, TCH-006. Version-pinned assignment/handover. *QA:* pinning; handover gating.
- **UC-TCH-015 — Unqualified Assignment Blocked (Exception)** · *High* · Rules: TCH-002. Assigning an unqualified teacher blocked. *QA:* blocked; qualified allowed.
- **UC-TCH-016 — Workload Limit Exceeded (Exception)** · *High* · Rules: TCH-003. Assignment beyond workload blocked. *QA:* blocked; at-limit allowed; exception path.
- **UC-TCH-017 — Second Class-Teacher Blocked (Exception)** · *High* · Rules: TCH-007. Only one class-teacher per section. *QA:* second blocked; replace path.

## 8. Module-level QA & Edge Themes
- **Ownership re-resolution (TCH-004 / C-05):** assignment confers ownership; change ends live ownership immediately while preserving historical attribution — the headline suite shared with Subject/Section.
- **Qualification + workload (TCH-002/003):** assignment gates on competence and capacity; exceptions are governed.
- **Attributed substitution (TCH-005):** temporary, time-boxed, acting-for ownership that reverts cleanly.
- **Mandatory handover (TCH-006):** no orphaned ownership on departure/reassignment.
- **Homeroom singularity (TCH-007):** exactly one class-teacher per section.
