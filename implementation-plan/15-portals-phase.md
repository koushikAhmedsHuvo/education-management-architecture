# 15 — Portals Phase

**Phase type:** Feature (vertical slice) · **Release:** R4 · **Maps to:** Blueprint Part 16

The parent/student self-service portals — high client value, built last among features because they aggregate everything prior. **Highest access-control surface in the product.**

## Objectives
- Deliver scoped student and parent portals showing exactly their own data — results, attendance, fees, notices.
- Enforce ownership so a parent sees only their children and a student only themselves.

## Scope
**In:** portal routing & access (ownership predicates); student portal (results, attendance, fees, profile, notices); parent portal (per-child views, fee-payment entry point, notifications); portal dashboards. Mostly **read** APIs over existing data, scoped hard; UIs reuse existing components.
**Out:** new domain logic (portals surface existing data); native mobile (R5).

## Deliverables
- Parents and students log into scoped portals showing only their own data.

## Dependencies
Essentially all prior feature modules (surfaces their data) + guardian links from Student Lifecycle (07).

## Risks
- **Cross-child / cross-student access — a severe breach involving minors (highest access surface).** *Mitigation:* exhaustive ownership tests on every portal endpoint — the most scrutinized suite after Core Platform's isolation suite.

## Acceptance Criteria
- A parent provably cannot access a non-child's data (exhaustive ownership test).
- A student sees only their own records.
- Portals reuse existing components (no parallel UI stack).

## Exit Criteria
- Portal ownership suite green and signed off by the architect (treated as security-critical).
- Parent + student portals demoed end-to-end.
- **Release 4 (Enterprise) feature set advanced** — proceed toward Production Readiness for the enterprise release.
