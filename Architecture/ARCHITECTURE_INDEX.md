# Architecture Index

A complete, navigable index of the Enterprise Education ERP architecture blueprint. Eight parts, mapped to the original 23-section brief, with 87 recorded decisions.

| Part | Title | Sections | Decisions | File |
|---|---|---|---|---|
| A | Strategy & Domain | 8 | D1–D7 | [part-A.md](./parts/part-A.md) |
| B | Backend & Data | 34 | D8–D18 (+D8-bis) | [part-B.md](./parts/part-B.md) |
| C | Frontend, Auth & Access | 25 | D19–D29 | [part-C.md](./parts/part-C.md) |
| D | Platform Core: Identity, Access, Config, Workflow | 29 | D30–D40 | [part-D.md](./parts/part-D.md) |
| E | Cross-Cutting Services | 25 | D41–D52 | [part-E.md](./parts/part-E.md) |
| F | Security | 25 | D53–D68 | [part-F.md](./parts/part-F.md) |
| G | Performance, Scalability, Infra, DevOps & Ops | 33 | D69–D83 | [part-G.md](./parts/part-G.md) |
| H | Standards, Roadmap & Critical Review | 20 | D84–D87 | [part-H.md](./parts/part-H.md) |

Total: **87 decisions**, continuous and gap-free.

---

## Part A — Strategy & Domain Design
1. Executive Architecture Summary
2. Domain-Driven Design
3. Domain Map
4. Subdomain Classification
5. Bounded Context Design
6. Context Relationships
7. Modular Monolith Architecture
8. Module Dependency Rules

## Part B — Backend & Data Architecture
**B-1 NestJS Application:** 1. NestJS Architecture · 2. Folder Structure · 3. Module Structure · 11. Shared Module · 12. Common Module
**B-2 Layered Module Internals:** 4. Domain Layer · 5. Application Layer · 6. Infrastructure Layer · 7. DTO Strategy · 8. Entity Strategy · 9. Repository Strategy · 10. Service Strategy
**B-3 Request Pipeline:** 13. Guards · 14. Interceptors · 15. Exception Filters · 16. Middleware · 17. Validation Architecture
**B-4 Asynchronous & Infra:** 18. Event Architecture · 19. Redis Strategy · 20. Queue Architecture · 21. Background Jobs
**B-5 PostgreSQL:** 22. PostgreSQL Architecture · 23. ERD · 24. Table Design · 25. Relationships · 26. Index Strategy · 27. Partitioning · 28. Soft Delete · 29. Audit Trail · 30. Archival · 31. Migration Strategy
**B-6 TypeORM & Dynamic Config:** 32. TypeORM Best Practices · 33. Dynamic Configuration Storage · 34. JSONB Strategy

## Part C — Frontend, Authentication & Access
**C-1 Next.js App:** 1. Next.js Architecture · 2. App Router Structure · 3. Folder Structure · 5. Layout Strategy · 17. Feature-Based Structure
**C-2 Portals & Routing:** 4. Dashboard · 22. Multi-Portal · 20. Route Protection · 10. Authorization Flow · 11. Permission Architecture
**C-3 State & Data:** 6. State Management · 7. RTK Query · 8. API Layer · 19. Caching
**C-4 Auth:** 9. Authentication Flow
**C-5 UI System:** 12. Component Architecture · 13. Design System · 14. Table Architecture · 15. Form Architecture · 16. Modal Architecture
**C-6 Cross-cutting:** 18. Error Handling · 21. White-Labeling · 23. Theme Engine · 24. Internationalization · 25. Mobile-Ready

## Part D — Platform Core: Identity, Access, Configuration & Workflow
**D-1 Authentication:** 1. Architecture · 2. JWT · 3. Refresh Tokens · 4. Sessions · 5. Devices · 6. MFA · 7. Password Recovery · 8. SSO-Ready
**D-2 Authorization:** 9. Architecture · 10. RBAC · 11. Dynamic Permissions · 12. Institute Access · 13. Campus Access · 14. Data Access Controls
**D-3 Configuration Engine:** 15. Engine Design · 16. Versioning · 17. Rollback · 18. Dynamic Forms · 19. Custom Fields · 20. Dynamic Academic Structures · 21. Dynamic Fee Structures
**D-4 Workflow Engine:** 22. Engine Design · 23. State Machine · 24. Approval Engine · 25. Escalation · 26. Delegation · 27. Timeout · 28. Versioning · 29. Audit Trail

## Part E — Cross-Cutting Services
**E-1 Reporting:** 1. Architecture · 2. Dynamic Report Builder · 3. Templates · 4. Excel Export · 5. PDF Export · 6. Scheduled Reports
**E-2 Notifications:** 7. Architecture · 8. Email · 9. SMS · 10. Push · 11. In-App · 12. Preferences · 13. Queue
**E-3 File Management:** 14. Architecture · 15. Document Storage · 16. Certificate Storage · 17. Media Storage · 18. Lifecycle Management
**E-4 Integration:** 19. Architecture · 20. Public API · 21. API Gateway · 22. Webhooks · 23. External Integrations · 24. Future Marketplace · 25. Internal Plugin Architecture

## Part F — Enterprise Security
**F-1 Foundation:** 1. Security Architecture · 2. Threat Model
**F-2 OWASP & Attacks:** 3. OWASP Top 10 · 4. XSS · 5. CSRF · 6. SQL Injection · 7. SSRF
**F-3 AuthN/AuthZ/Session/Password:** 8. Auth Security · 9. Authz Security · 10. Session Security · 11. Password Security
**F-4 Secrets & Encryption:** 12. Secrets Management · 13. Encryption at Rest · 14. Encryption in Transit · 15. Key Rotation
**F-5 Audit & Compliance:** 16. Audit Logs · 17. Security Monitoring · 18. Compliance Readiness · 19. GDPR Readiness · 20. Retention · 21. Export
**F-6 DR, IR & Testing:** 22. DR Security · 23. Incident Response · 24. Security Testing · 25. Penetration Testing

## Part G — Performance, Scalability, Infrastructure, DevOps & Operations
**G-1 Performance:** 1. Architecture · 2. Redis · 3. Cache Layers · 4. DB Optimization · 5. Query Optimization · 6. N+1 Prevention · 7. Read Replica · 8. CDN
**G-2 Scalability:** 9. Roadmap · 10. Current · 11. Medium · 12. Large · 13. Horizontal · 14. Vertical
**G-3 Infra & DevOps:** 15. Infrastructure · 16. Docker · 17. Nginx · 18. GitHub Actions · 19. CI/CD · 20. Deployment · 21. Blue-Green · 22. Rollback · 23. Environments
**G-4 Observability:** 24. Monitoring · 25. Logging · 26. Metrics · 27. Tracing · 28. Alerting
**G-5 Backup, DR & Cost:** 29. Backup · 30. Restore · 31. Disaster Recovery · 32. Cost Optimization · 33. Cost Planning

## Part H — Engineering Standards, Roadmap & Critical Review
**H-1 Standards:** 1. Engineering Standards · 2. Coding · 3. Naming · 4. Module Rules · 5. Folder Rules · 6. Documentation · 7. Git Workflow · 8. Branch Strategy · 9. PR Review · 10. Testing Standards · 11. Backend Testing · 12. Frontend Testing · 13. Governance · 14. ADR Strategy · 15. Technical Debt
**H-2 Roadmap:** 16. Development Roadmap (Phase 1 Foundation → Phase 5 Platform Expansion)
**H-3 Critical Review:** 17. CTO Review (Weaknesses, Risks, Bottlenecks, Scaling, Security, Maintainability)
**H-4 Verdict:** 18. Final Recommendations · 19. Architecture Scorecard · 20. Go/No-Go Assessment

---

## Cross-cutting threads (where to follow a topic across parts)

- **Configuration-driven thesis:** A (D2) → B (D8-bis, dynamic storage) → C (D26, dynamic forms) → D (D36–D38, engine/versioning/structures)
- **Tenancy & isolation:** A (two planes) → B (D15, one DB/client) → F (D53, isolation as primary control) → G (D75/D83, fleet ops & cost)
- **Authorization & access:** A (Identity Open Host) → B (D10, request pipeline) → C (D22, UX gating) → D (D34–D35, RBAC + three-layer) → F (D53, defense-in-depth)
- **Events & reliability:** B (D13, outbox) → D (D40, workflow audit) → E (D41/D44/D50, reporting/notifications/webhooks) → F (D17, audit) → G (D79, tracing)
- **Compliance & minor-data:** F (D54/D64–D68) → B (D17, audit) → E (D47, file retention) → G (D81, backups)
- **Cost & team-fit reality:** G (D83, isolation cost floor) → H (§17–20, critical review and go/no-go)

*See the [Decision Records](./decisions/decision-records.md) for the full D1–D87 log.*
