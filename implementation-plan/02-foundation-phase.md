# 02 — Foundation Phase

**Phase type:** Platform spine (pre-business-module) · **Release:** R1 · **Maps to:** Blueprint Part 3

The cross-cutting capabilities every business module will assume exists — built once as a thin walking skeleton, reused forever. Your best engineers belong here.

## Objectives
- Deliver the reliability, security, and UI primitives that all features depend on.
- Prove the **entire** stack works end-to-end (DB → domain → application → API → auth → UI → deploy) before any feature leans on it.
- Keep it **thin** — a working minimum, hardened later as features exercise it (this is the program's #1 schedule risk).

## Scope
**In:**
*Backend:* config loader + env validation; structured logging + correlation id (no PII); error handling + exception filter; request pipeline (guards/interceptors/pipes/filters); static validation tier (dynamic stubbed); **transactional outbox + in-process event bus**; **append-only audit pipeline**; Redis + BullMQ wiring; file service (object-storage adapter, signed URLs, scoped paths, scan hook); notification framework skeleton (decide/deliver split + in-app channel; email/SMS stubbed); event conventions (past-tense, idempotent handlers).
*Frontend:* tokens-driven design system; layout + navigation shell; RTK Query base API + auth slice (token-in-memory + refresh cookie); auth foundation UI + permission-gating component; table + form engine skeletons (dynamic renderer stubbed).
*Infra:* hardened local Compose; live staging environment; one backup+restore drill on staging.
**Out:** real email/SMS channels (R4); full dynamic form renderer (Config phase); business logic of any kind.

## Deliverables
- A deployed staging app: log in → see the app shell → one trivial capability writes a record, emits an audit event, and shows it in a table.
- Working outbox, audit, file service, queue, and the two UI engine skeletons.

## Dependencies
Project Setup (01) complete — rails proven on staging.

## Risks
- **Foundation overruns and starves features (top program risk).** *Mitigation:* build the walking skeleton thin; grow engines feature-driven; aggressive timebox.
- **Table/form engines over-built speculatively.** *Mitigation:* build only what the first real feature needs.
- **Notifications/email/SMS built too early.** *Mitigation:* in-app + framework skeleton only now.

## Acceptance Criteria
- Login → silent refresh → authenticated request → audited write → table read works end-to-end on staging.
- Killing the app mid-write still delivers the audit event on restart (at-least-once outbox proven).
- The design system renders from tokens (change a token → branding shifts, no code change).
- A staging backup is restored successfully in a drill.

## Exit Criteria
- Walking skeleton demoed on staging and signed off.
- Outbox/audit/file/queue primitives documented with usage examples in their context READMEs.
- Table and form engine skeletons ready for the Config phase to extend.
- Shared Definition of Done validated against the skeleton slice.
- **Core Platform Phase unblocked.**
