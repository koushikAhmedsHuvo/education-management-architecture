# 14 — Examination Business Rules

## 1. Module Purpose
Govern **examinations** — defining assessments, scheduling them, gating eligibility, capturing marks, and locking them. Examination is the integrity-critical precursor to results (Doc 15): it produces the raw marks that grading (Doc 16) and result processing consume. The dominant design concern is **record integrity** — marks are tamper-sensitive data about minors that drive life-affecting outcomes, so the module enforces ownership-scoped entry, validation against maxima, submit-and-lock immutability, governed moderation, and complete auditability.

## 2. Actors
- **Exam Controller / Academic Coordinator** — defines exams, schedules, eligibility, moderation policy, and locks marks.
- **Subject Teacher** — enters and submits marks for their assigned subjects/sections (ownership, Doc 13/SUB-004).
- **Verifier / Moderator** — reviews/moderates marks before lock where configured.
- **Student / Guardian** — receive admit cards and (post-publish) results; read-scoped to own.
- **System** — enforces eligibility, validation, ownership, locking, moderation bounds, and audit.

## 3. Use Cases

**Use Case ID:** UC-EXM-01 — Define & Schedule an Exam
**Actors:** Exam Controller, System
**Description:** Configure an exam (term/type), its subjects with maxima/pass marks/weightages, and its schedule.
**Preconditions:** Session active; curriculum mapped (Doc 13); actor holds `exam.manage`.
**Main Flow:** 1) Define exam (type/term, applicable classes/streams). 2) For each subject: max marks, pass marks, component split (theory/practical), weightage toward the term/final. 3) Schedule dates/times per subject. 4) Exam becomes available for eligibility and (later) mark entry.
**Alternative Flow:** A1) Continuous-assessment exam with multiple small components over time.
**Exception Flow:** E1) Subject not in curriculum for the class → reject. E2) Component weightages inconsistent → reject (links SUB-003).
**Post Conditions:** Exam defined and scheduled; audited.
**Business Rules Applied:** EXM-001, EXM-002, EXM-008.

**Use Case ID:** UC-EXM-02 — Determine Eligibility & Issue Admit Cards
**Actors:** Exam Controller, System
**Description:** Decide which students may sit the exam and issue admit cards.
**Preconditions:** Exam scheduled; eligibility rules configured.
**Main Flow:** 1) System evaluates eligibility (attendance threshold ATT-009, fee clearance, registration). 2) Eligible students get admit cards (document pipeline). 3) Ineligible students are flagged with the reason; overrides are authorized + audited.
**Exception Flow:** E1) Below-threshold attendance → ineligible unless overridden (ATT-009). E2) Fee hold → ineligible per policy until cleared/waived.
**Post Conditions:** Eligibility set; admit cards issued; audited.
**Business Rules Applied:** EXM-003.

**Use Case ID:** UC-EXM-03 — Enter, Verify & Lock Marks
**Actors:** Subject Teacher, Verifier, Exam Controller, System
**Description:** Capture marks, verify/moderate, then lock for results.
**Preconditions:** Exam conducted; teacher owns the subject/section; mark-entry window open.
**Main Flow:** 1) Teacher enters marks per student per component, validated against maxima. 2) Teacher submits. 3) Verifier/moderator reviews; moderation/grace applied within bounds if configured. 4) Controller locks marks → immutable; results may proceed.
**Alternative Flow:** A1) Absent students marked `AB`; malpractice flagged (EXM-006).
**Exception Flow:** E1) Mark > max or invalid → reject the entry (EXM-004). E2) Submit with missing students → block or flag per policy. E3) Edit after lock → governed correction only (EXM-007).
**Post Conditions:** Marks locked and immutable; audit complete; results unblocked.
**Business Rules Applied:** EXM-004, EXM-005, EXM-006, EXM-007.

## 4. Business Rules

**Rule ID:** EXM-001
**Rule Name:** Configurable Exam Definition
**Description:** Exams are configurable (type/term, applicable scope, weightage) per institute via the Configuration Engine.
**Priority:** High
**Category:** Configuration
**Preconditions:** Exam creation.
**Business Rule:** Exam types/terms (unit test, midterm, final, continuous assessment), their applicability (class/stream), and their weightage toward a term/final result are configurable and version-stamped; results reference the exam configuration in effect.
**System Action:** Resolve exam config; stamp the version on the exam.
**Validation:** Type/term valid; applicable scope exists; weightages consistent.
**Failure Behavior:** Reject inconsistent definitions.
**Audit Requirement:** Log `EXAM_DEFINED/EDITED` with config version.
**Example Scenario:** A "Final Term" exam weights 60% with two midterms at 20% each.
**Related Rules:** EXM-008, RES-001, GRD-007.

**Rule ID:** EXM-002
**Rule Name:** Per-Subject Maxima, Pass Marks & Components
**Description:** Each exam-subject defines max marks, pass marks, and component splits with their own maxima.
**Priority:** Critical
**Category:** Assessment integrity
**Preconditions:** Exam-subject definition.
**Business Rule:** Each subject in an exam has a total max, a pass mark, and (if componented) per-component maxima and pass rules (SUB-003). Marks are validated against these; component pass requirements may independently fail a subject.
**System Action:** Store maxima/pass/component rules; enforce on entry and result computation.
**Validation:** Component maxima sum to subject max; pass marks ≤ max.
**Failure Behavior:** Reject inconsistent maxima/pass configuration.
**Audit Requirement:** Captured in `EXAM_DEFINED`.
**Example Scenario:** Science exam: theory max 70 (pass 23), practical max 30 (pass 10); failing practical fails the subject.
**Related Rules:** SUB-003, EXM-004, RES-002.

**Rule ID:** EXM-003
**Rule Name:** Exam Eligibility & Admit Card Gating
**Description:** Eligibility (attendance, fees, registration) gates sitting the exam; admit cards are eligibility-bound.
**Priority:** High
**Category:** Eligibility / fairness
**Preconditions:** Eligibility rules configured.
**Business Rule:** A student must satisfy configured eligibility (e.g., minimum attendance ATT-009, fee clearance, exam registration) to be eligible; admit cards issue only to eligible students; overrides require authorization + reason and are audited.
**System Action:** Evaluate eligibility; issue admit cards; flag/override with audit.
**Validation:** Eligibility criteria evaluated against current data.
**Failure Behavior:** Block admit card for ineligible students unless overridden.
**Audit Requirement:** Log `ELIGIBILITY_EVALUATED`, `ADMIT_CARD_ISSUED`, eligibility overrides.
**Example Scenario:** A student below the attendance threshold cannot sit finals without an authorized override.
**Related Rules:** ATT-009, FEE clearance, Doc 30 (admit card).

**Rule ID:** EXM-004
**Rule Name:** Mark Validation Against Maxima
**Description:** Entered marks must be within defined bounds and of valid form.
**Priority:** Critical
**Category:** Data integrity
**Preconditions:** Mark entry.
**Business Rule:** A mark cannot exceed its component/subject max or be negative (unless negative marking is explicitly configured for MCQ exams); decimal precision follows configuration; special markers (`AB`, exempt) are distinct from numeric zero.
**System Action:** Validate each entry against bounds/precision; accept special markers as typed values.
**Validation:** 0 ≤ mark ≤ max (or per negative-marking config); precision valid.
**Failure Behavior:** Reject out-of-bound/invalid marks at entry.
**Audit Requirement:** Captured in mark-entry events.
**Example Scenario:** Entering 105 in a 100-max subject is rejected immediately.
**Related Rules:** EXM-002, EXM-005.

**Rule ID:** EXM-005
**Rule Name:** Ownership-Scoped Mark Entry
**Description:** Only the assigned subject teacher (or an authorized controller) may enter/edit a subject's marks.
**Priority:** Critical
**Category:** Authorization (ownership)
**Preconditions:** Mark entry/edit.
**Business Rule:** Mark entry for a subject in a section is restricted to the assigned subject teacher (SUB-004) within the entry window; controllers/admins may enter on behalf with audit. No teacher enters marks for subjects/sections they don't own.
**System Action:** Enforce ownership on entry; allow authorized on-behalf entry with attribution.
**Validation:** Marker owns the subject/section or holds controller permission.
**Failure Behavior:** Reject out-of-ownership entry.
**Audit Requirement:** Log `MARKS_ENTERED/UPDATED` with marker identity.
**Example Scenario:** Only the assigned Math teacher of 9-A enters its Math marks; the controller can enter on their behalf, attributed.
**Related Rules:** SUB-004, AUTHZ-003, EXM-007.

**Rule ID:** EXM-006
**Rule Name:** Absence, Exemption & Malpractice Are Distinct Outcomes
**Description:** Exam absence, subject exemption, and malpractice/disqualification are recorded as distinct, non-numeric outcomes.
**Priority:** High
**Category:** Assessment integrity
**Preconditions:** A student does not produce a normal numeric mark.
**Business Rule:** `AB` (absent in exam) ≠ 0 and is handled per policy (incomplete vs fail); exempted subjects (`EX`) are excluded from aggregates per policy; malpractice/disqualification is flagged, may void the subject/exam result, and follows a governed process. These never silently become zeros.
**System Action:** Record the distinct marker; apply policy in result computation; route malpractice through governance.
**Validation:** Marker valid for the context; malpractice authorized.
**Failure Behavior:** Block treating special markers as numeric without policy.
**Audit Requirement:** Log `EXAM_ABSENCE`, `EXEMPTION_APPLIED`, `MALPRACTICE_FLAGGED`.
**Example Scenario:** An absent student is `AB` (incomplete), not a scoring zero, preserving the chance of a makeup exam.
**Related Rules:** RES-002, RES-007 (makeup/supplementary).

**Rule ID:** EXM-007
**Rule Name:** Submit-and-Lock Immutability
**Description:** Submitted marks are locked and become immutable; changes after lock require governed correction.
**Priority:** Critical
**Category:** Integrity (tamper-evidence)
**Preconditions:** Marks submitted/locked.
**Business Rule:** On lock, marks are read-only and feed results. Post-lock corrections require elevated permission, a recorded reason, and (if a session is closed) a governed reopen; corrections re-trigger affected result recomputation and notifications. Step-up MFA is required for lock and post-lock correction.
**System Action:** Lock on submission/controller action; gate corrections; cascade recomputation.
**Validation:** Corrector authorized; reason provided; step-up satisfied.
**Failure Behavior:** Reject ungoverned post-lock edits.
**Audit Requirement:** Log `MARKS_LOCKED`, `MARKS_CORRECTED` (before/after, reason, actor).
**Example Scenario:** A discovered entry error after lock is fixed only via an authorized, audited correction that recomputes the result.
**Related Rules:** EXM-005, RES-006, SESS-005.

**Rule ID:** EXM-008
**Rule Name:** Bounded, Transparent Moderation & Grace
**Description:** Moderation/grace marks are configurable, bounded, applied transparently, and audited — never silent.
**Priority:** High
**Category:** Fairness / integrity
**Preconditions:** A moderation/grace policy is applied.
**Business Rule:** Where configured, moderation (uniform adjustment) or grace (to reach a pass/boundary) is bounded by policy limits, applied as a distinct, visible adjustment (original mark preserved), authorized, and audited. SoD may require a different actor than the entering teacher.
**System Action:** Apply bounded adjustment as a separate layer over the raw mark; preserve the original.
**Validation:** Adjustment within policy bounds; authorized; SoD respected.
**Failure Behavior:** Reject out-of-bounds or unauthorized adjustments.
**Audit Requirement:** Log `MODERATION_APPLIED`/`GRACE_APPLIED` with amount, basis, actor.
**Example Scenario:** A 2-mark grace to lift a student to the pass boundary is recorded distinctly from the raw mark, within the policy cap.
**Related Rules:** AUTHZ-009 (SoD), RES-002.

**Rule ID:** EXM-009
**Rule Name:** Mark-Entry Window & Completeness
**Description:** Marks are entered within a window; results require completeness (all eligible students marked).
**Priority:** High
**Category:** Process integrity
**Preconditions:** Mark entry.
**Business Rule:** Entry occurs within a configurable window; result computation requires every eligible student to have a mark or a valid special marker — no silent gaps. Incomplete subjects block result publication for affected students.
**System Action:** Enforce the window; block result computation on incomplete data.
**Validation:** All eligible students have a mark/marker; within window or authorized.
**Failure Behavior:** Block results on incompleteness; flag missing entries.
**Audit Requirement:** Log entry-window events and completeness checks.
**Example Scenario:** A subject missing two students' marks blocks those students' results until entered.
**Related Rules:** EXM-006, RES-001, RES-005.

## 5. Validation Rules
- Exam-subject maxima/pass/components consistent; weightages valid.
- Marks within bounds and precision; special markers distinct from numeric.
- Entry restricted to subject owner / authorized controller, within window.
- Eligibility evaluated before admit cards; overrides authorized + audited.
- Moderation/grace within policy bounds, authorized, transparent.
- Completeness required before result computation; lock requires step-up MFA.

## 6. State Machine

**State Name:** DEFINED
**Description:** Exam configured (subjects, maxima, weightage) but not scheduled.
**Allowed Transitions:** → SCHEDULED; → CANCELLED.
**Forbidden Transitions:** mark entry before scheduling/conduct.
**System Actions:** Validate definition; await scheduling.

**State Name:** SCHEDULED
**Description:** Dates set; eligibility/admit cards processed.
**Allowed Transitions:** → IN_PROGRESS; → RESCHEDULED; → CANCELLED.
**Forbidden Transitions:** results before marks.
**System Actions:** Evaluate eligibility; issue admit cards.

**State Name:** IN_PROGRESS / CONDUCTED
**Description:** Exam being conducted; mark entry opens after conduct.
**Allowed Transitions:** → MARK_ENTRY.
**Forbidden Transitions:** definition edits that change maxima after conduct.
**System Actions:** Open mark-entry window.

**State Name:** MARK_ENTRY
**Description:** Teachers entering/submitting marks.
**Allowed Transitions:** → UNDER_VERIFICATION; back to MARK_ENTRY on return.
**Forbidden Transitions:** results before lock.
**System Actions:** Validate entries; track completeness.

**State Name:** UNDER_VERIFICATION
**Description:** Marks being verified/moderated.
**Allowed Transitions:** → LOCKED; → MARK_ENTRY (corrections requested).
**Forbidden Transitions:** publish before lock.
**System Actions:** Apply bounded moderation; SoD checks.

**State Name:** LOCKED
**Description:** Marks immutable; results may proceed.
**Allowed Transitions:** → CORRECTION (governed, elevated) → re-LOCKED; → ARCHIVED.
**Forbidden Transitions:** routine edits.
**System Actions:** Freeze marks; feed result processing; require step-up for changes.

**State Name:** CANCELLED / ARCHIVED
**Description:** Exam cancelled, or archived with the session.
**Allowed Transitions:** CANCELLED → ARCHIVED.
**Forbidden Transitions:** mark entry.
**System Actions:** Preserve any data read-only.

## 7. Status Definitions
Exam: `DEFINED` · `SCHEDULED` · `RESCHEDULED` · `IN_PROGRESS` · `MARK_ENTRY` · `UNDER_VERIFICATION` · `LOCKED` · `CANCELLED` · `ARCHIVED`. Mark markers: numeric · `AB` (absent) · `EX` (exempt) · `MAL` (malpractice). Mark record: `DRAFT` · `SUBMITTED` · `LOCKED` · `CORRECTED`.

## 8. Workflow Rules
- Mark verification/moderation is a configurable workflow step (verifier ≠ entering teacher under SoD, AUTHZ-009).
- Lock and post-lock correction require step-up MFA (EXM-007).
- Eligibility overrides and malpractice handling are governed, audited actions.
- Re-exam/supplementary scheduling follows a governed flow (links RES-007).

## 9. Permission Rules
- `exam.manage` — define/schedule/cancel exams, set eligibility/moderation policy.
- `exam.marks.enter` — enter/submit marks (subject owner or controller).
- `exam.marks.verify` — verify/moderate marks.
- `exam.marks.lock` — lock marks (controller; step-up).
- `exam.marks.correct` — post-lock correction (elevated; step-up).
- `exam.eligibility.override` — override eligibility (elevated).

## 10. Notification Rules
- `ADMIT_CARD_ISSUED` → notify eligible students/guardians.
- Eligibility shortfall → notify guardian/student (attendance/fee reason) before the exam.
- `MARKS_LOCKED` → notify the exam controller (results ready to process).
- `MALPRACTICE_FLAGGED` → notify governance/admin (not the student directly until process completes).
- Result publication notifications are handled in Result Processing (Doc 15).

## 11. Audit Requirements
Mandatory: `EXAM_DEFINED/EDITED` (config version), `EXAM_SCHEDULED/RESCHEDULED/CANCELLED`, `ELIGIBILITY_EVALUATED`/overrides, `ADMIT_CARD_ISSUED`, `MARKS_ENTERED/UPDATED` (marker), `MODERATION/GRACE_APPLIED` (amount/basis/actor), `MARKS_LOCKED`, `MARKS_CORRECTED` (before/after/reason/step-up), `EXAM_ABSENCE/EXEMPTION/MALPRACTICE`. Marks are tamper-sensitive — every touch is attributable.

## 12. Data Retention Rules
- Marks and exam records retained long-term per academic-record retention (basis of transcripts).
- Raw marks plus any moderation layer retained distinctly (the original is never overwritten).
- Malpractice records retained per disciplinary policy with restricted access.
- Partitioned for scale; archived with the session; anonymized only per lawful erasure preserving aggregate integrity.

## 13. Edge Cases
- **Student admitted mid-term:** eligibility for already-conducted exams handled per policy (often `AB`/exempt for missed components).
- **Optional/best-of questions:** captured at the source; the recorded mark already reflects the exam's internal rules.
- **Negative marking (MCQ):** allowed only where explicitly configured; otherwise negatives are rejected (EXM-004).
- **Decimal/fractional marks:** precision per configuration; rounding deferred to result stage (RES) to avoid double-rounding.
- **Absent in one component:** component-level `AB` interacts with component pass rules (SUB-003) — may fail the subject.
- **Malpractice mid-exam:** flagged, may void the subject/exam, governed process; never a silent zero (EXM-006).
- **Mark entered then student withdraws:** historical mark retained; result handling per withdrawal policy.
- **Re-exam/supplementary:** a separate exam instance whose result merges per policy (RES-007), not an edit of the original.
- **Grace pushing past max:** capped at max; grace never exceeds the maximum mark.

## 14. Failure Scenarios
- **Mark > max / invalid:** rejected at entry (EXM-004).
- **Out-of-ownership entry:** rejected (EXM-005).
- **Incomplete marks at result time:** results blocked for affected students (EXM-009).
- **Post-lock edit without governance:** rejected (EXM-007).
- **Moderation beyond bounds:** rejected (EXM-008).
- **Concurrent entry by two markers:** serialized; attributed; no silent overwrite.

## 15. Exception Handling Rules
- Validation (bounds, ownership, window, completeness) precedes acceptance of any mark.
- Special outcomes (AB/EX/MAL) are explicit, never coerced to zero.
- Lock and corrections are step-up-gated and audited; closed sessions need governed reopen.
- Moderation is bounded, transparent, and SoD-respecting.

## 16. Compliance Considerations
- **Record integrity:** submit-lock immutability, distinct moderation layer, and complete audit defend against tampering and grade fraud — a top threat for minors' academic records.
- **Fairness:** transparent eligibility, bounded moderation, and distinct absence/exemption handling support dispute resolution and appeals.
- **Malpractice due process:** governed, restricted handling protects the student's rights while preserving integrity.
- **Transparency:** the exam/grading configuration in force is recorded for accountability.

## 17. Future Considerations
- Online/CBT exams with auto-scoring feeding the same mark pipeline.
- Question-bank and item-analysis integration.
- OMR/scanned mark-sheet ingestion with validation.
- Anonymized (roll-number-blind) marking to reduce bias.
