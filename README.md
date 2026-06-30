# Enterprise Education ERP — Documentation Repository

> A configurable **Student Management / Education ERP** system designed to serve schools, colleges, universities, madrasas, coaching centers, and training institutes — driven by configuration, not per-client code.

---

## What this repository contains

This repo is the complete documentation baseline for the ERP system. No source code lives here — only the design, rules, and execution plan that a development team builds from.

| Folder | What it covers |
|---|---|
| [Architecture/](./Architecture/) | Full system architecture blueprint (Parts A–H), 87 decision records (ADRs), and a navigable index |
| [business-rules-repository/](./business-rules-repository/) | 30 functional modules, 248 business rules, governance process, and conflict register |
| [implementation-plan/](./implementation-plan/) | 22 phase-by-phase execution documents — from project setup to first paying client |
| [use-cases/](./use-cases/) | Per-module use case specifications |

---

## System overview

- **Architecture style:** Modular Monolith with a two-plane split (Control Plane + per-client Application Plane)
- **Tenancy:** One isolated deployment per client — separate database, storage, and config
- **Core thesis:** Configuration-driven; no hard-coded business logic
- **Tech stack:** NestJS · Next.js · TypeScript · PostgreSQL · Redis · BullMQ · Docker
- **Scale target:** 200+ deployments by year 5; up to 30,000+ students per client
- **Team / timeline:** 4–8 engineers; first paying client targeted ~6 months

---

## Where to start

- **New to the project?** Read [Architecture/README.md](./Architecture/README.md) then [Part A](./Architecture/parts/part-A.md) for the strategic shape.
- **Need business rules?** Go to [business-rules-repository/README.md](./business-rules-repository/README.md) and pick the relevant module.
- **Planning implementation?** Follow [implementation-plan/README.md](./implementation-plan/README.md) phase by phase.
- **Want everything in one file?** See [Architecture/FINAL_ARCHITECTURE_BLUEPRINT.md](./Architecture/FINAL_ARCHITECTURE_BLUEPRINT.md).

---

## Status

| Artifact | Status |
|---|---|
| Architecture (Parts A–H) | Approved and merged |
| Decision Records (D1–D87) | Complete, gap-free |
| Business Rules (248 rules / 30 modules) | Version 1.0.0 — complete |
| Implementation Plan (22 phases) | Complete |
| Open items | 1 institutional policy decision pending (C-02 in Conflict Register) |
