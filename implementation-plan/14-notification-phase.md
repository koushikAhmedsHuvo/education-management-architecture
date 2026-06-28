# 14 — Notification Phase

**Phase type:** Feature (vertical slice) · **Release:** R4 · **Maps to:** Blueprint Part 15

Activating the multi-channel delivery the framework was scaffolded for. In-app exists from Foundation; this phase lights up email/SMS/push.

## Objectives
- Deliver per-channel templates (bn/en), real email and Bangladesh SMS, preferences, and event-to-notification rules.
- Enforce SMS cost controls (dedup, rate limits, opt-out).

## Scope
**In:** template management (per-channel, bn/en); email adapter (real provider); **SMS adapter (Bangladesh gateways, Bangla Unicode, dedup/cost controls)**; notification preferences (per user/category/channel); event→notification rules (result published → notify parents, fee due, admission); push adapter (stub until mobile).
**Out:** live push delivery (needs the R5 mobile app); marketing campaigns (out of product scope).

## Deliverables
- Result-publish, fee-due, and admission events notify the right people on preferred channels, with Bangla SMS and cost controls.

## Dependencies
Foundation (02) — decide/deliver split, queue. The triggering events (Result/Fee/Admission).

## Risks
- **SMS cost runaway / flooding.** *Mitigation:* dedup, rate limits, opt-out (D45).
- **Provider lock-in.** *Mitigation:* adapter port.

## Acceptance Criteria
- A result publish triggers deduplicated parent notifications respecting preferences.
- SMS sends Bangla Unicode within cost-control limits.
- Transactional/security notifications override preferences.

## Exit Criteria
- Email + SMS live and demoed with cost controls; preference center working.
- Push adapter seam documented for the R5 mobile app.
- **Portals Phase unblocked** (notification surfacing available to portals).
