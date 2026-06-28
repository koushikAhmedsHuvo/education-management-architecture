# Enterprise Education ERP — Architecture Repository

> **Configurable Education ERP / Student Management System** — one product serving schools, colleges, universities, madrasas, coaching centers, and training institutes, driven by configuration rather than per-client code.

This repository is the single source of truth for the system's architecture. It consolidates the complete enterprise architecture blueprint (Parts A–H), the navigable index, and the full decision record (D1–D87), all cross-checked for consistency.

---

## What this is

A production-grade, enterprise architecture blueprint intended to guide a professional engineering team for 10+ years. It contains no source code — it is the design, the reasoning behind every significant decision, and an unsparing critical review. Every architectural decision is recorded in a consistent format (Recommendation → Why → Pros → Cons → Alternatives → Final Decision) and is traceable as a founding ADR.

## System at a glance

| Aspect | Decision |
|---|---|
| Architecture style | Two-plane **Modular Monolith** (Control Plane + per-client Application Plane) |
| Tenancy | **Isolated deployment per client** — separate database, storage, and configuration |
| Within a client | Multiple institutes, campuses, and academic sessions |
| Core thesis | **Configuration-driven**, no hard-coded business rules (Level-B configurability) |
| Backend | NestJS · TypeScript · TypeORM · PostgreSQL |
| Frontend | Next.js (App Router) · TypeScript · Tailwind · RTK Query |
| Platform | Redis · BullMQ · S3/MinIO · Docker · Nginx · GitHub Actions · pnpm |
| Auth | JWT (asymmetric, short-lived) + rotating refresh tokens + optional TOTP MFA, SSO-ready |
| Compliance | GDPR-aligned baseline, Bangladesh-first, heightened minor-data protection |
| Scale target | 200+ deployments by year 5; largest client 30,000+ students, 3,000 peak concurrent |
| Team / timeline | 4–8 engineers; first paying client targeted ~6 months |

## Repository structure

```
architecture-repo/
├── README.md                      # this file
├── ARCHITECTURE_INDEX.md          # full navigable index of all parts & sections
├── FINAL_ARCHITECTURE_BLUEPRINT.md# all eight parts merged into one document
├── decisions/
│   └── decision-records.md        # consolidated ADR log (D1–D87)
└── parts/
    ├── part-A.md   Strategy & Domain
    ├── part-B.md   Backend & Data
    ├── part-C.md   Frontend, Auth & Access
    ├── part-D.md   Platform Core (Identity, Access, Config, Workflow)
    ├── part-E.md   Cross-Cutting Services (Reporting, Notifications, Files, Integration)
    ├── part-F.md   Security
    ├── part-G.md   Performance, Scalability, Infra, DevOps & Ops
    └── part-H.md   Engineering Standards, Roadmap & Critical Review
```

## How to read it

- **New to the project?** Start with the [Architecture Index](./ARCHITECTURE_INDEX.md), then read [Part A](./parts/part-A.md) for the strategic shape.
- **Need a specific decision?** Go to the [Decision Records](./decisions/decision-records.md) and follow the link to the part with the full reasoning.
- **Want everything in one file?** See the [Final Architecture Blueprint](./FINAL_ARCHITECTURE_BLUEPRINT.md).
- **Evaluating whether to build?** Read **Part H, Sections 17–20** — the critical CTO review, scorecard, and go/no-go.

## The five load-bearing decisions

Everything else hangs off these (full reasoning in the linked parts):

1. **Two-plane split** ([D1](./decisions/decision-records.md), Part A) — fleet management (Control Plane) is separate from the per-client ERP (Application Plane).
2. **Isolated deployment per client** (Part A) — physical isolation gives security and compliance; the Control Plane absorbs the operational cost.
3. **Modular monolith, not microservices** ([D5](./decisions/decision-records.md), Part A) — the right balance of structure and simplicity for the team.
4. **Configuration as core upstream domain** ([D2](./decisions/decision-records.md), Part A/D) — the Configuration and Workflow engines are Platform Core; the product's differentiation lives there.
5. **Event-and-interface integration** ([D4](./decisions/decision-records.md), Part A/B) — modules integrate via published interfaces and domain events, never shared data access.

## The honest verdict

From the Part H critical review: **the architecture is strong; the risk is execution.** This is a coherent, decade-durable design whose dominant risk is the gap between its ambition and a 4–8 person team on a six-month timeline. The recommendation is a **Conditional GO** — build it, but with ruthless MVP scoping, the two platform engines sequenced first, deliberate small-client isolation economics, managed infrastructure to reduce ops load, and a realistic timeline for the full vision. Read [Part H §17–20](./parts/part-H.md) for the full assessment and scorecard.

## Status

- All eight parts: **approved and merged.**
- Decisions: **D1–D87, continuous and gap-free.**
- Consistency: cross-checked — tenancy model, configuration thesis, tiered dependency graph, and decision numbering are uniform across all parts.
- Diagrams: all Mermaid diagrams validated.

*This repository is the design baseline. Implementation follows the build order and roadmap in Part H.*
