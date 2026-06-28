# 21 — Team Allocation

**Document type:** Organization · **Maps to:** Blueprint Part 22

The team structure that executes this plan, how it ramps, and the honest size/timeline/scope trade.

## Objectives
- Define roles, counts, and responsibilities at full ramp and at a small start.
- Place dedicated owners on the two concentration-risk engines and the access-control test suites.

## Scope
**In:** ideal full-ramp structure (~8–9); start-small ramp (4–5); responsibilities; the size/timeline/scope trade.
**Out:** individual hiring plans; compensation.

## Deliverables (the structure)
| Role | Count | Responsibilities |
|---|---|---|
| Solution Architect / Principal Engineer | 1 | Architecture integrity; owns config & workflow engine design; reviews security PRs; ADRs + per-phase review; must pair, not silo (bus-factor) |
| Backend Engineers | 3 | Domain modules, engines, API contracts. One anchors the **config engine**, one the **workflow engine** |
| Frontend Engineers | 2 | Design system, table/form engines, portals, feature slices against published contracts |
| QA Engineer | 1 | Test pyramid; especially **authz/scope/ownership suites** + critical-path e2e; load-test scenarios |
| DevOps Engineer | 1 | CI/CD, Docker, Nginx, Control Plane provisioning/migration/rollout, observability, backups/DR |
| Engineering/Delivery Manager | 1 | Sprint cadence, dependency unblocking, stakeholder comms, **scope discipline** (the key non-code job) |

**Start-small ramp (4–5):** Architect + 2 backend + 1 frontend + 1 DevOps(shared QA). Add 2nd frontend, dedicated QA, 3rd backend as R2 features fan out.

## Dependencies
Funding and hiring aligned to the release plan's ramp.

## Risks
- **At 4–5, Release 1 stretches and first-client slips.** *Mitigation:* flex exactly one of scope/timeline/people — don't pretend all three hold.
- **Engine knowledge siloed in one person.** *Mitigation:* pairing + ADRs on config/workflow.

## Acceptance Criteria
- The two engines each have a named senior owner.
- The access-control test suites have a named QA owner.
- Scope discipline has a named owner (the delivery manager).

## Exit Criteria
- Team staffed to at least the start-small ramp before Foundation begins; full ramp reached before R2 features fan out.
