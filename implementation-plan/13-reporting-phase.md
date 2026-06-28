# 13 — Reporting Phase

**Phase type:** Feature (vertical slice) · **Release:** R3 · **Maps to:** Blueprint Part 14

Turning operational data into reports without loading transactional tables. Establishes the projection framework reused by all later analytics.

## Objectives
- Build the event-fed read-model/projection framework (once, shared) and a config-driven report builder over curated datasets.
- Deliver async Excel/PDF export and scheduled reports.

## Scope
**In:** projection framework (event-fed read models); curated datasets (attendance, results, fees, enrollment); dynamic report builder over datasets (config-driven); export (Excel/PDF async); scheduled reports.
**Out:** cross-client analytics (impossible by design / R5 consent-bound); raw SQL access (forbidden, D57).

## Deliverables
- Users build, save, run, export, and schedule reports over curated datasets, served off projections/replica.
- The reusable projection framework.

## Dependencies
The reported modules (Attendance/Exam/Result/Fee/Student/HR). Foundation (02) — events, queue, files.

## Risks
- **Projection lag at peak makes reports stale when refreshed.** *Mitigation:* monitor projection lag; scale projection workers.
- **Over-flexible builder becomes a query/security hazard.** *Mitigation:* curated, permission-scoped datasets only.

## Acceptance Criteria
- Reports never query transactional tables directly.
- A large export runs async and respects the user's scope/permissions (no row they couldn't see on screen).
- Projection lag is observable.

## Exit Criteria
- Report builder + export + schedule demoed; projection framework documented as reusable.
- Projection-lag monitoring wired into observability.
- **Notification Phase unblocked** (report-ready events available).
