# 21 — Teacher Business Rules

> **Prefix note:** HR-phase prefixes are distinct — Staff `STF`, Teacher `TCH`, HR/Payroll `HR`, Leave `LEV` — to keep rule IDs unique and testable.

## 1. Module Purpose
Govern the **teacher** — a **Staff member specialized with teaching responsibilities** (Doc 22 is the base employment record; this module adds the teaching layer). The teacher record owns qualifications and subject expertise, **subject/section assignments** (the authoritative *ownership source* for attendance ATT-002/003, marks EXM-005, class-teacher duties SEC-004, and subject duties SUB-004), workload limits, and **substitute handling** (temporary, audited transfer of ownership). When a teacher is assigned or unassigned, downstream access and ownership re-resolve immediately.

## 2. Actors
- **Academic / HR Administrator** — manages teacher profiles, qualifications, and assignments.
- **Head of Department / Academic Coordinator** — proposes/approves teaching assignments and substitutes.
- **Teacher** — reads own profile, schedule, assignments, and assigned rosters.
- **Substitute Teacher** — temporarily holds another teacher's ownership for a defined period.
- **System** — enforces assignment validity, workload limits, ownership resolution, and substitute governance.

## 3. Use Cases

**Use Case ID:** UC-TCH-01 — Assign a Teacher to Subjects/Sections
**Actors:** Academic Coordinator, System
**Description:** Give a teacher authoritative responsibility for teaching (and thus ownership of) specific subject-section combinations.
**Preconditions:** Teacher is `ACTIVE` staff (Doc 22); subject mapped to the class (SUB-002); within scope.
**Main Flow:** 1) Coordinator assigns (teacher, subject, section) and/or class-teacher (homeroom) duty. 2) System validates the curriculum mapping, scope, and workload limits. 3) Assignment confers ownership (attendance/marks/roster) per AUTHZ-003.
**Alternative Flow:** A1) Class-teacher (homeroom) assignment for daily attendance ownership (SEC-004).
**Exception Flow:** E1) Subject not mapped to the section's class → reject (SUB-002). E2) Workload limit exceeded → warn/block (TCH-003).
**Post Conditions:** Assignment active; ownership established; audited.
**Business Rules Applied:** TCH-001, TCH-002, TCH-003.

**Use Case ID:** UC-TCH-02 — Assign a Substitute (Temporary Ownership Transfer)
**Actors:** Coordinator, System
**Description:** Cover a teacher's duties temporarily (leave, absence) without losing accountability.
**Preconditions:** A covered teacher is unavailable; a qualified substitute is available.
**Main Flow:** 1) Coordinator assigns a substitute for a defined period and scope. 2) System grants the substitute temporary ownership (attendance/marks) for that period, clearly attributed as substitute action. 3) On period end, ownership reverts to the original teacher.
**Exception Flow:** E1) Substitute lacks scope/qualification → reject. E2) Overlapping permanent + substitute marks → both attributed distinctly (TCH-004).
**Post Conditions:** Temporary, audited ownership transfer; auto-reversion; full attribution.
**Business Rules Applied:** TCH-004, TCH-005.

**Use Case ID:** UC-TCH-03 — Reassign on Teacher Departure / Mid-Term Change
**Actors:** Academic / HR Administrator, System
**Description:** Hand over a departing or reassigned teacher's duties so nothing is orphaned.
**Preconditions:** Teacher separating (STF-008) or being reassigned.
**Main Flow:** 1) System lists the teacher's open assignments and pending mark/attendance duties. 2) Admin reassigns each to other teachers (or substitutes). 3) Original teacher's live ownership ends; their historical actions remain audited.
**Exception Flow:** E1) Open locked-pending marks → must be reassigned/completed before separation finalizes (STF-008).
**Post Conditions:** Duties handed over; continuity preserved; audited.
**Business Rules Applied:** TCH-005, TCH-006.

## 4. Business Rules

**Rule ID:** TCH-001
**Rule Name:** Teacher Specializes Staff
**Description:** A teacher record is a staff record (Doc 22) extended with teaching attributes; it is not a separate person entity.
**Priority:** High
**Category:** Data model integrity
**Preconditions:** Teacher designation.
**Business Rule:** Becoming a teacher adds qualifications, subject expertise, and teaching assignments to an existing staff record; employment lifecycle, access, and sensitive-data rules are inherited from Staff/HR. One person is never both a separate staff and a separate teacher record.
**System Action:** Extend the staff record with the teaching layer; inherit lifecycle.
**Validation:** Underlying staff record exists and is active.
**Failure Behavior:** Block teacher attributes on a non-existent/inactive staff record.
**Audit Requirement:** Log `TEACHER_PROFILE_CREATED` linked to the staff record.
**Example Scenario:** Promoting a staff member to teaching adds assignments without duplicating their identity.
**Related Rules:** STF-001, STF-004, HR (lifecycle).

**Rule ID:** TCH-002
**Rule Name:** Qualification-Aware Subject Assignment
**Description:** Teaching assignments respect curriculum mapping, scope, and (where configured) qualification suitability.
**Priority:** High
**Category:** Assignment integrity
**Preconditions:** Assignment creation.
**Business Rule:** A (teacher, subject, section) assignment requires the subject to be curriculum-mapped to the section's class (SUB-002) and the teacher to be in scope; qualification/expertise checks may warn or block per policy. Cross-campus assignment is explicit (STF-005).
**System Action:** Validate mapping/scope/qualification; create the assignment.
**Validation:** Subject mapped; teacher in scope; qualification policy satisfied.
**Failure Behavior:** Reject unmapped/out-of-scope assignments; warn/block on qualification.
**Audit Requirement:** Log `TEACHING_ASSIGNMENT_CREATED`.
**Example Scenario:** A physics-qualified teacher is assigned Physics for 11-A (Science stream).
**Related Rules:** SUB-002, SUB-004, STF-005.

**Rule ID:** TCH-003
**Rule Name:** Workload Limits
**Description:** Teacher assignments respect configurable workload limits (max sections/subjects/periods).
**Priority:** Medium
**Category:** Fairness / feasibility
**Preconditions:** Assignment.
**Business Rule:** Configurable limits cap how many sections/subjects/periods a teacher may hold; exceeding warns or blocks per policy to prevent over-allocation and protect timetable feasibility.
**System Action:** Track current load; enforce limits on assignment.
**Validation:** Limits defined; current load computed.
**Failure Behavior:** Warn/block over-limit assignment per policy.
**Audit Requirement:** Log workload-limit warnings/blocks.
**Example Scenario:** A teacher at the maximum period load cannot take another section without an override.
**Related Rules:** TCH-002, timetable (future).

**Rule ID:** TCH-004
**Rule Name:** Assignment Confers Ownership (Re-Resolves on Change)
**Description:** A teaching assignment is the authoritative ownership source for attendance, marks, and roster; changes re-resolve ownership immediately.
**Priority:** Critical
**Category:** Authorization (ownership)
**Preconditions:** Assignment created/changed/ended.
**Business Rule:** Ownership (ATT-002/003 attendance, EXM-005 marks, SEC-004 class-teacher roster, SUB-004 subject) derives from current assignments; ending an assignment ends live ownership at once, while historical actions remain attributed to the actor who performed them.
**System Action:** Re-resolve ownership on every assignment change; preserve historical attribution.
**Validation:** Assignment current; ownership derived consistently.
**Failure Behavior:** No stale ownership after unassignment.
**Audit Requirement:** Log assignment changes; ownership re-resolution implicit.
**Example Scenario:** Unassigning a teacher from a section immediately stops their attendance access there; their prior marks stay theirs in the record.
**Related Rules:** AUTHZ-003, SEC-004, SUB-004, EXM-005, ATT-002.

**Rule ID:** TCH-005
**Rule Name:** Substitute = Temporary, Attributed Ownership Transfer
**Description:** A substitute holds another teacher's ownership for a defined period, with all actions distinctly attributed, reverting automatically.
**Priority:** High
**Category:** Continuity / integrity
**Preconditions:** Substitution.
**Business Rule:** A substitute receives time-boxed ownership over the covered scope; their attendance/mark entries are attributed to them as substitute (not silently to the original teacher); ownership reverts on period end. Accountability is never ambiguous.
**System Action:** Grant time-boxed ownership; attribute substitute actions; auto-revert.
**Validation:** Substitute in scope/qualified; period defined.
**Failure Behavior:** Reject unqualified/out-of-scope substitutes; block indefinite substitution.
**Audit Requirement:** Log `SUBSTITUTE_ASSIGNED/REVERTED` with attribution.
**Example Scenario:** A substitute marks attendance during a teacher's leave; the register shows the substitute as the marker for those days.
**Related Rules:** TCH-004, LEV (leave), ATT-002.

**Rule ID:** TCH-006
**Rule Name:** Mandatory Handover on Departure/Reassignment
**Description:** A teacher's open duties must be reassigned before their access ends; no orphaned sections or pending marks.
**Priority:** High
**Category:** Continuity
**Preconditions:** Teacher separation/reassignment.
**Business Rule:** Before a teacher's access ends, open teaching assignments and pending attendance/mark duties are reassigned/handed over; uncompleted locked-pending marks must be completed or reassigned. Reinforces STF-008.
**System Action:** Enumerate open duties; require reassignment; then finalize departure.
**Validation:** No open ownership remains at finalization.
**Failure Behavior:** Block finalization with orphaned duties.
**Audit Requirement:** Log handover of each assignment.
**Example Scenario:** A teacher leaving mid-term has all sections reassigned and pending marks completed before access is revoked.
**Related Rules:** STF-008, EXM-009, TCH-004.

**Rule ID:** TCH-007
**Rule Name:** Class-Teacher (Homeroom) Singularity
**Description:** A section has one class teacher of record at a time (optional co-teacher per policy); the role drives daily-attendance and pastoral ownership.
**Priority:** Medium
**Category:** Ownership clarity
**Preconditions:** Class-teacher assignment.
**Business Rule:** Exactly one class teacher per section at a time owns daily attendance and section pastoral duties (SEC-004); reassignment transfers the role cleanly; a co-teacher, if enabled, has a defined secondary scope.
**System Action:** Enforce single class teacher; transfer cleanly; support optional co-teacher.
**Validation:** One active class teacher per section.
**Failure Behavior:** Block a second concurrent class teacher (unless co-teacher policy).
**Audit Requirement:** Log `CLASS_TEACHER_ASSIGNED/CHANGED`.
**Example Scenario:** Section 7-B has one homeroom teacher responsible for its daily register.
**Related Rules:** SEC-004, ATT-002.

## 5. Validation Rules
- Teacher extends an existing active staff record (no duplicate person).
- Assignments require curriculum mapping, scope, and qualification policy satisfaction.
- Workload within configured limits (or override).
- Ownership re-resolves on every assignment change; no stale access.
- Substitutes are time-boxed, qualified, attributed; auto-revert.
- Departure requires complete handover; single class teacher per section.

## 6. State Machine
*(Teaching-assignment lifecycle; the teacher's employment lifecycle is Staff/HR.)*

**State Name:** ASSIGNED
**Description:** Assignment created; ownership active.
**Allowed Transitions:** → SUBSTITUTED (temporary cover); → ENDED (reassigned/term over).
**Forbidden Transitions:** ownership without a valid mapping/scope.
**System Actions:** Confer ownership; enforce workload.

**State Name:** SUBSTITUTED
**Description:** A substitute temporarily holds ownership.
**Allowed Transitions:** → ASSIGNED (revert on period end); → ENDED.
**Forbidden Transitions:** indefinite substitution.
**System Actions:** Time-boxed ownership; attribute substitute actions.

**State Name:** ENDED
**Description:** Assignment concluded (reassignment/term end/departure).
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** revived ownership (a new assignment is created instead).
**System Actions:** End live ownership; preserve historical attribution.

**State Name:** ARCHIVED
**Description:** Historical assignment record.
**Allowed Transitions:** none.
**Forbidden Transitions:** edits.
**System Actions:** Read-only retention.

## 7. Status Definitions
Assignment: `ASSIGNED` · `SUBSTITUTED` · `ENDED` · `ARCHIVED`. Teaching roles: `SUBJECT_TEACHER` · `CLASS_TEACHER` · `CO_TEACHER` · `SUBSTITUTE`. (Teacher employment status is inherited from Staff: `ONBOARDING`/`ACTIVE`/`SUSPENDED`/...).

## 8. Workflow Rules
- Teaching assignments may require HoD/coordinator approval (configurable).
- Substitute assignments are typically expedited but time-boxed and audited.
- Departure/reassignment runs the handover workflow (TCH-006/STF-008) before access revocation.
- Workload-override needs elevated authorization.

## 9. Permission Rules
- `teacher.profile.manage` — manage qualifications/teaching attributes.
- `teacher.assignment.manage` — create/change teaching and class-teacher assignments.
- `teacher.substitute.manage` — assign substitutes.
- `teacher.view` — read teacher profiles/assignments (scoped; teachers read own).
- Assignment management is scoped to institute/campus.

## 10. Notification Rules
- `TEACHING_ASSIGNMENT_CREATED/ENDED` → notify the teacher.
- `SUBSTITUTE_ASSIGNED` → notify both the substitute and (where appropriate) the covered teacher.
- `CLASS_TEACHER_CHANGED` → notify the teacher and affected section's guardians per policy.
- Handover tasks on departure → notify the receiving teachers.

## 11. Audit Requirements
Mandatory: `TEACHER_PROFILE_CREATED`, `TEACHING_ASSIGNMENT_CREATED/ENDED`, workload warnings/overrides, `SUBSTITUTE_ASSIGNED/REVERTED` (attribution), `CLASS_TEACHER_ASSIGNED/CHANGED`, handover records. Ownership-bearing changes are fully attributable.

## 12. Data Retention Rules
- Teaching-assignment history retained (who taught/owned what, when) — needed for academic-record accountability and safeguarding inquiries.
- Substitute attributions retained to preserve accurate authorship of attendance/marks.
- Retention tied to academic-record retention; anonymization follows the staff record (Doc 22).

## 13. Edge Cases
- **Teacher leaves mid-term:** mandatory handover; pending marks completed/reassigned before access ends (TCH-006).
- **Substitute marks attendance/marks:** attributed to the substitute, not silently to the original teacher (TCH-005).
- **Unassignment mid-session:** live ownership ends immediately; historical actions remain the original actor's (TCH-004).
- **Cross-campus teaching:** explicit multi-campus assignment; ownership does not silently span campuses (STF-005).
- **Co-teaching:** secondary ownership scope defined to avoid conflicting double-marks (TCH-007).
- **Teacher also a guardian of a student they teach:** staff/teacher scope and guardian scope kept distinct; conflict-of-interest noted for grading (links GRD/RES override governance).
- **Overlapping substitute and returned teacher on the same day:** both periods attributed distinctly.

## 14. Failure Scenarios
- **Assignment to an unmapped subject/out-of-scope section:** rejected (TCH-002).
- **Over-workload assignment:** warned/blocked (TCH-003).
- **Departure with orphaned duties:** blocked (TCH-006).
- **Indefinite/unqualified substitution:** rejected (TCH-005).
- **Stale ownership after unassignment:** prevented by re-resolution (TCH-004).

## 15. Exception Handling Rules
- Assignments validated against mapping/scope/qualification/workload before commit.
- Ownership always derives from current assignments; never stale.
- Substitutes are bounded and attributed; accountability is unambiguous.
- Departures gate on complete handover.

## 16. Compliance Considerations
- **Academic accountability:** authorship of attendance/marks is always traceable to the responsible (or substitute) teacher — vital for grade-integrity and safeguarding.
- **Continuity:** mandatory handover protects students from orphaned duties on staff change.
- **Conflict of interest:** a teacher related to/guardian of a student they assess is flagged for grading governance (RES/GRD overrides).
- **Minors:** teacher access to student rosters/marks is ownership-scoped and audited.

## 17. Future Considerations
- Timetable/period scheduling bound to assignments.
- Skills-based auto-suggestion for substitutes.
- Teacher performance linked to defined, fair metrics (Doc 23).
- Cross-institute teaching within a deployment (governed multi-employment).
