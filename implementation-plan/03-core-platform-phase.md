# 03 — Core Platform Phase

**Phase type:** Platform · **Release:** R1 · **Maps to:** Blueprint Part 4

Identity, access, and the organizational backbone (institute / campus / session) every business module scopes against. Second-highest risk concentration after the engines.

## Objectives
- Provide authentication, RBAC, and the multi-institute scope model that all features enforce against.
- Make cross-institute data access **structurally impossible** before any feature stores data.
- Establish scope columns (`institute_id`, `campus_id`, `session_id`) as the convention for all later tables.

## Scope
**In:** authentication (asymmetric short-lived JWT + rotating refresh with reuse detection + sessions/devices; MFA scaffolded); user management + invitation/activation; RBAC (permission registry, data-defined roles, version-cached effective-permission resolution); institute management; campus management; academic session; the mandatory scope filter + ownership predicates wired into the data layer.
**Out:** MFA enforcement (R4); SSO adapters (R5); any business domain.

## Deliverables
- A working multi-institute identity platform: create institutes/campuses/sessions/users/roles; users log in scoped to an institute.
- The scope-isolation + ownership **test suite** (the most important security code in the system), gating all later merges.
- Build/DB/API/UI orders executed per blueprint Part 4.

## Dependencies
Foundation (02) — request pipeline, audit, permission-gating component.

## Risks
- **Scope/ownership gap leaks data across institutes (minors' data).** *Mitigation:* isolation suite written **here, before** dependent features; gates every merge.
- **Hard-coded roles break the config thesis.** *Mitigation:* roles/permissions are seed data; arch-test forbids role-name literals in logic.
- **Refresh-rotation race conditions.** *Mitigation:* family/reuse-detection with grace window; concurrent-refresh tested.

## Acceptance Criteria
- A user scoped to Institute A provably receives 403/empty for Institute B across every endpoint pattern (automated).
- A custom role grants exactly its registered permissions — no escalation.
- Refresh-token reuse revokes the whole family (tested).
- All identity actions appear in the audit trail.

## Exit Criteria
- Isolation + ownership suite green and wired as a required CI gate for all future phases.
- Auth, RBAC, and org-backbone APIs published as OpenAPI; admin UIs demoed.
- Org setup wizard groundwork in place (completed alongside Config phase).
- **Configuration Engine Phase unblocked.**
