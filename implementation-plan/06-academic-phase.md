# 06 — Academic Phase

**Phase type:** Feature (vertical slice) · **Release:** R2 · **Maps to:** Blueprint Part 7

The academic structure every student, attendance record, exam, and fee attaches to. Built before student lifecycle because students enroll *into* this structure.

## Objectives
- Implement the timeless **structure definition** separately from per-session **instances** (prevents year-rollover duplication).
- Let a school define classes, sections, subjects, and curriculum as configuration, with the enrollment leaf marked.

## Scope
**In:** structure definition vs session instance (D38); classes/grades (terminology-driven labels); sections; subjects; curriculum (subject-to-structure mapping); enrollment-leaf marking; academic structure builder UI.
**Out:** timetable/routine (R3); cross-type structure templates beyond school (R3).

## Deliverables
- A school defines its academic structure for a session — classes, sections, subjects, curriculum — all configurable.
- Build/DB/API/UI executed per blueprint Part 7.

## Dependencies
Core Platform (03) — scope, session. Configuration Engine (04) — structures configurable, terminology relabeling.

## Risks
- **Structure/instance conflation re-creates the rollover-duplication bug.** *Mitigation:* enforce separation in the model; test session rollover.
- **Over-rigid hierarchy one type can't express.** *Mitigation:* drive levels from definitions.

## Acceptance Criteria
- Rolling over to a new session creates new instances **without duplicating the structure definition**.
- Historical sessions remain intact and queryable.
- A roll-up query (all students in a class) resolves via the tree.
- Shared Definition of Done met.

## Exit Criteria
- Academic structure demoed for a school across two sessions (rollover proven).
- Enrollment leaf documented as the attach point for Student Lifecycle.
- **Student Lifecycle Phase unblocked.**
