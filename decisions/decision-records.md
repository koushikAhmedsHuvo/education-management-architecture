# Architecture Decision Records (ADR Log)

**Product:** Enterprise Education ERP — Configurable Student Management System  
**Purpose:** A single, consolidated index of every significant architectural decision (D1–D87) recorded across Blueprint Parts A–H. Each entry links to the part where its full Recommendation / Why / Pros / Cons / Alternatives / Final-Decision reasoning lives.

> These are the founding ADRs. Each is immutable once accepted; a superseding decision gets a new record referencing the old. The numbering is continuous and gap-free across all parts.

## Summary Table

| # | Decision | Part | Area |
|---|---|---|---|
| D1 | Adopt strategic DDD with explicit bounded contexts as the organizing principle. | [Part A](../parts/part-A.md) | Strategy & Domain |
| D2 | Treat Configuration and Workflow as Platform Core, not as shared utilities. | [Part A](../parts/part-A.md) | Strategy & Domain |
| D3 | The Configuration context publishes effective values; consumers never read configuration internals. | [Part A](../parts/part-A.md) | Strategy & Domain |
| D4 | Integrate contexts via published interfaces and domain events only; never via shared data access. | [Part A](../parts/part-A.md) | Strategy & Domain |
| D5 | Modular Monolith per plane, not microservices. | [Part A](../parts/part-A.md) | Strategy & Domain |
| D6 | One Application Plane artifact for all clients; differentiation by configuration, never by per-client code or branches. | [Part A](../parts/part-A.md) | Strategy & Domain |
| D7 | Enforce an acyclic, tiered module dependency graph with platform contexts strictly below domain contexts. | [Part A](../parts/part-A.md) | Strategy & Domain |
| D8 | Persistence model and domain model are separated for Core contexts; pragmatically merged for Generic/Supporting contexts. | [Part B](../parts/part-B.md) | Backend & Data |
| D9 | Four named roots (`modules`, `platform`, `shared`, `common`) with distinct change-control. | [Part B](../parts/part-B.md) | Backend & Data |
| D10 | Separate, explicit request and response DTOs; never expose persistence entities across the API. | [Part B](../parts/part-B.md) | Backend & Data |
| D11 | Repository ports in the application layer, TypeORM implementations in infrastructure, returning domain models for Core contexts. | [Part B](../parts/part-B.md) | Backend & Data |
| D12 | Two-tier validation: static DTO validation for fixed contracts, definition-driven dynamic validation for configurable fields and forms. | [Part B](../parts/part-B.md) | Backend & Data |
| D13 | In-process domain events with a transactional outbox for reliability; a real broker deferred. | [Part B](../parts/part-B.md) | Backend & Data |
| D14 | BullMQ on Redis for all asynchronous and deferred work. | [Part B](../parts/part-B.md) | Backend & Data |
| D15 | One database per client; institute/campus/session as scoping columns; UUIDv7 keys; PgBouncer; read replica at scale. | [Part B](../parts/part-B.md) | Backend & Data |
| D16 | Range-partition the high-volume time-series tables; keep the rest unpartitioned. | [Part B](../parts/part-B.md) | Backend & Data |
| D17 | Event-sourced, append-only audit trail fed by the outbox, separate from configuration version history. | [Part B](../parts/part-B.md) | Backend & Data |
| D18 | Explicit, versioned TypeORM migrations using expand-contract, rolled out fleet-wide by the Control Plane in health-gated waves; synchronize is never used in any environment. | [Part B](../parts/part-B.md) | Backend & Data |
| D19 | One unified Next.js app with portal-segmented routing, not separate apps per portal. | [Part C](../parts/part-C.md) | Frontend, Auth & Access |
| D20 | Public pages use Server Components; the authenticated portal is client-rendered with RTK Query, using App Router for layout, routing, and code-splitting. | [Part C](../parts/part-C.md) | Frontend, Auth & Access |
| D21 | Feature-sliced architecture: features own their components, hooks, API slices, and types; routing only composes them. | [Part C](../parts/part-C.md) | Frontend, Auth & Access |
| D22 | Centralized, declarative permission gating mirroring backend permission strings; client gating is UX-only. | [Part C](../parts/part-C.md) | Frontend, Auth & Access |
| D23 | RTK Query as the single server-state layer, organized as one base API with per-feature endpoint injection and tag-based invalidation. | [Part C](../parts/part-C.md) | Frontend, Auth & Access |
| D24 | Access token in memory, refresh token in an httpOnly secure cookie, with silent refresh; never store tokens in localStorage. | [Part C](../parts/part-C.md) | Frontend, Auth & Access |
| D25 | A single configurable, headless-driven table engine; feature tables are configuration, not bespoke code. | [Part C](../parts/part-C.md) | Frontend, Auth & Access |
| D26 | A unified form engine with a definition-driven dynamic renderer mirroring the backend's two-tier validation. | [Part C](../parts/part-C.md) | Frontend, Auth & Access |
| D27 | Runtime white-labeling driven by per-client bootstrap configuration; no rebuild for branding or terminology. | [Part C](../parts/part-C.md) | Frontend, Auth & Access |
| D28 | Two distinct layers: i18n for UI language (English/Bangla) and terminology mapping for per-client domain relabeling, resolved together by one label resolver. | [Part C](../parts/part-C.md) | Frontend, Auth & Access |
| D29 | Responsive-web-first now; API and design system shaped for a future React Native app; no native build in early phases. | [Part C](../parts/part-C.md) | Frontend, Auth & Access |
| D30 | Short-lived access tokens signed with rotating asymmetric keys; tokens carry identity, active scope, roles, and a permissions-version, not the full permission set. | [Part D](../parts/part-D.md) | Platform Core (Identity, Access, Config, Workflow) |
| D31 | Rotating refresh tokens with token-family reuse detection; stored hashed, never as plaintext. | [Part D](../parts/part-D.md) | Platform Core (Identity, Access, Config, Workflow) |
| D32 | TOTP-based MFA with recovery codes, enabled per client and optionally step-up for sensitive actions. | [Part D](../parts/part-D.md) | Platform Core (Identity, Access, Config, Workflow) |
| D33 | Abstract authentication behind an identity-provider port with adapters; ship local credentials now, design the seam for OIDC/SAML. | [Part D](../parts/part-D.md) | Platform Core (Identity, Access, Config, Workflow) |
| D34 | Permissions and roles are data, not code; effective permissions are resolved and cached, invalidated by a version stamp. | [Part D](../parts/part-D.md) | Platform Core (Identity, Access, Config, Workflow) |
| D35 | Three-layer data access: RBAC permission, mandatory scope filter, and ownership/relationship rules enforced in the data layer. | [Part D](../parts/part-D.md) | Platform Core (Identity, Access, Config, Workflow) |
| D36 | Change-set-based configuration versioning: every publish creates an immutable version snapshot with an active pointer. | [Part D](../parts/part-D.md) | Platform Core (Identity, Access, Config, Workflow) |
| D37 | Custom field values stored as validated JSONB on the owning entity; frequently-queried fields promoted to real columns. | [Part D](../parts/part-D.md) | Platform Core (Identity, Access, Config, Workflow) |
| D38 | Separate the timeless structure definition from the per-session instance; the hierarchy is data with a marked enrollment leaf. | [Part D](../parts/part-D.md) | Platform Core (Identity, Access, Config, Workflow) |
| D39 | Declarative finite state machine defined as data; instances are pinned to the definition version they started under. | [Part D](../parts/part-D.md) | Platform Core (Identity, Access, Config, Workflow) |
| D40 | Every workflow event is recorded immutably and feeds the central audit trail, with full lineage of decisions, escalations, delegations, and timeouts. | [Part D](../parts/part-D.md) | Platform Core (Identity, Access, Config, Workflow) |
| D41 | Reporting reads from dedicated read models (a reporting projection), never from transactional tables directly; heavy generation runs on the queue; the largest client uses the read replica. | [Part E](../parts/part-E.md) | Cross-Cutting Services |
| D42 | Report definitions are configuration: datasets, filters, columns, and grouping defined as data and resolved through the configuration engine. | [Part E](../parts/part-E.md) | Cross-Cutting Services |
| D43 | Two-tier PDF generation: HTML-template rendering via a headless browser for rich documents, a lightweight generator for simple ones; all heavy generation async to object storage. | [Part E](../parts/part-E.md) | Cross-Cutting Services |
| D44 | Two-layer notifications: the Communication context decides what to send; a provider-agnostic Notification Delivery layer with per-channel adapters sends it, all queue-driven and event-fed. | [Part E](../parts/part-E.md) | Cross-Cutting Services |
| D45 | SMS via a provider-agnostic adapter with Bangladesh gateways, Bangla Unicode templates, and strict cost controls. | [Part E](../parts/part-E.md) | Cross-Cutting Services |
| D46 | Private object storage (S3/MinIO) behind an adapter; access only via short-lived signed URLs; scoped paths; server-side encryption; virus scanning on upload. | [Part E](../parts/part-E.md) | Cross-Cutting Services |
| D47 | Configurable, retention-class-driven file lifecycle with tiering, archival, and compliant erasure. | [Part E](../parts/part-E.md) | Cross-Cutting Services |
| D48 | Versioned REST public API governed by the same RBAC/scoping model, authenticated by scoped API credentials, rate-limited, and documented via OpenAPI. | [Part E](../parts/part-E.md) | Cross-Cutting Services |
| D49 | Per-deployment gateway concerns split between Nginx (edge) and the application; no separate API-gateway product at current scale. | [Part E](../parts/part-E.md) | Cross-Cutting Services |
| D50 | Outbound webhooks driven by the outbox, signed, retried with backoff, and subscribable per event type. | [Part E](../parts/part-E.md) | Cross-Cutting Services |
| D51 | Build the marketplace's foundations now (public API, webhooks, entitlements); defer the marketplace product itself. | [Part E](../parts/part-E.md) | Cross-Cutting Services |
| D52 | Internal modular extensibility through trusted extension points and configuration; explicitly no sandboxed third-party code execution in early phases. | [Part E](../parts/part-E.md) | Cross-Cutting Services |
| D53 | Defense-in-depth with deployment isolation as the primary control and scoped, least-privilege access as the secondary. | [Part F](../parts/part-F.md) | Security |
| D54 | STRIDE-based threat modeling plus an education-specific threat catalog, prioritized by impact on minors' data and on academic/financial integrity. | [Part F](../parts/part-F.md) | Security |
| D55 | Output-encoding by default, a strict Content Security Policy, and treating all dynamic and rich content (custom fields, notices, PDF templates) as untrusted. | [Part F](../parts/part-F.md) | Security |
| D56 | Bearer-token API calls are CSRF-immune; the cookie-based refresh path is protected by SameSite plus an anti-CSRF token. | [Part F](../parts/part-F.md) | Security |
| D57 | Parameterized access only, through TypeORM and the query builder; no string-concatenated SQL anywhere; reporting uses curated datasets, never raw user SQL. | [Part F](../parts/part-F.md) | Security |
| D58 | Strict egress controls for every server-side outbound request: allowlists, internal-range blocking, and URL validation; this explicitly covers webhooks, file-URL fetches, integration callbacks, and PDF rendering. | [Part F](../parts/part-F.md) | Security |
| D59 | Secrets are externalized, never in code or images; per-client integration credentials are encrypted with envelope encryption; access is least-privilege and audited. | [Part F](../parts/part-F.md) | Security |
| D60 | Layered encryption at rest: volume/disk + database storage encryption + application-level field encryption for the most sensitive data + encrypted backups and object storage. | [Part F](../parts/part-F.md) | Security |
| D61 | TLS everywhere, internal and external, with strong ciphers and HSTS. | [Part F](../parts/part-F.md) | Security |
| D62 | Scheduled rotation for all key classes with overlap windows; per-deployment keys; no shared keys across clients. | [Part F](../parts/part-F.md) | Security |
| D63 | Per-deployment security monitoring with anomaly alerting, plus aggregated security signals to the Control Plane carrying no student data. | [Part F](../parts/part-F.md) | Security |
| D64 | A GDPR-aligned baseline applied everywhere, with Bangladesh-first specifics and explicit minor-data protections, designed so jurisdiction-specific rules are configuration. | [Part F](../parts/part-F.md) | Security |
| D65 | Machine-readable, scoped, audited data export for subject-access/portability and client data ownership. | [Part F](../parts/part-F.md) | Security |
| D66 | A defined incident-response process with severity classification, breach-notification timelines, runbooks, and clear vendor/client responsibilities. | [Part F](../parts/part-F.md) | Security |
| D67 | Security testing integrated into CI: SAST, dependency and secret scanning, plus dedicated authorization and isolation tests; nothing ships without passing them. | [Part F](../parts/part-F.md) | Security |
| D68 | Regular independent penetration testing at a cadence and rigor appropriate to handling minors' data, with tracked remediation. | [Part F](../parts/part-F.md) | Security |
| D69 | Performance is engineered to named peak events with explicit budgets: cache-first reads, async-everything for heavy work, and queue-based load-shedding. | [Part G](../parts/part-G.md) | Performance, Scalability & Ops |
| D70 | Three cache layers with clear ownership: in-process for tiny hot data, Redis for shared resolved data, CDN for static and signed media. | [Part G](../parts/part-G.md) | Performance, Scalability & Ops |
| D71 | N+1 queries are prevented structurally: no lazy relations, explicit batched loading, and detection in tests and monitoring. | [Part G](../parts/part-G.md) | Performance, Scalability & Ops |
| D72 | A read replica for the large-tier deployments, serving reporting, exports, and heavy reads; transactional reads/writes stay on the primary. | [Part G](../parts/part-G.md) | Performance, Scalability & Ops |
| D73 | CDN for static assets and cacheable signed media; the application never serves bulk static or media transfer. | [Part G](../parts/part-G.md) | Performance, Scalability & Ops |
| D74 | Horizontal scaling within a deployment via stateless app and worker instances behind Nginx; workers scale independently by queue depth. | [Part G](../parts/part-G.md) | Performance, Scalability & Ops |
| D75 | Per-deployment containerized topology orchestrated by the Control Plane; Docker Compose now, with a clear migration path to a cluster orchestrator as the fleet grows. | [Part G](../parts/part-G.md) | Performance, Scalability & Ops |
| D76 | Build-and-validate once in CI; the Control Plane orchestrates fleet rollout in health-gated waves with coupled expand-contract migrations. | [Part G](../parts/part-G.md) | Performance, Scalability & Ops |
| D77 | Per-deployment zero-downtime releases via blue-green (or rolling) with health-gated cutover and instant rollback. | [Part G](../parts/part-G.md) | Performance, Scalability & Ops |
| D78 | Per-deployment observability (logs, metrics, traces) with non-sensitive signal aggregation to the Control Plane; built on open standards. | [Part G](../parts/part-G.md) | Performance, Scalability & Ops |
| D79 | Distributed tracing with OpenTelemetry, propagating the correlation id across modules, the queue, and external calls — even within the monolith. | [Part G](../parts/part-G.md) | Performance, Scalability & Ops |
| D80 | SLO-based alerting on symptoms users feel, routed by severity to on-call, fleet-aware, and tuned against fatigue. | [Part G](../parts/part-G.md) | Performance, Scalability & Ops |
| D81 | Per-deployment automated, encrypted backups with point-in-time recovery, tested by restore drills, orchestrated by the Control Plane. | [Part G](../parts/part-G.md) | Performance, Scalability & Ops |
| D82 | Tiered DR with RPO/RTO targets per tier; the large tier uses the read replica as a warm standby and cross-region capability; isolation preserved throughout. | [Part G](../parts/part-G.md) | Performance, Scalability & Ops |
| D83 | Cost is optimized by tiering, right-sizing, storage lifecycle, worker autoscaling, and a shared Control Plane — while accepting that isolation has an irreducible per-deployment cost. | [Part G](../parts/part-G.md) | Performance, Scalability & Ops |
| D84 | Trunk-based development with short-lived feature branches and CI-gated merges. | [Part H](../parts/part-H.md) | Standards & Governance |
| D85 | Every change is peer-reviewed against a security- and architecture-aware checklist before merge; no direct pushes to main. | [Part H](../parts/part-H.md) | Standards & Governance |
| D86 | A risk-weighted test pyramid: heavy unit testing of domain logic, integration tests of critical flows and authorization, and focused end-to-end tests of key journeys, all gating releases. | [Part H](../parts/part-H.md) | Standards & Governance |
| D87 | Lightweight architecture governance: enforced automated rules, ADRs for significant decisions, and a periodic architecture review — proportionate to a small team. | [Part H](../parts/part-H.md) | Standards & Governance |

## Decisions by Part

### Part A — Strategy & Domain

- **D1** — Adopt strategic DDD with explicit bounded contexts as the organizing principle.
- **D2** — Treat Configuration and Workflow as Platform Core, not as shared utilities.
- **D3** — The Configuration context publishes effective values; consumers never read configuration internals.
- **D4** — Integrate contexts via published interfaces and domain events only; never via shared data access.
- **D5** — Modular Monolith per plane, not microservices.
- **D6** — One Application Plane artifact for all clients; differentiation by configuration, never by per-client code or branches.
- **D7** — Enforce an acyclic, tiered module dependency graph with platform contexts strictly below domain contexts.

### Part B — Backend & Data

- **D8** — Persistence model and domain model are separated for Core contexts; pragmatically merged for Generic/Supporting contexts.
- **D9** — Four named roots (`modules`, `platform`, `shared`, `common`) with distinct change-control.
- **D10** — Separate, explicit request and response DTOs; never expose persistence entities across the API.
- **D11** — Repository ports in the application layer, TypeORM implementations in infrastructure, returning domain models for Core contexts.
- **D12** — Two-tier validation: static DTO validation for fixed contracts, definition-driven dynamic validation for configurable fields and forms.
- **D13** — In-process domain events with a transactional outbox for reliability; a real broker deferred.
- **D14** — BullMQ on Redis for all asynchronous and deferred work.
- **D15** — One database per client; institute/campus/session as scoping columns; UUIDv7 keys; PgBouncer; read replica at scale.
- **D16** — Range-partition the high-volume time-series tables; keep the rest unpartitioned.
- **D17** — Event-sourced, append-only audit trail fed by the outbox, separate from configuration version history.
- **D18** — Explicit, versioned TypeORM migrations using expand-contract, rolled out fleet-wide by the Control Plane in health-gated waves; synchronize is never used in any environment.

### Part C — Frontend, Auth & Access

- **D19** — One unified Next.js app with portal-segmented routing, not separate apps per portal.
- **D20** — Public pages use Server Components; the authenticated portal is client-rendered with RTK Query, using App Router for layout, routing, and code-splitting.
- **D21** — Feature-sliced architecture: features own their components, hooks, API slices, and types; routing only composes them.
- **D22** — Centralized, declarative permission gating mirroring backend permission strings; client gating is UX-only.
- **D23** — RTK Query as the single server-state layer, organized as one base API with per-feature endpoint injection and tag-based invalidation.
- **D24** — Access token in memory, refresh token in an httpOnly secure cookie, with silent refresh; never store tokens in localStorage.
- **D25** — A single configurable, headless-driven table engine; feature tables are configuration, not bespoke code.
- **D26** — A unified form engine with a definition-driven dynamic renderer mirroring the backend's two-tier validation.
- **D27** — Runtime white-labeling driven by per-client bootstrap configuration; no rebuild for branding or terminology.
- **D28** — Two distinct layers: i18n for UI language (English/Bangla) and terminology mapping for per-client domain relabeling, resolved together by one label resolver.
- **D29** — Responsive-web-first now; API and design system shaped for a future React Native app; no native build in early phases.

### Part D — Platform Core (Identity, Access, Config, Workflow)

- **D30** — Short-lived access tokens signed with rotating asymmetric keys; tokens carry identity, active scope, roles, and a permissions-version, not the full permission set.
- **D31** — Rotating refresh tokens with token-family reuse detection; stored hashed, never as plaintext.
- **D32** — TOTP-based MFA with recovery codes, enabled per client and optionally step-up for sensitive actions.
- **D33** — Abstract authentication behind an identity-provider port with adapters; ship local credentials now, design the seam for OIDC/SAML.
- **D34** — Permissions and roles are data, not code; effective permissions are resolved and cached, invalidated by a version stamp.
- **D35** — Three-layer data access: RBAC permission, mandatory scope filter, and ownership/relationship rules enforced in the data layer.
- **D36** — Change-set-based configuration versioning: every publish creates an immutable version snapshot with an active pointer.
- **D37** — Custom field values stored as validated JSONB on the owning entity; frequently-queried fields promoted to real columns.
- **D38** — Separate the timeless structure definition from the per-session instance; the hierarchy is data with a marked enrollment leaf.
- **D39** — Declarative finite state machine defined as data; instances are pinned to the definition version they started under.
- **D40** — Every workflow event is recorded immutably and feeds the central audit trail, with full lineage of decisions, escalations, delegations, and timeouts.

### Part E — Cross-Cutting Services

- **D41** — Reporting reads from dedicated read models (a reporting projection), never from transactional tables directly; heavy generation runs on the queue; the largest client uses the read replica.
- **D42** — Report definitions are configuration: datasets, filters, columns, and grouping defined as data and resolved through the configuration engine.
- **D43** — Two-tier PDF generation: HTML-template rendering via a headless browser for rich documents, a lightweight generator for simple ones; all heavy generation async to object storage.
- **D44** — Two-layer notifications: the Communication context decides what to send; a provider-agnostic Notification Delivery layer with per-channel adapters sends it, all queue-driven and event-fed.
- **D45** — SMS via a provider-agnostic adapter with Bangladesh gateways, Bangla Unicode templates, and strict cost controls.
- **D46** — Private object storage (S3/MinIO) behind an adapter; access only via short-lived signed URLs; scoped paths; server-side encryption; virus scanning on upload.
- **D47** — Configurable, retention-class-driven file lifecycle with tiering, archival, and compliant erasure.
- **D48** — Versioned REST public API governed by the same RBAC/scoping model, authenticated by scoped API credentials, rate-limited, and documented via OpenAPI.
- **D49** — Per-deployment gateway concerns split between Nginx (edge) and the application; no separate API-gateway product at current scale.
- **D50** — Outbound webhooks driven by the outbox, signed, retried with backoff, and subscribable per event type.
- **D51** — Build the marketplace's foundations now (public API, webhooks, entitlements); defer the marketplace product itself.
- **D52** — Internal modular extensibility through trusted extension points and configuration; explicitly no sandboxed third-party code execution in early phases.

### Part F — Security

- **D53** — Defense-in-depth with deployment isolation as the primary control and scoped, least-privilege access as the secondary.
- **D54** — STRIDE-based threat modeling plus an education-specific threat catalog, prioritized by impact on minors' data and on academic/financial integrity.
- **D55** — Output-encoding by default, a strict Content Security Policy, and treating all dynamic and rich content (custom fields, notices, PDF templates) as untrusted.
- **D56** — Bearer-token API calls are CSRF-immune; the cookie-based refresh path is protected by SameSite plus an anti-CSRF token.
- **D57** — Parameterized access only, through TypeORM and the query builder; no string-concatenated SQL anywhere; reporting uses curated datasets, never raw user SQL.
- **D58** — Strict egress controls for every server-side outbound request: allowlists, internal-range blocking, and URL validation; this explicitly covers webhooks, file-URL fetches, integration callbacks, and PDF rendering.
- **D59** — Secrets are externalized, never in code or images; per-client integration credentials are encrypted with envelope encryption; access is least-privilege and audited.
- **D60** — Layered encryption at rest: volume/disk + database storage encryption + application-level field encryption for the most sensitive data + encrypted backups and object storage.
- **D61** — TLS everywhere, internal and external, with strong ciphers and HSTS.
- **D62** — Scheduled rotation for all key classes with overlap windows; per-deployment keys; no shared keys across clients.
- **D63** — Per-deployment security monitoring with anomaly alerting, plus aggregated security signals to the Control Plane carrying no student data.
- **D64** — A GDPR-aligned baseline applied everywhere, with Bangladesh-first specifics and explicit minor-data protections, designed so jurisdiction-specific rules are configuration.
- **D65** — Machine-readable, scoped, audited data export for subject-access/portability and client data ownership.
- **D66** — A defined incident-response process with severity classification, breach-notification timelines, runbooks, and clear vendor/client responsibilities.
- **D67** — Security testing integrated into CI: SAST, dependency and secret scanning, plus dedicated authorization and isolation tests; nothing ships without passing them.
- **D68** — Regular independent penetration testing at a cadence and rigor appropriate to handling minors' data, with tracked remediation.

### Part G — Performance, Scalability & Ops

- **D69** — Performance is engineered to named peak events with explicit budgets: cache-first reads, async-everything for heavy work, and queue-based load-shedding.
- **D70** — Three cache layers with clear ownership: in-process for tiny hot data, Redis for shared resolved data, CDN for static and signed media.
- **D71** — N+1 queries are prevented structurally: no lazy relations, explicit batched loading, and detection in tests and monitoring.
- **D72** — A read replica for the large-tier deployments, serving reporting, exports, and heavy reads; transactional reads/writes stay on the primary.
- **D73** — CDN for static assets and cacheable signed media; the application never serves bulk static or media transfer.
- **D74** — Horizontal scaling within a deployment via stateless app and worker instances behind Nginx; workers scale independently by queue depth.
- **D75** — Per-deployment containerized topology orchestrated by the Control Plane; Docker Compose now, with a clear migration path to a cluster orchestrator as the fleet grows.
- **D76** — Build-and-validate once in CI; the Control Plane orchestrates fleet rollout in health-gated waves with coupled expand-contract migrations.
- **D77** — Per-deployment zero-downtime releases via blue-green (or rolling) with health-gated cutover and instant rollback.
- **D78** — Per-deployment observability (logs, metrics, traces) with non-sensitive signal aggregation to the Control Plane; built on open standards.
- **D79** — Distributed tracing with OpenTelemetry, propagating the correlation id across modules, the queue, and external calls — even within the monolith.
- **D80** — SLO-based alerting on symptoms users feel, routed by severity to on-call, fleet-aware, and tuned against fatigue.
- **D81** — Per-deployment automated, encrypted backups with point-in-time recovery, tested by restore drills, orchestrated by the Control Plane.
- **D82** — Tiered DR with RPO/RTO targets per tier; the large tier uses the read replica as a warm standby and cross-region capability; isolation preserved throughout.
- **D83** — Cost is optimized by tiering, right-sizing, storage lifecycle, worker autoscaling, and a shared Control Plane — while accepting that isolation has an irreducible per-deployment cost.

### Part H — Standards & Governance

- **D84** — Trunk-based development with short-lived feature branches and CI-gated merges.
- **D85** — Every change is peer-reviewed against a security- and architecture-aware checklist before merge; no direct pushes to main.
- **D86** — A risk-weighted test pyramid: heavy unit testing of domain logic, integration tests of critical flows and authorization, and focused end-to-end tests of key journeys, all gating releases.
- **D87** — Lightweight architecture governance: enforced automated rules, ADRs for significant decisions, and a periodic architecture review — proportionate to a small team.

