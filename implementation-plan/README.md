# Implementation Plan

The phase-by-phase execution playbook for the Enterprise Education ERP, derived from the [Implementation & Execution Blueprint](../Implementation_Execution_Blueprint.md) and governed by the approved architecture (Blueprint Parts A–H). Each document is a runnable phase gate, not a description.

## How these documents work

Every phase document follows the same seven sections so a delivery manager can run any phase the same way:

| Section | Answers |
|---|---|
| **Objectives** | Why this phase exists; what it must achieve |
| **Scope** | What is in, and explicitly what is out (the anti-scope-creep fence) |
| **Deliverables** | The concrete artifacts produced |
| **Dependencies** | What must be done before this phase can start |
| **Risks** | What can go wrong, with mitigation |
| **Acceptance Criteria** | The work is **correct** — functional and quality bars met |
| **Exit Criteria** | The phase is **closed** — accepted, demoed, signed off, next phase unblocked |

> **Acceptance vs Exit — the key distinction.** Acceptance criteria prove the *deliverables work*. Exit criteria prove the *phase is finished and the team can safely move on* (acceptance met **plus** demoed on staging, ADRs/docs updated, the shared Definition of Done satisfied, and downstream dependencies unblocked). A phase is not over until its Exit Criteria are signed off.

## The shared Definition of Done (applies to every feature slice)

Referenced by every phase rather than repeated. A slice is Done only when: expand-contract migration reversible (D18); domain unit-tested with no framework imports; **authorization/scope/ownership tests pass** (D35); every state change emits an audit event (D13/D17); configurable behavior reads through the config engine (D2); API published as OpenAPI with explicit DTOs (D10); UI consumes the contract with permission gating (D22); the critical path has an e2e test; peer-reviewed and merged behind a feature flag if incomplete (D84/D85).

## Index

| # | Document | Maps to Blueprint Part |
|---|---|---|
| 01 | [Project Setup](./01-project-setup.md) | Part 1–2 |
| 02 | [Foundation Phase](./02-foundation-phase.md) | Part 3 |
| 03 | [Core Platform Phase](./03-core-platform-phase.md) | Part 4 |
| 04 | [Configuration Engine Phase](./04-configuration-engine-phase.md) | Part 5 |
| 05 | [Workflow Engine Phase](./05-workflow-engine-phase.md) | Part 6 |
| 06 | [Academic Phase](./06-academic-phase.md) | Part 7 |
| 07 | [Student Lifecycle Phase](./07-student-lifecycle-phase.md) | Part 8 |
| 08 | [Attendance Phase](./08-attendance-phase.md) | Part 9 |
| 09 | [Examination Phase](./09-examination-phase.md) | Part 10 |
| 10 | [Result Phase](./10-result-phase.md) | Part 11 |
| 11 | [Fee Phase](./11-fee-phase.md) | Part 12 |
| 12 | [HR Phase](./12-hr-phase.md) | Part 13 |
| 13 | [Reporting Phase](./13-reporting-phase.md) | Part 14 |
| 14 | [Notification Phase](./14-notification-phase.md) | Part 15 |
| 15 | [Portals Phase](./15-portals-phase.md) | Part 16 |
| 16 | [Production Readiness](./16-production-readiness.md) | Part 17 |
| 17 | [Client Deployment](./17-client-deployment.md) | Part 18 |
| 18 | [Release Plan](./18-release-plan.md) | Part 19 |
| 19 | [Sprint Plan](./19-sprint-plan.md) | Part 20 |
| 20 | [Risk Management](./20-risk-management.md) | Part 21 |
| 21 | [Team Allocation](./21-team-allocation.md) | Part 22 |
| 22 | [Final Execution Blueprint](./22-final-execution-blueprint.md) | Part 23 |

## Phase sequencing rule

Engines before consumers; foundation thin; one institution type (school) end-to-end before breadth; every phase a shippable vertical slice. First paying client lands at the **end of Release 2** (after document 11).
