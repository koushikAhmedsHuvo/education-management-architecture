# Use Case Specification Repository

The **operational source of truth** for Development, QA, UI/UX, Product Validation, and UAT. It transforms the approved **Business Rules Catalog** (the *what must be true*) into **use case specifications** (the *how users accomplish goals, step by step, with every flow, edge case, and acceptance test*). It does **not** create new business rules — every use case cites existing rule IDs (e.g., `AUTH-002`, `SESS-006`).

## Relationship to the other artifacts

```
Architecture Blueprint (how it's built)
Implementation Plan (when it's built)
Business Rules Catalog (what must be true)  ──►  Use Case Repository (how users do it) ──► Dev · QA · UI · UAT
```

## Conventions

**Use Case ID:** `UC-<MODULE>-NNN` (3-digit, stable, never reused). These are the authoritative, expanded specs; the short use cases embedded in the Business Rules Catalog were summaries. **Prefix disambiguation (mirrors Catalog Conflict C-09):** Guardian use cases use `UC-GRD-N-NNN` and Grading use cases use `UC-GRD-NNN` — always write the Guardian form with the `-N-` segment so the two never collide.

**Each module document provides:** Primary Actors · Secondary Actors · Goals · User Journeys · then a **Use Case Inventory** organized into the nine required categories (Core · CRUD · Approval · Search · Reporting · Bulk · Import · Export · Workflow · Administrative · Exception), followed by the specifications.

**Full use case template (applied to high-value/complex use cases):**
Use Case ID · Name · Module · Priority · Actors · Goal · Description · Business Rules Applied · Preconditions · Trigger · Main Success Scenario · Alternative Flows · Exception Flows · Validation Rules · Permissions Required · Notifications Triggered · Audit Events Generated · Data Created/Updated/Deleted · Post Conditions · Related Use Cases · Acceptance Criteria (Given/When/Then) · **Edge Case Analysis** (Invalid Input · Permission Failure · Concurrent Update · Duplicate Data · System Failure · Workflow Failure) · **QA Coverage** (Positive · Negative · Boundary).

**Compact specs:** routine CRUD/search/export use cases use a condensed block (Goal · Rules · Main flow · Permissions · Validation · key Edge/QA) — complete enough to build and test, without ceremony. This keeps the repository usable; complexity, not category, decides depth.

## Delivery phases

- **Phase 1 — Core Platform (this delivery):** 01 Authentication · 02 Authorization · 03 Institute · 04 Campus · 05 Academic Session
- Phase 2 Student Lifecycle · Phase 3 Academic · Phase 4 Exam & Results · Phase 5 Finance · Phase 6 HR · Phase 7 Configuration Engine · Phase 8 Workflow Engine · Phase 9 Reporting/Notification/Audit/Files

## How each audience uses this

- **Developers** build to the Main Success Scenario, Alternative/Exception Flows, and Validation Rules; the cited business rules are the authoritative behavior.
- **QA** derives test suites from the Acceptance Criteria, Edge Case Analysis, and QA Coverage (Positive/Negative/Boundary).
- **UI/UX** design from the User Journeys, Main/Alternative flows, and Notifications.
- **Product/UAT** validate against Acceptance Criteria and Post Conditions.
