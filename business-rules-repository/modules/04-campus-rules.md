# 04 — Campus Business Rules

## 1. Module Purpose
Govern the **campus** — a physical branch or location within an institute. Campus is the **secondary scope boundary** beneath institute, enabling branch-level administration and access. Campus is **optional**: institutes that operate as a single location use one auto-created default campus and never see the distinction; multi-branch institutes use campuses to scope staff, students, sections, and access.

## 2. Actors
- **Organization / Institute Administrator** — creates and manages campuses within an institute.
- **Campus Administrator** — manages a single campus's operations.
- **System** — enforces uniqueness, the default-campus rule, scope cascade, and lifecycle.

## 3. Use Cases

**Use Case ID:** UC-CAMP-01 — Create a Campus
**Actors:** Institute Administrator, System
**Description:** Add a branch to an institute.
**Preconditions:** Institute is `ACTIVE` or in setup; actor holds `institute.campus.manage`.
**Main Flow:** 1) Admin creates a campus (name, unique code within institute, address, contact). 2) System registers it under the institute. 3) Campus becomes available as a scope.
**Alternative Flow:** A1) First institute setup auto-creates a **default campus** so single-location institutes work transparently (CAMP-002).
**Exception Flow:** E1) Duplicate code within the institute → reject.
**Post Conditions:** Campus `ACTIVE`; available for section/student/staff scoping; audited.
**Business Rules Applied:** CAMP-001, CAMP-002, CAMP-003.

**Use Case ID:** UC-CAMP-02 — Deactivate / Archive a Campus
**Actors:** Institute Administrator, System
**Description:** Take a campus out of operation.
**Preconditions:** Actor holds `institute.campus.manage`.
**Main Flow:** 1) Admin requests deactivation/archive. 2) System checks for active sections/enrollments at the campus. 3) If clear (or after handling), campus → `INACTIVE`/`ARCHIVED`; data preserved.
**Exception Flow:** E1) Active sections/students present → block; require transfer/handling first (CAMP-006). E2) Attempt to deactivate the only/default campus while the institute is active → block (CAMP-002).
**Post Conditions:** Campus deactivated/archived; cascade applied; audited.
**Business Rules Applied:** CAMP-002, CAMP-006, CAMP-007.

## 4. Business Rules

**Rule ID:** CAMP-001
**Rule Name:** Campus Belongs to Exactly One Institute
**Description:** A campus is owned by a single institute and cannot be shared or moved between institutes.
**Priority:** Critical
**Category:** Scope integrity
**Preconditions:** Campus creation.
**Business Rule:** Each campus references exactly one `institute_id`; campuses are never shared across institutes, and a campus cannot be reparented to a different institute (its data is institute-bound).
**System Action:** Bind campus to institute on creation; forbid reparenting.
**Validation:** `institute_id` valid and accessible to the actor.
**Failure Behavior:** Reject shared/reparented campuses.
**Audit Requirement:** Log `CAMPUS_CREATED` with institute and code.
**Example Scenario:** A campus of School A can never be attached to School B.
**Related Rules:** INST-006, CAMP-004.

**Rule ID:** CAMP-002
**Rule Name:** Default Campus Guarantee
**Description:** Every active institute always has at least one campus; single-location institutes use an auto default.
**Priority:** High
**Category:** Lifecycle invariant
**Preconditions:** Institute activation or campus deactivation.
**Business Rule:** An active institute must have ≥1 active campus at all times. A default campus is auto-created at institute setup. The last remaining active campus of an active institute cannot be deactivated.
**System Action:** Auto-create default campus; block removal of the last active campus.
**Validation:** Count active campuses before deactivation.
**Failure Behavior:** Block deactivation that would leave an active institute with zero campuses.
**Audit Requirement:** Log `DEFAULT_CAMPUS_CREATED`.
**Example Scenario:** A single-building school operates without ever thinking about campuses, on its silent default.
**Related Rules:** INST-003, CAMP-007.

**Rule ID:** CAMP-003
**Rule Name:** Campus Code Uniqueness Within Institute
**Description:** Campus codes are unique within their institute and immutable after creation.
**Priority:** Medium
**Category:** Identity
**Preconditions:** Campus creation/edit.
**Business Rule:** Code unique per institute (not globally — two institutes may both have "MAIN"), immutable once set.
**System Action:** Enforce per-institute uniqueness; lock after creation.
**Validation:** Format + uniqueness within institute.
**Failure Behavior:** Reject duplicates within the institute; reject code edits.
**Audit Requirement:** Captured in `CAMPUS_CREATED`.
**Example Scenario:** School A and School B can each have a "MAIN" campus; School A cannot have two.
**Related Rules:** INST-001, CAMP-001.

**Rule ID:** CAMP-004
**Rule Name:** Campus Scope Filtering
**Description:** Where campus scoping is used, campus-tagged data is filtered by the active campus, bounded by institute.
**Priority:** Critical
**Category:** Isolation
**Preconditions:** A campus-scoped resource is accessed.
**Business Rule:** A campus admin sees only their campus's data; an institute admin sees all campuses of their institute. Campus access always implies and is bounded by institute access (cannot exceed it).
**System Action:** Apply campus filter from active scope, nested within the institute filter.
**Validation:** Active campus ∈ active institute; membership grants campus access.
**Failure Behavior:** Out-of-campus access filtered/denied.
**Audit Requirement:** Inherit scope-violation auditing (AUTHZ-002).
**Example Scenario:** A campus admin cannot view another campus's attendance, even within the same institute.
**Related Rules:** AUTHZ-002, AUTHZ-003, CAMP-001.

**Rule ID:** CAMP-005
**Rule Name:** Campus-Level Configuration Override
**Description:** A campus may override institute configuration for genuinely branch-specific settings.
**Priority:** Medium
**Category:** Configuration resolution
**Preconditions:** A configurable value resolved at campus scope.
**Business Rule:** Resolution is most-specific-wins: campus beats institute beats org beats default. Only settings flagged campus-overridable may be set at campus level.
**System Action:** Resolve via Configuration Engine; cache per scope.
**Validation:** Setting is campus-overridable; value valid against its definition.
**Failure Behavior:** Disallowed or invalid override rejected; inherit institute value.
**Audit Requirement:** Log campus-level config changes.
**Example Scenario:** A campus sets its own daily start time while inheriting the institute grading scale.
**Related Rules:** INST-005, CFG (future).

**Rule ID:** CAMP-006
**Rule Name:** Inter-Campus Movement Is a Transfer
**Description:** Moving a student or section between campuses is a controlled, audited transfer — not an edit.
**Priority:** High
**Category:** Data integrity
**Preconditions:** A student/section is reassigned to a different campus within the same institute.
**Business Rule:** Such moves are modeled as transfers with effective dates and full audit, preserving the historical association (the student *was* at Campus X until date Y).
**System Action:** Record a transfer event; update current campus; retain history.
**Validation:** Target campus active and within the same institute; effective date valid.
**Failure Behavior:** Cross-institute "campus move" rejected (that is a different process); invalid target rejected.
**Audit Requirement:** Log `CAMPUS_TRANSFER` with from/to, subject, effective date, actor.
**Example Scenario:** A student moving from the city campus to the suburb campus mid-year is transferred, and reports correctly attribute each period.
**Related Rules:** CAMP-001, ENR/STU transfer rules (Doc 06/08).

**Rule ID:** CAMP-007
**Rule Name:** Deactivate/Archive Requires No Active Dependents
**Description:** A campus with active sections/students cannot be deactivated/archived until they are handled.
**Priority:** High
**Category:** Lifecycle integrity
**Preconditions:** Campus deactivation/archive requested.
**Business Rule:** Active sections, enrollments, or assignments at the campus must be transferred or closed before deactivation; data is then preserved read-only.
**System Action:** Check dependents; block or guide resolution; archive on clearance.
**Validation:** Zero active dependents (or explicit handling) before transition.
**Failure Behavior:** Block with a list of blocking dependents.
**Audit Requirement:** Log `CAMPUS_DEACTIVATED/ARCHIVED`.
**Example Scenario:** A campus being closed mid-session requires its students to be transferred before it can be archived.
**Related Rules:** CAMP-002, CAMP-006, SESS-007.

## 5. Validation Rules
- Campus code unique within institute, format-valid, immutable post-creation.
- `institute_id` valid and accessible; no reparenting.
- Active institute must retain ≥1 active campus.
- Campus-level overrides allowed only for campus-overridable settings.
- Inter-campus moves require same-institute target and an effective date.

## 6. State Machine

**State Name:** ACTIVE
**Description:** Operational branch.
**Allowed Transitions:** → INACTIVE (temporarily closed); → ARCHIVED (permanently closed, with dependents handled).
**Forbidden Transitions:** deactivation if it is the last active campus of an active institute (CAMP-002).
**System Actions:** Enable campus scoping; accept sections/students/staff.

**State Name:** INACTIVE
**Description:** Temporarily not operating; data intact, not accepting new operations.
**Allowed Transitions:** → ACTIVE (reopen); → ARCHIVED.
**Forbidden Transitions:** none beyond the default-campus invariant.
**System Actions:** Block new operational assignments; preserve data.

**State Name:** ARCHIVED
**Description:** Permanently closed; read-only history.
**Allowed Transitions:** → ACTIVE only via explicit audited reactivation policy.
**Forbidden Transitions:** hard-delete if data ever existed.
**System Actions:** Read-only; retain for retention period.

## 7. Status Definitions
`ACTIVE` (operational) · `INACTIVE` (temporarily closed) · `ARCHIVED` (permanently closed, retained). Plus the implicit `DEFAULT` flag distinguishing the auto-created default campus.

## 8. Workflow Rules
- Campus creation is a direct admin action, audited.
- Campus closure (deactivate/archive) may require approval via the Workflow Engine when configured, given its student-impact.
- Inter-campus student transfers may follow a transfer-approval workflow (configurable) — see Student Lifecycle (Doc 08).

## 9. Permission Rules
- `institute.campus.manage` — create/edit/deactivate/archive campuses within an institute.
- `campus.operations.manage` — manage a single campus's operations (campus admin, scoped to that campus).
- Campus admins are scoped strictly to their campus (CAMP-004); they cannot view or act on sibling campuses.

## 10. Notification Rules
- `CAMPUS_CREATED` → notify institute admins.
- `CAMPUS_DEACTIVATED/ARCHIVED` → notify institute admins and affected campus staff.
- `CAMPUS_TRANSFER` (student) → notify the relevant guardians (per preference) and receiving campus admin.

## 11. Audit Requirements
Mandatory: `CAMPUS_CREATED`, `DEFAULT_CAMPUS_CREATED`, `CAMPUS_UPDATED`, `CAMPUS_DEACTIVATED/REACTIVATED`, `CAMPUS_ARCHIVED`, `CAMPUS_TRANSFER`, campus-level config changes. With actor, institute, campus, before/after, timestamp.

## 12. Data Retention Rules
- Archived campuses and their historical records retained per the institute's retention policy.
- Transfer history retained to preserve accurate period-by-period attribution (a student's campus on any past date).
- Campus deletion (only for an empty, never-used campus) is the sole hard-delete case; otherwise archive.

## 13. Edge Cases
- **The default campus:** must never be deletable while the institute is active; single-location clients depend on it silently.
- **Last active campus:** cannot be deactivated without first deactivating/archiving the institute itself.
- **Mid-session campus closure:** requires student transfers first (CAMP-007); reports must still correctly attribute the pre-closure period to the old campus.
- **Campus code collision across institutes:** allowed (uniqueness is per-institute), a common false-bug report.
- **Campus admin scope leakage:** a campus admin must never see sibling-campus data; a frequent authorization test target.
- **Reparenting temptation:** "move this campus to another institute" must be rejected — institute-bound data makes it semantically impossible; the correct path is recreate + transfer.

## 14. Failure Scenarios
- **Deactivating the last campus:** blocked by the default-campus invariant; never leave an active institute campus-less.
- **Transfer to an inactive/cross-institute campus:** rejected.
- **Partial archive with lingering dependents:** transactional check prevents half-archived state.
- **Config override on a non-overridable setting:** rejected, inherits institute value.

## 15. Exception Handling Rules
- Lifecycle transitions validated and transactional.
- Forbidden moves (reparenting, last-campus deactivation, cross-institute transfer) rejected with explicit messages.
- Deactivation never deletes or mutates student/academic data — only changes operational availability.

## 16. Compliance Considerations
- **Accurate historical attribution:** transfer history ensures records correctly reflect where a minor studied at any time — relevant for transcripts and audits.
- **Minors' data on closure:** retained under retention rules, not orphaned or prematurely purged.
- **Scope isolation:** campus-level scoping reinforces least-privilege for branch staff.

## 17. Future Considerations
- Campus-level capacity/room/facility modeling (feeds timetabling).
- Geo/locale per campus (multi-city institutes spanning time zones).
- Campus-level dashboards and KPIs for branch managers.
- Shared resources across campuses (a teacher serving two campuses) with explicit multi-campus assignment.
