# Module Functional Specification Repository

Implementation-ready functional specifications for the Enterprise Education ERP — the layer engineering, QA, and design build directly from. Each module follows a fixed 22-section template and references (never redefines) the approved **Business Rules Catalog** and **Use Case Repository**.

## Conventions
- **FR-IDs** (`FR-001…`) are functional-requirement identifiers local to each module.
- **Business Rule References** (§12) and **Use Case References** (§13) cite approved IDs only — no new rules are created here.
- **API Requirements** (§14) list endpoint purpose / method / actor only — no request/response contracts (per scope).
- Permissions follow the registry namespacing (`module.entity.action`) enforced by the Authorization module (deny-by-default + scope + ownership).
- Architecture is fixed; decisions are cited by ID (e.g., D35 three-layer access, D30/31 tokens, D36 config versioning) where they constrain implementation.

## Delivery phases
1. **Core Platform — Authentication, Authorization, Institute, Campus, Academic Session** ✅
2. Student Lifecycle — Admission, Student, Enrollment, Guardian
3. Academic — Class, Section, Subject, Attendance
4. Examination & Results
5. Finance
6. HR
7. Configuration Engine
8. Workflow Engine
9. Reporting, Notification, Audit, Files

## The 22-section template
1 Overview · 2 Actors · 3 Functional Requirements · 4 Features · 5 Screens · 6 Screen Actions · 7 Forms · 8 Search & Filter · 9 Table · 10 Workflow · 11 Permissions · 12 Business Rule References · 13 Use Case References · 14 API Requirements · 15 Database Requirements · 16 Notifications · 17 Audit · 18 Reporting · 19 Error Handling · 20 Edge Cases · 21 Acceptance Criteria · 22 Future Enhancements
