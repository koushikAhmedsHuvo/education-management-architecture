# 28 — Workflow Engine Use Cases

Transforms the Workflow Engine business rules (`WFL-001`…`WFL-011`) into use cases. The generic, declarative engine that runs every approval/review/multi-step process: version-pinned FSMs, dynamic approver resolution with SoD, escalation/delegation/timeout, sequential/parallel/conditional routing, withdrawal/cancellation/reassignment, idempotent reliability with no orphaned instances, and bounded return-for-correction loops.

## 1. Primary Actors
Initiator / Requester (starts an instance), Approver / Reviewer (acts on a step), Delegate (temporary authority), Workflow Administrator (defines/versions workflows).

## 2. Secondary Actors
Consuming Modules (declare workflows, initiate instances, react to outcomes), Staff hierarchy (approver resolution), Configuration Engine (workflow definitions/versioning), Notification & Audit services.

## 3. Goals
Run domain processes generically from declarative definitions; pin the definition version at initiation (fairness); execute deterministic FSMs; resolve approvers dynamically with SoD; escalate/delegate/time-out so nothing stalls; never silently auto-approve sensitive actions; support sequential/parallel/conditional routing; allow governed withdrawal/cancellation/reassignment; guarantee reliability/idempotency with no orphaned instances; bound return-for-correction loops.

## 4. User Journeys
- **Define:** a workflow admin declares a workflow as states, transitions, approver rules, SoD, routing, and escalation/delegation/timeout policies → published as an immutable version.
- **Run:** a module initiates an instance → the engine pins the definition version → resolves approvers (SoD) → notifies → progresses through the FSM to a terminal outcome.
- **Exception-handle:** a step times out → escalates; an approver is away → delegates; a stuck instance → reassigned; an initiator → withdraws; a request → returned for correction (bounded).
- **Guarantee:** every instance reaches a terminal state or is flagged; double-submissions decide once; sensitive steps never auto-approve.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-WFL-001 | Define / Version a Workflow | High |
| Core | UC-WFL-002 | Initiate Instance (version-pinned) | Critical |
| Core | UC-WFL-003 | Progress Instance (deterministic FSM) | Critical |
| Core | UC-WFL-004 | Resolve Approver with SoD | Critical |
| Core | UC-WFL-005 | Escalate on Timeout | High |
| Core | UC-WFL-006 | Delegate Authority (bounded) | High |
| Core | UC-WFL-007 | Safe Timeout (no silent auto-approve) | Critical |
| Core | UC-WFL-008 | Sequential / Parallel / Conditional Routing | High |
| Core | UC-WFL-009 | Withdraw / Cancel / Reassign | Medium |
| Core | UC-WFL-010 | Return-for-Correction (bounded loop) | Medium |
| CRUD | UC-WFL-011 | View Instance / Task (scoped) | Medium |
| Admin | UC-WFL-012 | Manage Pending Tasks (inbox) | High |
| Search | UC-WFL-013 | Search Instances / Tasks | Low |
| Reporting | UC-WFL-014 | SLA / Bottleneck & Audit Report | Medium |
| Bulk | UC-WFL-015 | Bulk Approve / Act (where permitted) | Medium |
| Workflow | UC-WFL-016 | Orphan Detection & Recovery | High |
| Exception | UC-WFL-017 | Self-Approval (SoD) Blocked | Critical |
| Exception | UC-WFL-018 | Invalid Transition Blocked | High |
| Exception | UC-WFL-019 | Unsafe Auto-Approve Definition Blocked | Critical |

---

## 6. Detailed Specifications (high-value use cases)

### UC-WFL-002 — Initiate Instance (version-pinned)
- **Module:** Workflow Engine · **Priority:** Critical
- **Actors:** Initiator / Consuming Module (primary), System
- **Goal:** Start a process instance that runs on the definition version in force at initiation (fairness).
- **Description:** On initiation, the engine pins the workflow definition version so later definition changes never alter this in-flight instance; resolves the first approver and starts timers.
- **Business Rules Applied:** WFL-002, WFL-001, WFL-003, WFL-004.
- **Preconditions:** A published workflow version; valid initiation context from the module.
- **Trigger:** A module initiates a process (e.g., admission decision, leave request, discount approval).
- **Main Success Scenario:**
  1. A module initiates an instance with context.
  2. The engine pins the current definition version to the instance (WFL-002).
  3. The engine resolves the first step's approver(s) with SoD (WFL-004) and starts SLA timers.
  4. The instance enters `IN_PROGRESS`; approvers are notified.
- **Alternative Flows:** A1) Conditional first step branches on context (WFL-008).
- **Exception Flows:** E1) No eligible approver → escalate/flag (WFL-007). E2) Definition changed after initiation → instance keeps its pinned version (WFL-002).
- **Validation Rules:** Version pinned; first approver resolvable (SoD); FSM valid (WFL-002/003/004).
- **Permissions Required:** `workflow.instance.initiate` (often implicit in the domain action).
- **Notifications Triggered:** Pending-task notice to resolved approvers.
- **Audit Events Generated:** `WORKFLOW_INITIATED` (pinned version).
- **Data Created:** Workflow instance.
- **Data Updated:** Approver assignment; timers.
- **Data Deleted:** None.
- **Post Conditions:** Instance running on its pinned version; first step pending.
- **Related Use Cases:** UC-WFL-003, UC-WFL-004; invoked by ADM/LEV/DSC/SCH/RES/HR/CFG.
- **Acceptance Criteria:**
  - Given a workflow at version v2, When an instance is initiated, Then it pins v2 and runs on v2 to completion.
  - Given the definition changes to v3 mid-flight, When this instance progresses, Then it still uses v2.
  - Given initiation, When the first step has no eligible approver, Then it escalates/flags (never auto-approves).
- **Edge Case Analysis:**
  - *Invalid Input:* initiation without required context rejected.
  - *Permission Failure:* unauthorized initiation → 403.
  - *Concurrent Update:* definition edit during initiation → instance pins the version at its initiation instant.
  - *Duplicate Data:* duplicate initiation (idempotency key) → one instance.
  - *System Failure:* atomic creation with audit (outbox).
  - *Workflow Failure:* no approver → escalate/flag.
- **QA Coverage:**
  - *Positive:* initiate; pinned-version isolation.
  - *Negative:* mid-flight definition change (must not affect); duplicate initiation.
  - *Boundary:* initiation exactly at a version publish boundary.

### UC-WFL-004 — Resolve Approver with SoD
- **Module:** Workflow Engine · **Priority:** Critical
- **Actors:** System (primary), Approver
- **Goal:** Resolve the eligible approver(s) for a step at runtime, enforcing separation of duties.
- **Description:** Approvers resolve dynamically from roles/reporting-line/scope/amount-tier; SoD ensures the requester (and any incompatible-duty party) cannot approve; if the only eligible approver is the requester, route to the next authority or block — never self-approval.
- **Business Rules Applied:** WFL-004, AUTHZ-009, STF-006.
- **Preconditions:** A step requiring approval; approver rules defined.
- **Trigger:** A step becomes pending.
- **Main Success Scenario:**
  1. The engine resolves eligible approvers (by role, reporting manager STF-006, scope, or amount tier).
  2. The engine applies SoD: requester ≠ approver; incompatible-duty pairs excluded (AUTHZ-009).
  3. The engine routes the task to the resolved approver(s) and notifies them.
- **Alternative Flows:** A1) Amount-tiered routing (larger amounts → higher authority). A2) Committee/parallel approvers (WFL-008).
- **Exception Flows:** E1) Only eligible approver is the requester → route to next authority or block (UC-WFL-017). E2) No eligible approver → escalate/flag (WFL-007).
- **Validation Rules:** ≥1 SoD-compliant approver resolvable (else escalate/flag); requester excluded (WFL-004, AUTHZ-009).
- **Permissions Required:** Resolved approver holds `workflow.act`.
- **Notifications Triggered:** Pending-task notice.
- **Audit Events Generated:** Approver resolution + SoD exclusions recorded.
- **Data Created:** Task assignment.
- **Data Updated:** Step approver.
- **Data Deleted:** None.
- **Post Conditions:** Task routed to an SoD-compliant approver, or escalated/flagged.
- **Related Use Cases:** UC-WFL-005, UC-WFL-017; STF-006 hierarchy.
- **Acceptance Criteria:**
  - Given a request, When resolving the approver, Then the requester is excluded (SoD).
  - Given the requester is the only eligible approver, When resolving, Then it routes to the next authority or blocks (never self-approval).
  - Given an amount tier, When resolving, Then the appropriate authority level is selected.
- **Edge Case Analysis:**
  - *Invalid Input:* malformed approver rule rejected at definition time.
  - *Permission Failure:* resolved approver lacking `workflow.act` → re-resolve/escalate.
  - *Concurrent Update:* hierarchy change mid-resolution → uses current hierarchy.
  - *Duplicate Data:* idempotent resolution.
  - *System Failure:* resolution failure → fail closed (escalate/flag), never auto-approve.
  - *Workflow Failure:* vacant role → escalation.
- **QA Coverage:**
  - *Positive:* role/hierarchy/tier resolution.
  - *Negative:* self-approval; incompatible-duty; no eligible approver.
  - *Boundary:* amount exactly at a tier threshold.

### UC-WFL-007 — Safe Timeout (no silent auto-approve)
- **Module:** Workflow Engine · **Priority:** Critical
- **Actors:** System (primary), Administrator
- **Goal:** Apply timeout behavior safely — sensitive steps never silently auto-approve.
- **Description:** Timeout actions (escalate, hold, auto-reject, or — only for explicitly low-risk steps — auto-advance) are configurable, but sensitive/financial/academic-integrity steps are barred from auto-approval; the safe default is escalate or hold-and-flag. A definition attempting auto-approve on a sensitive step is rejected at definition time.
- **Business Rules Applied:** WFL-007, WFL-005, WFL-001.
- **Preconditions:** A timeout policy applies to a pending step.
- **Trigger:** A step exceeds its SLA.
- **Main Success Scenario:**
  1. On SLA breach, the engine applies the configured timeout action.
  2. For sensitive steps, auto-approval is disallowed; the engine escalates or holds-and-flags (WFL-007).
  3. The action is recorded; relevant parties are notified.
- **Alternative Flows:** A1) Low-risk step → configured auto-advance permitted.
- **Exception Flows:** E1) Definition attempts auto-approve on a sensitive step → rejected at definition time (UC-WFL-019). E2) No escalation target → hold-and-flag.
- **Validation Rules:** Sensitivity classification honored; auto-approve disallowed where unsafe; safe default escalate/hold (WFL-007).
- **Permissions Required:** System; `workflow.definition.manage` to set policies.
- **Notifications Triggered:** Escalation/hold notices.
- **Audit Events Generated:** Timeout action recorded; any auto-advance (low-risk only).
- **Data Created:** None.
- **Data Updated:** Step state.
- **Data Deleted:** None.
- **Post Conditions:** Safe outcome on timeout; sensitive steps never silently approved.
- **Related Use Cases:** UC-WFL-005, UC-WFL-019.
- **Acceptance Criteria:**
  - Given a sensitive step timing out, When the SLA is breached, Then it escalates or holds (never auto-approves).
  - Given a low-risk step configured to auto-advance, When it times out, Then it advances as configured.
  - Given a definition attempting auto-approve on a sensitive step, When published, Then it is rejected.
- **Edge Case Analysis:**
  - *Invalid Input:* unsafe timeout config rejected at definition.
  - *Permission Failure:* policy change without permission → 403.
  - *Concurrent Update:* late action + timeout → first valid decision wins.
  - *Duplicate Data:* idempotent timeout handling.
  - *System Failure:* default to escalate/hold on uncertainty.
  - *Workflow Failure:* no target → hold-and-flag.
- **QA Coverage:**
  - *Positive:* escalate-on-timeout; low-risk auto-advance.
  - *Negative:* sensitive auto-approve (must be impossible); silent approval.
  - *Boundary:* timeout exactly at SLA; no-target hold.

### UC-WFL-010 — Return-for-Correction (bounded loop)
- **Module:** Workflow Engine · **Priority:** Medium
- **Actors:** Approver (primary), Initiator, System
- **Goal:** Send a request back for correction and resubmission, bounded to prevent infinite ping-pong.
- **Description:** A return routes the instance back to the initiator for fixes; rounds are bounded (configurable) and tracked; exceeding the bound escalates or terminates per policy; the pinned version persists across rounds.
- **Business Rules Applied:** WFL-011, WFL-003, WFL-002.
- **Preconditions:** A workflow with a return/rework path; an in-progress instance.
- **Trigger:** An approver returns the request for correction.
- **Main Success Scenario:**
  1. Approver returns the instance with required corrections.
  2. The engine routes it back to the initiator (round counter increments, WFL-011).
  3. The initiator corrects and resubmits; the instance re-enters review on its pinned version.
  4. On exceeding the round bound, the engine escalates or terminates per policy.
- **Alternative Flows:** A1) Initiator withdraws instead of resubmitting (UC-WFL-009).
- **Exception Flows:** E1) Round bound exceeded → escalate/terminate (no infinite loop).
- **Validation Rules:** Rounds bounded and tracked; pinned version persists; loop terminates (WFL-011).
- **Permissions Required:** `workflow.act` (return) / initiator (resubmit).
- **Notifications Triggered:** `WORKFLOW_RETURNED` to initiator; resubmission notice to approver.
- **Audit Events Generated:** `WORKFLOW_RETURNED` (round count).
- **Data Created:** None.
- **Data Updated:** Round counter; instance state.
- **Data Deleted:** None.
- **Post Conditions:** Correction loop progresses within bounds; resolves or escalates.
- **Related Use Cases:** UC-WFL-003, UC-WFL-009, UC-ADM-017.
- **Acceptance Criteria:**
  - Given a return, When routed, Then it goes back to the initiator with the round counter incremented.
  - Given the round bound is exceeded, When reached, Then the instance escalates or terminates (no infinite loop).
  - Given a resubmission, When re-entering review, Then the pinned version still governs.
- **Edge Case Analysis:**
  - *Invalid Input:* return without required corrections (if mandated) blocked.
  - *Permission Failure:* non-approver returning → 403.
  - *Concurrent Update:* return + withdraw racing → terminal action wins.
  - *Duplicate Data:* idempotent round increment.
  - *System Failure:* reliable round tracking.
  - *Workflow Failure:* bound exceeded → escalate/terminate.
- **QA Coverage:**
  - *Positive:* return-correct-resubmit; resolve within bound.
  - *Negative:* infinite loop (must not occur).
  - *Boundary:* exactly at the round bound (escalation triggers).

---

## 7. Compact Specifications (routine use cases)

- **UC-WFL-001 — Define / Version a Workflow** · *High* · Rules: WFL-001, WFL-003, WFL-007, CFG-004. Declare states/transitions/approver-rules/SoD/routing/timeouts; validate FSM (reachable, terminating); publish immutable version. *Permissions:* `workflow.definition.manage`. *Audit:* `WORKFLOW_DEFINED`. *Edge:* unsafe auto-approve rejected (UC-WFL-019); non-terminating FSM rejected. *QA:* valid define; bad FSM; unsafe timeout.
- **UC-WFL-003 — Progress Instance (deterministic FSM)** · *Critical* · Rules: WFL-003, WFL-002. Move via declared transitions only; no invalid jumps. *Audit:* `WORKFLOW_TRANSITION` (from/to, actor). *Edge:* invalid transition blocked (UC-WFL-018); only pinned-FSM transitions. *QA:* valid transitions; invalid jump blocked.
- **UC-WFL-005 — Escalate on Timeout** · *High* · Rules: WFL-005, WFL-007. Escalate a stalled step to the next target; bounded; never silent. *Audit:* `WORKFLOW_ESCALATED`. *Edge:* no target → hold-and-flag. *QA:* escalate; bounded; no-target hold.
- **UC-WFL-006 — Delegate Authority (bounded)** · *High* · Rules: WFL-006, AUTHZ-009. Time-boxed, attributed delegation; chains bounded; auto-expires; SoD-compliant. *Audit:* `WORKFLOW_DELEGATED`. *Edge:* ineligible/looping delegate blocked. *QA:* delegate; acting-for attribution; bounded chain.
- **UC-WFL-008 — Sequential / Parallel / Conditional Routing** · *High* · Rules: WFL-008, WFL-003. Chain steps; parallel/committee with quorum; data-conditional branching (deterministic). *Edge:* quorum/branch explicit; no ambiguous routing. *QA:* sequence; quorum; branch determinism.
- **UC-WFL-009 — Withdraw / Cancel / Reassign** · *Medium* · Rules: WFL-009. Initiator withdraws (pre-decision); admin cancels (reason) or reassigns a stuck step. *Audit:* `WORKFLOW_WITHDRAWN/CANCELLED/REASSIGNED`. *Edge:* withdrawal after terminal decision blocked. *QA:* withdraw; cancel; reassign; post-terminal blocked.
- **UC-WFL-011 — View Instance / Task (scoped)** · *Medium* · Rules: AUTHZ-003. Initiators see own; approvers see assigned; admins scoped. *QA:* scope; own-instance visibility.
- **UC-WFL-012 — Manage Pending Tasks (inbox)** · *High* · Rules: WFL-004, AUTHZ-003. Approver's task inbox (assigned/escalated/delegated tasks). *Edge:* shows only authorized tasks; SLA indicators. *QA:* inbox correctness; scope.
- **UC-WFL-013 — Search Instances / Tasks** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-WFL-014 — SLA / Bottleneck & Audit Report** · *Medium* · Rules: REP-002, AUD. SLA performance, bottlenecks, decision audit. *Edge:* scoped; correlated end-to-end. *QA:* SLA accuracy; bottleneck detection.
- **UC-WFL-015 — Bulk Approve / Act (where permitted)** · *Medium* · Rules: WFL-004, AUTHZ-009. Bulk-act on permitted low-risk tasks (SoD per task). *Edge:* sensitive tasks excluded from bulk; per-task SoD. *QA:* bulk low-risk; sensitive excluded.
- **UC-WFL-016 — Orphan Detection & Recovery** · *High* · Rules: WFL-010. Detect instances stuck beyond limits; flag/escalate; idempotent recovery (no double-decide). *Audit:* orphan flags/recovery. *Edge:* crash-stuck instance flagged; recovery never double-decides. *QA:* orphan flag; idempotent recovery.
- **UC-WFL-017 — Self-Approval (SoD) Blocked (Exception)** · *Critical* · Rules: WFL-004, AUTHZ-009. Requester cannot approve own instance. *QA:* self-approval blocked; distinct approver allowed.
- **UC-WFL-018 — Invalid Transition Blocked (Exception)** · *High* · Rules: WFL-003. Undeclared/out-of-band transitions rejected. *QA:* invalid jump blocked; declared transition allowed.
- **UC-WFL-019 — Unsafe Auto-Approve Definition Blocked (Exception)** · *Critical* · Rules: WFL-007. Auto-approve on a sensitive step rejected at definition time. *QA:* unsafe definition rejected; safe definition allowed.

## 8. Module-level QA & Edge Themes
- **Fairness — version-pinning (WFL-002):** in-flight instances immune to mid-process definition changes — the headline fairness suite (admissions, awards).
- **Safety — no silent auto-approve (WFL-007 / WFL-019):** sensitive/financial/academic steps never auto-approve; unsafe definitions rejected at design time.
- **Control — SoD (WFL-004 / WFL-017):** requester ≠ approver, enforced at approver resolution; the machinery behind every domain SoD (P9).
- **Reliability — no orphans + idempotency (WFL-010 / WFL-016):** every instance terminal or flagged; double-submissions decide once.
- **Determinism (WFL-003 / WFL-008):** only declared transitions; deterministic quorum/branch routing; bounded loops (WFL-011).
