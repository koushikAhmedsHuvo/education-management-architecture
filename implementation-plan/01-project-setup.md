# 01 — Project Setup

**Phase type:** Initialization (pre-feature) · **Release:** R1 · **Maps to:** Blueprint Parts 1–2

Everything that must exist before a single feature is written — the rails the team runs on. Includes the chosen delivery strategy that governs every later phase.

## Objectives
- Stand up the repository, toolchain, CI/CD, and environments so feature work can begin without retrofitting.
- Lock the delivery method: **foundation-first walking skeleton, then contract-first vertical slices** (micro-order: schema → domain → application → API → UI → DoD).
- Make architecture and coding rules enforced by tooling from the first commit, not by memory.

## Scope
**In:** monorepo (pnpm workspaces: `apps/api`, `apps/web`, `packages/shared`, `packages/config`); trunk-based branch strategy + feature flags; ESLint/Prettier/strict-TS; **architecture tests** (dependency-direction, no cross-module internals, no domain framework imports); GitHub Actions CI (lint → typecheck → test → SAST/dep/secret scan → build image); multi-stage Docker image (web+worker entrypoints); local Compose stack (api, web, Postgres, Redis, MinIO) + seed; dev→staging→prod environment strategy; documentation/ADR standards; a manual-trigger staging deploy stub.
**Out:** any business feature; full fleet-rollout orchestration (deferred, D76); the Control Plane product (built incrementally later).

## Deliverables
- A repo where `pnpm install && docker compose up` runs the full stack locally.
- Green CI on an empty walking-skeleton commit, with all gates active.
- One Docker image building and deploying to a single staging environment.
- ADR-0001…0087 seeded from the architecture blueprint; per-context README template.

## Dependencies
None — this is the root of the program.

## Risks
- **Over-engineering init (gold-plating CI/monorepo).** *Mitigation:* timebox to ~1–1.5 sprints; defer fleet orchestration.
- **Weak architecture-test enforcement lets boundaries rot later.** *Mitigation:* dependency-direction and no-cross-internals tests written here and gating every PR thereafter.

## Acceptance Criteria
- A new engineer goes clone → running stack in under 30 minutes via the README.
- A PR violating a module-dependency rule **fails CI**.
- A PR containing a secret **fails CI**.
- The staging deployment serves a health endpoint from the built image.

## Exit Criteria
- All acceptance criteria signed off by the architect and DevOps owner.
- Branch protection, required reviews, and CI gates enforced on `main`.
- Every engineer has run the stack locally and merged one trivial PR through the full pipeline.
- Delivery method and Definition of Done circulated and agreed by the team.
- **Foundation Phase is unblocked** (rails proven end-to-end on staging).
