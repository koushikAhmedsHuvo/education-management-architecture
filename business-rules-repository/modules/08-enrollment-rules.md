# 08 — Enrollment Business Rules

## 1. Module Purpose
Govern **enrollment** — placing a student into a specific academic session's structure (the enrollment leaf: a class/grade and section) and maintaining that placement across the lifecycle, including promotion, transfer, and withdrawal. Enrollment is the link between a timeless student (Doc 06) and a time-bound session instance (Doc 05); it is what makes a student "in Class 5, Section A, for 2026." Attendance, exams, results, and fees all attach to the enrollment, not merely the student.

## 2. Actors
- **Admission Officer / Administrator** — creates enrollments (via conversion, Doc 07, or directly).
- **Institute / Campus Administrator** — manages section placement, transfers, promotion.
- **System** — enforces single-active-enrollment, capacity, structure validity, and history preservation.

## 3. Use Cases

**Use Case ID:** UC-ENR-01 — Enroll a Student into a Session/Level/Section
**Actors:** Administrator/Officer, System
**Description:** Place a student at the enrollment leaf for a session.
**Preconditions:** Student exists (Doc 06); target session `ACTIVE`/admission-open; target level/section instance exists; seat available.
**Main Flow:** 1) Select student, session, level, section. 2) System validates structure + capacity + single-active-enrollment. 3) Enrollment created `ACTIVE`; roll number assigned (configurable). 4) Fee schedule for the level/session is associated (Finance).
**Alternative Flow:** A1) Section auto-assigned by policy (balanced fill) instead of manual.
**Exception Flow:** E1) Section full → block or choose another section (ENR-003). E2) Student already actively enrolled in the same session → block (ENR-002).
**Post Conditions:** Active enrollment exists; attendance/exam/fee context established; audited.
**Business Rules Applied:** ENR-001, ENR-002, ENR-003, ENR-004.

**Use Case ID:** UC-ENR-02 — Section Transfer (Within Institute)
**Actors:** Administrator, System
**Description:** Move an enrolled student between sections/campuses within the institute, with history.
**Preconditions:** Active enrollment; target section valid + has capacity; same institute.
**Main Flow:** 1) Admin initiates transfer with effective date. 2) System closes the current placement as-of the date and opens the new one. 3) Historical attendance/marks remain attributed to the prior section for the prior period.
**Exception Flow:** E1) Target full → block. E2) Cross-institute "transfer" → not this flow (TC/withdrawal + new admission, ENR-006).
**Post Conditions:** New placement active; period-accurate history; audited.
**Business Rules Applied:** ENR-005, CAMP-006.

**Use Case ID:** UC-ENR-03 — Promotion at Rollover
**Actors:** Administrator, System
**Description:** Carry enrollments forward to the next session per outcomes.
**Preconditions:** Session rollover initiated (Doc 05); outcomes known.
**Main Flow:** 1) For each active enrollment, evaluate outcome (advance/retain/graduate/exit). 2) Create the next-session enrollment at the appropriate level/section instance. 3) Close the prior enrollment as completed.
**Exception Flow:** E1) Unresolved results → block per policy or override (audited). E2) Next-level instance missing → block until structure ready.
**Post Conditions:** Next-session enrollments exist; prior ones closed; full audit.
**Business Rules Applied:** ENR-007, SESS-006.

## 4. Business Rules

**Rule ID:** ENR-001
**Rule Name:** Enrollment Binds Student to Session/Level/Section Instance
**Description:** An enrollment references a student and a concrete session-instance leaf, never the bare structure definition.
**Priority:** Critical
**Category:** Integrity (definition/instance)
**Preconditions:** Enrollment creation.
**Business Rule:** Enrollment points to the session's class/section *instance* (Doc 05/SESS-004), so it is historically faithful: editing the structure definition later does not alter past enrollments. Attendance/exam/fee context derive from this binding.
**System Action:** Bind enrollment to the instance; stamp session/level/section.
**Validation:** Target instance exists and is active for the session.
**Failure Behavior:** Block enrollment to a non-existent/inactive instance.
**Audit Requirement:** Log `ENROLLMENT_CREATED` with student, session, level, section.
**Example Scenario:** A 2026 enrollment in "Class 5 / Section A" stays attributed correctly even if the class is renamed for 2027.
**Related Rules:** SESS-004, ENR-002.

**Rule ID:** ENR-002
**Rule Name:** One Active Enrollment per Student per Session
**Description:** A student has at most one active enrollment within a given session.
**Priority:** Critical
**Category:** Integrity
**Preconditions:** Enrollment creation for a session.
**Business Rule:** A student cannot be actively enrolled in two levels/sections of the same session simultaneously (a student is in one place). Multiple enrollments may exist across *different* sessions (history) and, where the institution model allows (e.g., a student also taking a separate course program), only via an explicitly supported multi-program model — not by accident.
**System Action:** Enforce single-active-per-session; reject conflicting concurrent enrollment.
**Validation:** No existing active enrollment for the student in that session.
**Failure Behavior:** Block the second active enrollment; suggest transfer instead.
**Audit Requirement:** Log conflicts (`ENROLLMENT_CONFLICT_BLOCKED`).
**Example Scenario:** A student in Class 5 cannot also be actively enrolled in Class 6 for the same year.
**Related Rules:** ENR-001, ENR-005.

**Rule ID:** ENR-003
**Rule Name:** Section Capacity Enforcement
**Description:** Enrollment respects configurable section/level capacity.
**Priority:** High
**Category:** Capacity
**Preconditions:** Enrollment/transfer into a section.
**Business Rule:** When capacity is configured, the system blocks enrollment beyond the limit (counting active enrollments) or requires choosing another section. Consistent with admission capacity (ADM-004).
**System Action:** Check live active-enrollment count vs capacity.
**Validation:** Capacity defined; count current.
**Failure Behavior:** Block over-capacity placement.
**Audit Requirement:** Log `SECTION_CAPACITY_BLOCK`.
**Example Scenario:** Section A is full at 40; the next student goes to Section B.
**Related Rules:** ADM-004, SEC (Doc 12).

**Rule ID:** ENR-004
**Rule Name:** Roll Number Assignment
**Description:** Each active enrollment gets a roll/registration number unique within its section/session per a configurable scheme.
**Priority:** Medium
**Category:** Identity
**Preconditions:** Enrollment created.
**Business Rule:** A roll number (configurable: by name order, merit, or sequence) is assigned, unique within the section for the session; it may be re-sequenced by an authorized action (audited) but not duplicated.
**System Action:** Assign per scheme; enforce within-section uniqueness.
**Validation:** Uniqueness within section/session.
**Failure Behavior:** Block duplicate roll numbers.
**Audit Requirement:** Log `ROLL_NUMBER_ASSIGNED/RESEQUENCED`.
**Example Scenario:** Roll numbers regenerate alphabetically after late admissions, as an audited action.
**Related Rules:** ENR-001.

**Rule ID:** ENR-005
**Rule Name:** Transfer Preserves Period-Accurate History
**Description:** Section/campus transfers close the old placement and open a new one with effective dates — history is never overwritten.
**Priority:** High
**Category:** Integrity
**Preconditions:** Within-institute transfer.
**Business Rule:** A transfer is a dated close-and-open: the prior placement's attendance/marks remain attributed to it for its period; the new placement owns subsequent records. No silent reassignment that rewrites the past.
**System Action:** Close prior placement as-of date; open new; retain both.
**Validation:** Effective date valid; target valid + same institute + capacity.
**Failure Behavior:** Invalid/cross-institute transfer rejected.
**Audit Requirement:** Log `ENROLLMENT_TRANSFERRED` with from/to and effective date.
**Example Scenario:** A mid-year section move keeps January–March attendance under Section A and April onward under Section B.
**Related Rules:** CAMP-006, ENR-002.

**Rule ID:** ENR-006
**Rule Name:** Cross-Institution Exit Is Withdrawal + Transfer Certificate
**Description:** Leaving the institution is a withdrawal that closes enrollment and may issue a transfer certificate — not an in-place edit.
**Priority:** High
**Category:** Lifecycle
**Preconditions:** Student leaves the institution (or moves to another institute in the deployment).
**Business Rule:** The active enrollment is closed (`WITHDRAWN`/`TRANSFERRED_OUT`), dues/clearance handled per policy, and a transfer certificate generated where required. Joining elsewhere is a fresh admission/enrollment, not a mutation of the old.
**System Action:** Close enrollment; trigger clearance + certificate; update student status.
**Validation:** Clearance policy evaluated; certificate generated per template.
**Failure Behavior:** Block exit on unmet hard clearance items (configurable) or warn.
**Audit Requirement:** Log `ENROLLMENT_CLOSED` with reason and certificate reference.
**Example Scenario:** A student moving cities is withdrawn, cleared of dues, and issued a transfer certificate.
**Related Rules:** STU-007, FEE clearance, Doc 30 (certificates).

**Rule ID:** ENR-007
**Rule Name:** Promotion Creates Next-Session Enrollment by Outcome
**Description:** Rollover produces the next enrollment according to each student's outcome, atomically and reversibly.
**Priority:** High
**Category:** Promotion
**Preconditions:** Session rollover (Doc 05).
**Business Rule:** Advance → next level instance; retain → same level's new-session instance; graduate → no new enrollment (exit); withdrawn → excluded. Prior enrollment closed as completed. The batch is atomic/recoverable and reversible within the rollback window.
**System Action:** Evaluate outcomes; create appropriate enrollments; close prior.
**Validation:** Outcomes finalized (or audited override); target instances exist.
**Failure Behavior:** Block students with unresolved outcomes; never drop silently.
**Audit Requirement:** Log per-student `ENROLLMENT_PROMOTED/RETAINED/COMPLETED`.
**Example Scenario:** A class rolls into the next grade, two students retained in the same grade's new-year section.
**Related Rules:** SESS-006, ENR-001.

**Rule ID:** ENR-008
**Rule Name:** Enrollment Status Drives Operational Eligibility
**Description:** Only an `ACTIVE` enrollment makes a student eligible for attendance, exams, and recurring fees in that session.
**Priority:** High
**Category:** Lifecycle integrity
**Preconditions:** Any operation that targets enrolled students.
**Business Rule:** Operational modules act on active enrollments; a `SUSPENDED`/`CLOSED` enrollment excludes the student from new attendance/exam/fee generation for that session (history remains).
**System Action:** Filter operational targets by active enrollment status.
**Validation:** Enrollment status current.
**Failure Behavior:** Operations on non-active enrollments blocked/excluded.
**Audit Requirement:** Log `ENROLLMENT_STATUS_CHANGED`.
**Example Scenario:** A suspended student is excluded from new fee invoices while suspended.
**Related Rules:** ATT, EXM, FEE, ENR-002.

## 5. Validation Rules
- Target session active/admission-open; target level/section instance exists.
- Single active enrollment per student per session.
- Section capacity respected.
- Roll number unique within section/session.
- Transfers same-institute, dated, target valid + capacity.
- Promotion targets exist; outcomes finalized or overridden (audited).

## 6. State Machine

**State Name:** ACTIVE
**Description:** Current placement in a session; operationally eligible.
**Allowed Transitions:** → SUSPENDED (hold); → TRANSFERRED (section/campus move, internal close-open); → COMPLETED (promotion/graduation); → CLOSED (withdrawal/transfer-out).
**Forbidden Transitions:** two ACTIVE for the same student+session.
**System Actions:** Drive attendance/exam/fee eligibility; assign roll number.

**State Name:** SUSPENDED
**Description:** Temporarily on hold (discipline, non-payment, leave).
**Allowed Transitions:** → ACTIVE (reinstated); → CLOSED.
**Forbidden Transitions:** → COMPLETED (cannot complete while suspended without reinstatement).
**System Actions:** Exclude from new operational records; retain history.

**State Name:** TRANSFERRED
**Description:** Internal placement changed; represented as prior placement closed-as-of-date.
**Allowed Transitions:** (the new placement is a fresh ACTIVE record).
**Forbidden Transitions:** reopening the old placement to accept new current-period records.
**System Actions:** Preserve prior-period attribution.

**State Name:** COMPLETED
**Description:** Session finished for this enrollment (promoted or graduated).
**Allowed Transitions:** → ARCHIVED (with session).
**Forbidden Transitions:** → ACTIVE.
**System Actions:** Freeze as historical record.

**State Name:** CLOSED
**Description:** Ended early (withdrawal/transfer-out).
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** → ACTIVE (return = new enrollment).
**System Actions:** Trigger clearance/certificate; retain.

**State Name:** ARCHIVED
**Description:** Long-term historical enrollment record.
**Allowed Transitions:** none (terminal, retained).
**Forbidden Transitions:** edits/hard-delete.
**System Actions:** Read-only retention.

## 7. Status Definitions
`ACTIVE` · `SUSPENDED` · `TRANSFERRED` (internal move marker) · `COMPLETED` (promoted/graduated) · `CLOSED` (withdrawn/transferred-out) · `ARCHIVED`.

## 8. Workflow Rules
- Enrollment via admission conversion is automatic (Doc 07); direct enrollment requires elevated permission, audited.
- Internal transfers may require approval (configurable), especially cross-campus (CAMP-006).
- Withdrawal/transfer-out runs a clearance workflow (dues/library/asset clearance) before certificate issuance.
- Promotion is a governed batch (Doc 05), reversible within the rollback window.

## 9. Permission Rules
- `enrollment.manage` — create/transfer/close enrollments within scope.
- `enrollment.promote` — run promotion (often the same as session rollover permission).
- `enrollment.view` — read enrollments (teachers see their sections; students/guardians see their own).
- All scoped to institute/campus; ownership applies for non-admin roles.

## 10. Notification Rules
- `ENROLLMENT_CREATED` → welcome/placement notice to guardian (class, section, roll number).
- `ENROLLMENT_TRANSFERRED` → notify guardian of the new placement.
- `ENROLLMENT_SUSPENDED` (e.g., non-payment hold) → notify guardian with reason.
- `ENROLLMENT_PROMOTED/RETAINED` → notify guardian of the outcome at rollover.
- `ENROLLMENT_CLOSED` → notify guardian with clearance/certificate details.

## 11. Audit Requirements
Mandatory: `ENROLLMENT_CREATED`, `ROLL_NUMBER_ASSIGNED/RESEQUENCED`, `ENROLLMENT_TRANSFERRED`, `ENROLLMENT_STATUS_CHANGED`, `ENROLLMENT_PROMOTED/RETAINED/COMPLETED`, `ENROLLMENT_CLOSED` (with reason + certificate ref), capacity/conflict blocks. With actor, student, session, level, section, effective dates.

## 12. Data Retention Rules
- Enrollment history retained long-term (progression record across sessions) — core to transcripts.
- Closed/transferred enrollments retained, not purged early; tied to the student's retention.
- Transfer-period attributions retained to keep historical attendance/marks accurate.
- Anonymization follows the student's erasure (Doc 06), preserving non-identifying progression integrity.

## 13. Edge Cases
- **Mid-session admission:** a late student enrolls into the active session; roll numbers may re-sequence (ENR-004).
- **Retained student:** stays at the same level but in the *new session's* instance — a new enrollment, not the old one continuing (ENR-007, SESS-006).
- **Repeating a withdrawn year:** re-admission creates a fresh enrollment; the old closed one remains for history.
- **Section merge/split mid-session:** treated as transfers with preserved period attribution, not data rewrites.
- **Dual-program students (e.g., coaching + regular):** only via an explicitly supported multi-program model; not an accidental second active enrollment (ENR-002).
- **Capacity counted on active enrollments vs approvals:** active enrollments are the truth for section capacity (ENR-003).
- **Promotion target level absent (final grade):** graduates exit; must not be force-promoted into a non-existent level.
- **Suspended enrollment and fees:** new recurring fees pause while suspended; prior dues remain (ENR-008).

## 14. Failure Scenarios
- **Promotion batch partial failure:** atomic/recoverable; reversible within the window; never leave a cohort half-promoted.
- **Concurrent enrollment attempts:** single-active-per-session enforced atomically.
- **Transfer to a full/closed section:** rejected; the student stays put until a valid target exists.
- **Missing fee schedule for the level:** enrollment may proceed but flags finance; never silently skip fee association where required.

## 15. Exception Handling Rules
- Enrollment violating single-active or capacity rules is rejected with a clear reason.
- Transfers and promotions are transactional; no partial placements.
- Operations targeting non-active enrollments are excluded, not errored silently.
- Cross-institute "transfer" attempts are redirected to the withdrawal + new-admission path.

## 16. Compliance Considerations
- **Accurate academic record:** period-accurate placement history underpins truthful transcripts and certificates — a legal/accreditation necessity.
- **Minors:** enrollment finalization for a minor requires guardian linkage (STU-006); guardian notifications keep responsible adults informed.
- **Clearance & certificates:** withdrawal/transfer clearance and transfer certificates follow documented, audited processes.
- **Retention:** progression history retained per academic-record retention, anonymized only per lawful erasure.

## 17. Future Considerations
- Elective/optional-subject enrollment within a level (subject-level enrollment beyond the section).
- Multi-program enrollment model (regular + coaching/skills tracks) made first-class.
- Automated section balancing and roll-number policies.
- Cross-institute enrollment history within a deployment (seamless inter-institute moves).
