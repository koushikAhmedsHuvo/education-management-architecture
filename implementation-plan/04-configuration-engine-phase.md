# 04 — Configuration Engine Phase

**Phase type:** Platform Core (highest leverage) · **Release:** R1 · **Maps to:** Blueprint Part 5

The engine that makes the product configuration-driven. Every later module reads its behavior from here. Build early, with the strongest engineer, and over-test it.

## Objectives
- Deliver definition registry, scoped value store, resolution, caching, versioning, and rollback.
- Enable adding custom fields and forms with **zero code change**, validated identically on frontend and backend.
- Guarantee historical fidelity: records stamp the config version they used; rollback never rewrites history.

## Scope
**In:** definition registry (relational); scoped value store (JSONB, EAV forbidden); most-specific-wins resolution engine; Redis + in-process cache with event-driven invalidation; dynamic fields (validated JSONB on owning entity, GIN-indexed, promote-to-column hatch); dynamic forms (one definition source feeding the FE renderer and BE validator); change-set versioning to immutable snapshots; validated rollback; Configuration Center UI.
**Out:** fee/grading/workflow *definitions* (those are consumers, built in their own phases) — this phase delivers the generic engine they use.

## Deliverables
- A working Configuration Center: define a custom field → it renders and validates everywhere with no code change.
- Resolution engine with near-100% unit coverage; versioning + rollback operational.
- The dynamic form **renderer**, reused by every later feature.

## Dependencies
Core Platform (03) — scope columns, RBAC. Foundation (02) — events, cache, audit.

## Risks
- **Resolution/rollback bugs radiate into every module (concentration risk).** *Mitigation:* near-100% coverage with edge-case tables; rollback tested against "data exists under newer version."
- **FE renderer and BE validator drift.** *Mitigation:* one definition source; contract test asserts identical validation outcomes.
- **Config-driven ceiling — a need config can't express.** *Mitigation:* define the **pressure-release process** (promote recurring custom needs to first-class config) now.

## Acceptance Criteria
- Adding a custom field via UI makes it render+validate on FE and BE with zero code change.
- Most-specific-wins resolution proven across all scope combinations.
- A published change-set is rollback-able; a record created under v2 still references v2 after rollback to v1.
- Resolved-value reads are cached and invalidated within one event cycle.

## Exit Criteria
- Engine acceptance signed off; coverage report on resolution/rollback reviewed by the architect.
- Dynamic form renderer documented and demonstrated as reusable.
- Pressure-release process documented as an ADR.
- **Workflow Engine and all feature phases unblocked** (they consume this engine).
