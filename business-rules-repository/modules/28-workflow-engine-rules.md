# 28 — Workflow Engine Business Rules

## 1. Module Purpose
Govern the **Workflow Engine** — the platform capability that runs every approval, review, and multi-step process in the system. Wherever another module says *approval*, *escalation*, *delegation*, *timeout*, or *separation of duties*, it resolves here. The engine is **generic and declarative**: domain modules define their workflows as configuration (states, transitions, approvers, conditions); the engine executes them as **version-pinned finite state machines** with dynamic approver resolution, escalation, delegation, timeouts, parallel/sequential/conditional routing, and complete audit. Its defining guarantees are **fairness** (in-flight instances are immune to mid-process definition changes), **safety** (timeouts never silently auto-approve sensitive actions), and **reliability** (no instance is ever orphaned).

## 2. Actors
- **Initiator / Requester** — starts a workflow instance (e.g., submits a leave request, a discount, an admission decision).
- **Approver / Reviewer** — acts on a pending step (approve/reject/return).
- **Delegate** — temporarily holds an approver's authority.
- **Workflow Administrator** — defines/versions workflow definitions (a platform/governance role).
- **Consuming Modules** — declare workflows, initiate instances, and react to outcomes.
- **System** — executes the FSM, resolves approvers, enforces SoD, escalates/delegates/times out, and audits.

## 3. Use Cases

**Use Case ID:** UC-WFL-01 — Define & Version a Workflow
**Actors:** Workflow Administrator, System
**Description:** Declare a workflow as states, transitions, approver rules, and timeouts.
**Preconditions:** Actor holds `workflow.definition.manage`.
**Main Flow:** 1) Define states, allowed transitions, per-step approver resolution rules, SoD constraints, routing conditions, escalation/delegation/timeout policies. 2) System validates the FSM (reachable, terminating, no forbidden cycles). 3) Publish an immutable version with an effective date.
**Alternative Flow:** A1) Parallel/committee step with quorum policy (WFL-008).
**Exception Flow:** E1) Unreachable/non-terminating FSM → reject (WFL-003). E2) Timeout set to auto-approve a sensitive step → blocked/flagged (WFL-007).
**Post Conditions:** Workflow version available; audited.
**Business Rules Applied:** WFL-001, WFL-003, WFL-007, WFL-008.

**Use Case ID:** UC-WFL-02 — Initiate & Progress an Instance
**Actors:** Initiator, Approver, System
**Description:** Run a process instance to a terminal outcome.
**Preconditions:** A published workflow version; valid initiation context.
**Main Flow:** 1) Initiator starts an instance; the engine **pins the current definition version** (WFL-002). 2) The engine resolves the first step's approver(s) with SoD, notifies them, starts timers. 3) Approver acts; the engine transitions per the FSM. 4) Repeat until a terminal state (approved/rejected/withdrawn).
**Alternative Flow:** A1) Return-for-correction loops back to the initiator (bounded, WFL-011). A2) Conditional routing branches on data (WFL-008).
**Exception Flow:** E1) Step times out → escalate per policy (WFL-005). E2) Approver away → delegate (WFL-006). E3) Approver resolves to requester → SoD skips/blocks (WFL-004).
**Post Conditions:** Instance reaches a terminal state; outcome returned to the module; fully audited.
**Business Rules Applied:** WFL-002, WFL-003, WFL-004, WFL-005, WFL-006, WFL-010.

**Use Case ID:** UC-WFL-03 — Escalate, Delegate, Reassign, or Withdraw
**Actors:** Approver, Administrator, Initiator, System
**Description:** Handle exceptions so instances never stall.
**Preconditions:** An instance is pending.
**Main Flow:** 1) On timeout, escalate to the next target. 2) An away approver delegates; an admin may reassign a stuck step. 3) The initiator may withdraw before a terminal decision; an admin may cancel with reason.
**Exception Flow:** E1) No escalation target resolvable → flag/hold (never silent auto-approve, WFL-007). E2) Delegate also unavailable → chain/escalate (bounded).
**Post Conditions:** Instance progresses or is safely flagged; audited.
**Business Rules Applied:** WFL-005, WFL-006, WFL-009, WFL-010.

## 4. Business Rules

**Rule ID:** WFL-001
**Rule Name:** Declarative Workflow Definitions
**Description:** Workflows are configured definitions (states, transitions, approver rules, conditions), never hard-coded process logic.
**Priority:** Critical
**Category:** Governance / flexibility
**Preconditions:** Workflow use.
**Business Rule:** Each workflow is a registered definition describing its FSM and policies; domain modules reference a workflow by key. Process behavior changes are configuration (via the Configuration Engine patterns), not code changes; the engine has no domain-specific branching.
**System Action:** Store/validate definitions; execute generically from them.
**Validation:** Definition complete and structurally valid.
**Failure Behavior:** Reject malformed definitions; reject references to unknown workflows.
**Audit Requirement:** Log `WORKFLOW_DEFINED/EDITED`.
**Example Scenario:** Admission approval steps are changed by editing the workflow definition, not the codebase.
**Related Rules:** CFG-001, WFL-003.

**Rule ID:** WFL-002
**Rule Name:** Version-Pinning at Initiation (Fairness)
**Description:** An instance pins the workflow definition version at initiation; later definition changes never alter in-flight instances.
**Priority:** Critical
**Category:** Fairness / integrity
**Preconditions:** Instance initiation.
**Business Rule:** The version in effect when an instance starts governs it to completion; a mid-process policy change applies only to instances started afterward. This is the fairness guarantee admission and other decisions rely on.
**System Action:** Snapshot/pin the version on initiation; execute against the pinned version.
**Validation:** Pinned version recorded; immutable for the instance.
**Failure Behavior:** Never re-bind a running instance to a new version.
**Audit Requirement:** Instance records its pinned workflow version.
**Example Scenario:** Applications submitted under last month's approval rules complete under those rules even after the workflow changes.
**Related Rules:** ADM-003, CFG-004.

**Rule ID:** WFL-003
**Rule Name:** Deterministic FSM Execution
**Description:** Instances move only via defined transitions; invalid state jumps are impossible.
**Priority:** Critical
**Category:** Integrity
**Preconditions:** A transition is attempted.
**Business Rule:** Only transitions declared in the (pinned) definition are allowed; the FSM is validated at definition time to be reachable and terminating (every instance can reach a terminal state) with only bounded, intentional loops. No out-of-band or arbitrary state changes.
**System Action:** Enforce declared transitions; reject undefined ones.
**Validation:** Transition exists in the pinned FSM; FSM reachable/terminating.
**Failure Behavior:** Reject invalid transitions; never strand an instance in a dead state.
**Audit Requirement:** Log every `WORKFLOW_TRANSITION` with from/to, actor.
**Example Scenario:** An approval cannot jump from "submitted" straight to "disbursed" without the defined steps.
**Related Rules:** WFL-001, WFL-010, WFL-011.

**Rule ID:** WFL-004
**Rule Name:** Dynamic Approver Resolution with Separation of Duties
**Description:** Approvers are resolved at runtime from roles/reporting-line/scope, and the requester can never approve their own instance.
**Priority:** Critical
**Category:** Authorization / control
**Preconditions:** A step needs an approver.
**Business Rule:** Approver(s) resolve dynamically (by role, reporting manager STF-006, scope, or amount tier); SoD constraints (AUTHZ-009) ensure requester ≠ approver and any declared incompatible-duty pairs are enforced; if the only eligible approver is the requester, the engine routes to the next eligible authority or blocks — never self-approval.
**System Action:** Resolve eligible approvers; apply SoD; route accordingly.
**Validation:** At least one eligible, SoD-compliant approver resolvable (else escalate/flag).
**Failure Behavior:** Block self-approval; if no eligible approver, escalate/flag (WFL-007), never auto-approve.
**Audit Requirement:** Log resolved approvers and SoD exclusions.
**Example Scenario:** A discount requested by an accountant routes to a different authorized approver, never back to the requester.
**Related Rules:** AUTHZ-009, STF-006, WFL-005.

**Rule ID:** WFL-005
**Rule Name:** Escalation on Timeout
**Description:** A step that stalls beyond its SLA escalates to a defined target so nothing waits indefinitely.
**Priority:** High
**Category:** Reliability
**Preconditions:** A pending step exceeds its configured time.
**Business Rule:** Each step has a timeout/SLA; on breach, the engine escalates to the next-level approver/role (or a defined fallback) and notifies; escalation is bounded and recorded. The default for sensitive steps is escalate/hold, not auto-decide.
**System Action:** Track SLA; escalate on breach; notify; record.
**Validation:** Escalation target resolvable; SLA defined.
**Failure Behavior:** If no escalation target, flag/hold and alert (WFL-007).
**Audit Requirement:** Log `WORKFLOW_ESCALATED` with reason/target.
**Example Scenario:** A leave request unactioned for two days escalates to the next manager.
**Related Rules:** LEV-005, WFL-006, WFL-007.

**Rule ID:** WFL-006
**Rule Name:** Bounded, Audited Delegation
**Description:** An approver may delegate their authority (e.g., while away); delegation is bounded, attributed, and revocable.
**Priority:** High
**Category:** Reliability / accountability
**Preconditions:** An approver delegates.
**Business Rule:** Delegation transfers approval authority for a defined scope/period to an eligible delegate; the delegate's decisions are attributed as acting-for, within the same SoD constraints; delegation chains are bounded to prevent loops; it is revocable and auto-expires.
**System Action:** Apply time-boxed delegation; attribute decisions; bound chains; auto-expire.
**Validation:** Delegate eligible and SoD-compliant; period defined; chain bounded.
**Failure Behavior:** Reject ineligible/looping delegation; escalate if no valid delegate.
**Audit Requirement:** Log `WORKFLOW_DELEGATED` and acting-for decisions.
**Example Scenario:** A manager on leave delegates approvals to a deputy for a week; decisions are recorded as acting-for the manager.
**Related Rules:** WFL-004, LEV-005, STF-006.

**Rule ID:** WFL-007
**Rule Name:** Safe Timeout Policies — No Silent Auto-Approval of Sensitive Actions
**Description:** Timeout behavior is configurable but defaults to safe handling; sensitive steps never silently auto-approve.
**Priority:** Critical
**Category:** Safety
**Preconditions:** A timeout policy applies.
**Business Rule:** Timeout actions (escalate, hold, auto-reject, or — only for explicitly low-risk steps — auto-advance) are configurable, but sensitive/financial/academic-integrity steps are barred from auto-approval; the safe default is escalate or hold-and-flag. A definition attempting auto-approve on a sensitive step is rejected at definition time.
**System Action:** Apply the timeout action; forbid auto-approve on sensitive steps.
**Validation:** Sensitivity classification honored; auto-approve disallowed where unsafe.
**Failure Behavior:** Default to escalate/hold; reject unsafe auto-approve definitions.
**Audit Requirement:** Log timeout actions and any auto-advance (low-risk only).
**Example Scenario:** A stalled result-revision approval escalates and holds; it never auto-approves itself.
**Related Rules:** WFL-005, RES-006, DSC-003.

**Rule ID:** WFL-008
**Rule Name:** Sequential, Parallel & Conditional Routing
**Description:** Workflows support multi-step sequences, parallel/committee steps with quorum, and data-conditional branching.
**Priority:** High
**Category:** Expressiveness / integrity
**Preconditions:** A multi-step or branching workflow.
**Business Rule:** Definitions may chain steps (sequential), require multiple approvers in parallel with a declared quorum (e.g., majority/unanimous for a committee, SCH-003), and branch on instance data (e.g., amount thresholds route to higher authority). Quorum and branch rules are explicit and deterministic.
**System Action:** Execute sequence/parallel/quorum/branch per the definition.
**Validation:** Quorum/branch rules defined; deterministic.
**Failure Behavior:** Block ambiguous routing; resolve branches deterministically.
**Audit Requirement:** Log parallel decisions and the branch taken.
**Example Scenario:** A large scholarship needs a 3-of-5 committee quorum; a small discount needs one approver.
**Related Rules:** SCH-003, DSC-003, WFL-003.

**Rule ID:** WFL-009
**Rule Name:** Withdrawal, Cancellation & Reassignment
**Description:** Instances can be withdrawn by the initiator, cancelled by an admin, or reassigned when stuck — all governed.
**Priority:** Medium
**Category:** Lifecycle
**Preconditions:** A pending instance.
**Business Rule:** The initiator may withdraw before a terminal decision; an administrator may cancel (with reason) or reassign a stuck step to another eligible approver; all such actions are governed, attributed, and move the instance to a clear state.
**System Action:** Apply withdrawal/cancellation/reassignment; record reason/actor.
**Validation:** Action permitted for the instance state; target eligible (reassignment).
**Failure Behavior:** Reject withdrawal/cancellation after a terminal decision.
**Audit Requirement:** Log `WORKFLOW_WITHDRAWN/CANCELLED/REASSIGNED` with reason.
**Example Scenario:** A guardian withdraws an admission application before the decision; the instance closes cleanly.
**Related Rules:** ADM-008, WFL-010.

**Rule ID:** WFL-010
**Rule Name:** Reliability, Idempotency & No Orphaned Instances
**Description:** Workflow execution is reliable and idempotent; every instance reaches a terminal state or is safely flagged — never lost or stalled forever.
**Priority:** Critical
**Category:** Reliability
**Preconditions:** Instance execution.
**Business Rule:** Transitions are durably recorded (no lost actions); duplicate action submissions are idempotent (a double-click/retry decides once); orphan detection flags instances stuck beyond limits for intervention; the engine recovers cleanly from failures without skipping steps or double-deciding.
**System Action:** Durable transitions; idempotent actions; orphan monitoring; safe recovery.
**Validation:** Idempotency keys on actions; terminal-reachability guaranteed.
**Failure Behavior:** Stuck instances flagged/escalated; never silently dropped or double-decided.
**Audit Requirement:** Log orphan flags and recovery actions.
**Example Scenario:** A double-submitted approval click records one decision; an instance stuck for a week is flagged for an admin.
**Related Rules:** WFL-003, WFL-005, PAY-008 (idempotency parallel).

**Rule ID:** WFL-011
**Rule Name:** Bounded Return-for-Correction Loops
**Description:** Send-back loops (e.g., return an application for correction) are supported but bounded to prevent infinite ping-pong.
**Priority:** Medium
**Category:** Integrity
**Preconditions:** A workflow with a return/rework path.
**Business Rule:** Return-for-correction routes back to the initiator for fixes and resubmission; the number of rounds is bounded (configurable) and tracked; exceeding the bound escalates or terminates per policy. The pinned version persists across rounds (or re-pins per explicit policy).
**System Action:** Track correction rounds; bound them; escalate/terminate on exceed.
**Validation:** Round limit defined; loop bounded.
**Failure Behavior:** Block infinite loops; escalate on bound breach.
**Audit Requirement:** Log each `WORKFLOW_RETURNED` and round count.
**Example Scenario:** An application returned for corrections three times is escalated rather than looping forever.
**Related Rules:** ADM (RETURNED_FOR_CORRECTION), WFL-003.

## 5. Validation Rules
- Workflows are declarative, registered, and structurally valid (reachable, terminating).
- Instances pin the definition version at initiation; immune to later changes.
- Only declared transitions execute; no invalid jumps.
- Approvers resolve dynamically with SoD; never self-approval.
- Escalation/delegation/timeout configured; sensitive steps never auto-approve.
- Parallel quorum/branch rules deterministic; loops bounded; actions idempotent; no orphans.

## 6. State Machine
*(Generic workflow-instance lifecycle; domain states are defined per workflow.)*

**State Name:** INITIATED
**Description:** Instance created; definition version pinned.
**Allowed Transitions:** → IN_PROGRESS; → WITHDRAWN (initiator); → CANCELLED (admin).
**Forbidden Transitions:** terminal decision without steps.
**System Actions:** Pin version; resolve first approver; start SLA.

**State Name:** IN_PROGRESS (step: PENDING / ESCALATED)
**Description:** Awaiting decisions; steps may escalate/delegate.
**Allowed Transitions:** → IN_PROGRESS (next step); → RETURNED (correction); → APPROVED; → REJECTED; → CANCELLED.
**Forbidden Transitions:** invalid/out-of-band transitions.
**System Actions:** Resolve approvers (SoD); escalate/delegate/time out; record decisions.

**State Name:** RETURNED
**Description:** Sent back to the initiator for correction (bounded).
**Allowed Transitions:** → IN_PROGRESS (resubmitted); → WITHDRAWN; → terminated on bound breach.
**Forbidden Transitions:** unbounded looping.
**System Actions:** Track rounds; enforce the loop bound.

**State Name:** APPROVED / REJECTED
**Description:** Terminal decision reached.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** re-deciding a terminal instance.
**System Actions:** Return outcome to the consuming module; lock.

**State Name:** WITHDRAWN / CANCELLED
**Description:** Ended before decision (initiator/admin), with reason.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** reactivation (a new instance is created).
**System Actions:** Record reason; close cleanly.

**State Name:** ARCHIVED
**Description:** Historical workflow instance, immutable.
**Allowed Transitions:** none.
**Forbidden Transitions:** edits.
**System Actions:** Retain for audit.

## 7. Status Definitions
Instance: `INITIATED` · `IN_PROGRESS` · `RETURNED` · `APPROVED` · `REJECTED` · `WITHDRAWN` · `CANCELLED` · `ARCHIVED`. Step: `PENDING` · `ESCALATED` · `DELEGATED` · `DECIDED`. Definition: `DRAFT` · `PUBLISHED` (versioned) · `SUPERSEDED` · `ARCHIVED`.

## 8. Workflow Rules
*(Meta — the engine's own governance.)*
- Workflow definitions are versioned; high-impact definition changes may require approval (CFG-007 pattern).
- Instances always run on their pinned version (WFL-002).
- Escalation/delegation/timeout are mandatory safety nets; sensitive steps cannot auto-approve (WFL-007).
- Return loops are bounded (WFL-011); orphans are monitored (WFL-010).

## 9. Permission Rules
- `workflow.definition.manage` — define/version workflows (governance; tightly held).
- `workflow.instance.initiate` — start instances (often implicit in domain actions).
- `workflow.act` — approve/reject/return a pending step (resolved per approver rules).
- `workflow.delegate` — delegate one's own approval authority.
- `workflow.admin` — cancel/reassign stuck instances (elevated).
- `workflow.view` — read instance status/history (scoped; initiators see their own).

## 10. Notification Rules
- Pending-task notifications to resolved approvers (and on escalation, to the escalation target).
- `WORKFLOW_DECISION` (approved/rejected/returned) → notify the initiator.
- Delegation/reassignment → notify the new actor.
- Orphan/stuck-instance alerts → notify administrators.
- The engine is the source of process notifications; domain modules supply context.

## 11. Audit Requirements
Mandatory: `WORKFLOW_DEFINED/EDITED`, instance `INITIATED` (pinned version), every `WORKFLOW_TRANSITION` (from/to, actor), approver resolution + SoD exclusions, `WORKFLOW_ESCALATED/DELEGATED/RETURNED/WITHDRAWN/CANCELLED/REASSIGNED`, timeout actions, orphan flags. Every decision is attributable, with the pinned version, for fairness and dispute defense.

## 12. Data Retention Rules
- Workflow instances and their full transition history retained per the related domain's retention (e.g., admission/financial instances follow those retentions).
- Definitions and versions retained so any historical instance is reconstructable to the rules that governed it.
- Audit of decisions retained long-term (who approved what, when, under which version).
- Anonymization follows the related records' erasure, preserving non-identifying process integrity.

## 13. Edge Cases
- **Mid-process definition change:** the instance keeps its pinned version (WFL-002) — fairness preserved.
- **Approver vacant/role empty:** escalate/flag, never silent auto-approve (WFL-004/WFL-007).
- **Approver = requester:** SoD routes elsewhere or blocks (WFL-004).
- **Approver suspended/leaves mid-instance:** delegate/reassign/escalate; nothing stalls (WFL-006/WFL-009).
- **Delegate also unavailable:** bounded chain then escalate (WFL-006).
- **Committee partial quorum then a reject:** resolved by the declared quorum policy (any-reject vs majority) (WFL-008).
- **Timeout with no escalation target:** hold-and-flag, never auto-decide sensitive (WFL-007).
- **Double-click/retry on a decision:** idempotent; decided once (WFL-010).
- **Endless return-for-correction:** bounded; escalates on breach (WFL-011).
- **Instance stuck due to a crash:** orphan detection flags it; safe recovery without double-deciding (WFL-010).

## 14. Failure Scenarios
- **No eligible approver:** escalate/flag; never auto-approve (WFL-004/WFL-007).
- **Invalid transition attempt:** rejected (WFL-003).
- **Unsafe auto-approve definition:** rejected at definition time (WFL-007).
- **Lost/duplicate action:** prevented by durability + idempotency (WFL-010).
- **Re-binding a running instance to a new version:** forbidden (WFL-002).
- **Unbounded loop:** prevented (WFL-011).

## 15. Exception Handling Rules
- Instances always reach a terminal state or are flagged; nothing is silently dropped or stalled.
- Sensitive steps never auto-approve; the safe default is escalate/hold.
- SoD is enforced at approver resolution; self-approval is impossible.
- Actions are idempotent and durable; recovery never double-decides.

## 16. Compliance Considerations
- **Fairness:** version-pinning ensures consistent rules within a process lifecycle (admissions, awards) — defensible against bias/appeals.
- **Controls & SoD:** enforced separation of duties and governed approvals support financial and academic-integrity audits.
- **Accountability:** complete, attributable decision trails with pinned versions support disputes and regulator inquiries.
- **Safety:** the no-silent-auto-approval rule protects sensitive financial/academic actions.

## 17. Future Considerations
- Visual workflow designer for administrators (with structural validation).
- SLA dashboards and bottleneck analytics across processes.
- Event-/time-triggered workflows (e.g., auto-initiate renewal reviews).
- Cross-module orchestrations (sagas) for multi-domain processes with compensation.
