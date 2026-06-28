# 05 — Workflow Engine Phase

**Phase type:** Platform Core · **Release:** R1 · **Maps to:** Blueprint Part 6

The generic, domain-agnostic approval engine that admission, leave, fee-waiver, and HR approvals all run on. Built once; never re-implemented per module.

## Objectives
- Deliver a declarative finite-state-machine engine driven entirely by data (definitions), with version-pinned instances.
- Provide approvals (sequential/parallel/conditional), escalation, delegation, and timeout as declarative rules.
- Record every workflow event immutably into the central audit trail.

## Scope
**In:** workflow definitions (states/transitions/guards/actions as data); transition engine with a small safe condition grammar; version-pinned instances; approval engine (approver resolution by role/user/dynamic resolver); escalation (time-triggered); delegation (bounded, audited); timeout rules (remind/escalate/auto-approve/auto-expire); workflow audit; task inbox UI + instance timeline + (admin) designer.
**Out:** BPMN / graph-shaped orchestration (explicitly out); domain-specific approval logic (lives in consuming modules).

## Deliverables
- A generic engine: a declarative definition drives a multi-step approval with escalation and delegation, fully audited.
- The **task inbox** component, reused across all portals.
- First consumer (Admission) ready to integrate in the Student Lifecycle phase.

## Dependencies
Configuration Engine (04) — shared versioning pattern. Foundation (02) — events, scheduled jobs, audit. Core Platform (03) — approver resolution needs roles/scope.

## Risks
- **State-machine races / stuck instances.** *Mitigation:* transactional idempotent transitions; reconciliation sweep; concurrency tested.
- **Condition grammar becomes an unsafe mini-language.** *Mitigation:* keep it small, declarative, sandboxed — no arbitrary code.
- **Speculative generality (built before a consumer).** *Mitigation:* build the minimum Admission needs; grow escalation/delegation/timeout when modules require them.

## Acceptance Criteria
- A definition change does **not** affect in-flight instances (version-pinning proven).
- An unactioned task escalates after its timeout; a delegated task routes to the delegate with both authorities recorded.
- Every workflow event is in the audit trail with actor, authority, and version.

## Exit Criteria
- Engine acceptance signed off; concurrency/stuck-instance tests green.
- Task inbox demonstrated and documented as reusable.
- Admission integration contract agreed with the Student Lifecycle phase.
- **Release 1 platform complete** — feature phases begin.
