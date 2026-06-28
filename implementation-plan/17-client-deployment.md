# 17 — Client Deployment

**Phase type:** Operations (repeatable process) · **Release:** R2 onward · **Maps to:** Blueprint Part 18

The repeatable, increasingly-automated process to stand up a new isolated client deployment — the Control Plane's core job.

## Objectives
- Provision an isolated, branded, healthy client deployment from a signed contract via a documented, automated path.
- Guarantee per-client isolation, backups, monitoring, and a passing smoke test before go-live.

## Scope
**In:** client onboarding (tier selection, config collection); new-deployment process (provision tier infra → deploy current image → run migrations → seed template/roles/terminology/branding → provision secrets/keys → health-check); database provisioning (dedicated DB, full migrations, fleet-inventory registration); domain + SSL setup (automated TLS, HSTS); go-live checklist.
**Out:** the full canary→staged→full fleet-rollout of *releases* (that's the CD process, D76) — this phase is about standing up a *new client*.

## Deliverables
- A documented, increasingly-automated path from signed contract to a live, isolated, branded client deployment.
- A completed go-live checklist per client.

## Dependencies
Production Readiness (16) passed. Control Plane provisioning + migration capability (built incrementally from the CD stub in phase 01).

## Risks
- **Manual provisioning errors at scale.** *Mitigation:* automate provisioning early; runbook-as-code.
- **SSL/domain misconfiguration.** *Mitigation:* automated verification step in the checklist.

## Acceptance Criteria
- A new deployment is provisioned, seeded, and taken live following the checklist, with an isolated DB.
- A backup is configured and a restore drill passes for **this** deployment.
- A smoke test of the critical path (login → admit → enroll → mark → result → fee) passes.
- DPA signed; controller/processor roles confirmed.

## Exit Criteria
- First client (end of R2) live and operating on its own isolated deployment.
- The onboarding runbook validated and repeatable for subsequent clients.
- Monitoring/alerting active; non-sensitive signals flowing to the Control Plane.
