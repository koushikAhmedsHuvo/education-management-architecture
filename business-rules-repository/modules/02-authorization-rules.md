# 02 — Authorization Business Rules

## 1. Module Purpose
Decide **what an authenticated identity may do, and on which records**. Authorization answers three independent questions for every protected operation: **(a) permission** — may this action be performed at all? **(b) scope** — within which institute/campus/session? **(c) ownership** — which specific rows? An operation is allowed only when all three pass (three-layer model, D35). Authorization is **deny-by-default**; the backend is the sole authority (UI gating is convenience only).

## 2. Actors
- **End User** — the subject whose permissions are evaluated.
- **Role/Access Administrator** — manages roles, permissions, and memberships within their scope.
- **System** — resolves and caches effective permissions, enforces scope and ownership in the data layer.

## 3. Use Cases

**Use Case ID:** UC-AUTHZ-01 — Authorize an Operation
**Actors:** End User, System
**Description:** Evaluate a request against permission, scope, and ownership.
**Preconditions:** Authenticated identity with an active scope selected.
**Main Flow:** 1) Resolve the user's effective permissions for the active scope (cached by version). 2) Check the required permission. 3) Apply the mandatory institute/campus scope filter. 4) Apply ownership predicates for the resource. 5) Allow only if all pass.
**Alternative Flow:** A1) Org-wide role → scope filter spans all institutes.
**Exception Flow:** E1) Missing permission → 403. E2) Out-of-scope → 403/empty. E3) Not owner → 403/filtered out.
**Post Conditions:** Operation proceeds or is denied; denial of a sensitive resource is audited.
**Business Rules Applied:** AUTHZ-001, AUTHZ-002, AUTHZ-003, AUTHZ-004.

**Use Case ID:** UC-AUTHZ-02 — Create a Custom Role
**Actors:** Access Administrator, System
**Description:** An admin composes a new role from registered permissions.
**Preconditions:** Admin holds `authz.role.manage` in the target scope.
**Main Flow:** 1) Admin selects permissions from the registry. 2) System enforces that every selected permission is one the admin **themselves holds** in that scope (no escalation). 3) Role saved as data, scoped, versioned.
**Exception Flow:** E1) A selected permission exceeds the admin's own grant → reject the whole operation (AUTHZ-005). E2) Unregistered permission string → reject.
**Post Conditions:** Role exists; permissions-version bumped; event audited.
**Business Rules Applied:** AUTHZ-005, AUTHZ-007, AUTHZ-008.

**Use Case ID:** UC-AUTHZ-03 — Switch Active Scope
**Actors:** End User, System
**Description:** A user who is a member of multiple institutes switches their active institute/campus.
**Preconditions:** User holds memberships in more than one scope.
**Main Flow:** 1) User selects a new active scope. 2) System verifies a membership grants access there. 3) System re-resolves effective permissions for the new scope.
**Exception Flow:** E1) No membership for the requested scope → deny.
**Post Conditions:** Active scope updated; subsequent requests evaluated against it.
**Business Rules Applied:** AUTHZ-002, AUTHZ-006.

## 4. Business Rules

**Rule ID:** AUTHZ-001
**Rule Name:** Deny-by-Default Permission Check
**Description:** Every protected operation requires an explicit permission; absence means denial.
**Priority:** Critical
**Category:** Access control
**Preconditions:** Authenticated request to a protected operation.
**Business Rule:** Permissions are fine-grained strings `module.resource.action`. A user's effective permissions for a scope are the union of their roles' permissions in that scope. No permission → deny.
**System Action:** Resolve effective permissions (cached by permissions-version); check the required permission.
**Validation:** Required permission exists in the registry; effective-permission cache is current for the user's permissions-version.
**Failure Behavior:** 403 Forbidden; sensitive denials audited.
**Audit Requirement:** Log `ACCESS_DENIED` for sensitive resources with required permission and reason.
**Example Scenario:** A teacher without `finance.invoice.view` cannot open fee invoices.
**Related Rules:** AUTHZ-006, AUTHZ-008.

**Rule ID:** AUTHZ-002
**Rule Name:** Mandatory Scope Filter
**Description:** Every scoped query is filtered by the active institute (and campus where applicable) from request context — applied in the data layer, not per-controller.
**Priority:** Critical
**Category:** Multi-institute isolation
**Preconditions:** A scoped resource is accessed.
**Business Rule:** A user scoped to Institute A can never read or write Institute B's rows, even by crafting the request. The scope filter is applied centrally and cannot be omitted by a forgetful query.
**System Action:** Inject the active scope into every repository operation; reject or empty out-of-scope access.
**Validation:** Resource carries `institute_id` (and `campus_id`/`session_id` where relevant); active scope is established and authorized by a membership.
**Failure Behavior:** Out-of-scope access returns 403 or an empty/filtered result; attempts on sensitive data are audited.
**Audit Requirement:** Log `SCOPE_VIOLATION_ATTEMPT` when an explicit out-of-scope identifier is supplied.
**Example Scenario:** An institute admin of School A requesting a student id belonging to School B gets nothing.
**Related Rules:** AUTHZ-003, INST-006, CAMP-004.

**Rule ID:** AUTHZ-003
**Rule Name:** Ownership / Relationship Predicates
**Description:** Within a permitted scope, access is further limited to the rows a user owns or relates to.
**Priority:** Critical
**Category:** Row-level access
**Preconditions:** Permission and scope already satisfied.
**Business Rule:** Role-specific ownership applies: a teacher accesses only their assigned classes/sections/subjects; a student only their own record; a parent only their linked children; an accountant only their permitted financial scope. Enforced in the data layer.
**System Action:** Apply ownership predicates derived from assignments/links; filter accordingly.
**Validation:** Ownership source-of-truth (e.g., staff assignments, guardian links) is current.
**Failure Behavior:** Non-owned rows are filtered out (not merely hidden in UI).
**Audit Requirement:** Log `ACCESS_DENIED` (ownership) for sensitive cross-owner attempts.
**Example Scenario:** A teacher cannot view marks for a class they do not teach, even within their own institute.
**Related Rules:** AUTHZ-002, HR (assignment), STU (self), Doc 09 (guardian links).

**Rule ID:** AUTHZ-004
**Rule Name:** Deny Overrides
**Description:** If any of the three layers denies, the operation is denied — there is no "allow" that overrides a "deny."
**Priority:** Critical
**Category:** Evaluation policy
**Preconditions:** Multi-layer evaluation.
**Business Rule:** Permission-grant does not override a scope or ownership denial, and vice versa. All three must independently pass.
**System Action:** Short-circuit to deny on the first failing layer.
**Validation:** Each layer evaluated independently; no implicit elevation.
**Failure Behavior:** Deny; reason recorded as the first failing layer.
**Audit Requirement:** Sensitive denials audited with the failing layer.
**Example Scenario:** A user with `student.record.view` but scoped to the wrong institute is still denied.
**Related Rules:** AUTHZ-001, AUTHZ-002, AUTHZ-003.

**Rule ID:** AUTHZ-005
**Rule Name:** No Privilege Escalation via Role/Membership Management
**Description:** An administrator cannot grant a permission, role, or scope that exceeds their own authority.
**Priority:** Critical
**Category:** Escalation prevention
**Preconditions:** An admin creates/edits a role or assigns a membership.
**Business Rule:** The set of permissions an admin may place in a role, or assign to a user, is bounded by the permissions the admin themselves holds **in that scope**. An institute admin cannot mint org-level permissions; no one can self-elevate.
**System Action:** Validate the requested grant ⊆ the actor's effective grant in scope; reject otherwise.
**Validation:** Compare requested permission set against the actor's effective set; verify scope boundary.
**Failure Behavior:** Reject the entire operation (not a silent partial grant); audit the attempt.
**Audit Requirement:** Log `PRIVILEGE_ESCALATION_BLOCKED` with actor, requested grant, scope.
**Example Scenario:** An institute admin tries to create a role with `org.deployment.configure`; the system rejects it because they lack that permission.
**Related Rules:** AUTHZ-007, AUTHZ-008.

**Rule ID:** AUTHZ-006
**Rule Name:** Permissions-Version Cache Invalidation
**Description:** Permission/role/membership changes propagate quickly via a version stamp, not at token expiry.
**Priority:** High
**Category:** Consistency
**Preconditions:** A role's permissions, a user's roles, or a membership changes.
**Business Rule:** Each change bumps the affected users' permissions-version; cached effective-permissions for that version are invalidated, so the next request resolves fresh.
**System Action:** Increment version; invalidate cache; access tokens carry the version for fast comparison.
**Validation:** Version monotonic per user/scope; cache keyed by version.
**Failure Behavior:** On cache miss/uncertainty, resolve from source (fail safe toward correctness).
**Audit Requirement:** Log `PERMISSIONS_VERSION_BUMPED` with cause.
**Example Scenario:** Removing `result.publish` from a role takes effect on the user's next request, not 15 minutes later.
**Related Rules:** AUTHZ-001, AUTH-001.

**Rule ID:** AUTHZ-007
**Rule Name:** Scoped Role Assignment Authority
**Description:** Assigning a role to a user requires role-management permission **in the scope of the assignment**, and the user gains the role only within that scope.
**Priority:** High
**Category:** Membership management
**Preconditions:** Admin assigns a (user, institute/campus, role) membership.
**Business Rule:** The same person can be Institute Admin in A and Teacher in B; each membership is independently scoped. The assigner must hold `authz.role.manage` in the assignment's scope.
**System Action:** Create the scoped membership; resolve permissions per scope thereafter.
**Validation:** Assigner authority in scope; role exists and is `ACTIVE`; target user exists.
**Failure Behavior:** Reject out-of-scope or unauthorized assignments; audit.
**Audit Requirement:** Log `MEMBERSHIP_GRANTED/REVOKED` with user, scope, role, actor.
**Example Scenario:** A campus admin assigns a teacher role at their campus but cannot assign roles at another campus.
**Related Rules:** AUTHZ-005, CAMP-003.

**Rule ID:** AUTHZ-008
**Rule Name:** Permission Registry Integrity
**Description:** Only registered permissions are enforceable; custom roles cannot invent permissions the code does not check.
**Priority:** High
**Category:** Governance
**Preconditions:** Role definition references permissions.
**Business Rule:** Every permission used in a role must exist in the permission registry (the catalog of code-enforced permissions). New capabilities register their permission before use.
**System Action:** Validate role permissions against the registry on save.
**Validation:** Each permission string ∈ registry.
**Failure Behavior:** Reject roles referencing unregistered permissions.
**Audit Requirement:** Log registry changes (`PERMISSION_REGISTERED/DEPRECATED`).
**Example Scenario:** A role cannot grant `library.book.issue` until that permission is registered by the library module.
**Related Rules:** AUTHZ-001, CFG (future) registry management.

**Rule ID:** AUTHZ-009
**Rule Name:** Separation of Duties (SoD) Hooks
**Description:** Certain action pairs must not be performable by the same user on the same object (e.g., requesting and approving the same waiver).
**Priority:** Medium
**Category:** Controls / fraud prevention
**Preconditions:** A defined SoD constraint exists for an action pair.
**Business Rule:** Where an SoD constraint is configured, the system prevents the same actor from performing both sides on the same instance (the requester cannot be the approver). SoD constraints are configurable per client.
**System Action:** Evaluate SoD at the point of the second action; block if violated.
**Validation:** SoD pairs defined in configuration; actor identity compared against the first action's actor.
**Failure Behavior:** Block the second action with an SoD violation; audit.
**Audit Requirement:** Log `SOD_VIOLATION_BLOCKED` with actor, action pair, object.
**Example Scenario:** The accountant who requests a fee waiver cannot also approve it when SoD is enabled for waivers.
**Related Rules:** WFL (workflow approver resolution), FEE/DSC (waivers).

## 5. Validation Rules
- Required permission must exist in the registry before enforcement.
- Active scope must be backed by a current membership for the user.
- Role permission sets validated ⊆ assigner's grant on create/edit (anti-escalation).
- Ownership predicates derive from current source-of-truth (assignments, guardian links); stale sources must not silently widen access.
- Deny-by-default: any unmatched operation is denied.

## 6. State Machine

**State Name:** ROLE_DRAFT
**Description:** Role being defined, not yet assignable.
**Allowed Transitions:** → ROLE_ACTIVE (published).
**Forbidden Transitions:** → ROLE_DEPRECATED (cannot deprecate an unpublished role).
**System Actions:** Validate permissions ⊆ author's grant and ∈ registry.

**State Name:** ROLE_ACTIVE
**Description:** Assignable role.
**Allowed Transitions:** → ROLE_DEPRECATED (retired); edits create a new version.
**Forbidden Transitions:** → ROLE_DRAFT.
**System Actions:** Bump permissions-version of affected users on edit.

**State Name:** ROLE_DEPRECATED
**Description:** No longer assignable; existing assignments handled per policy.
**Allowed Transitions:** → ROLE_ARCHIVED.
**Forbidden Transitions:** → ROLE_ACTIVE (a new role/version is created instead).
**System Actions:** Block new assignments; optionally migrate or revoke existing.

**State Name:** MEMBERSHIP_ACTIVE / MEMBERSHIP_SUSPENDED / MEMBERSHIP_REVOKED
**Description:** Lifecycle of a (user, scope, role) link.
**Allowed Transitions:** ACTIVE → SUSPENDED → ACTIVE; ACTIVE/SUSPENDED → REVOKED (terminal).
**Forbidden Transitions:** REVOKED → ACTIVE (re-grant creates a new membership).
**System Actions:** Re-resolve permissions on any transition; bump version.

## 7. Status Definitions
Roles: `DRAFT` / `ACTIVE` / `DEPRECATED` / `ARCHIVED`. Memberships: `ACTIVE` / `SUSPENDED` / `REVOKED`. Permission registry entries: `REGISTERED` / `DEPRECATED`.

## 8. Workflow Rules
- Role creation/edit may optionally require approval (configurable) for high-impact roles (those bearing critical permissions like `result.publish` or `finance.*`).
- Membership grants of elevated roles may require approval via the Workflow Engine (Doc 28) when configured.
- SoD constraints are enforced inline (not a workflow) at the point of the conflicting action.

## 9. Permission Rules
- `authz.role.manage` — create/edit/deprecate roles within scope (bounded by AUTHZ-005).
- `authz.permission.assign` — assign permissions to roles within scope.
- `authz.membership.manage` — grant/revoke memberships within scope.
- `authz.policy.view` — read the access model (auditors).
- Reading the authorization model is itself permissioned; not everyone can enumerate roles/permissions.

## 10. Notification Rules
- `MEMBERSHIP_GRANTED` of an elevated role → notify the affected user and the granting admin.
- `MEMBERSHIP_REVOKED` → notify the affected user.
- `PRIVILEGE_ESCALATION_BLOCKED` and `SOD_VIOLATION_BLOCKED` → notify a security/admin channel.
- Role changes affecting critical permissions → notify org admins (configurable).

## 11. Audit Requirements
Mandatory: `ROLE_CREATED/EDITED/DEPRECATED`, `PERMISSION_REGISTERED/DEPRECATED`, `MEMBERSHIP_GRANTED/SUSPENDED/REVOKED`, `PERMISSIONS_VERSION_BUMPED`, `ACCESS_DENIED` (sensitive), `SCOPE_VIOLATION_ATTEMPT`, `PRIVILEGE_ESCALATION_BLOCKED`, `SOD_VIOLATION_BLOCKED`. Each with actor, target, scope, before/after where applicable.

## 12. Data Retention Rules
- Roles/permissions/memberships are configuration-like; full version history retained (who could do what, when) for the audit window — critical for "who had access on date X" inquiries.
- Revoked memberships retained (not hard-deleted) for historical access reconstruction.
- Authorization audit follows the long, immutable audit retention class.

## 13. Edge Cases
- **Conflicting roles across scopes:** resolved per scope independently; no global merge (a user is Admin in A, Teacher in B simultaneously).
- **Permission removed from a role while users hold it:** version bump invalidates caches; access narrows on next request.
- **Ownership after reassignment:** when a teacher is unassigned from a class, future access ends immediately; whether they retain read access to *historical* data they created is a configurable policy decision (default: no live access, audit trail preserves their past actions).
- **Orphaned memberships when an institute is archived:** memberships scoped only to an archived institute become inert (no live access) but are retained for history (see INST-008).
- **Super/org admin breadth:** org-wide roles span all institutes; this breadth is itself a high-value target and must be tightly held and audited.
- **Self-management loophole:** a user must not be able to grant themselves a role or permission, even with role-management rights (AUTHZ-005 blocks self-escalation).
- **Stale ownership source:** if a guardian link or staff assignment is revoked, dependent access must re-resolve, not linger from cache.

## 14. Failure Scenarios
- **Cache unavailable:** resolve permissions from source (correctness over speed); never default-allow.
- **Registry/source inconsistency:** unknown permission → treat as not-granted (deny).
- **Scope context missing:** if the active scope cannot be established, deny scoped operations.
- **Partial membership update:** transactional; never leave a user with half-applied access.

## 15. Exception Handling Rules
- Authorization failures are explicit denials, never silent allows.
- A missing or malformed scope context is a denial, not a fallback to "all scopes."
- Escalation and SoD violations reject the **entire** operation, not a partial subset.
- Denials of non-sensitive resources may be quiet (UX), but sensitive denials are always audited.

## 16. Compliance Considerations
- **Least privilege & SoD** support financial and academic-integrity controls expected by auditors.
- **Access reconstruction:** retained role/membership history answers "who could access this minor's data on this date?" — a GDPR/accountability necessity.
- **Minors' data:** access to student records is doubly gated (scope + ownership), reflecting heightened protection.
- **Auditability:** every access-control change is immutable and attributable.

## 17. Future Considerations
- Attribute-based (ABAC) refinements layered onto RBAC for narrow cases without abandoning the RBAC spine.
- Time-bound / just-in-time elevated access (temporary admin) with auto-expiry.
- Access certification campaigns (periodic review/attestation of who holds elevated roles).
- Policy-as-code testing to prove no role grants an unintended sensitive permission.
