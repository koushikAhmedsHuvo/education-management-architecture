# 13 — Subject Business Rules

## 1. Module Purpose
Govern the **subject** — a course of study taught and assessed (Mathematics, Physics, Quran, English). Subjects are defined per institute, mapped to classes via **curriculum**, taught by assigned teachers per section, and assessed under grading schemes (Doc 16). This module owns subject identity, types (core/elective), **components** (theory/practical), credit/weightage, language-option grouping, stream applicability, and the rules that make assessment and results coherent downstream.

## 2. Actors
- **Institute Administrator / Academic Coordinator** — defines subjects, curriculum mapping, components, and credits.
- **Subject Teacher** — assigned to teach a subject in a section (owns its marks/attendance for period mode).
- **Student / Guardian** — may select electives where offered (read-scoped to own).
- **System** — enforces curriculum integrity, component aggregation, credit rules, and assessment coherence.

## 3. Use Cases

**Use Case ID:** UC-SUB-01 — Define a Subject and Map to Classes
**Actors:** Academic Coordinator, System
**Description:** Create a subject and map it to the classes/streams that study it.
**Preconditions:** Institute `ACTIVE`; classes defined (Doc 11).
**Main Flow:** 1) Define subject (name/code, type core/elective, components, credit/weightage). 2) Map to classes (and streams) via curriculum, marking mandatory/optional per class. 3) Subject becomes assessable for mapped classes.
**Alternative Flow:** A1) Subject with components (theory + practical) defined with per-component weightage (SUB-003).
**Exception Flow:** E1) Duplicate subject code → reject. E2) Mapping to a non-existent class/stream → reject.
**Post Conditions:** Subject defined and curriculum-mapped; audited.
**Business Rules Applied:** SUB-001, SUB-002, SUB-003.

**Use Case ID:** UC-SUB-02 — Assign Subject Teacher to a Section
**Actors:** Administrator, System
**Description:** Assign a teacher to teach a subject in a specific section.
**Preconditions:** Subject mapped to the section's class; teacher is active staff.
**Main Flow:** 1) Admin assigns (subject, section, teacher). 2) The assignment confers ownership over that subject's marks and (period-mode) attendance for that section (AUTHZ-003). 3) Timetable/assessment reference the assignment.
**Exception Flow:** E1) Subject not mapped to the section's class → reject. E2) Teacher over assignment limit → warn/block.
**Post Conditions:** Subject-teacher ownership established; audited.
**Business Rules Applied:** SUB-004.

**Use Case ID:** UC-SUB-03 — Elective Selection
**Actors:** Student/Guardian (or admin on their behalf), System
**Description:** A student selects an elective where the curriculum offers choices.
**Preconditions:** Class offers electives; selection window open; prerequisites met.
**Main Flow:** 1) Student selects from the offered elective group within limits. 2) System validates prerequisites and capacity. 3) The student's subject set for the session is recorded.
**Exception Flow:** E1) Prerequisite unmet → block (SUB-006). E2) Elective full → waitlist/alternative.
**Post Conditions:** Student subject set recorded; assessment scoped to it; audited.
**Business Rules Applied:** SUB-005, SUB-006.

## 4. Business Rules

**Rule ID:** SUB-001
**Rule Name:** Subject Identity & Code
**Description:** Each subject has a unique code within the institute and a terminology-resolvable name.
**Priority:** High
**Category:** Identity
**Preconditions:** Subject creation/edit.
**Business Rule:** Subject code is unique within the institute and immutable; the display name is terminology/i18n-resolvable; subjects are institute-scoped (two institutes may both have "Math").
**System Action:** Enforce uniqueness; lock code.
**Validation:** Unique code within institute; valid name/label.
**Failure Behavior:** Reject duplicate codes.
**Audit Requirement:** Log `SUBJECT_CREATED`.
**Example Scenario:** "MATH-101" identifies Mathematics within an institute permanently.
**Related Rules:** SUB-002.

**Rule ID:** SUB-002
**Rule Name:** Curriculum Mapping (Subject ↔ Class/Stream)
**Description:** A subject is taught in a class only if mapped via curriculum, with mandatory/optional designation.
**Priority:** Critical
**Category:** Structure integrity
**Preconditions:** Curriculum definition.
**Business Rule:** Assessment and timetabling for a subject in a class require a curriculum mapping. A subject may be core (mandatory) in one class and elective in another, and stream-specific where streams apply. Unmapped subjects cannot be assessed for that class.
**System Action:** Enforce mapping before assessment/assignment; respect mandatory/optional and stream.
**Validation:** Class/stream exists; mapping consistent.
**Failure Behavior:** Block assessment/assignment for unmapped subject-class pairs.
**Audit Requirement:** Log `CURRICULUM_MAPPED/UNMAPPED`.
**Example Scenario:** Physics is core in Science stream of Class 11 but not offered in Commerce.
**Related Rules:** SUB-001, CLS-006, EXM (assessment scope).

**Rule ID:** SUB-003
**Rule Name:** Subject Components & Aggregation
**Description:** A subject may have components (e.g., theory + practical) with defined weightages that aggregate to the subject result.
**Priority:** High
**Category:** Assessment integrity
**Preconditions:** Subject with components.
**Business Rule:** Components carry weightages summing to 100% (or per the configured scheme); the subject's mark/grade aggregates components per the rule. Per-component pass requirements may apply (a student must pass practical and theory separately if configured).
**System Action:** Aggregate component results per weightage; enforce per-component pass rules where set.
**Validation:** Component weightages valid (sum per scheme); pass rules defined.
**Failure Behavior:** Reject inconsistent weightages; block result computation on missing component marks.
**Audit Requirement:** Log component configuration; results reference the aggregation used.
**Example Scenario:** Science = 70% theory + 30% practical; failing practical fails the subject if so configured.
**Related Rules:** GRD (grading), RES (results), EXM (marks).

**Rule ID:** SUB-004
**Rule Name:** Subject-Teacher Assignment Confers Marks Ownership
**Description:** The assigned subject teacher owns that subject's marks (and period attendance) for their section.
**Priority:** High
**Category:** Authorization (ownership)
**Preconditions:** Subject-teacher assignment.
**Business Rule:** Mark entry and period-mode attendance for a subject in a section are owned by the assigned subject teacher (AUTHZ-003); a teacher cannot enter marks for subjects/sections they are not assigned. Assignment limits are configurable.
**System Action:** Establish ownership from the assignment; enforce on mark/attendance entry.
**Validation:** Subject mapped to the class; assignee active in scope; within limits.
**Failure Behavior:** Reject mark/attendance entry outside assignment.
**Audit Requirement:** Log `SUBJECT_TEACHER_ASSIGNED/CHANGED`.
**Example Scenario:** Only the assigned Physics teacher of Class 11-A can enter its Physics marks.
**Related Rules:** AUTHZ-003, EXM (mark entry), ATT-003.

**Rule ID:** SUB-005
**Rule Name:** Subject Type, Credit & Weightage
**Description:** Subjects are typed (core/elective/co-curricular) and carry credit/weightage used in result computation and GPA.
**Priority:** High
**Category:** Assessment integrity
**Preconditions:** Subject definition.
**Business Rule:** Each subject's type and credit/weightage are configurable and feed grading/GPA (Doc 16). Co-curricular/non-graded subjects may be recorded without affecting GPA. Language-option groups (First/Second/Third language) constrain selection.
**System Action:** Apply credit/weightage in result aggregation; respect non-graded flags and language groups.
**Validation:** Credits valid; type consistent with grading scheme.
**Failure Behavior:** Block GPA computation on inconsistent credit configuration.
**Audit Requirement:** Log subject type/credit changes.
**Example Scenario:** A 4-credit core subject weighs more in GPA than a 2-credit elective; "Moral Science" is recorded but non-graded.
**Related Rules:** GRD, RES, SUB-006.

**Rule ID:** SUB-006
**Rule Name:** Elective Selection Rules & Prerequisites
**Description:** Where electives are offered, selection respects offered groups, limits, prerequisites, and capacity.
**Priority:** Medium
**Category:** Curriculum flexibility
**Preconditions:** Class offers electives; selection window open.
**Business Rule:** Students select electives from defined groups within min/max limits; prerequisites (a prior subject/level) are enforced; elective capacity may apply. The selected set defines the student's assessment scope for the session.
**System Action:** Validate selection against groups/limits/prerequisites/capacity; record the set.
**Validation:** Selection within rules; prerequisites met; capacity available.
**Failure Behavior:** Block invalid selections; waitlist/alternative on full electives.
**Audit Requirement:** Log `ELECTIVE_SELECTED/CHANGED`.
**Example Scenario:** A student must have passed Biology I to elect Biology II.
**Related Rules:** SUB-002, SUB-005, ENR (subject-level enrollment future).

**Rule ID:** SUB-007
**Rule Name:** Subject Retirement Preserves History
**Description:** Retiring/unmapping a subject affects future sessions only; historical assessments remain intact.
**Priority:** Medium
**Category:** Integrity
**Preconditions:** Subject deprecated or unmapped from a class.
**Business Rule:** A subject with assessment history cannot be hard-deleted; it is deprecated/unmapped for future sessions while past results referencing it remain valid and retrievable.
**System Action:** Deprecate/unmap forward; preserve historical records.
**Validation:** Detect dependent history; deprecate instead of delete.
**Failure Behavior:** Hard-delete of an assessed subject rejected.
**Audit Requirement:** Log `SUBJECT_DEPRECATED/UNMAPPED`.
**Example Scenario:** A subject dropped from the 2027 curriculum still appears correctly on 2025 marksheets.
**Related Rules:** SUB-002, SESS-005.

## 5. Validation Rules
- Subject code unique within institute, immutable.
- Assessment/assignment require a valid curriculum mapping.
- Component weightages valid (sum per scheme); per-component pass rules consistent.
- Credit/type consistent with the grading scheme.
- Elective selection within groups/limits/prerequisites/capacity.
- Deprecate (not delete) subjects with history.

## 6. State Machine

**State Name:** DRAFT
**Description:** Subject being defined; not yet assessable.
**Allowed Transitions:** → ACTIVE (defined + mapped); → ARCHIVED (abandoned).
**Forbidden Transitions:** assessment while unmapped.
**System Actions:** Validate code/components/credits.

**State Name:** ACTIVE
**Description:** Live subject, mapped and assessable.
**Allowed Transitions:** → DEPRECATED (retired for future sessions).
**Forbidden Transitions:** edits that alter historical assessment semantics.
**System Actions:** Enable assignment/assessment for mapped classes.

**State Name:** DEPRECATED
**Description:** Not offered in future sessions; historical results remain valid.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** → ACTIVE (re-introduce as a new/updated subject).
**System Actions:** Block new mapping/assignment; preserve history.

**State Name:** ARCHIVED
**Description:** Retained historical subject definition.
**Allowed Transitions:** none (terminal).
**Forbidden Transitions:** edits/hard-delete (if assessed).
**System Actions:** Read-only retention.

## 7. Status Definitions
`DRAFT` · `ACTIVE` · `DEPRECATED` · `ARCHIVED`. Attributes: `type` (core/elective/co-curricular/non-graded), `components`, `credit/weightage`, `language_group`, `stream applicability`.

## 8. Workflow Rules
- Subject definition/curriculum mapping is a direct academic-admin action, audited; changes apply to future sessions.
- Elective selection may run a selection window with approval where configured.
- Subject-teacher assignment is immediate but audited (it confers marks ownership).

## 9. Permission Rules
- `academic.subject.manage` — define subjects, components, credits, curriculum mapping.
- `academic.subject.assign_teacher` — assign subject teachers.
- `academic.subject.view` — read subjects/curriculum.
- `academic.elective.select` — student/guardian elective selection (scoped to self).

## 10. Notification Rules
- `SUBJECT_TEACHER_ASSIGNED/CHANGED` → notify the teacher.
- Elective selection window open/close → notify eligible students/guardians.
- `ELECTIVE_SELECTED` confirmation → notify student/guardian.
- Curriculum changes affecting a class → notify academic staff.

## 11. Audit Requirements
Mandatory: `SUBJECT_CREATED/EDITED`, `CURRICULUM_MAPPED/UNMAPPED`, component/credit changes, `SUBJECT_TEACHER_ASSIGNED/CHANGED`, `ELECTIVE_SELECTED/CHANGED`, `SUBJECT_DEPRECATED/ARCHIVED`. With actor, subject, class/stream, before/after.

## 12. Data Retention Rules
- Subject definitions and curriculum mappings retained as configuration history (what was taught in year X).
- Component/credit configurations version-stamped so historical results aggregate per the rules then in effect.
- Deprecated subjects retained while any historical result references them.

## 13. Edge Cases
- **Components with separate pass rules:** failing practical can fail the subject even with a high theory score (SUB-003) — a common results bug if not modeled.
- **Same subject, different role per class:** core in one class, elective in another (SUB-002).
- **Language options:** First/Second/Third language groups constrain mutually-exclusive choices.
- **Non-graded/co-curricular subjects:** recorded but excluded from GPA (SUB-005) — must not silently inflate/deflate results.
- **Subject taught across streams differently:** Physics in Science vs not in Commerce; assessment scope must follow the student's stream.
- **Mid-session curriculum change:** affects future sessions; the current session's assessment scope is fixed (SUB-007/SESS-004).
- **Elective changed after assessment started:** restricted; switching electives mid-assessment needs governed handling to avoid orphaned marks.
- **Subject with no assigned teacher:** assessment blocked until assignment (SUB-004) — flagged in readiness checks.

## 14. Failure Scenarios
- **Component weightages not summing correctly:** rejected; result computation blocked until fixed.
- **Marks entry for an unmapped/unassigned subject:** rejected (SUB-002/SUB-004).
- **Elective prerequisite chain broken:** blocked at selection (SUB-006).
- **Hard-delete of an assessed subject:** rejected; deprecate instead (SUB-007).

## 15. Exception Handling Rules
- Assessment and assignment validated against curriculum mapping and teacher assignment before commit.
- Component/credit inconsistencies block result computation rather than producing wrong results.
- Forbidden operations (delete-assessed-subject, unmapped assessment) rejected with explicit messages.
- Historical aggregation always uses the version-stamped configuration.

## 16. Compliance Considerations
- **Accurate transcripts:** version-stamped subjects/components/credits ensure historical marksheets reflect the rules then in force.
- **Fair assessment:** documented component and pass rules support grade-dispute resolution.
- **Curriculum record:** retained mappings evidence what was taught for accreditation.

## 17. Future Considerations
- Subject-level enrollment (beyond section) for fully elective/modular programs.
- Cross-class subject pooling (a subject taught to mixed-class groups).
- Outcome-based assessment (learning outcomes per subject).
- Prerequisite graphs across multiple levels for university-style programs.
