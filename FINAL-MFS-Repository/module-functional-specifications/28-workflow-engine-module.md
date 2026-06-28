# 28 — Workflow Engine Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint (Platform Core; declarative version-pinned FSM, D2/D39/D40), Business Rules Catalog (`WFL-001…011`), and Use Case Repository (`UC-WFL-001…019`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Provide the generic, declarative engine that runs every approval/review/multi-step process: version-pinned FSMs, dynamic approver resolution with SoD, escalation/delegation/timeout, sequential/parallel/conditional routing, withdrawal/cancellation/reassignment, idempotent reliability with no orphaned instances, and bounded return-for-correction loops.

**Business Goal.** Let every module run governed processes generically and fairly — pinned to the rules in force at initiation, never silently auto-approving sensitive actions, never stalling, never orphaning an instance.

**Scope.** Declarative workflow definitions; version-pinning at initiation; deterministic FSM execution; dynamic approver resolution + SoD; escalation on timeout; bounded audited delegation; safe timeout (no silent auto-approve); sequential/parallel/conditional routing; withdrawal/cancellation/reassignment; reliability/idempotency/no-orphans; bounded return-for-correction.

**Out of Scope.** Domain meaning of any process (consuming modules own it; the engine executes the FSM). Approver-identity source (Staff hierarchy / Authorization provide it; engine resolves against it). Notification delivery (Notification module). Config of workflow definitions storage (Configuration Engine — definitions are configuration).

---

## 2. Actors

**Primary Actors.** Initiator/Requester (starts an instance, usually via a domain action), Approver/Reviewer (acts on a step), Delegate (temporary authority), Workflow Administrator (defines/versions workflows), System (FSM execution, escalation/timeout, orphan recovery).

**Secondary Actors.** All consuming modules (declare workflows, initiate instances, react to outcomes), Staff hierarchy (approver resolution), Configuration Engine (definitions/versioning), Authorization (SoD), Notification, Audit.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Declarative definitions | Define workflows as states/transitions/approver-rules/SoD/routing/timeouts. | High | Configuration Engine |
| FR-002 | Version-pinning at initiation | Pin the definition version at initiation (fairness). | Critical | FR-001 |
| FR-003 | Deterministic FSM execution | Progress only via declared transitions; no invalid jumps. | Critical | FR-001 |
| FR-004 | Approver resolution + SoD | Resolve approvers dynamically; enforce requester ≠ approver. | Critical | Staff (STF-006), AUTHZ-009 |
| FR-005 | Escalation on timeout | Escalate stalled steps; bounded. | High | FR-004 |
| FR-006 | Bounded audited delegation | Time-boxed, attributed delegation; bounded chains. | High | AUTHZ-009 |
| FR-007 | Safe timeout (no silent auto-approve) | Sensitive steps never auto-approve; unsafe definitions rejected. | Critical | FR-001 |
| FR-008 | Sequential/parallel/conditional routing | Chain/parallel(quorum)/conditional branches deterministically. | High | FR-003 |
| FR-009 | Withdrawal/cancellation/reassignment | Governed withdraw/cancel/reassign. | Medium | FR-003 |
| FR-010 | Reliability/idempotency/no-orphans | Every instance terminal or flagged; double-submissions decide once. | High | Outbox (D13) |
| FR-011 | Bounded return-for-correction | Bounded rework loops; escalate/terminate on breach. | Medium | FR-003 |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Declarative workflows | Config-defined FSMs. | New processes without code. |
| Version-pinning | Frozen at initiation. | Fairness (admissions/awards). |
| Deterministic FSM | Only declared transitions. | Predictability. |
| SoD approver resolution | Requester ≠ approver. | Fraud prevention. |
| Escalation/delegation/timeout | Stall-proof routing. | Never stuck. |
| Safe timeout | No silent auto-approve. | Integrity. |
| Flexible routing | Sequence/parallel/conditional. | Complex processes. |
| No orphans + idempotency | Reliable execution. | Trustworthy outcomes. |
| Bounded rework | Capped return loops. | No infinite ping-pong. |

---

## 5. Screens

Workflow Definition (designer); Define/Version Workflow; My Tasks (approver inbox); Instance Detail / Task; Approve/Reject/Return; Delegate Authority; Withdraw/Cancel/Reassign; Instance/Task Search; SLA/Bottleneck & Audit Report; Bulk Approve/Act; Orphan Detection & Recovery; Change Approvals (for definition changes).

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Workflow Definition | Define, Version, Validate FSM, Publish | — |
| My Tasks (inbox) | Open, Approve, Reject, Return, Delegate | Bulk Approve (low-risk) |
| Instance Detail | View State/Lineage, Act, Reassign, Cancel | — |
| Approve/Reject/Return | Approve, Reject, Return-for-Correction (reason) | — |
| Delegate | Set Delegate/Period, Confirm | — |
| Withdraw/Cancel/Reassign | Withdraw (initiator), Cancel (admin), Reassign | — |
| SLA Report | Run, Filter, Export | Export |
| Orphan Recovery | Detect, Recover (idempotent) | Bulk Recover |

---

## 7. Forms

**Define/Version Workflow** — states, transitions, `approverRules`, `sodConstraints`, `routing` (sequential/parallel/conditional), `timeoutPolicy`, `returnBound`. Validation: FSM reachable/terminating (WFL-001/003); unsafe auto-approve on sensitive step rejected (WFL-007 / UC-WFL-019); immutable version on publish (CFG-004).

**Initiate Instance** (via domain action) — `workflowKey`, `context`. Validation: pins current definition version (WFL-002); resolves first approver with SoD (WFL-004); starts timers.

**Act on Task** — `decision` (approve/reject/return), `reason`. Validation: only declared transitions (WFL-003 / UC-WFL-018); requester ≠ approver (WFL-004 / UC-WFL-017); return increments bounded counter (WFL-011).

**Delegate** — `delegate` (eligible), `period`. Validation: time-boxed, attributed, bounded chain (WFL-006).

**Withdraw/Cancel/Reassign** — `action`, `reason`. Validation: withdrawal pre-terminal only; cancel/reassign governed (WFL-009).

---

## 8. Search & Filter Requirements

**Instances/Tasks:** by workflow, status (in-progress/approved/rejected/escalated/withdrawn), initiator, approver, SLA breach, date. Sorting: SLA/created/status. Pagination: server-side, 25 default. Scope-bound (initiators see own; approvers see assigned).

---

## 9. Table Requirements

**Task inbox:** Workflow, Subject, Step, Assigned, SLA, Status, Action. **Instance table:** Workflow, Initiator, Current State, Version, Created, SLA. Sorting on SLA/Created. Filtering as above. Export (governed). Bulk: approve/act (low-risk only — sensitive excluded).

---

## 10. Workflow Requirements

**Trigger events:** define/version, initiate, progress, resolve approver, escalate/delegate/timeout, withdraw/cancel/reassign, return-for-correction, orphan recovery. **Status changes:** instance `IN_PROGRESS → APPROVED/REJECTED/WITHDRAWN/CANCELLED`; step transitions per FSM. **Approvals:** definition changes governed; instances ARE the approval mechanism. **Notifications:** pending task, escalation, delegation, decision, return. **Audit:** initiation (pinned version), every transition/decision, escalations/delegations/timeouts, SoD/auto-approve blocks (immutable, full lineage — D40).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Define/version workflow | `workflow.definition.manage` |
| Initiate instance | `workflow.instance.initiate` (often implicit in domain action) |
| Act on task (approve/reject/return) | `workflow.act` (resolved approver) |
| Delegate authority | `workflow.delegate` |
| Withdraw/cancel/reassign | `workflow.instance.manage` |
| View instance/task (scoped) | `workflow.view` |
| Bulk act (low-risk) | `workflow.bulk.act` |
| SLA/audit report | `workflow.report.view` |

Approver resolution enforces SoD (WFL-004); sensitive steps never auto-approve (WFL-007).

---

## 12. Business Rule References

WFL-001 (declarative definitions), WFL-002 (version-pinning at initiation — fairness), WFL-003 (deterministic FSM execution), WFL-004 (dynamic approver resolution with SoD), WFL-005 (escalation on timeout), WFL-006 (bounded audited delegation), WFL-007 (safe timeout — no silent auto-approve), WFL-008 (sequential/parallel/conditional routing), WFL-009 (withdrawal/cancellation/reassignment), WFL-010 (reliability/idempotency/no orphaned instances), WFL-011 (bounded return-for-correction). Cross-cutting: CFG-004 (definition versioning), STF-006 (approver hierarchy), AUTHZ-009 (SoD), AUD-001 (lineage), and every consuming module (ADM-003, LEV-004/005, DSC-003, SCH-003, RES-006, HR-006, CFG-007).

## 13. Use Case References

UC-WFL-001 (Define/Version), UC-WFL-002 (Initiate — version-pinned), UC-WFL-003 (Progress — deterministic FSM), UC-WFL-004 (Resolve Approver with SoD), UC-WFL-005 (Escalate on Timeout), UC-WFL-006 (Delegate — bounded), UC-WFL-007 (Safe Timeout — no silent auto-approve), UC-WFL-008 (Sequential/Parallel/Conditional Routing), UC-WFL-009 (Withdraw/Cancel/Reassign), UC-WFL-010 (Return-for-Correction — bounded), UC-WFL-011 (View Instance/Task — scoped), UC-WFL-012 (Manage Pending Tasks — inbox), UC-WFL-013 (Search), UC-WFL-014 (SLA/Bottleneck & Audit Report), UC-WFL-015 (Bulk Approve/Act), UC-WFL-016 (Orphan Detection & Recovery), UC-WFL-017 (Self-Approval Blocked), UC-WFL-018 (Invalid Transition Blocked), UC-WFL-019 (Unsafe Auto-Approve Definition Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Define / version workflow | POST/PUT | Workflow Admin |
| Initiate instance (version-pinned) | POST (internal) | System (consuming module) |
| Progress instance (act on task) | POST | Approver |
| Resolve approver (SoD) | GET (internal) | System |
| Escalate / delegate | POST | System / Approver |
| Withdraw / cancel / reassign | POST | Initiator / Admin |
| Return-for-correction | POST | Approver |
| Get pending tasks (inbox) | GET | Approver |
| View / search instances & tasks | GET | Authorized roles |
| SLA / bottleneck & audit report | GET | Admin |
| Bulk act (low-risk) | POST | Approver |
| Orphan detection & recovery | POST | System / Admin |

Initiation pins the definition version (WFL-002); resolution enforces SoD (WFL-004); recovery is idempotent and never double-decides (WFL-010).

---

## 15. Database Requirements

**Entities:** `WorkflowDefinition` (states, transitions, approverRules, sod, routing, timeout, version), `WorkflowInstance` (definitionVersion pinned, currentState, context, status), `Task` (instanceId, step, assignee, sla, status), `Decision` (actor, outcome, reason), `Delegation` (delegator, delegate, period), `EscalationLog`, `ReturnLog` (round count). **Relationships:** Definition 1—* Instance; Instance 1—* Task; Task 1—1 Decision. **Indexes:** index(WorkflowInstance.status, definitionVersion), index(Task.assigneeId, status), index(Task.sla), unique(WorkflowInstance idempotencyKey). Version pinned per instance (WFL-002); every event recorded immutably feeding audit (D40); orphan detection on stuck instances (WFL-010).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Pending task, escalation, delegation, decision, return-for-correction. |
| In-App | Approver task inbox; SLA warnings; escalation/delegation notices. |
| SMS/Push | Urgent approval/escalation alerts (minimized). |

Notifications ride the outbox (non-blocking, NOT-005); escalation is never silent (WFL-007).

---

## 17. Audit Requirements

Log: definition publishes (version), instance initiation (pinned version), every transition/decision (actor, outcome, reason), approver resolution + SoD exclusions, escalations/delegations/timeouts, withdrawals/cancellations/reassignments, return rounds, orphan recovery, every SoD/unsafe-auto-approve block. Record who/when/before/after. Full decision lineage retained (D40). Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** SLA performance, Bottleneck analysis, Decision audit/lineage, Escalation/delegation frequency, Orphan/recovery log. **Exports:** governed instance/task export. **Dashboards:** workflow health (in-flight, breached SLAs, escalations, orphans, approval throughput).

---

## 19. Error Handling

**Validation:** invalid transition, self-approval, unsafe auto-approve definition → specific errors (UC-WFL-017/018/019). **Permission:** acting without resolved authority → 403. **Workflow:** no eligible approver → escalate/flag (never auto-approve); return-bound exceeded → escalate/terminate (WFL-011). **System:** resolution failure → fail closed (escalate/flag); crash mid-instance → orphan detection + idempotent recovery (WFL-010).

---

## 20. Edge Cases

**Concurrent updates:** definition edit during initiation → instance pins version at initiation instant. **Duplicate data:** duplicate initiation (idempotency key) → one instance; double approval → idempotent single decision. **Partial failures:** bulk act partial → per-task report; sensitive tasks excluded from bulk. **Rollback:** return-correct-resubmit on pinned version; bound exceeded → escalate/terminate. **Stall race:** SLA breach + late action → first valid decision wins; no escalation target → hold-and-flag (never auto-approve).

---

## 21. Acceptance Criteria

**Functional.** Workflows are declarative and versioned; an instance pins the definition version at initiation and runs on it to completion; only declared transitions execute; approvers resolve dynamically with requester ≠ approver enforced; stalled steps escalate, away approvers delegate (bounded/attributed), and sensitive steps never silently auto-approve (unsafe definitions rejected at design time); routing supports sequence/parallel(quorum)/conditional; instances can be withdrawn/cancelled/reassigned; every instance reaches a terminal state or is flagged and double-submissions decide once; return-for-correction loops are bounded.

**Business.** Every governed process across the platform is fair (version-pinned), safe (no silent auto-approval), controlled (SoD), and reliable (no orphans) — the trustworthy substrate all module approvals run on.

---

## 22. Future Enhancements

Visual workflow/FSM designer with simulation; SLA-driven predictive escalation; workload-balanced approver assignment; conditional-routing rule builder; mobile approvals with biometric step-up; process-mining analytics; cross-workflow dependency orchestration; configurable delegation policies.
