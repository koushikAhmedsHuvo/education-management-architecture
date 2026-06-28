# 03 — Institute Business Rules

## 1. Module Purpose
Govern the **institute** — a distinct educational entity (school, college, university, madrasa, coaching center, training institute) operating inside one client deployment. The institute is the **primary scope boundary**: nearly every business object belongs to an institute, and access is gated by it. A deployment hosts one or many institutes; the institution **type** seeds the institute's templates (structure, terminology, defaults) but never hard-codes behavior (configuration-driven).

## 2. Actors
- **Organization Administrator** — manages all institutes in the deployment (the client-level admin).
- **Institute Administrator** — manages one institute's setup and operations.
- **System** — enforces uniqueness, lifecycle, scope cascade, and configuration seeding.

## 3. Use Cases

**Use Case ID:** UC-INST-01 — Create & Set Up an Institute
**Actors:** Organization Administrator, System
**Description:** Stand up a new institute and complete its mandatory setup before activation.
**Preconditions:** Actor holds `org.institute.manage`; the deployment is provisioned.
**Main Flow:** 1) Admin creates an institute (name, unique code, **type**, locale). 2) System seeds type-based templates (structure, terminology, defaults). 3) Admin completes mandatory setup (at least one campus or the default campus, locale/terminology, required config). 4) Admin activates the institute.
**Alternative Flow:** A1) Single-institute deployment → a default institute may be auto-created at provisioning.
**Exception Flow:** E1) Duplicate code → reject. E2) Activation attempted with incomplete mandatory setup → block with a checklist of what's missing.
**Post Conditions:** Institute `ACTIVE`; scope available for downstream modules; events audited.
**Business Rules Applied:** INST-001, INST-002, INST-003, INST-005.

**Use Case ID:** UC-INST-02 — Suspend / Archive an Institute
**Actors:** Organization Administrator, System
**Description:** Temporarily suspend or permanently archive an institute.
**Preconditions:** Actor holds `org.institute.manage`.
**Main Flow (Suspend):** 1) Admin suspends. 2) System blocks operational access for users scoped only to this institute; org admins retain management access. **Main Flow (Archive):** 1) Admin requests archive. 2) System verifies no blocking active dependencies (open sessions/enrollments) or guides resolution. 3) Institute → `ARCHIVED`; data preserved immutably.
**Exception Flow:** E1) Archive attempted with an active session/enrollments → block or require explicit handling (INST-007/INST-008). E2) Hard-delete attempted → forbidden (INST-008).
**Post Conditions:** Institute suspended or archived; cascade applied; audited.
**Business Rules Applied:** INST-006, INST-007, INST-008.

## 4. Business Rules

**Rule ID:** INST-001
**Rule Name:** Institute Code Uniqueness
**Description:** Each institute has an immutable, human-meaningful code unique within the deployment.
**Priority:** High
**Category:** Identity
**Preconditions:** Institute creation/edit.
**Business Rule:** Code is unique across the deployment, case-insensitive, and not re-assignable once set (used in references and exports).
**System Action:** Enforce uniqueness on create; lock the code after creation.
**Validation:** Format per client config; uniqueness check.
**Failure Behavior:** Reject duplicates; reject code edits after creation.
**Audit Requirement:** Log `INSTITUTE_CREATED` with code.
**Example Scenario:** "DHK-HS-01" cannot be reused for a second institute.
**Related Rules:** INST-002.

**Rule ID:** INST-002
**Rule Name:** Institution Type Drives Templates, Not Code
**Description:** The institute's type seeds configurable templates; it never selects hard-coded behavior.
**Priority:** Critical
**Category:** Configuration
**Preconditions:** Institute creation with a chosen type.
**Business Rule:** Type (school/college/university/madrasa/coaching/training) seeds default academic structure, terminology, grading, and fee templates via configuration. All seeded values are editable; the engine never branches on type in code.
**System Action:** Apply the type template as initial configuration values.
**Validation:** Type ∈ supported catalog; template exists for the type.
**Failure Behavior:** Unknown type rejected; missing template flagged.
**Audit Requirement:** Log `INSTITUTE_TYPE_TEMPLATE_APPLIED`.
**Example Scenario:** Choosing "madrasa" seeds madrasa-appropriate terminology and structure, all of which the admin can adjust.
**Related Rules:** INST-004, CFG (future) templates.

**Rule ID:** INST-003
**Rule Name:** Mandatory Setup Before Activation
**Description:** An institute cannot be activated until required setup is complete.
**Priority:** High
**Category:** Lifecycle gating
**Preconditions:** Institute in `DRAFT`.
**Business Rule:** Activation requires the configured minimum: at least one campus (or the auto default campus), locale/terminology set, and any type-required configuration. Operational modules are unavailable until `ACTIVE`.
**System Action:** Validate the setup checklist; block activation otherwise.
**Validation:** Checklist items satisfied (client-configurable set).
**Failure Behavior:** Activation blocked with a precise list of missing items.
**Audit Requirement:** Log `INSTITUTE_ACTIVATED` with the checklist snapshot.
**Example Scenario:** An institute with no campus and no locale cannot be activated.
**Related Rules:** INST-005, CAMP-001, SESS-001.

**Rule ID:** INST-004
**Rule Name:** Institute Type Change Is Restricted
**Description:** Changing an institute's type after setup is forbidden or tightly constrained.
**Priority:** High
**Category:** Integrity
**Preconditions:** A type-change is requested on a non-draft institute.
**Business Rule:** Once an institute has operational data (structure, enrollments), its type is **locked**, because type drove the structural template and a change would invalidate existing records. In `DRAFT` with no data, a type change re-seeds templates.
**System Action:** Allow type change only in `DRAFT` with no dependent data; otherwise reject.
**Validation:** Check for dependent data before permitting change.
**Failure Behavior:** Reject the change with explanation; suggest a new institute instead.
**Audit Requirement:** Log `INSTITUTE_TYPE_CHANGED` (rare, draft-only).
**Example Scenario:** A live school cannot be flipped to "university" mid-year.
**Related Rules:** INST-002, INST-008.

**Rule ID:** INST-005
**Rule Name:** Configuration Inheritance & Override
**Description:** Institute configuration inherits organization defaults and may override them; campus may further override.
**Priority:** Medium
**Category:** Configuration resolution
**Preconditions:** A configurable value is resolved for an institute.
**Business Rule:** Resolution is most-specific-wins: campus → institute → organization → definition default. Institute-level overrides apply only within that institute.
**System Action:** Resolve via the Configuration Engine; cache by scope.
**Validation:** Override values valid against their definitions.
**Failure Behavior:** Invalid override rejected; falls back to inherited value.
**Audit Requirement:** Log institute-level configuration changes.
**Example Scenario:** The organization default grading scale applies unless an institute sets its own.
**Related Rules:** CAMP-005, CFG (future).

**Rule ID:** INST-006
**Rule Name:** Institute Is a Mandatory Scope
**Description:** Every scoped business object belongs to exactly one institute and is filtered by it.
**Priority:** Critical
**Category:** Isolation
**Preconditions:** Any scoped record is created.
**Business Rule:** Records carry `institute_id`; cross-institute reads/writes are impossible (enforced by AUTHZ-002). No business object is institute-less except deployment-global config.
**System Action:** Stamp `institute_id`; apply the scope filter on all access.
**Validation:** `institute_id` present and references an existing, accessible institute.
**Failure Behavior:** Reject institute-less scoped records; deny cross-institute access.
**Audit Requirement:** Inherit AUTHZ-002 scope-violation auditing.
**Example Scenario:** A student created without an institute is rejected.
**Related Rules:** AUTHZ-002, CAMP-004.

**Rule ID:** INST-007
**Rule Name:** Suspension Cascade
**Description:** Suspending an institute blocks operational access for its scoped users while preserving data and org-admin control.
**Priority:** High
**Category:** Lifecycle / access
**Preconditions:** Institute → `SUSPENDED`.
**Business Rule:** Users whose only access is to the suspended institute lose operational access (via scope evaluation); org admins retain management access to resolve the suspension. No data is altered or deleted.
**System Action:** Evaluate scope so suspended-institute access is denied; retain records.
**Validation:** Suspension is reversible; reinstatement restores access.
**Failure Behavior:** N/A (access simply denied while suspended).
**Audit Requirement:** Log `INSTITUTE_SUSPENDED/REINSTATED` with reason.
**Example Scenario:** A non-paying institute is suspended; its teachers cannot log in to operate it, but the org admin can reinstate after payment.
**Related Rules:** AUTH-005, AUTHZ-002.

**Rule ID:** INST-008
**Rule Name:** Archive, Never Hard-Delete
**Description:** Institutes with any operational history are archived, never destroyed.
**Priority:** Critical
**Category:** Data integrity / compliance
**Preconditions:** Institute removal requested.
**Business Rule:** An institute that ever held data cannot be hard-deleted; it is `ARCHIVED`, preserving all historical records immutably (academic and financial history must survive). Hard-delete is reserved only for an empty `DRAFT` institute with no data.
**System Action:** Transition to `ARCHIVED`; make data read-only; block new operations.
**Validation:** Detect any dependent data; if present, archive only.
**Failure Behavior:** Hard-delete attempt on a data-bearing institute → rejected.
**Audit Requirement:** Log `INSTITUTE_ARCHIVED`; archival is itself reversible-to-read only via audited reactivation policy.
**Example Scenario:** A closed college is archived; its alumni transcripts remain retrievable years later.
**Related Rules:** INST-004, SESS-007, Doc 29 (audit), Doc 30 (retention).

## 5. Validation Rules
- Institute code: unique, format-valid, immutable post-creation.
- Type ∈ supported catalog; template must exist.
- Activation blocked unless the mandatory-setup checklist passes.
- Type change permitted only in `DRAFT` with no dependent data.
- Every scoped record must carry a valid `institute_id`.

## 6. State Machine

**State Name:** DRAFT
**Description:** Created, undergoing setup; not operational.
**Allowed Transitions:** → ACTIVE (setup complete); → ARCHIVED (abandoned draft with no data may be hard-deleted instead).
**Forbidden Transitions:** → SUSPENDED (cannot suspend a never-active institute).
**System Actions:** Seed templates; allow type change; block operational modules.

**State Name:** ACTIVE
**Description:** Fully operational.
**Allowed Transitions:** → SUSPENDED; → ARCHIVED (with dependency handling).
**Forbidden Transitions:** → DRAFT.
**System Actions:** Enable modules; lock type once data exists.

**State Name:** SUSPENDED
**Description:** Temporarily disabled; data intact.
**Allowed Transitions:** → ACTIVE (reinstate); → ARCHIVED.
**Forbidden Transitions:** → DRAFT.
**System Actions:** Deny scoped operational access; preserve data.

**State Name:** ARCHIVED
**Description:** Permanently closed; read-only history.
**Allowed Transitions:** → ACTIVE only via explicit, audited reactivation policy (rare).
**Forbidden Transitions:** hard-delete (forbidden if data ever existed).
**System Actions:** Make read-only; block new operations; retain for retention period.

## 7. Status Definitions
`DRAFT` (setup) · `ACTIVE` (operational) · `SUSPENDED` (temporary block) · `ARCHIVED` (closed, read-only, retained).

## 8. Workflow Rules
- Institute creation is a direct org-admin action (no approval) but fully audited.
- Suspension/archival of an institute may require approval via the Workflow Engine when configured (high-impact).
- Reactivation of an `ARCHIVED` institute is an explicit, audited, policy-gated action — not a casual toggle.

## 9. Permission Rules
- `org.institute.manage` — create/activate/suspend/archive institutes (org admin).
- `institute.setup.manage` — complete an institute's setup (institute admin, scoped).
- `institute.config.manage` — manage institute-level configuration overrides.
- Viewing institute lists is scoped: an institute admin sees only their institute(s).

## 10. Notification Rules
- `INSTITUTE_ACTIVATED` → notify org admins and the institute admin.
- `INSTITUTE_SUSPENDED` → notify the institute's admins (and optionally all scoped users) with reason.
- `INSTITUTE_ARCHIVED` → notify org admins.
- Setup-incomplete reminders to the institute admin during `DRAFT`.

## 11. Audit Requirements
Mandatory: `INSTITUTE_CREATED`, `INSTITUTE_TYPE_TEMPLATE_APPLIED`, `INSTITUTE_ACTIVATED`, `INSTITUTE_SUSPENDED/REINSTATED`, `INSTITUTE_ARCHIVED`, `INSTITUTE_TYPE_CHANGED` (draft only), institute-level config changes. With actor, code, before/after, timestamp.

## 12. Data Retention Rules
- Archived institutes and all their historical records are retained for the configured retention period (often long, for academic/financial/legal reasons) — never auto-purged early.
- Retention is per-client configurable but bounded by legal minimums for academic/financial records.
- Erasure of an archived institute's personal data (if ever required) follows retention-driven, export-before-purge anonymization, preserving non-identifying integrity references.

## 13. Edge Cases
- **Single-institute deployment:** a default institute exists; the institute concept is invisible to that client but still scopes data.
- **Archiving with an active session:** blocked until the session is closed or explicitly handled (SESS-007), preventing orphaned live academics.
- **Type lock-in:** a wrong type chosen and discovered after data exists cannot be changed — the remedy is a new institute, not a mutation (INST-004).
- **Suspended institute's pending workflows:** in-flight approvals are paused, not lost; they resume on reinstatement.
- **Org admin with zero institutes:** a freshly provisioned deployment may have no institute until one is created.
- **Cross-institute reporting:** is per-institute by default; any "all institutes in this deployment" view is an org-admin capability and still never crosses deployments.

## 14. Failure Scenarios
- **Template missing for a type:** creation blocked with a clear error; never silently create an untemplated institute.
- **Activation race:** two admins activating simultaneously → idempotent; one wins, the other is a no-op.
- **Config override invalid:** rejected; institute falls back to inherited value, never to an undefined state.
- **Archive with unresolved dependencies:** transactional check prevents partial archival.

## 15. Exception Handling Rules
- Lifecycle transitions are validated and transactional; no partial state.
- Forbidden transitions (e.g., hard-delete of a data-bearing institute, type change with data) are rejected with explicit, actionable messages.
- Suspension never mutates business data; it only affects access.

## 16. Compliance Considerations
- **Record permanence:** academic and financial history must outlive an institute's operation; archive-not-delete enforces this.
- **Minors' data within an institute:** scoped and protected; archival retains it under retention rules, not indefinitely beyond legal need.
- **Data ownership:** the institution owns its data; archival and export support clean handover/offboarding.

## 17. Future Considerations
- Institute cloning (spin up a similar institute from an existing one's configuration).
- Institute groups/federations within a deployment for shared reporting (still deployment-bound).
- Per-institute branding overrides beyond per-deployment branding.
- Automated lifecycle (auto-suspend on non-payment via Control Plane signals), carefully audited.
