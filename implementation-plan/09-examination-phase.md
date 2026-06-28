# 09 — Examination Phase

**Phase type:** Feature (vertical slice) · **Release:** R2 · **Maps to:** Blueprint Part 10

Defining exams and capturing marks — the integrity-critical precursor to results.

## Objectives
- Configure and schedule exams; capture marks scoped to each teacher's assignments.
- Begin record immutability: submit-and-lock makes marks tamper-evident.

## Scope
**In:** exam configuration (types, terms, weightages — config-driven); scheduling; mark-entry schema; mark entry (per subject, scoped); submission/lock.
**Out:** result calculation/grading (phase 10); question banks/online exams (R5).

## Deliverables
- Exams configured and scheduled; teachers enter and submit marks within scope.
- `marks` table partitioned; submit-lock immutability in place.

## Dependencies
Academic (06) — subjects/structure. Student Lifecycle (07) — students. Configuration (04) — exam config. HR assignment (12) provides ownership truth (or interim assignment until phase 12).

## Risks
- **Marks are tamper-sensitive.** *Mitigation:* submit-and-lock immutability, full audit, step-up MFA on later publish.
- **Mark-entry UX errors.** *Mitigation:* validation from config (max marks, etc.).

## Acceptance Criteria
- A teacher enters marks only for assigned subjects/sections.
- Submitted marks are immutable; every change is audited.
- Marks validate against configured maxima.
- Shared Definition of Done met.

## Exit Criteria
- Exam setup + mark entry + lock demoed; immutability and scope tests green.
- Mark schema documented for the Result phase.
- **Result Processing Phase unblocked.**
