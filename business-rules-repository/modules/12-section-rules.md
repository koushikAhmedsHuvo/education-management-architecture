# 12 — Section Business Rules

## 1. Module Purpose
Govern the **section** — the subdivision of a class and the **enrollment leaf**: the lowest structural node where students actually attach (Doc 08/ENR-001). A section belongs to one class instance, sits at one campus, and may belong to a **shift** (morning/day/evening) and a **stream** (for streamed classes). The section is the operative unit for **capacity**, **roll numbers**, the **class teacher (homeroom)** assignment, attendance, and timetabling. This module also formalizes the **Shift** concept that institutions in the target market depend on but generic ERPs routinely omit.

## 2. Actors
- **Institute / Campus Administrator** — creates and manages sections, capacity, shifts, class-teacher assignment.
- **Class Teacher (Homeroom)** — the staff member responsible for a section (daily attendance, pastoral).
- **System** — enforces capacity, uniqueness, the default-section guarantee, shift integrity, and enrollment-leaf rules.

## 3. Use Cases

**Use Case ID:** UC-SEC-01 — Create a Section
**Actors:** Campus/Institute Administrator, System
**Description:** Add a section under a class instance at a campus.
**Preconditions:** Class instance exists for the session (Doc 11); actor authorized.
**Main Flow:** 1) Admin creates a section (name, capacity, campus, shift, stream if applicable). 2) Optionally assigns a class teacher. 3) Section becomes an enrollment target.
**Alternative Flow:** A1) Auto default section created by the class when none exists (CLS-003).
**Exception Flow:** E1) Duplicate section name within class+campus+shift → reject (SEC-002). E2) Shift not defined for the institute → reject (SEC-005).
**Post Conditions:** Section active; enrollment can target it; audited.
**Business Rules Applied:** SEC-001, SEC-002, SEC-005.

**Use Case ID:** UC-SEC-02 — Assign / Change Class Teacher
**Actors:** Administrator, System
**Description:** Designate the staff member responsible for the section.
**Preconditions:** Section active; staff member exists and is assignable.
**Main Flow:** 1) Admin assigns a class teacher. 2) The assignment grants the teacher ownership over the section's attendance and roster (AUTHZ-003). 3) Change reassigns ownership.
**Exception Flow:** E1) Assigning a non-staff or out-of-scope user → reject. E2) Teacher already over assignment limit (configurable) → warn/block.
**Post Conditions:** Class-teacher ownership established; audited.
**Business Rules Applied:** SEC-004.

**Use Case ID:** UC-SEC-03 — Merge / Split / Close a Section
**Actors:** Administrator, System
**Description:** Restructure sections mid- or end-of-session with preserved history.
**Preconditions:** Actor authorized; target sections valid.
**Main Flow:** 1) Admin merges two sections or splits one. 2) Affected students are **transferred** (Doc 08/ENR-005) with effective dates. 3) Historical attendance/marks stay attributed to the original section for their period.
**Exception Flow:** E1) Merge exceeding target capacity → block or raise capacity first. E2) Closing a section with active students → require transfers first (SEC-006).
**Post Conditions:** New section layout; period-accurate history; audited.
**Business Rules Applied:** SEC-003, SEC-006, ENR-005.

## 4. Business Rules

**Rule ID:** SEC-001
**Rule Name:** Section Is the Enrollment Leaf
**Description:** Students enroll into a section, never directly into a class; the section is the lowest structural node.
**Priority:** Critical
**Category:** Structure integrity
**Preconditions:** Enrollment (Doc 08).
**Business Rule:** A section belongs to exactly one class instance, one campus, one shift (and one stream if streamed). All student-level operations (attendance, roll number, timetable) operate at the section level.
**System Action:** Bind section to class-instance/campus/shift; treat as enrollment target.
**Validation:** Parent class instance active; campus/shift valid.
**Failure Behavior:** Reject sections without a valid parent class instance.
**Audit Requirement:** Log `SECTION_CREATED` with class, campus, shift, stream.
**Example Scenario:** A student is enrolled in "Class 5 / Section A / Morning shift".
**Related Rules:** CLS-003, ENR-001, SEC-005.

**Rule ID:** SEC-002
**Rule Name:** Section Name Uniqueness Within Class+Campus+Shift
**Description:** Section names are unique within their class instance at a campus and shift.
**Priority:** Medium
**Category:** Identity
**Preconditions:** Section creation/edit.
**Business Rule:** "Section A" can exist in many classes/campuses/shifts but not twice within the same class+campus+shift combination.
**System Action:** Enforce composite uniqueness.
**Validation:** Unique (class instance, campus, shift, name).
**Failure Behavior:** Reject duplicates within the combination.
**Audit Requirement:** Captured in `SECTION_CREATED`.
**Example Scenario:** Morning and Day shifts of Class 5 can each have a "Section A".
**Related Rules:** SEC-001, SEC-005.

**Rule ID:** SEC-003
**Rule Name:** Section Capacity Is the Operative Enrollment Limit
**Description:** Section capacity governs how many students may be actively enrolled in it.
**Priority:** High
**Category:** Capacity
**Preconditions:** Enrollment/transfer into the section.
**Business Rule:** Active enrollments cannot exceed the configured section capacity; this is the operative limit referenced by admission and enrollment (ADM-004/ENR-003). Lowering capacity below current enrollment is blocked.
**System Action:** Count active enrollments vs capacity on enroll/transfer.
**Validation:** Capacity ≥ current active enrollments; ≥0.
**Failure Behavior:** Block over-capacity enrollment; block capacity reduction below current count.
**Audit Requirement:** Log `SECTION_CAPACITY_CHANGED` and capacity blocks.
**Example Scenario:** Section A at 40/40 sends the next student to Section B.
**Related Rules:** ADM-004, ENR-003, CLS-007.

**Rule ID:** SEC-004
**Rule Name:** Class-Teacher Assignment Confers Ownership
**Description:** The assigned class teacher owns the section's daily attendance and roster for access purposes.
**Priority:** High
**Category:** Authorization (ownership)
**Preconditions:** Class-teacher assignment.
**Business Rule:** The class teacher is the ownership source for daily attendance and section roster access (AUTHZ-003). One section has one class teacher (optionally a co-teacher per policy); a teacher may hold a configurable maximum of assignments.
**System Action:** Establish ownership from the assignment; re-resolve on change.
**Validation:** Assignee is active staff in scope; within assignment limits.
**Failure Behavior:** Reject invalid/over-limit assignments.
**Audit Requirement:** Log `CLASS_TEACHER_ASSIGNED/CHANGED`.
**Example Scenario:** Only the class teacher of Section A (and admins) can take its daily attendance.
**Related Rules:** AUTHZ-003, HR (assignments), ATT-003.

**Rule ID:** SEC-005
**Rule Name:** Shift Is a First-Class Section Attribute
**Description:** Sections may belong to a shift (e.g., Morning / Day / Evening); shifts are configurable and scope timetable, attendance windows, and capacity.
**Priority:** High
**Category:** Structure (market-specific)
**Preconditions:** Institute uses shifts.
**Business Rule:** A shift is a configurable institute attribute defining operating windows. A section belongs to exactly one shift; the same class can run parallel sections across shifts with independent rosters, capacity, and timetables. Institutes that don't use shifts operate on a single default shift transparently.
**System Action:** Enforce one shift per section; default-shift for non-shift institutes; scope timetable/attendance windows by shift.
**Validation:** Shift defined for the institute; section references a valid shift.
**Failure Behavior:** Reject sections with undefined shifts; auto-default where shifts unused.
**Audit Requirement:** Log shift configuration and section-shift assignment.
**Example Scenario:** A school runs "Class 6 / Section A / Morning" and "Class 6 / Section A / Day" as distinct sections with separate students and schedules.
**Related Rules:** SEC-001, ATT-001 (attendance windows), timetable (future).

**Rule ID:** SEC-006
**Rule Name:** Merge/Split/Close Preserves History via Transfers
**Description:** Restructuring sections moves students by transfer with effective dates; history is never overwritten.
**Priority:** High
**Category:** Integrity
**Preconditions:** Section merge/split/close.
**Business Rule:** Students moved during merge/split are transferred (ENR-005); prior-period attendance/marks stay attributed to the original section. A section with active students cannot be closed until they are transferred.
**System Action:** Execute dated transfers; block closure with active dependents.
**Validation:** Targets valid + capacity; effective dates valid.
**Failure Behavior:** Block closure/merge violating capacity or leaving active students stranded.
**Audit Requirement:** Log `SECTION_MERGED/SPLIT/CLOSED` and the student transfers.
**Example Scenario:** Two under-filled sections merge mid-year; January–March stays under the old sections, April onward under the merged section.
**Related Rules:** ENR-005, CAMP-006, SEC-003.

## 5. Validation Rules
- Section bound to a valid active class instance, campus, and shift (and stream if streamed).
- Section name unique within class+campus+shift.
- Active enrollments ≤ section capacity; no capacity reduction below current count.
- Class-teacher assignee is active in-scope staff, within assignment limits.
- Closure requires zero active students (transfer first).

## 6. State Machine

**State Name:** ACTIVE
**Description:** Operational enrollment leaf for the session.
**Allowed Transitions:** → INACTIVE (temporarily closed); → MERGED (into another section); → CLOSED (end/restructure, students transferred); → COMPLETED (session closes).
**Forbidden Transitions:** closure with active students.
**System Actions:** Accept enrollments; drive attendance/timetable; enforce capacity.

**State Name:** INACTIVE
**Description:** Temporarily not accepting operations; data intact.
**Allowed Transitions:** → ACTIVE; → CLOSED.
**Forbidden Transitions:** new enrollments while inactive.
**System Actions:** Preserve roster/history; block new operations.

**State Name:** MERGED
**Description:** Folded into another section; students transferred out.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** reactivation as a live target.
**System Actions:** Preserve prior-period attribution.

**State Name:** COMPLETED / CLOSED
**Description:** Session ended or section restructured/closed.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** edits.
**System Actions:** Freeze as historical record.

**State Name:** ARCHIVED
**Description:** Historical section, retained read-only.
**Allowed Transitions:** none (terminal).
**Forbidden Transitions:** edits/hard-delete.
**System Actions:** Read-only retention.

## 7. Status Definitions
`ACTIVE` · `INACTIVE` · `MERGED` · `CLOSED` · `COMPLETED` · `ARCHIVED`. Attributes: `shift`, `stream`, `capacity`, `class_teacher`.

## 8. Workflow Rules
- Section creation/edit is a direct admin action, audited.
- Merge/split/close may require approval (configurable) due to student impact and runs the transfer flow (Doc 08).
- Class-teacher changes are immediate but audited (they shift ownership).

## 9. Permission Rules
- `academic.section.manage` — create/edit/merge/close sections within scope.
- `academic.section.assign_teacher` — assign class teachers.
- `academic.section.view` — read sections (teachers see their own; admins see scope).
- Scoped to institute/campus; campus admins manage only their campus's sections.

## 10. Notification Rules
- `CLASS_TEACHER_ASSIGNED/CHANGED` → notify the affected teacher.
- `SECTION_MERGED/SPLIT/CLOSED` → notify affected students' guardians (transfer notices) and the class teacher.
- Capacity-reached → notify admission/enrollment admins (informational).

## 11. Audit Requirements
Mandatory: `SECTION_CREATED`, `DEFAULT_SECTION_CREATED`, `SECTION_CAPACITY_CHANGED`, `CLASS_TEACHER_ASSIGNED/CHANGED`, `SECTION_MERGED/SPLIT/CLOSED`, shift/stream assignment changes, capacity blocks. With actor, class, campus, shift, before/after.

## 12. Data Retention Rules
- Sections retained with their session (long-term academic record).
- Class-teacher assignment history retained (who was responsible when) — relevant for audits and safeguarding inquiries.
- Merge/split history retained to preserve period-accurate attribution.

## 13. Edge Cases
- **Shift omission (the classic miss):** institutions running morning and day shifts need parallel same-named sections with independent rosters/timetables — handled by SEC-005; a generic ERP that ignores shifts forces hacky workarounds.
- **Default section:** single-section institutions never see sections explicitly (CLS-003).
- **Streamed section:** belongs to a stream; a student changing stream is a section transfer with subject reconciliation.
- **Mid-session merge/split:** transfers preserve period attribution (SEC-006); reports must respect the split timeline.
- **Class-teacher leaves mid-year:** reassignment shifts ownership immediately; the prior teacher loses live access but their past actions remain audited.
- **Co-teaching / shared sections across campuses:** a teacher serving two campuses needs explicit multi-assignment (HR); ownership must not silently span campuses.
- **Capacity vs shift:** capacity is per section (per shift), not per class across shifts — a frequent miscount.

## 14. Failure Scenarios
- **Closing a section with active students:** blocked until transfers complete (SEC-006).
- **Capacity reduced below current enrollment:** blocked; never auto-drop students.
- **Section referencing an undefined shift/stream:** rejected.
- **Two admins assigning different class teachers simultaneously:** last-write-wins is unacceptable for ownership; the operation is serialized and audited.

## 15. Exception Handling Rules
- Structural changes are transactional with preserved history.
- Capacity and shift integrity are validated before commit.
- Forbidden operations (close-with-students, undefined-shift) rejected with explicit messages.
- Ownership changes are atomic and audited.

## 16. Compliance Considerations
- **Accurate attribution:** period-accurate section history underpins truthful attendance and academic records.
- **Safeguarding (class teacher of record):** knowing the responsible adult for a section at any past date supports safeguarding and accountability.
- **Minors:** section rosters are sensitive student data, scoped and access-controlled.

## 17. Future Considerations
- Room/facility and timetable binding per section+shift.
- Co-teacher and substitute-teacher models with scoped ownership.
- Dynamic section balancing and auto-roll-number policies.
- Shift-level operating calendars (different holidays/hours per shift).
