# 18 — Release Plan

**Document type:** Planning · **Maps to:** Blueprint Part 19

Five releases mapped to the architecture roadmap. First paying client lands at the **end of Release 2**. Effort is in team-weeks at ~8 engineers — planning ranges, not commitments.

## Objectives
- Sequence delivery into client-shippable releases, each with clear business and client value.
- Make the first revenue milestone (R2) explicit and protected from scope creep.

## Scope
**In:** R1 Foundation & Platform Core; R2 Core ERP (school) → first client; R3 Advanced ERP + more institution types; R4 Enterprise features; R5 Platform expansion.
**Out:** detailed sprint decomposition for R3–R5 (done at each release start — see doc 19).

## Deliverables
| Release | Features | Business value | Client value | Effort |
|---|---|---|---|---|
| **R1** | Init, foundation spine, auth/RBAC, org backbone, **Config + Workflow engines**, setup wizard | Platform exists; clients provisionable | Admin sets up org, structure, roles, branding | ~16–22 wk |
| **R2** | Academic, Student Lifecycle, Attendance, Exam, Result, basic Fees (school) → **First Client** | Sellable ERP; first revenue | School runs core academic+financial ops | ~16–20 wk |
| **R3** | HR/Staff, payroll, leave/waiver workflows, Reporting, more institution types, routine | Market widens beyond schools | Richer ops; multiple types | ~14–18 wk |
| **R4** | Notifications (email/SMS/push), Portals, payment gateway, certificates, MFA, compliance hardening | Enterprise-grade, multi-channel | Self-service portals, online payments, polish | ~14–18 wk |
| **R5** | Public API + webhooks, mobile, SSO, analytics/data-mart, LMS, library/transport/hostel, AI | Platform & ecosystem | Integrations, mobile, adjacent capabilities | Multi-quarter track |

## Dependencies
Each release depends on the prior (engines before consumers; one type before breadth).

## Risks
- **R1 scope pressure starves the platform.** *Mitigation:* protect R1; it compounds into everything.
- **R2 breadth creep delays first revenue.** *Mitigation:* one institution type, thin slice, hard scope.

## Acceptance Criteria
- Each release is independently demoable and client-shippable.
- R2 produces a live paying client.

## Exit Criteria
- Each release closed only when its phases' Exit Criteria are met and Production Readiness (for client-facing releases) is passed.
