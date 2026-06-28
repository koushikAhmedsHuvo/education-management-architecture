# 02 — Authorization Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint (three-layer access, D35), Business Rules Catalog (`AUTHZ-001…009`), and Use Case Repository (`UC-AUTHZ-001…020`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Decide what an authenticated identity may do: evaluate permissions deny-by-default, enforce a mandatory scope filter, apply ownership/relationship predicates, and govern role/permission/membership administration with separation of duties.

**Business Goal.** Guarantee that no user ever sees or acts on data outside their authorized scope and relationships — the structural safeguard for minors' data and multi-institute isolation — while letting administrators manage access safely.

**Scope.** Runtime authorization (permission + scope + ownership); effective-permission resolution; scope switching; role definition (system + custom); membership grants/revocations; permission registry integrity; SoD constraints; privilege-escalation prevention; permissions-version cache invalidation; access-matrix reporting/audit.

**Out of Scope.** Identity/credential verification (Authentication). Data values of domain entities (owning modules). Workflow execution mechanics (Workflow Engine — invoked for high-impact change approvals). Per-field masking rules (owning modules/Reporting apply them; this module authorizes access).

---

## 2. Actors

**Primary Actors.** Security/Access Administrator, Org/Institute/Campus Administrator (scoped role assignment), System (runtime checks, cache invalidation).

**Secondary Actors.** Every other module (calls the authorization check), Authentication (supplies identity, active scope, permissions-version), Workflow Engine (approves high-impact changes), Audit, Configuration Engine (permissions-version, SoD config).

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Deny-by-default check | Every protected operation requires an explicit grant; absence = deny. | Critical | Permission registry |
| FR-002 | Mandatory scope filter | Inject an unbypassable scope predicate into every data access. | Critical | Institute/Campus scope, Session |
| FR-003 | Ownership/relationship predicates | Enforce owner/teacher-of/guardian-of/manager-of relationships in the data layer. | Critical | Domain relationships |
| FR-004 | Deny overrides | An explicit deny always beats any allow. | High | FR-001 |
| FR-005 | Effective-permission resolution | Resolve a user's net permissions for an active scope (roles + memberships − denies). | High | FR-001..004 |
| FR-006 | Scope switching | Let multi-scope users switch active scope; re-resolve permissions. | High | Authentication (claims) |
| FR-007 | Role management | Define/clone/update/deprecate system and custom roles from registry permissions. | High | Permission registry |
| FR-008 | Membership management | Grant/revoke (user, scope, role) within the granter's authority. | High | FR-002, AUTHZ-007 |
| FR-009 | No privilege escalation | Prevent granting permissions/scope the granter does not themselves hold. | Critical | FR-008 |
| FR-010 | Permission registry integrity | Only registered permissions are referenceable; no orphan/undeclared permissions. | High | Configuration Engine |
| FR-011 | SoD constraints | Define and enforce incompatible-duty pairs (requester ≠ approver, etc.). | Critical | Workflow Engine |
| FR-012 | Permissions-version invalidation | Bump a permissions-version on change to invalidate caches/stale tokens fast. | High | Cache (Redis), Authentication |
| FR-013 | High-impact change approval | Route sensitive role/membership changes through governed approval. | High | Workflow Engine |
| FR-014 | Access-matrix reporting/audit | Report "who can do what where" and audit all changes/escalation attempts. | Medium | Reporting, Audit |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Three-layer access control | Permission + scope + ownership enforced together in the data layer. | Structural data isolation; minor-data safety. |
| Effective-permission resolver | Single source of truth for a user's net rights per scope. | Predictable, debuggable access. |
| Custom roles | Clone/compose roles from registered permissions. | Per-deployment org models without code. |
| Scoped assignment authority | Granters can only assign within their own scope/rights. | Prevents privilege creep. |
| SoD engine | Incompatible-duty enforcement. | Fraud/abuse prevention; audit compliance. |
| Escalation prevention | Hard block on granting beyond one's own rights. | Closes the classic admin-escalation hole. |
| Fast revocation propagation | Permissions-version invalidation. | Access changes take effect immediately. |
| Access matrix & SoD reports | Visibility for audits/reviews. | Governance and certification readiness. |

---

## 5. Screens

Role List; Role Detail; Role Create/Clone; Role Edit; Membership List (per user / per scope); Grant Membership; My Access / Scope Switcher; Permission Registry (admin); SoD Constraint Configuration; "Who Has Permission X" Explorer; Access Matrix Report; Access Change Approvals (inbox); Access Audit Export.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Role List | Create Role, Clone, Open, Deprecate, Search | Bulk Deprecate |
| Role Detail/Edit | Add/Remove Permission, Save (versioned), View Usage, Deprecate | — |
| Membership List | Grant Membership, Revoke, Filter by scope/role | Bulk Revoke |
| Grant Membership | Select User/Scope/Role, Validate Authority, Submit (→ approval if high-impact) | Bulk Assign |
| My Access / Scope Switcher | Switch Scope, View Effective Permissions | — |
| Permission Registry | Register, Deprecate, View References | — |
| SoD Configuration | Add Constraint, Edit, Disable, Save | — |
| Who-Has-Permission Explorer | Search permission, List holders by scope, Export | Export |
| Access Matrix Report | Run, Filter, Export | Export |
| Access Change Approvals | Approve, Reject, Return, Comment | — |

---

## 7. Forms

**Role Create/Clone** — `name` (text, required, unique within scope-level), `description` (text), `basis` (clone-from select, optional), `permissions` (multi-select from registry, required ≥1), `scopeLevels` (multi-select, required). Validation: name unique; all permissions registered (AUTHZ-008); creator may only include permissions they hold (AUTHZ-009/005).

**Grant Membership** — `user` (search-select, required), `scope` (scope picker, required), `role` (select, required). Defaults: effective-now. Validation: granter authorized for the scope (AUTHZ-007); no escalation beyond granter's rights (AUTHZ-005); SoD not violated (AUTHZ-009); high-impact → approval (UC-AUTHZ-010).

**SoD Constraint** — `dutyA` (permission/role, required), `dutyB` (permission/role, required), `type` (mutually-exclusive / requester≠approver), `scope` (optional). Validation: A≠B; non-contradictory with existing constraints.

**Permission Registry Entry** — `key` (text, required, unique, namespaced e.g. `student.view`), `description` (text, required), `category` (select). Validation: unique key; immutable once referenced (deprecate-not-delete).

---

## 8. Search & Filter Requirements

**Roles:** by name, scope-level, system/custom, status (active/deprecated). **Memberships:** by user, scope, role, granted-by, date. **Permissions:** by key, category, "held by user/role". **Who-has-X:** input permission → filter by scope. Sorting: name/date/scope. Pagination: server-side, 25 default.

---

## 9. Table Requirements

**Roles:** Name, Type (system/custom), #Permissions, Scope Levels, Status, Updated. **Memberships:** User, Scope, Role, Granted By, Granted At, Status; bulk revoke. **Access Matrix:** Subject (user/role) × Permission × Scope; exportable (governed, audited — UC-AUTHZ-015). All tables scope-filtered (a viewer sees only memberships/roles within their authority).

---

## 10. Workflow Requirements

**Trigger events:** role create/update/deprecate, membership grant/revoke, SoD config change, escalation attempt, scope switch. **Status changes:** Role `ACTIVE → DEPRECATED`; Membership `PENDING(→approval) → ACTIVE → REVOKED`. **Approvals:** high-impact role/membership changes via Workflow Engine (SoD enforced). **Notifications:** membership granted/revoked, approval requests/decisions, escalation-blocked alerts to security. **Audit:** all changes + every denied escalation/SoD violation (immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| View roles/permissions/memberships | `authz.role.view`, `authz.membership.view` |
| Create/update/deprecate role | `authz.role.manage` |
| Grant/revoke membership | `authz.membership.grant`, `authz.membership.revoke` |
| Approve high-impact change | `authz.change.approve` |
| Manage permission registry | `authz.registry.manage` |
| Configure SoD | `authz.sod.manage` |
| Export access matrix/audit | `authz.matrix.export` |

All capabilities are themselves scope-bound; granting authority never exceeds the granter's own rights (AUTHZ-005/007).

---

## 12. Business Rule References

AUTHZ-001 (deny-by-default), AUTHZ-002 (mandatory scope filter), AUTHZ-003 (ownership/relationship predicates), AUTHZ-004 (deny overrides), AUTHZ-005 (no escalation via role/membership mgmt), AUTHZ-006 (permissions-version cache invalidation), AUTHZ-007 (scoped role assignment authority), AUTHZ-008 (permission registry integrity), AUTHZ-009 (SoD hooks). Cross-cutting: INST-006/CAMP-004 (scope sources), CFG-001/004 (registry/version), WFL-004 (approver resolution), AUD-001.

## 13. Use Case References

UC-AUTHZ-001 (Authorize Operation), UC-AUTHZ-002 (Switch Scope), UC-AUTHZ-003 (Resolve Effective Permissions), UC-AUTHZ-004/006/007 (Role create/update/deprecate), UC-AUTHZ-008/009 (Membership grant/revoke), UC-AUTHZ-010 (Approve High-Impact Change), UC-AUTHZ-011 (Who-has-permission search), UC-AUTHZ-012 (Access Matrix & SoD Report), UC-AUTHZ-005 (View Roles/Permissions), UC-AUTHZ-013 (Bulk Assignment), UC-AUTHZ-014 (Import Role/Permission Sets), UC-AUTHZ-015 (Export Access Matrix), UC-AUTHZ-016 (Role-Change Approval Workflow), UC-AUTHZ-017 (Permission Registry), UC-AUTHZ-018 (Configure SoD), UC-AUTHZ-019 (Escalation Blocked), UC-AUTHZ-020 (SoD Violation Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Authorize an operation (internal check) | POST (internal) | System (any module) |
| Resolve effective permissions for scope | GET | End User / Admin |
| Switch active scope | POST | Multi-scope User |
| List/create/update/deprecate role | GET/POST/PUT/DELETE | Admin |
| List/grant/revoke membership | GET/POST/DELETE | Admin |
| Approve/return/reject access change | POST | Approver |
| List/register/deprecate permission | GET/POST/DELETE | Security Admin |
| Get/set SoD constraints | GET/PUT | Security Admin |
| Who-has-permission search | GET | Admin/Auditor |
| Access-matrix report / export | GET | Admin/Auditor |

The internal authorize check is the hot path used by all modules; deny-by-default, scope, and ownership are evaluated in the data layer (D35).

---

## 15. Database Requirements

**Entities:** `Permission` (registry), `Role` + `RolePermission`, `Membership` (user×scope×role), `ScopeNode` (institute/campus/…, referenced), `SoDConstraint`, `DenyRule`, `PermissionsVersion` (counter per user/tenant). **Relationships:** Role *—* Permission; User 1—* Membership *—1 Role; Membership *—1 ScopeNode; SoDConstraint references permissions/roles. **Indexes:** unique(Permission.key), index(Membership.userId, scopeId, status), index(Membership.scopeId, roleId), unique(Role.name, scopeLevel), index(DenyRule.subject). Effective-permission resolution cached in Redis keyed by (userId, scopeId, permissionsVersion).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Membership granted/revoked; high-impact change approval request/decision. |
| In-App | Effective-access change; approval inbox items; escalation-blocked notice. |
| SMS/Push | Not used by default (security alerts ride Authentication/Notification). |

Access-change notices are mandatory-category where they affect a user's own privileges.

---

## 17. Audit Requirements

Log: role create/update/deprecate (before/after permission sets), membership grant/revoke (user, scope, role, granted-by), SoD constraint changes, permission registry changes, **every denied privilege-escalation and SoD violation** (UC-AUTHZ-019/020), scope switches for sensitive contexts. Record who/when/before/after. Escalation/SoD denials are first-class audit events (security-relevant). Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Access Matrix (subject × permission × scope), SoD compliance (violations/exceptions), Role usage, Orphaned/over-privileged accounts, Escalation-attempt log. **Exports:** governed access-matrix/audit export (signed URL, audited). **Dashboards:** access-governance overview (custom-role count, pending approvals, recent escalation attempts).

---

## 19. Error Handling

**Validation:** unregistered permission, duplicate role name, contradictory SoD → specific errors. **Permission:** action beyond authority → 403; out-of-scope target → 403/not-found. **Workflow:** high-impact change pending → clear pending state; approver = requester → SoD block. **System:** registry/cache unavailable → fail closed (deny); permissions-version mismatch → re-resolve before allowing sensitive ops.

---

## 20. Edge Cases

**Concurrent updates:** simultaneous role edits → versioned, last-coherent wins; concurrent grant + revoke → revoke wins (deny overrides). **Duplicate data:** duplicate membership → idempotent (one active). **Partial failures:** grant approved but cache bump fails → re-resolve forces source read; never grants stale-wider access. **Rollback:** failed bulk assignment → per-row report, no partial silent grants. **Escalation race:** granter's own rights reduced mid-grant → re-checked at commit (cannot grant what they no longer hold).

---

## 21. Acceptance Criteria

**Functional.** Any operation lacking an explicit grant is denied; every data read/write carries the scope filter and ownership predicates; a deny always overrides an allow; a user cannot grant a permission or scope they do not hold; SoD pairs block self-approval/incompatible duties; permission changes invalidate caches immediately; high-impact changes require distinct-approver sign-off.

**Business.** Cross-institute and out-of-relationship data is unreachable by construction (not by UI hiding); access governance is reportable and auditable end to end; administrators can model their org with custom roles without code changes and without ever escalating privileges.

---

## 22. Future Enhancements

Attribute/condition-based rules (ABAC) layered on RBAC; time-bound/just-in-time memberships with auto-expiry; access certification campaigns (periodic recertification); delegated administration templates; permission-usage analytics to prune over-grants; policy simulation ("what would change if…").
