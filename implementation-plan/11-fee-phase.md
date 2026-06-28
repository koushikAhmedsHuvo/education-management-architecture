# 11 — Fee Phase

**Phase type:** Feature (vertical slice) · **Release:** R2 (last module → First Client) · **Maps to:** Blueprint Part 12

The financial backbone — invoices, payments, ledgers. Immutability and auditability are paramount. Completing this module enables the **first paying client**.

## Objectives
- Configure fees, generate invoices in bulk (peak event), record payments, process waivers via approval, and surface dues.
- Guarantee financial immutability: invoices stamp the config version; historical invoices never change.

## Scope
**In:** fee structure configuration (types, amounts per node, effective-dating, schedules — config-driven); invoice generation (bulk, version-stamped, immutable); payment recording (manual); discounts/waivers (via **workflow** approval); fee ledgers & dues.
**Out:** online payment-gateway integration (bKash/Nagad) — R4; advanced installment automation — R4.

## Deliverables
- A school configures fees, generates invoices in bulk, records payments, processes waivers via approval, and sees dues.
- Build/DB/API/UI executed per blueprint Part 12.

## Dependencies
Academic (06) — structure nodes. Student Lifecycle (07) — students. Configuration (04) — fee config + versioning. Workflow (05) — waivers.

## Risks
- **Financial errors and disputes.** *Mitigation:* immutable version-stamped invoices, full audit, reconciliation views.
- **Bulk invoice generation spikes writes.** *Mitigation:* async batching (D69).

## Acceptance Criteria
- A published invoice is immutable and references the fee-config version used.
- A mid-session fee change does **not** alter historical invoices (effective-dating proven).
- Waivers require workflow approval and are audited.
- Ledgers reconcile.

## Exit Criteria
- Fee config → bulk invoice → payment → waiver → dues demoed end-to-end.
- **Release 2 (Core ERP for school) feature-complete** — proceed to Production Readiness, then First Client onboarding.
