# 05 — Academic Session Business Rules

## 1. Module Purpose
Govern the **academic session** — the time dimension (year / term / semester) that academics, attendance, examinations, results, and fees attach to. The session is what makes the institute's structure *operational for a period*: the timeless structure **definition** is realized as concrete **instances** per session (D38). Sessions also encode the realities ERP teams routinely under-model: **two sessions can be non-archived at once** (the current one running while the next one's admissions are open), and **promotion is not duplication** (rollover creates new instances, it does not copy the structure).

## 2. Actors
- **Organization / Institute Administrator** — creates, activates, closes, and rolls over sessions.
- **System** — enforces session lifecycle, the single-current invariant, rollover/promotion, and post-close immutability.

## 3. Use Cases

**Use Case ID:** UC-SESS-01 — Create & Activate a Session
**Actors:** Institute Administrator, System
**Description:** Define a new academic session and make it operational.
**Preconditions:** Institute `ACTIVE`; actor holds `institute.session.manage`.
**Main Flow:** 1) Admin defines a session (name, start/end dates, type — year/term/semester). 2) System validates dates and overlap policy. 3) Admin instantiates the academic structure for the session (classes/sections per the definition). 4) Admin marks the session **current** (or schedules it). 5) Session `ACTIVE`.
**Alternative Flow:** A1) Admission-open session created alongside the current one (SESS-002) for next-cycle admissions.
**Exception Flow:** E1) Invalid/overlapping dates beyond policy → reject. E2) Activating a second *current* session → block (SESS-003).
**Post Conditions:** Session active; structure instances exist; events audited.
**Business Rules Applied:** SESS-001, SESS-002, SESS-003, SESS-004.

**Use Case ID:** UC-SESS-02 — Session Rollover & Promotion
**Actors:** Institute Administrator, System
**Description:** Move from one session to the next, promoting students.
**Preconditions:** A next session exists (or is created in-flow); current session ready to close.
**Main Flow:** 1) Admin initiates rollover to the next session. 2) System instantiates next-session structure from the **definition** (no duplication). 3) Students are promoted per promotion rules (advance, retain, graduate, or transfer-out). 4) Carry-forward items (dues, pending records) handled per policy. 5) Old session → `CLOSED`.
**Alternative Flow:** A1) Selective promotion — some students retained in the same level (SESS-006). A2) Graduating cohort exits the structure.
**Exception Flow:** E1) Unpublished results or blocking dues at close → warn/block per policy (SESS-007). E2) Promotion target level missing in next session → block until structure is ready.
**Post Conditions:** New session active with promoted enrollments; old session closed and immutable; full audit.
**Business Rules Applied:** SESS-005, SESS-006, SESS-007, SESS-008.

## 4. Business Rules

**Rule ID:** SESS-001
**Rule Name:** Session Date Validity
**Description:** A session has a coherent start and end (or an explicit open-ended mode) in the deployment's locale.
**Priority:** High
**Category:** Temporal integrity
**Preconditions:** Session creation/edit.
**Business Rule:** Start < end; dates interpreted in the deployment/institute locale and time zone. An open-ended (rolling) mode is allowed only for institution types that use it (e.g., coaching) and is explicitly flagged.
**System Action:** Validate dates; store with locale/time-zone context.
**Validation:** start < end (unless open-ended); within sane bounds.
**Failure Behavior:** Reject invalid date ranges.
**Audit Requirement:** Log `SESSION_CREATED` with dates.
**Example Scenario:** A 2026 academic year runs Jan–Dec in the institute's time zone.
**Related Rules:** SESS-002, SESS-003.

**Rule ID:** SESS-002
**Rule Name:** Controlled Session Overlap (Current + Admission-Open)
**Description:** More than one non-archived session may exist so next-cycle admissions run while the current session is live.
**Priority:** Critical
**Category:** Lifecycle (commonly mis-modeled)
**Preconditions:** A future session is created during an active session.
**Business Rule:** The system supports a **current** session (operational academics) and one or more **future/admission-open** sessions simultaneously. Overlap is permitted by policy for admissions and planning; academic operations (attendance/exams) run against the *current* session, while admissions can target a future session.
**System Action:** Allow concurrent non-archived sessions; tag exactly one as `current`.
**Validation:** Overlap permitted per configurable policy; future session dates are after (or planned).
**Failure Behavior:** Disallowed overlap (per policy) rejected.
**Audit Requirement:** Log creation of overlapping/admission-open sessions.
**Example Scenario:** In Nov 2026, the 2026 session is current while 2027 admissions are open against the 2027 session.
**Related Rules:** SESS-003, ADM (admission targets a session, Doc 07).

**Rule ID:** SESS-003
**Rule Name:** Single Current Session per Institute
**Description:** Exactly one session is the operational "current" per institute at any time, even when others are open.
**Priority:** Critical
**Category:** Lifecycle invariant
**Preconditions:** A session is marked current.
**Business Rule:** Among an institute's sessions, exactly one carries the `current` designation that drives default academic operations. Marking a new current session demotes the previous (typically as part of rollover).
**System Action:** Enforce one-current; reassign atomically on rollover.
**Validation:** No two sessions of one institute hold `current` simultaneously.
**Failure Behavior:** Block a second concurrent current designation.
**Audit Requirement:** Log `CURRENT_SESSION_CHANGED`.
**Example Scenario:** Attendance "today" always resolves to the single current session unambiguously.
**Related Rules:** SESS-002, SESS-005.

**Rule ID:** SESS-004
**Rule Name:** Structure Instances Are Per-Session, From the Definition
**Description:** Each session instantiates the institute's structure; it never duplicates the definition.
**Priority:** Critical
**Category:** Definition/instance separation (D38)
**Preconditions:** A session is set up.
**Business Rule:** Classes/sections/subject offerings for a session are *instances* created from the timeless structure definition. Editing the definition does not retroactively alter past sessions' instances; each session's academics are self-contained and historically faithful.
**System Action:** Generate session instances from the current definition; pin them to the session.
**Validation:** Definition exists; instances reference it without copying it.
**Failure Behavior:** Block session activation if structure cannot be instantiated.
**Audit Requirement:** Log `SESSION_STRUCTURE_INSTANTIATED`.
**Example Scenario:** Renaming a class in the definition for 2027 does not rewrite the 2026 session's records.
**Related Rules:** INST-002, Academic rules (Docs 11–13).

**Rule ID:** SESS-005
**Rule Name:** Post-Close Immutability
**Description:** A closed session's academic and financial records are immutable.
**Priority:** Critical
**Category:** Record integrity
**Preconditions:** Session `CLOSED`.
**Business Rule:** Once closed, marks, results, attendance, and invoices for the session cannot be altered through normal operations. Corrections require an explicit, audited reopen or a correcting entry, never silent edits.
**System Action:** Make session records read-only; route corrections through governed reopen/correction flows.
**Validation:** Operations on closed-session records are blocked unless via the correction path.
**Failure Behavior:** Edit attempts on closed records rejected.
**Audit Requirement:** Log any `SESSION_REOPENED` and correcting entries with justification.
**Example Scenario:** A 2025 transcript cannot be quietly changed in 2027; a correction is a documented, audited event.
**Related Rules:** SESS-007, RES/EXM/FEE immutability rules.

**Rule ID:** SESS-006
**Rule Name:** Promotion Honors Outcomes (Advance / Retain / Graduate / Exit)
**Description:** Rollover promotes each student per their result/eligibility, not blindly.
**Priority:** High
**Category:** Promotion logic
**Preconditions:** Rollover initiated; results/eligibility known.
**Business Rule:** Promotion is per-student: eligible students advance to the next level's instance; failing/ineligible students are **retained** in the same level's new-session instance; final-level students **graduate** (exit the structure); withdrawn students are excluded. Promotion rules are configurable.
**System Action:** Evaluate each student's outcome; create the appropriate next-session enrollment.
**Validation:** Promotion policy configured; results finalized for affected students (or explicit override).
**Failure Behavior:** Block promotion for students with unresolved results unless overridden (audited).
**Audit Requirement:** Log `STUDENT_PROMOTED/RETAINED/GRADUATED` per student with the basis.
**Example Scenario:** Of a class, most advance to the next grade, two are retained, and the top cohort graduates — all in one audited rollover.
**Related Rules:** SESS-005, RES (results), ENR (enrollment, Doc 08).

**Rule ID:** SESS-007
**Rule Name:** Close-Readiness Checks
**Description:** Closing a session warns or blocks on unfinished business (unpublished results, unhandled dues).
**Priority:** High
**Category:** Lifecycle gating
**Preconditions:** Session close requested.
**Business Rule:** Before close, the system checks for unpublished results, in-flight workflows, and outstanding fees, and warns or blocks per configurable policy. Carry-forward of dues to the next session is an explicit choice, not an accident.
**System Action:** Run the close-readiness checklist; warn/block; record carry-forward decisions.
**Validation:** Checklist evaluated; overrides require justification.
**Failure Behavior:** Block close on hard-stop items; warn on soft items.
**Audit Requirement:** Log `SESSION_CLOSE_CHECK` results and the close decision.
**Example Scenario:** An admin closing a session is warned that one class's results are unpublished and chooses to resolve them first.
**Related Rules:** SESS-005, SESS-008, FEE carry-forward.

**Rule ID:** SESS-008
**Rule Name:** Reopen Is Exceptional and Audited
**Description:** Reopening a closed session is a rare, governed, fully-audited action.
**Priority:** High
**Category:** Integrity / governance
**Preconditions:** A closed session needs correction.
**Business Rule:** Reopen requires elevated permission (and may require approval), a recorded justification, and re-closure afterward; it temporarily lifts immutability only for the intended correction. Backdating new operational data into a reopened session is constrained and logged.
**System Action:** Transition `CLOSED → ACTIVE` under audit; restore `CLOSED` after correction.
**Validation:** Actor authorized; justification provided; (optional) approval obtained.
**Failure Behavior:** Unauthorized/ungoverned reopen rejected.
**Audit Requirement:** Log `SESSION_REOPENED` and `SESSION_RECLOSED` with actor, justification, scope of change.
**Example Scenario:** A discovered marking error in a closed session is fixed only via an approved, audited reopen.
**Related Rules:** SESS-005, AUTHZ-009 (SoD on reopen approval).

## 5. Validation Rules
- start < end (unless explicitly open-ended for a permitted type).
- Exactly one `current` session per institute.
- Overlap permitted only per configured policy.
- Structure must be instantiable before activation.
- Closed-session records are read-only except via governed reopen/correction.
- Promotion requires finalized results (or an audited override).

## 6. State Machine

**State Name:** DRAFT
**Description:** Session defined, structure not yet instantiated; not operational.
**Allowed Transitions:** → ACTIVE (instantiated and started/scheduled); → ARCHIVED (abandoned draft).
**Forbidden Transitions:** → CLOSED (cannot close a never-active session).
**System Actions:** Validate dates; allow structure instantiation.

**State Name:** ACTIVE
**Description:** Operational session (may be the current one or a future/admission-open one).
**Allowed Transitions:** → CLOSED (end-of-cycle/rollover).
**Forbidden Transitions:** → DRAFT.
**System Actions:** Permit academic/financial operations; enforce single-current for the `current` flag.

**State Name:** CLOSED
**Description:** Ended; records immutable.
**Allowed Transitions:** → ACTIVE only via governed `SESSION_REOPENED` (SESS-008); → ARCHIVED.
**Forbidden Transitions:** silent edits to records.
**System Actions:** Make records read-only; allow only governed corrections.

**State Name:** ARCHIVED
**Description:** Long-term closed; read-only history, retained.
**Allowed Transitions:** none typical (terminal); reactivation only by exceptional audited policy.
**Forbidden Transitions:** hard-delete (academic/financial history must persist).
**System Actions:** Retain per retention class; serve historical reads.

## 7. Status Definitions
`DRAFT` (defined, not operational) · `ACTIVE` (operational; one is also flagged `current`) · `CLOSED` (ended, immutable) · `ARCHIVED` (long-term, read-only, retained). Flag: `current` (exactly one per institute among ACTIVE sessions).

## 8. Workflow Rules
- Session creation/activation is a direct admin action, audited.
- **Rollover/promotion** is a governed batch operation; it may require approval (configurable) given its breadth, and is reversible within a defined window (bulk-operation rollback).
- **Session close** runs the close-readiness checklist; hard-stops block, soft items warn.
- **Reopen** requires elevated permission and may require approval + recorded justification (SESS-008), with SoD on the approver where configured.

## 9. Permission Rules
- `institute.session.manage` — create/activate/close sessions and run rollover.
- `institute.session.reopen` — the elevated, separately-held permission to reopen a closed session.
- `institute.session.view` — read session data (most academic roles).
- Reopen and rollover are high-impact and should be narrowly granted and audited.

## 10. Notification Rules
- `SESSION_ACTIVATED` / `CURRENT_SESSION_CHANGED` → notify institute admins and academic staff.
- `SESSION_ROLLOVER_COMPLETED` → notify admins with a promotion summary; notify guardians of promotion outcomes per preference.
- `SESSION_CLOSED` → notify admins; `SESSION_REOPENED` → notify admins and a governance/security channel.
- Close-readiness warnings → notify the initiating admin.

## 11. Audit Requirements
Mandatory: `SESSION_CREATED`, `SESSION_STRUCTURE_INSTANTIATED`, `SESSION_ACTIVATED`, `CURRENT_SESSION_CHANGED`, `SESSION_ROLLOVER_STARTED/COMPLETED`, `STUDENT_PROMOTED/RETAINED/GRADUATED`, `SESSION_CLOSE_CHECK`, `SESSION_CLOSED`, `SESSION_REOPENED/RECLOSED`. With actor, institute, session, before/after, justification (for reopen), timestamp.

## 12. Data Retention Rules
- Session records (academics, finance) follow long, immutable academic/financial retention — transcripts and ledgers may be required for many years.
- Closed and archived sessions are never auto-purged before their legal/retention minimum.
- Promotion history retained to reconstruct a student's progression across sessions.
- Erasure (if ever lawful) is retention-driven, export-before-purge anonymization, preserving non-identifying integrity.

## 13. Edge Cases
- **Overlap during admissions:** the current session runs while a future session takes admissions — explicitly supported (SESS-002), a frequent modeling miss.
- **Retention (failing a grade):** a retained student stays in the same level's *new-session* instance, not the old one — promotion is not all-or-nothing (SESS-006).
- **Graduating cohort:** final-level students exit the structure on rollover; they must not be auto-promoted into a non-existent next level.
- **Open-ended sessions (coaching):** no fixed end; rollover/close semantics differ and must be explicitly supported per type.
- **Closing with unpublished results:** caught by close-readiness checks, not discovered later (SESS-007).
- **Backdating into a reopened session:** constrained and logged; reopening is not a license to rewrite history freely (SESS-008).
- **Time zone at boundaries:** "today's attendance" near midnight must resolve in the institute's time zone, not the server's.
- **Mid-session structure change:** affects the *definition* for future sessions; the current session's instances remain stable (SESS-004).

## 14. Failure Scenarios
- **Rollover partially fails:** treated atomically/recoverably; never leave half-promoted cohorts; reversible within the rollback window.
- **Promotion target missing:** if the next level's instance doesn't exist, block and require structure setup — never silently drop a student.
- **Two admins marking current simultaneously:** single-current invariant enforced atomically; one wins.
- **Reopen left open:** governance requires re-closure; an open-too-long reopened session is flagged.

## 15. Exception Handling Rules
- Lifecycle transitions are validated, transactional, and audited.
- Closed-session edits outside the correction path are rejected with explicit messaging.
- Promotion exceptions (unresolved results) block per policy rather than guessing an outcome.
- Reopen without justification/authority is rejected.

## 16. Compliance Considerations
- **Academic record permanence & accuracy:** immutability and promotion history protect the integrity of transcripts — a core institutional and legal obligation.
- **Minors' progression data:** retained under retention rules; corrections are transparent and audited, never silent.
- **Auditability of reopen:** governed reopen with justification supports regulator and accreditation scrutiny.

## 17. Future Considerations
- Multi-term sessions (a year containing semesters/terms) with nested instance lifecycles.
- Automated promotion proposals (system suggests advance/retain from results for admin confirmation).
- Cross-session analytics (cohort progression) via the consent-bound analytics track.
- Configurable academic-calendar events (holidays, exam windows) bound to the session.
