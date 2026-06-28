# 02 — Authorization Use Cases

Transforms the Authorization business rules (`AUTHZ-001`…`AUTHZ-009`) into use cases. Decides *what* an identity may do and *on which records* — permission + scope + ownership, deny-by-default.

## 1. Primary Actors
Role/Access Administrator (manages roles, permissions, memberships in scope), End User (whose access is evaluated).

## 2. Secondary Actors
System (resolves/caches effective permissions, enforces scope & ownership), Auditor (reads the access model), Workflow Engine (role/membership approvals).

## 3. Goals
Enforce least-privilege access; prevent privilege escalation and cross-scope leakage; support custom roles from a controlled permission registry; enforce separation of duties; keep access changes fast (version-based) and fully auditable.

## 4. User Journeys
- **Standing up access:** admin composes roles from registered permissions → assigns scoped memberships → users gain exactly the access their role+scope grants.
- **Runtime enforcement:** user acts → engine checks permission → applies institute/campus scope filter → applies ownership predicate → allows only if all three pass.
- **Scope switching:** a multi-institute user selects an active scope → permissions re-resolve for that scope.
- **Governance:** auditor reviews "who can access this minor's data" → access matrix report; SoD and escalation attempts are blocked and surfaced.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-AUTHZ-001 | Authorize an Operation (runtime) | Critical |
| Core | UC-AUTHZ-002 | Switch Active Scope | High |
| Core | UC-AUTHZ-003 | Resolve Effective Permissions | High |
| CRUD | UC-AUTHZ-004 | Create Custom Role | High |
| CRUD | UC-AUTHZ-005 | View Roles / Permissions | Medium |
| CRUD | UC-AUTHZ-006 | Update Role | High |
| CRUD | UC-AUTHZ-007 | Deprecate / Archive Role | Medium |
| CRUD | UC-AUTHZ-008 | Grant Membership (user, scope, role) | High |
| CRUD | UC-AUTHZ-009 | Revoke Membership | High |
| Approval | UC-AUTHZ-010 | Approve High-Impact Role/Membership Change | High |
| Search | UC-AUTHZ-011 | Search Roles / Memberships / "Who has permission X" | Medium |
| Reporting | UC-AUTHZ-012 | Access Matrix & SoD Report | High |
| Bulk | UC-AUTHZ-013 | Bulk Role Assignment | Medium |
| Import | UC-AUTHZ-014 | Import Role/Permission Sets | Low |
| Export | UC-AUTHZ-015 | Export Access Matrix (audit) | Medium |
| Workflow | UC-AUTHZ-016 | Role-Change Approval Workflow | High |
| Admin | UC-AUTHZ-017 | Manage Permission Registry | High |
| Admin | UC-AUTHZ-018 | Configure SoD Constraints | High |
| Exception | UC-AUTHZ-019 | Privilege-Escalation Attempt Blocked | Critical |
| Exception | UC-AUTHZ-020 | SoD Violation Blocked | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-AUTHZ-001 — Authorize an Operation (runtime)
- **Module:** Authorization · **Priority:** Critical
- **Actors:** End User (primary), System (secondary)
- **Goal:** Decide whether a specific operation on specific records is allowed.
- **Description:** The engine evaluates permission, then mandatory scope filter, then ownership predicates; all three must pass (deny overrides).
- **Business Rules Applied:** AUTHZ-001, AUTHZ-002, AUTHZ-003, AUTHZ-004, AUTHZ-006.
- **Preconditions:** Authenticated identity with an active scope selected.
- **Trigger:** Any protected operation/request.
- **Main Success Scenario:**
  1. System resolves the user's effective permissions for the active scope (cached by permissions-version).
  2. System checks the required permission for the operation.
  3. System applies the mandatory institute/campus scope filter to the data.
  4. System applies ownership predicates (teacher→own sections, student→self, guardian→children).
  5. Operation proceeds only if all three layers pass.
- **Alternative Flows:** A1) Org-wide role → scope filter spans all institutes (still one deployment).
- **Exception Flows:** E1) Missing permission → 403. E2) Out of scope → 403/empty. E3) Not owner → filtered out. (Any failing layer denies.)
- **Validation Rules:** Required permission ∈ registry; active scope backed by a membership; ownership source current (AUTHZ-001/002/003).
- **Permissions Required:** The operation's own permission (evaluated here).
- **Notifications Triggered:** None routine; sensitive denials may alert.
- **Audit Events Generated:** `ACCESS_DENIED` (sensitive, with failing layer); `SCOPE_VIOLATION_ATTEMPT` (explicit out-of-scope id).
- **Data Created/Updated/Deleted:** None (evaluation only).
- **Post Conditions:** Operation allowed or denied; sensitive denial audited.
- **Related Use Cases:** UC-AUTHZ-002, UC-AUTHZ-003.
- **Acceptance Criteria:**
  - Given a user lacking the required permission, When they attempt the operation, Then it is denied (403) regardless of scope/ownership.
  - Given a permitted user in the wrong scope, When they attempt access, Then they receive nothing (scope filter).
  - Given a permitted, in-scope user who is not the owner, When they access a record, Then non-owned rows are filtered out.
- **Edge Case Analysis:**
  - *Invalid Input:* unknown permission string → treated as not-granted (deny).
  - *Permission Failure:* the core path; first failing layer denies.
  - *Concurrent Update:* permission change mid-session → next request re-resolves (version bump).
  - *Duplicate Data:* N/A.
  - *System Failure:* cache down → resolve from source (correctness over speed); never default-allow.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* permitted + in-scope + owner succeeds.
  - *Negative:* missing permission; wrong scope; non-owner; deny-overrides combinations.
  - *Boundary:* access immediately after a permission-version bump; org-wide vs institute-scoped boundary.

### UC-AUTHZ-004 — Create Custom Role
- **Module:** Authorization · **Priority:** High
- **Actors:** Access Administrator (primary), System
- **Goal:** Compose a new role from registered permissions without escalating privilege.
- **Description:** Admin selects permissions; the engine enforces that every selected permission is one the admin themselves holds in that scope (no escalation) and is registered.
- **Business Rules Applied:** AUTHZ-005, AUTHZ-007, AUTHZ-008.
- **Preconditions:** Admin holds `authz.role.manage` in the target scope.
- **Trigger:** Admin submits a new role definition.
- **Main Success Scenario:**
  1. Admin selects permissions from the registry and a target scope.
  2. System validates each permission ∈ registry (AUTHZ-008).
  3. System validates the selected set ⊆ the admin's own effective grant in scope (AUTHZ-005).
  4. System saves the role as data, scoped and versioned (`DRAFT`/`ACTIVE`).
- **Alternative Flows:** A1) High-impact role (critical permissions) routes through approval (UC-AUTHZ-010).
- **Exception Flows:** E1) A selected permission exceeds the admin's grant → reject the whole operation (UC-AUTHZ-019). E2) Unregistered permission → reject.
- **Validation Rules:** Permissions registered; requested set ⊆ actor's grant; scope authorized (AUTHZ-005/008).
- **Permissions Required:** `authz.role.manage` (in scope).
- **Notifications Triggered:** On high-impact creation, notify org admins (configurable).
- **Audit Events Generated:** `ROLE_CREATED`; on block `PRIVILEGE_ESCALATION_BLOCKED`.
- **Data Created:** Role definition (versioned).
- **Data Updated:** Permissions-version of affected scope (on assignment later).
- **Data Deleted:** None.
- **Post Conditions:** Role exists, assignable within scope; audited.
- **Related Use Cases:** UC-AUTHZ-006, UC-AUTHZ-008, UC-AUTHZ-019.
- **Acceptance Criteria:**
  - Given an admin, When they create a role using only permissions they hold, Then the role is created.
  - Given an admin, When they include a permission they do not hold, Then the entire operation is rejected and the attempt audited.
  - Given an unregistered permission, When referenced, Then the role is rejected.
- **Edge Case Analysis:**
  - *Invalid Input:* empty permission set → reject; malformed permission string → reject.
  - *Permission Failure:* no `authz.role.manage` → 403.
  - *Concurrent Update:* two admins editing the same role → versioned; last write creates a new version, not silent overwrite.
  - *Duplicate Data:* duplicate role name in scope → reject/version.
  - *System Failure:* registry unavailable → block creation (cannot validate).
  - *Workflow Failure:* approval step fails to resolve approver → role stays `DRAFT`, escalates.
- **QA Coverage:**
  - *Positive:* role from held permissions; high-impact role via approval.
  - *Negative:* escalation attempt; unregistered permission; out-of-scope.
  - *Boundary:* role with exactly the admin's full grant; single-permission role.

### UC-AUTHZ-008 — Grant Membership (user, scope, role)
- **Module:** Authorization · **Priority:** High
- **Actors:** Access Administrator (primary), System
- **Goal:** Give a user a role within a specific institute/campus scope.
- **Description:** Creates a scoped (user, scope, role) membership; the assigner must hold role-management in that scope and cannot exceed their own authority.
- **Business Rules Applied:** AUTHZ-007, AUTHZ-005, AUTHZ-006.
- **Preconditions:** Target user exists; role is `ACTIVE`; assigner holds `authz.membership.manage` in the assignment scope.
- **Trigger:** Admin assigns a membership.
- **Main Success Scenario:**
  1. Admin selects user, scope (institute/campus), and role.
  2. System validates assigner authority in scope and that the role's permissions ⊆ assigner's grant (AUTHZ-005/007).
  3. System creates the scoped membership and bumps the user's permissions-version (AUTHZ-006).
  4. Access takes effect on the user's next request.
- **Alternative Flows:** A1) Elevated role → approval workflow (UC-AUTHZ-010). A2) Same user gets different roles in different scopes (independent memberships).
- **Exception Flows:** E1) Out-of-scope assignment → reject. E2) Escalation via role → block (UC-AUTHZ-019).
- **Validation Rules:** Assigner authority in scope; role `ACTIVE`; no escalation (AUTHZ-005/007).
- **Permissions Required:** `authz.membership.manage` (in scope).
- **Notifications Triggered:** `MEMBERSHIP_GRANTED` to user and granting admin (elevated roles).
- **Audit Events Generated:** `MEMBERSHIP_GRANTED` (user, scope, role, actor); `PERMISSIONS_VERSION_BUMPED`.
- **Data Created:** Membership record.
- **Data Updated:** Permissions-version; cache invalidated.
- **Data Deleted:** None.
- **Post Conditions:** User has the role in scope; effect on next request.
- **Related Use Cases:** UC-AUTHZ-009, UC-AUTHZ-004, UC-AUTHZ-010.
- **Acceptance Criteria:**
  - Given an authorized admin, When they grant a membership in their scope, Then the user gains the role only within that scope.
  - Given an admin, When they attempt to grant a role exceeding their authority, Then it is blocked and audited.
  - Given a granted membership, When the user next requests, Then the new access is in effect (version bump).
- **Edge Case Analysis:**
  - *Invalid Input:* unknown user/role → reject.
  - *Permission Failure:* out-of-scope assigner → 403.
  - *Concurrent Update:* parallel grant + revoke → serialized; final state consistent; version bumped.
  - *Duplicate Data:* duplicate (user,scope,role) → idempotent/no-op.
  - *System Failure:* version/cache failure → resolve from source; never widen access.
  - *Workflow Failure:* approval unresolved → membership pending, escalates.
- **QA Coverage:**
  - *Positive:* scoped grant; multi-scope memberships.
  - *Negative:* out-of-scope; escalation; inactive role.
  - *Boundary:* grant then immediate access (version timing); self-grant attempt blocked.

### UC-AUTHZ-019 — Privilege-Escalation Attempt Blocked (Exception)
- **Module:** Authorization · **Priority:** Critical
- **Actors:** Access Administrator (acting), System
- **Goal:** Prevent any actor from granting access beyond their own authority.
- **Description:** When a role/membership operation would grant a permission/scope the actor lacks, the engine rejects the entire operation and audits it.
- **Business Rules Applied:** AUTHZ-005, AUTHZ-007.
- **Preconditions:** An actor attempts a role/membership change.
- **Trigger:** Requested grant exceeds the actor's effective grant in scope.
- **Main Success Scenario:**
  1. Actor submits a role/membership change including a permission/scope they do not hold.
  2. System compares the requested grant against the actor's effective grant in scope.
  3. System rejects the **entire** operation (not a partial grant).
  4. System audits `PRIVILEGE_ESCALATION_BLOCKED`.
- **Alternative Flows:** None (rejection is terminal).
- **Exception Flows:** N/A (this *is* the exception path).
- **Validation Rules:** Requested grant ⊄ actor grant → block (AUTHZ-005).
- **Permissions Required:** Holds `authz.role.manage`/`membership.manage` but cannot exceed own authority.
- **Notifications Triggered:** Security/admin channel notice.
- **Audit Events Generated:** `PRIVILEGE_ESCALATION_BLOCKED` (actor, requested grant, scope).
- **Data Created/Updated/Deleted:** None (operation rejected).
- **Post Conditions:** No access changed; attempt recorded.
- **Related Use Cases:** UC-AUTHZ-004, UC-AUTHZ-008.
- **Acceptance Criteria:**
  - Given an actor, When they attempt to grant a permission they lack, Then the whole operation is rejected and a security audit event is recorded.
  - Given a partial overlap, When some requested permissions exceed authority, Then nothing is granted (no partial application).
- **Edge Case Analysis:**
  - *Invalid Input:* malformed request → reject.
  - *Permission Failure:* the scenario itself.
  - *Concurrent Update:* actor's own grant changes mid-operation → re-evaluated against current grant.
  - *Duplicate Data:* repeated attempts each audited.
  - *System Failure:* grant resolution failure → fail closed (block).
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive (control):* in-authority grant succeeds.
  - *Negative:* org-level permission by institute admin; self-escalation; partial-overlap rejection.
  - *Boundary:* grant exactly equal to actor's authority (allowed) vs one permission beyond (blocked).

---

## 7. Compact Specifications (routine use cases)

- **UC-AUTHZ-002 — Switch Active Scope** · *High* · Rules: AUTHZ-002, AUTHZ-006. Multi-institute user selects an active scope backed by a membership; permissions re-resolve. *Validation:* membership exists for requested scope. *Audit:* scope switch. *Edge:* switch to unmembered scope denied. *QA:* valid switch; unauthorized scope; re-resolution correctness.
- **UC-AUTHZ-003 — Resolve Effective Permissions** · *High* · Rules: AUTHZ-001, AUTHZ-006. System computes the union of role permissions for the active scope, cached by version. *Edge:* cache miss → source resolution. *QA:* union correctness; version invalidation.
- **UC-AUTHZ-005 — View Roles / Permissions** · *Medium* · Rules: AUTHZ-001. Read the access model (permissioned). *Permissions:* `authz.policy.view`. *Edge:* not everyone may enumerate roles. *QA:* authorized read; unauthorized denied.
- **UC-AUTHZ-006 — Update Role** · *High* · Rules: AUTHZ-005, AUTHZ-006, AUTHZ-008. Edit a role; new version; affected users' permissions-version bumped. *Validation:* no escalation; permissions registered. *Audit:* `ROLE_EDITED`. *Edge:* editing a critical role may require approval; concurrent edits versioned. *QA:* valid edit; escalation blocked; version bump propagation.
- **UC-AUTHZ-007 — Deprecate / Archive Role** · *Medium* · Rules: AUTHZ (role lifecycle). Retire a role; block new assignments; handle existing per policy. *Audit:* `ROLE_DEPRECATED`. *Edge:* cannot deprecate a role with unmigrated assignments without policy. *QA:* deprecate blocks new grants; existing handled.
- **UC-AUTHZ-009 — Revoke Membership** · *High* · Rules: AUTHZ-007, AUTHZ-006. Remove a (user,scope,role); access narrows on next request. *Permissions:* `authz.membership.manage`. *Audit:* `MEMBERSHIP_REVOKED`. *Edge:* revoke retained (not hard-deleted); re-grant is a new membership. *QA:* revoke ends access; out-of-scope revoke denied.
- **UC-AUTHZ-010 — Approve High-Impact Role/Membership Change** · *High* · Rules: AUTHZ-009, WFL-004. Approver (≠ requester) approves elevated role/membership changes via the Workflow Engine. *Permissions:* `authz.*.approve`. *Audit:* approval. *Edge:* SoD blocks self-approval; escalation/timeout. *QA:* approval flow; self-approval blocked; timeout escalation.
- **UC-AUTHZ-011 — Search Roles/Memberships / "Who has permission X"** · *Medium* · Rules: AUTHZ-002/003. Scoped query incl. reverse permission lookup. *Permissions:* `authz.policy.view`. *Edge:* results scope-filtered. *QA:* reverse lookup correctness; scope respected.
- **UC-AUTHZ-012 — Access Matrix & SoD Report** · *High* · Rules: AUTHZ-009, REP-002. Report of who-can-do-what and SoD constraints/violations. *Permissions:* `authz.policy.view`/`report.run`. *Edge:* supports "who could access this minor's data on date X" via retained history. *QA:* matrix accuracy; historical reconstruction.
- **UC-AUTHZ-013 — Bulk Role Assignment** · *Medium* · Rules: AUTHZ-007, AUTHZ-005. Assign a role to many users in scope (each escalation-checked). *Validation:* per-user authority check; partial-failure report. *Edge:* any escalation in the batch is blocked per-row, not all-or-nothing unless configured. *QA:* clean bulk; mixed with escalation; large batch boundary.
- **UC-AUTHZ-014 — Import Role/Permission Sets** · *Low* · Rules: AUTHZ-008. Import role definitions referencing registered permissions. *Validation:* all permissions registered; no escalation beyond importer. *Edge:* unknown permissions rejected. *QA:* valid import; unregistered-permission import rejected.
- **UC-AUTHZ-015 — Export Access Matrix (audit)** · *Medium* · Rules: REP-005, AUD. Governed export of the access model. *Permissions:* `audit.export`. *Audit:* `REPORT_EXPORTED`. *QA:* scoped export; unauthorized denied.
- **UC-AUTHZ-016 — Role-Change Approval Workflow** · *High* · Rules: AUTHZ-009, WFL-002/004. Version-pinned approval for high-impact role changes. *Edge:* mid-flight definition change does not alter in-progress approval. *QA:* version-pinning; SoD; escalation.
- **UC-AUTHZ-017 — Manage Permission Registry** · *High* · Rules: AUTHZ-008, CFG-001. Register/deprecate code-enforced permissions. *Permissions:* platform/`config.definition.manage`. *Audit:* `PERMISSION_REGISTERED/DEPRECATED`. *Edge:* roles cannot use unregistered permissions. *QA:* register enables use; deprecate blocks new use.
- **UC-AUTHZ-018 — Configure SoD Constraints** · *High* · Rules: AUTHZ-009. Define incompatible action pairs (e.g., request≠approve waiver). *Permissions:* `authz.policy.manage`. *Audit:* SoD config change. *Edge:* constraint enforced at the second action. *QA:* configured pair blocks same-actor; non-constrained pairs unaffected.
- **UC-AUTHZ-020 — SoD Violation Blocked (Exception)** · *High* · Rules: AUTHZ-009. Same actor attempting both sides of an SoD pair on one object is blocked. *Audit:* `SOD_VIOLATION_BLOCKED`. *QA:* same-actor blocked; different-actor allowed.

## 8. Module-level QA & Edge Themes
- **Three-layer always:** every access test must cover permission, scope, and ownership independently and in deny-overrides combinations.
- **Escalation & SoD:** dedicated negative suites for AUTHZ-005 and AUTHZ-009 (the highest-value controls).
- **Version propagation:** test that role/membership changes take effect on the next request via permissions-version, not at token expiry.
- **Reporting cannot bypass:** the access matrix report itself respects scope (REP-002).
