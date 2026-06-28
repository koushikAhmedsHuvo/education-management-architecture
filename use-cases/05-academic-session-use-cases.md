# 05 — Academic Session Use Cases

Transforms the Academic Session business rules (`SESS-001`…`SESS-008`) into use cases. The session is the time dimension; structure instances are per-session; promotion honors outcomes; closed sessions are immutable except via governed reopen.

## 1. Primary Actors
Organization / Institute Administrator (creates, activates, rolls over, closes, reopens sessions).

## 2. Secondary Actors
System (lifecycle, single-current invariant, instance generation, promotion, immutability), Workflow Engine (rollover/reopen approvals), Result/Enrollment modules (promotion inputs/outputs), Audit Service.

## 3. Goals
Define academic time periods; instantiate structure per session; support a current session running alongside an admission-open future session; promote students by outcome (advance/retain/graduate/exit) atomically; close sessions cleanly; protect closed records while allowing governed corrections.

## 4. User Journeys
- **Open a year:** admin defines a session (dates, type) → instantiates classes/sections from the definition → marks it current → operations run.
- **Admissions overlap:** while the current session runs, a future session is created and admission-open (SESS-002) without disturbing current operations.
- **Roll over:** admin initiates rollover → next-session structure instantiated → each student advanced/retained/graduated per results → old session closed.
- **Correct history:** a discovered error in a closed session triggers a governed, audited reopen → fix → re-close.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core/CRUD | UC-SESS-001 | Create & Define Session | High |
| Core | UC-SESS-002 | Instantiate Structure for Session | High |
| Core | UC-SESS-003 | Activate / Set Current Session | Critical |
| Core | UC-SESS-004 | Session Rollover & Promotion | Critical |
| Core | UC-SESS-005 | Close Session (close-readiness) | High |
| Core | UC-SESS-006 | Reopen Closed Session (governed) | High |
| CRUD | UC-SESS-007 | View Session(s) | Medium |
| CRUD | UC-SESS-008 | Update Session (pre-close) | Medium |
| Approval | UC-SESS-009 | Approve Rollover / Reopen | High |
| Search | UC-SESS-010 | Search Sessions | Low |
| Reporting | UC-SESS-011 | Promotion Summary & Session Status Report | Medium |
| Bulk | UC-SESS-012 | Bulk Promotion (rollover batch) | Critical |
| Import | UC-SESS-013 | Import Session Structure | Low |
| Export | UC-SESS-014 | Export Session / Promotion Data | Low |
| Workflow | UC-SESS-015 | Rollover / Reopen Approval Workflow | High |
| Exception | UC-SESS-016 | Second-Current-Session Blocked | High |
| Exception | UC-SESS-017 | Close-With-Unpublished-Results Warning/Block | High |
| Exception | UC-SESS-018 | Promotion Target Missing (graduating cohort) | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-SESS-003 — Activate / Set Current Session
- **Module:** Academic Session · **Priority:** Critical
- **Actors:** Institute Administrator (primary), System
- **Goal:** Make exactly one session the operational "current" per institute, even when others (admission-open) coexist.
- **Description:** Activates a session and designates it current; marking a new current demotes the prior, atomically. Overlap with a future/admission-open session is allowed; only one is current.
- **Business Rules Applied:** SESS-003, SESS-002, SESS-001, SESS-004.
- **Preconditions:** Session defined and structure instantiated; institute `ACTIVE`.
- **Trigger:** Admin activates/sets-current a session.
- **Main Success Scenario:**
  1. Admin selects a session to activate / mark current.
  2. System validates a single-current invariant (no two current at once).
  3. System sets the session `ACTIVE` and `current`, atomically demoting any prior current (typically during rollover).
  4. Default academic operations (attendance "today", etc.) resolve to this session.
- **Alternative Flows:** A1) A future session is created and marked admission-open alongside the current (SESS-002) — not current.
- **Exception Flows:** E1) Marking a second concurrent current → blocked (UC-SESS-016).
- **Validation Rules:** Exactly one `current` per institute; structure instantiable; dates valid (SESS-001/003/004).
- **Permissions Required:** `institute.session.manage`.
- **Notifications Triggered:** `CURRENT_SESSION_CHANGED` to admins/academic staff.
- **Audit Events Generated:** `SESSION_ACTIVATED`, `CURRENT_SESSION_CHANGED`.
- **Data Created:** None (instances created in UC-SESS-002).
- **Data Updated:** Session state/current flag; prior current demoted.
- **Data Deleted:** None.
- **Post Conditions:** One current session drives default operations; overlaps coexist correctly.
- **Related Use Cases:** UC-SESS-002, UC-SESS-004, UC-SESS-016.
- **Acceptance Criteria:**
  - Given a defined, instantiated session, When it is set current, Then any previous current is demoted and exactly one current remains.
  - Given a future admission-open session, When it coexists with the current, Then operational defaults still resolve unambiguously to the current.
  - Given an attempt to set a second current, When requested, Then it is blocked.
- **Edge Case Analysis:**
  - *Invalid Input:* activating an uninstantiated session blocked.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* two admins setting current simultaneously → single-current enforced atomically; one wins.
  - *Duplicate Data:* re-setting the same current → idempotent.
  - *System Failure:* atomic transition; never two current.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* set current; coexist with admission-open future.
  - *Negative:* second current blocked; activate uninstantiated.
  - *Boundary:* "today" resolution at the session date boundary (institute time zone).

### UC-SESS-004 — Session Rollover & Promotion
- **Module:** Academic Session · **Priority:** Critical
- **Actors:** Institute Administrator (primary), System, Approver
- **Goal:** Move from one session to the next, promoting each student by outcome, atomically and reversibly.
- **Description:** Instantiates next-session structure from the definition (no duplication), evaluates each student's outcome (advance/retain/graduate/exit), creates next-session enrollments, and closes the old session.
- **Business Rules Applied:** SESS-006, SESS-004, SESS-005, ENR-007, RES (outcomes).
- **Preconditions:** A next session exists or is created in-flow; results finalized (or audited override); actor holds `institute.session.manage`/`enrollment.promote`.
- **Trigger:** Admin initiates rollover.
- **Main Success Scenario:**
  1. Admin initiates rollover to the next session.
  2. System instantiates next-session structure from the definition.
  3. For each active enrollment, System evaluates the outcome: advance → next level instance; retain → same level's new-session instance; graduate → exit; withdrawn → excluded.
  4. System creates the appropriate next-session enrollments and closes prior ones as completed.
  5. Carry-forward items (dues, pending) handled per policy; old session → `CLOSED`.
- **Alternative Flows:** A1) Selective/partial promotion. A2) Rollover requires approval (UC-SESS-009).
- **Exception Flows:** E1) Unresolved results → block per policy or audited override (UC-SESS-017). E2) Promotion target level missing → block until structure ready (UC-SESS-018).
- **Validation Rules:** Promotion policy configured; results finalized or overridden; target instances exist (SESS-006/ENR-007).
- **Permissions Required:** `institute.session.manage` / `enrollment.promote`.
- **Notifications Triggered:** `SESSION_ROLLOVER_COMPLETED` (summary); promotion outcomes to guardians per preference.
- **Audit Events Generated:** `SESSION_ROLLOVER_STARTED/COMPLETED`; per-student `STUDENT_PROMOTED/RETAINED/GRADUATED`.
- **Data Created:** Next-session structure instances; next-session enrollments.
- **Data Updated:** Prior enrollments → completed; old session → `CLOSED`.
- **Data Deleted:** None.
- **Post Conditions:** New session active with promoted cohort; old session closed/immutable; reversible within the rollback window.
- **Related Use Cases:** UC-SESS-003, UC-SESS-012, UC-ENR (promotion).
- **Acceptance Criteria:**
  - Given finalized results, When rollover runs, Then each student is advanced/retained/graduated per outcome and the old session is closed.
  - Given a retained student, When rollover runs, Then they land in the same level's new-session instance (not the old one continuing).
  - Given a final-level cohort, When rollover runs, Then they graduate and are not force-promoted into a non-existent level.
  - Given a partial failure, When rollover runs, Then it is recoverable/reversible within the window with no half-promoted cohort.
- **Edge Case Analysis:**
  - *Invalid Input:* rollover with unconfigured promotion policy blocked.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* concurrent rollover attempts → serialized; idempotent; one run.
  - *Duplicate Data:* re-running rollover does not double-promote (idempotent).
  - *System Failure:* atomic/recoverable batch; reversible within window; no half-promotion.
  - *Workflow Failure:* approval unresolved → rollover pending, escalates.
- **QA Coverage:**
  - *Positive:* full advance; mixed advance/retain/graduate; carry-forward dues.
  - *Negative:* unresolved results; missing target level; unauthorized.
  - *Boundary:* single-student cohort; entire cohort retained; rollback at the window edge.

### UC-SESS-006 — Reopen Closed Session (governed)
- **Module:** Academic Session · **Priority:** High
- **Actors:** Institute Administrator (elevated, primary), System, Approver
- **Goal:** Correct an error in a closed session under strict governance.
- **Description:** Reopen temporarily lifts immutability for an intended correction; requires elevated permission, recorded justification, optional approval, and re-closure.
- **Business Rules Applied:** SESS-008, SESS-005, AUTHZ-009.
- **Preconditions:** Session `CLOSED`; actor holds `institute.session.reopen`; justification provided; approval if configured.
- **Trigger:** A discovered error requires changing closed-session records.
- **Main Success Scenario:**
  1. Authorized admin requests reopen with a recorded justification.
  2. (If configured) approval obtained via the Workflow Engine (SoD).
  3. System transitions `CLOSED → ACTIVE` under audit, lifting immutability for the intended scope.
  4. The correction is made via the relevant governed correction flow.
  5. System re-closes the session (`SESSION_RECLOSED`).
- **Alternative Flows:** A1) Reopen scoped to a single record-type for the correction.
- **Exception Flows:** E1) Unauthorized/ungoverned reopen → rejected. E2) Backdating new operational data into the reopened session → constrained and logged.
- **Validation Rules:** Actor authorized; justification captured; approval obtained; re-closure required (SESS-008).
- **Permissions Required:** `institute.session.reopen` (elevated; + approval).
- **Notifications Triggered:** `SESSION_REOPENED` to admins + governance/security channel; `SESSION_RECLOSED`.
- **Audit Events Generated:** `SESSION_REOPENED`, `SESSION_RECLOSED` (actor, justification, scope of change).
- **Data Created:** Correcting entries (via governed correction).
- **Data Updated:** Session state CLOSED → ACTIVE → CLOSED; corrected records (as new linked entries).
- **Data Deleted:** None.
- **Post Conditions:** Correction applied transparently; session re-closed; full audit.
- **Related Use Cases:** UC-SESS-005, UC-SESS-009.
- **Acceptance Criteria:**
  - Given a closed session and a justified, authorized request, When reopened, Then immutability is lifted only for the correction and the session is re-closed afterward.
  - Given an unauthorized or unjustified reopen, When attempted, Then it is rejected.
  - Given a reopen, When complete, Then both reopen and re-close are audited with justification.
- **Edge Case Analysis:**
  - *Invalid Input:* reopen without justification rejected.
  - *Permission Failure:* lacks `session.reopen` → 403.
  - *Concurrent Update:* concurrent reopen requests → serialized; one governed reopen.
  - *Duplicate Data:* re-closure idempotent.
  - *System Failure:* if re-closure fails, session flagged as open-too-long and alerted.
  - *Workflow Failure:* approval unresolved → reopen not granted.
- **QA Coverage:**
  - *Positive:* governed reopen → correct → re-close.
  - *Negative:* no permission; no justification; left-open detection.
  - *Boundary:* reopen for a single correcting entry; SoD on approver.

---

## 7. Compact Specifications (routine use cases)

- **UC-SESS-001 — Create & Define Session** · *High* · Rules: SESS-001, SESS-002. Define name, dates, type; date validity; overlap permitted per policy. *Permissions:* `institute.session.manage`. *Audit:* `SESSION_CREATED`. *Edge:* start<end; open-ended only for permitted types. *QA:* valid dates; invalid range; overlap policy.
- **UC-SESS-002 — Instantiate Structure for Session** · *High* · Rules: SESS-004. Generate class/section instances from the definition (no duplication). *Audit:* `SESSION_STRUCTURE_INSTANTIATED`. *Edge:* incomplete definition blocks instantiation; definition edits don't alter past instances. *QA:* instances created; historical fidelity.
- **UC-SESS-005 — Close Session (close-readiness)** · *High* · Rules: SESS-007, SESS-005. Run close-readiness checklist (unpublished results, dues, in-flight workflows); warn/block; record carry-forward. *Audit:* `SESSION_CLOSE_CHECK`, `SESSION_CLOSED`. *Edge:* hard-stops block, soft items warn. *QA:* clean close; blocked on unpublished results; carry-forward recorded.
- **UC-SESS-007 — View Session(s)** · *Medium* · Rules: AUTHZ-002. Scoped read. *QA:* scope respected.
- **UC-SESS-008 — Update Session (pre-close)** · *Medium* · Rules: SESS-001, SESS-005. Edit dates/details before close; closed sessions immutable. *Audit:* `SESSION_UPDATED`. *Edge:* edits on closed session rejected. *QA:* pre-close edit; closed edit rejected.
- **UC-SESS-009 — Approve Rollover / Reopen** · *High* · Rules: WFL-004, AUTHZ-009. Approver (≠ requester) approves high-impact session actions. *QA:* approval gates; SoD; escalation.
- **UC-SESS-010 — Search Sessions** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-SESS-011 — Promotion Summary & Session Status Report** · *Medium* · Rules: REP-002. Report rollover outcomes and session status. *Edge:* scoped; as-of freshness. *QA:* counts match promotion audit.
- **UC-SESS-012 — Bulk Promotion (rollover batch)** · *Critical* · Rules: SESS-006, ENR-007. The batch engine behind UC-SESS-004; atomic/recoverable/reversible. *Edge:* partial-failure recovery; idempotent. *QA:* large cohort; partial failure; rollback.
- **UC-SESS-013 — Import Session Structure** · *Low* · Rules: SESS-004, CFG-002. Import a structure definition for instantiation. *QA:* valid import; invalid blocked.
- **UC-SESS-014 — Export Session / Promotion Data** · *Low* · Rules: REP-005. Governed export. *QA:* scoped export.
- **UC-SESS-015 — Rollover / Reopen Approval Workflow** · *High* · Rules: WFL-002/004. Version-pinned approval. *QA:* version-pinning; SoD; escalation.
- **UC-SESS-016 — Second-Current-Session Blocked (Exception)** · *High* · Rules: SESS-003. Cannot have two current sessions. *QA:* blocked; single-current preserved.
- **UC-SESS-017 — Close-With-Unpublished-Results Warning/Block (Exception)** · *High* · Rules: SESS-007. Close-readiness surfaces unpublished results. *QA:* warn/block per policy; override audited.
- **UC-SESS-018 — Promotion Target Missing (Exception)** · *High* · Rules: SESS-006, ENR-007. Graduating cohort exits; missing next-level blocks promotion (no silent drop). *QA:* graduation exit; missing target blocks.

## 8. Module-level QA & Edge Themes
- **Current vs overlap:** test that admission-open future sessions coexist with the single current without ambiguity (SESS-002/003).
- **Promotion by outcome:** advance/retain/graduate/exit each verified; retained student lands in the *new-session* instance.
- **Reopen governance:** the closed-session correction path is a key controlled-access suite.
- **Time zone:** "today" resolves in the institute/shift time zone at session boundaries.
