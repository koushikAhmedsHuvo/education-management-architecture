# 05 — Academic Session Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`SESS-001…008`), and Use Case Repository (`UC-SESS-001…018`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Manage the academic session/year — the temporal backbone every academic, financial, and reporting record is stamped to: session definition, structure instancing, single-current-session enforcement, controlled overlap, rollover/promotion, close-readiness and post-close immutability, and exceptional governed reopen.

**Business Goal.** Provide an authoritative, immutable-after-close time boundary so that historical academic and financial data remains correct and reproducible across years, and so promotion between years honors student outcomes.

**Scope.** Session CRUD with date validity; single current session per institute; controlled overlap (current + admission-open next); per-session structure instances from the definition; rollover with outcome-based promotion (advance/retain/graduate/exit); close-readiness checks; post-close immutability; exceptional, audited reopen.

**Out of Scope.** Class/section/subject definitions (Academic modules — Session instances them). Result computation (Result module — Session gates close on publish state). Fee structures (Fee module — stamped per session). Configuration mechanics (Configuration Engine).

---

## 2. Actors

**Primary Actors.** Institute/Academic Administrator (defines, activates, rolls over, closes sessions), Registrar (promotion execution), System (overlap/immutability enforcement, structure instancing).

**Secondary Actors.** Class/Section/Subject modules (structure definitions), Result module (close-readiness on published results), Fee module (per-session stamping), Workflow Engine (rollover/reopen approval), Audit, Reporting, Notification.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Create/define session | Define a session with valid, non-contradictory start/end dates. | Critical | Institute |
| FR-002 | Single current session | At most one current session per institute at a time. | Critical | Institute scope |
| FR-003 | Controlled overlap | Allow the current session to coexist only with the next session in admission-open state. | High | Admission module |
| FR-004 | Structure instancing | Instantiate per-session class/section/subject instances from the structure definition. | High | Class/Section/Subject |
| FR-005 | Activate / set current | Promote a session to current (enforcing single-current). | Critical | FR-002 |
| FR-006 | Rollover & promotion | Roll to the next session honoring outcomes (advance/retain/graduate/exit). | High | Result, Enrollment, Workflow |
| FR-007 | Close-readiness checks | Block close until readiness (e.g., results published, dues state) is satisfied or explicitly overridden. | High | Result, Fee |
| FR-008 | Post-close immutability | Make a closed session's records immutable. | Critical | All academic/financial modules |
| FR-009 | Governed reopen | Reopen a closed session only exceptionally, with approval and audit. | High | Workflow Engine |
| FR-010 | Outcome-driven targets | Ensure promotion has valid targets (next class) or graduate/exit handling. | High | Class structure |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Session lifecycle | Define → activate (current) → rollover → close → (governed) reopen. | Authoritative academic time boundary. |
| Single current + controlled overlap | One current; next only admission-open. | Prevents ambiguous "which year" data. |
| Structure instancing | Per-session instances from definitions. | Year-specific structure without redefining. |
| Outcome-based promotion | Advance/retain/graduate/exit honored. | Correct, fair year transitions. |
| Close-readiness gate | Block premature close. | Protects data completeness. |
| Post-close immutability | Frozen history. | Reproducible, defensible records. |
| Exceptional reopen | Governed, audited. | Rare corrections without losing integrity. |

---

## 5. Screens

Session List; Session Detail (status/dates/structure/readiness); Session Create/Define; Structure Instancing (per session); Activate / Set Current (confirm); Rollover & Promotion Wizard; Bulk Promotion; Close Session (readiness report); Reopen Session (governed); Session Edit (pre-close); Promotion Summary & Session Status Report; Session Structure Import/Export; Rollover/Reopen Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Session List | Create, Open, Search/Filter, Export | — |
| Session Detail | Edit (pre-close), Activate, Instance Structure, Start Rollover, Close, Reopen, View Readiness | — |
| Create/Define | Save, Validate Dates | — |
| Structure Instancing | Generate Instances, Review, Confirm | — |
| Rollover & Promotion | Map Outcomes, Set Targets, Validate, Promote (→ approval) | Bulk Promote |
| Close Session | Run Readiness Check, Override (governed), Confirm Close | — |
| Reopen | Request Reopen with Reason (→ approval) | — |
| Promotion Summary | Run, Filter, Export | Export |

---

## 7. Forms

**Create/Define Session** — `name` (text, required, e.g. "2026"), `startDate` (date, required), `endDate` (date, required), `admissionOpen` (toggle). Validation: end > start; no contradictory dates (SESS-001); overlap allowed only as current + admission-open next (SESS-002).

**Activate / Set Current** — confirm only. Validation: no other current session for the institute (SESS-003); if another is current, require explicit rollover/close path.

**Rollover & Promotion** — per-cohort `outcome` mapping (advance/retain/graduate/exit), `targetClass` (for advance), `nextSession` (select). Validation: advance has a valid next-class target (SESS-006); graduating cohort handled (UC-SESS-018); routed to approval.

**Close Session** — readiness checklist (results published, dues reconciled per policy), `overrideReason` (text, required if overriding). Validation: readiness satisfied or governed override recorded (SESS-007); close triggers immutability (SESS-005).

**Reopen Session** — `reason` (text, required), `scope` (what to reopen). Validation: exceptional path; approval required; fully audited (SESS-008).

---

## 8. Search & Filter Requirements

**Sessions:** by name, status (draft/current/admission-open/closed/reopened), date range, institute. Sorting: start date/status/name. Pagination: server-side, 25 default. Scoped to the institute.

---

## 9. Table Requirements

**Session table:** Name, Start, End, Status, Current?, #Enrolled, Readiness, Created. Sorting on Start/Status. Filtering as above. Export (governed). **Promotion summary table:** Cohort/Class, Advanced, Retained, Graduated, Exited, Target Session.

---

## 10. Workflow Requirements

**Trigger events:** create, structure-instanced, activate, rollover/promotion, close, reopen. **Status changes:** `DRAFT → CURRENT → CLOSED → (governed) REOPENED`; next session `ADMISSION_OPEN → CURRENT` at rollover. **Approvals:** rollover/promotion and reopen via Workflow Engine. **Notifications:** session activation, rollover/promotion results (to guardians — results honor outcomes), close, reopen. **Audit:** all transitions, promotion decisions, close-overrides, reopen (immutable, with full lineage).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| View session(s) | `session.view` |
| Create/define session | `session.create` |
| Update session (pre-close) | `session.update` |
| Instance structure | `session.structure.instance` |
| Activate / set current | `session.activate` |
| Rollover / promote | `session.rollover` |
| Close session | `session.close` |
| Reopen (governed) | `session.reopen` |
| Approve rollover/reopen | `session.change.approve` |
| Import/Export structure | `session.export` |

All capabilities are institute-scoped; reopen and override are elevated/governed.

---

## 12. Business Rule References

SESS-001 (date validity), SESS-002 (controlled overlap: current + admission-open), SESS-003 (single current session per institute), SESS-004 (per-session structure instances from definition), SESS-005 (post-close immutability), SESS-006 (promotion honors outcomes), SESS-007 (close-readiness checks), SESS-008 (reopen exceptional & audited). Cross-cutting: INST-006 (institute scope), RES-006/009 (results gate close), WFL-002/004 (version-pinned approvals), AUD-001, NOT-006 (promotion notifications).

## 13. Use Case References

UC-SESS-001 (Create & Define), UC-SESS-002 (Instantiate Structure), UC-SESS-003 (Activate/Set Current), UC-SESS-004 (Rollover & Promotion), UC-SESS-005 (Close — readiness), UC-SESS-006 (Reopen — governed), UC-SESS-007 (View), UC-SESS-008 (Update pre-close), UC-SESS-009 (Approve Rollover/Reopen), UC-SESS-010 (Search), UC-SESS-011 (Promotion Summary & Status Report), UC-SESS-012 (Bulk Promotion), UC-SESS-013/014 (Import/Export), UC-SESS-015 (Rollover/Reopen Approval Workflow), UC-SESS-016 (Second-Current-Session Blocked), UC-SESS-017 (Close-With-Unpublished-Results Warning/Block), UC-SESS-018 (Promotion Target Missing).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Create / define session | POST | Academic Admin |
| List / get session(s) | GET | Admin |
| Update session (pre-close) | PUT | Admin |
| Instance structure for session | POST | Admin |
| Activate / set current | POST | Admin |
| Rollover & promote (single/bulk) | POST | Registrar (→ approval) |
| Run close-readiness check | GET | Admin |
| Close session | POST | Admin |
| Reopen session (governed) | POST | Admin (→ approval) |
| Promotion summary / status report | GET | Admin |
| Import / export session structure | POST/GET | Admin |

---

## 15. Database Requirements

**Entities:** `AcademicSession` (instituteId, name, startDate, endDate, status, isCurrent, admissionOpen), `SessionStructureInstance` (class/section/subject instances per session), `PromotionRecord` (student, outcome, fromClass, toClass, fromSession, toSession), `SessionReopenLog`. **Relationships:** Institute 1—* AcademicSession; AcademicSession 1—* SessionStructureInstance; AcademicSession 1—* PromotionRecord; session id stamped on enrollments/results/invoices. **Indexes:** partial-unique(isCurrent true per institute), index(AcademicSession.instituteId, status), index(PromotionRecord.studentId), index(SessionStructureInstance.sessionId). Closed sessions are write-locked at the data layer (SESS-005).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Session activation; promotion outcome to guardians (custody-aware, batched); close/reopen notices to admins. |
| In-App | Readiness warnings; pending rollover/reopen approvals; structure-instancing status. |
| SMS/Push | Optional promotion-result alert (minimized content). |

Promotion notifications use bulk batching/dedup (NOT-006) for large cohorts.

---

## 17. Audit Requirements

Log: create/update, structure instancing, activation, rollover/promotion (per-student outcomes), close (with readiness result and any override reason), reopen (reason, approver, scope). Record who/when/before/after. Reopen carries full lineage. Closed-session write attempts are logged as blocked. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Promotion summary (advance/retain/graduate/exit by class), Session status, Close-readiness, Year-over-year enrollment movement. **Exports:** session/promotion data (governed). **Dashboards:** academic-year overview (current session, rollover progress, readiness).

---

## 19. Error Handling

**Validation:** invalid dates, second current session, missing promotion target, close with unpublished results → specific errors (UC-SESS-016/017/018). **Permission:** non-academic-admin activating/closing → 403. **Workflow:** rollover/reopen pending approval → clear state. **System:** Result/Fee readiness service unavailable → block close (fail safe); write to closed session → rejected (immutability).

---

## 20. Edge Cases

**Concurrent updates:** two activations racing → single-current guarantee, one wins (UC-SESS-016). **Duplicate data:** duplicate structure instancing → idempotent. **Partial failures:** bulk promotion partial → per-student report, completed promotions stand. **Rollback:** rollover approval reversed mid-flight → promotions not applied until approved/committed atomically. **Close/reopen race:** reopen during dependent reads → reads see consistent state; reopen scope explicit.

---

## 21. Acceptance Criteria

**Functional.** Session dates must be valid; at most one current session exists per institute; the only permitted overlap is current + admission-open next; structure instances derive from definitions per session; rollover honors each student's outcome with valid targets (or graduate/exit); a session cannot close until readiness is satisfied or explicitly overridden; closed sessions are immutable; reopen is exceptional, approved, and fully audited.

**Business.** Every academic and financial record is correctly stamped to a session and remains reproducible after close; year transitions are fair and outcome-honoring; historical integrity is structurally protected while rare corrections remain possible under governance.

---

## 22. Future Enhancements

Term/semester sub-periods within a session; configurable promotion rules per institution type; automated readiness remediation suggestions; multi-session financial carry-forward dashboards; predictive rollover planning; calendar integration (holidays/exam windows) feeding readiness.
