# 27 — Configuration Engine Use Cases

Transforms the Configuration Engine business rules (`CFG-001`…`CFG-011`) into use cases. This is the meta-engine that makes the system "configuration-driven with no hard-coded business rules": a definition registry, typed validation, scoped most-specific-wins resolution, immutable versioning with consumer stamping, effective-dating, governed rollback, change governance with immutable-after-use locks, fast cache propagation, secret protection, dynamic custom-field schemas, and per-deployment isolation.

## 1. Primary Actors
Platform Engineer / Developer (registers definitions at build time), Organization Administrator (org-level values, rollback, templates), Institute/Campus Administrator (scoped overrides).

## 2. Secondary Actors
Consuming Modules (resolve values, stamp versions), Workflow Engine (high-impact change approval), Audit service, all 29 other modules (consumers of resolution).

## 3. Goals
Declare every configurable setting before use; validate values by type; resolve deterministically across scopes (most-specific-wins, else default); version immutably so consumers can stamp the exact version used; effective-date changes forward; roll back safely; govern high-impact and immutable-after-use settings; propagate changes fast via a config-version; protect secrets; validate dynamic custom-field schemas; isolate configuration per deployment with type-template seeding.

## 4. User Journeys
- **Declare:** a developer registers a configuration definition (key, type, default, allowed scopes, sensitivity, immutable-after-use flag) — only then can values be set.
- **Set:** an admin sets a scoped value → typed validation → a new immutable version with an effective date → caches invalidate → consumers resolve fresh.
- **Resolve:** a consuming module requests a key for a scope → most-specific-wins resolution returns the value and its version → the consumer stamps it on any record it produces (reproducibility).
- **Govern:** a high-impact change routes through approval (SoD); an immutable-after-use setting is locked once dependent data exists; a bad change is rolled back (forward-only, governed).

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-CFG-001 | Register Configuration Definition | Critical |
| Core | UC-CFG-002 | Set Scoped Value (new immutable version) | Critical |
| Core | UC-CFG-003 | Resolve Value (most-specific-wins) | Critical |
| Core | UC-CFG-004 | Effective-Dated Change | High |
| Core | UC-CFG-005 | Governed Rollback (forward-only) | High |
| Core | UC-CFG-006 | High-Impact Change Approval & Immutable-After-Use | Critical |
| Core | UC-CFG-007 | Register / Validate Dynamic Custom-Field Schema | High |
| Core | UC-CFG-008 | Manage Sensitive / Secret Configuration | Critical |
| Core | UC-CFG-009 | Apply Type Template (per-deployment seed) | High |
| CRUD | UC-CFG-010 | View Configuration (resolved / by scope) | Medium |
| CRUD | UC-CFG-011 | Deprecate Definition | Medium |
| Admin | UC-CFG-012 | Manage Cache Invalidation / Config-Version | High |
| Search | UC-CFG-013 | Search Definitions / Values | Low |
| Reporting | UC-CFG-014 | Configuration Audit & Drift Report | Medium |
| Bulk/Import | UC-CFG-015 | Import / Bulk-Set Configuration | Medium |
| Export | UC-CFG-016 | Export Configuration (secrets redacted) | Medium |
| Workflow | UC-CFG-017 | Change-Approval Workflow | High |
| Exception | UC-CFG-018 | Unregistered-Key Use Blocked | Critical |
| Exception | UC-CFG-019 | Invalid Value Blocked | High |
| Exception | UC-CFG-020 | Immutable-After-Use Change Blocked | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-CFG-001 — Register Configuration Definition
- **Module:** Configuration Engine · **Priority:** Critical
- **Actors:** Platform Engineer / Developer (primary), System
- **Goal:** Declare a configurable setting before any value can be set or resolved.
- **Description:** Registers a definition with key, data type, allowed values/range, default, allowed scope levels, overridability, sensitivity, and an immutable-after-use flag — so no undeclared configuration can exist.
- **Business Rules Applied:** CFG-001, CFG-002, CFG-010.
- **Preconditions:** A setting is needed by a module; actor holds `config.definition.manage`.
- **Trigger:** A new configurable setting is introduced.
- **Main Success Scenario:**
  1. Engineer registers a definition (key, type, default, allowed scopes, sensitivity, immutable-after-use, validation).
  2. System validates the definition is complete and well-formed.
  3. System adds it to the registry; the key becomes settable within its allowed scopes.
- **Alternative Flows:** A1) Register a dynamic custom-field schema (UC-CFG-007).
- **Exception Flows:** E1) Duplicate/invalid definition → reject. E2) Setting a value for an unregistered key → blocked (UC-CFG-018).
- **Validation Rules:** Definition complete; key unique; type/scope/sensitivity valid (CFG-001/002).
- **Permissions Required:** `config.definition.manage` (platform; tightly held).
- **Notifications Triggered:** None routine.
- **Audit Events Generated:** `CONFIG_DEFINITION_REGISTERED`.
- **Data Created:** Definition.
- **Data Updated:** Registry.
- **Data Deleted:** None.
- **Post Conditions:** Key declared and configurable within allowed scopes.
- **Related Use Cases:** UC-CFG-002, UC-CFG-007, UC-CFG-011.
- **Acceptance Criteria:**
  - Given a complete definition, When registered, Then the key becomes settable within its allowed scopes.
  - Given an attempt to set a value for an unregistered key, When made, Then it is blocked.
  - Given a duplicate or malformed definition, When registered, Then it is rejected.
- **Edge Case Analysis:**
  - *Invalid Input:* incomplete/malformed definition rejected.
  - *Permission Failure:* lacks `config.definition.manage` → 403.
  - *Concurrent Update:* duplicate key registered twice → one wins (uniqueness).
  - *Duplicate Data:* duplicate definition blocked.
  - *System Failure:* atomic registration.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* register simple/typed/sensitive/immutable-after-use definitions.
  - *Negative:* duplicate key; unregistered-key value; malformed.
  - *Boundary:* definition with all scopes vs single scope.

### UC-CFG-002 — Set Scoped Value (new immutable version)
- **Module:** Configuration Engine · **Priority:** Critical
- **Actors:** Administrator (primary), System
- **Goal:** Set or change a configuration value at a scope, creating a new immutable version.
- **Description:** Validates the value against its definition, checks approval/immutability rules, publishes a new immutable version with an effective date, bumps the config-version, and invalidates caches.
- **Business Rules Applied:** CFG-002, CFG-003, CFG-004, CFG-005, CFG-007, CFG-008.
- **Preconditions:** Definition registered; actor authorized for the scope; value valid.
- **Trigger:** Admin sets/changes a value.
- **Main Success Scenario:**
  1. Admin sets a value at a scope (org/institute/campus/...).
  2. System validates against the definition (type/range/enum/required + consistency hooks, CFG-002).
  3. System checks approval (CFG-007) and immutability (immutable-after-use), then publishes a new immutable version with an effective date (CFG-004/005).
  4. System bumps the config-version and invalidates affected caches (CFG-008); consumers resolve fresh.
- **Alternative Flows:** A1) High-impact change routes to approval (UC-CFG-006). A2) Future-dated value (UC-CFG-004).
- **Exception Flows:** E1) Invalid value → reject (UC-CFG-019). E2) Immutable-after-use with dependents → blocked (UC-CFG-020). E3) Override at a disallowed scope → reject (CFG-003).
- **Validation Rules:** Typed/validated; allowed scope; immutable published versions; effective-dated; governed (CFG-002/003/004/005/007).
- **Permissions Required:** `config.value.manage` (scoped).
- **Notifications Triggered:** High-impact change/approval notices; future-dated activation notice.
- **Audit Events Generated:** `CONFIG_VALUE_PUBLISHED` (version, scope, before/after for non-sensitive).
- **Data Created:** New value version.
- **Data Updated:** Config-version; caches invalidated.
- **Data Deleted:** None (prior version retained).
- **Post Conditions:** New version effective; history retained; consumers resolve fresh.
- **Related Use Cases:** UC-CFG-003, UC-CFG-005, UC-CFG-006.
- **Acceptance Criteria:**
  - Given a valid value at an allowed scope, When set, Then a new immutable version is published and caches invalidate.
  - Given an invalid value, When set, Then it is rejected with the specific violation.
  - Given an immutable-after-use setting with dependents, When changed, Then it is blocked.
- **Edge Case Analysis:**
  - *Invalid Input:* type/range/enum violation rejected.
  - *Permission Failure:* unauthorized scope → 403.
  - *Concurrent Update:* two sets → last-write publishes a new version (no lost history).
  - *Duplicate Data:* identical re-set → no-op/new version per policy.
  - *System Failure:* atomic publish + version bump + audit.
  - *Workflow Failure:* high-impact approval pends; value not effective until approved.
- **QA Coverage:**
  - *Positive:* set at each allowed scope; future-dated.
  - *Negative:* invalid value; disallowed scope; immutable-after-use with dependents.
  - *Boundary:* value at range edges; effective-date boundary.

### UC-CFG-003 — Resolve Value (most-specific-wins)
- **Module:** Configuration Engine · **Priority:** Critical
- **Actors:** Consuming Module (primary), System
- **Goal:** Return the effective value for a scope deterministically and expose the version used.
- **Description:** Resolves most-specific-wins across the scope hierarchy (and effective date), returning the value and its version so the consumer can stamp it on any record it produces.
- **Business Rules Applied:** CFG-003, CFG-004, CFG-005, CFG-008.
- **Preconditions:** Definition exists; scope context available.
- **Trigger:** A module requests a key for a scope.
- **Main Success Scenario:**
  1. A module requests a key with a scope context (and optional effective date).
  2. System resolves most-specific-wins (e.g., section → class → campus → institute → org → default) and by effective date (CFG-003/005).
  3. System returns the value and its version; the consumer stamps the version (CFG-004).
- **Alternative Flows:** A1) Cached resolution served fast; cache keyed by config-version (CFG-008).
- **Exception Flows:** E1) No value at any scope → return the definition default (never null/ambiguous). E2) Cache uncertainty → resolve from source.
- **Validation Rules:** Deterministic resolution; default fallback; version exposed (CFG-003/004).
- **Permissions Required:** System (consumed internally); `config.view` for admin reads.
- **Notifications Triggered:** None.
- **Audit Events Generated:** Resolution provenance available; not individually audited (high-volume).
- **Data Created/Updated/Deleted:** None.
- **Post Conditions:** Deterministic value + version returned; consumer can stamp it.
- **Related Use Cases:** UC-CFG-002, UC-CFG-012; consumed by every module.
- **Acceptance Criteria:**
  - Given a value set at campus scope, When resolved for that campus, Then the campus value wins over the institute/org/default.
  - Given no value at any scope, When resolved, Then the definition default is returned (never null).
  - Given a resolved value, When returned, Then its version is exposed for stamping.
- **Edge Case Analysis:**
  - *Invalid Input:* unknown key → not-resolved (treated as unregistered, UC-CFG-018).
  - *Permission Failure:* N/A (system) / admin read scoped.
  - *Concurrent Update:* value change mid-resolution → in-flight operations use the pinned version; next resolve is fresh.
  - *Duplicate Data:* one value per key/scope (no ambiguity).
  - *System Failure:* cache down → resolve from source (correctness over speed).
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* most-specific-wins at each level; default fallback; effective-dated resolution.
  - *Negative:* unknown key; ambiguous resolution (must not occur).
  - *Boundary:* exactly one level set; effective-date edge; cache-version boundary.

### UC-CFG-006 — High-Impact Change Approval & Immutable-After-Use
- **Module:** Configuration Engine · **Priority:** Critical
- **Actors:** Administrator (requester), Approver, System
- **Goal:** Govern high-impact changes via approval (SoD) and lock immutable-after-use settings once dependents exist.
- **Description:** High-impact settings (grading, fee, security policy) route through approval (requester ≠ approver); definitions flagged immutable-after-use (e.g., institute type INST-004, ID schemes) cannot change once dependent data exists.
- **Business Rules Applied:** CFG-007, AUTHZ-009, WFL-004.
- **Preconditions:** A change to a high-impact or immutable-after-use setting.
- **Trigger:** Admin attempts such a change.
- **Main Success Scenario:**
  1. Admin requests a change to a high-impact setting.
  2. System routes it through the approval workflow (SoD: requester ≠ approver, CFG-007/AUTHZ-009).
  3. On approval, the new version publishes; for immutable-after-use settings, System first checks no dependent data exists.
  4. If dependents exist for an immutable-after-use setting, the change is blocked.
- **Alternative Flows:** A1) Low-impact change → no approval needed (UC-CFG-002).
- **Exception Flows:** E1) Self-approval → blocked (SoD). E2) Immutable-after-use with dependents → blocked (UC-CFG-020).
- **Validation Rules:** High-impact approved (SoD); immutable-after-use locked with dependents (CFG-007).
- **Permissions Required:** `config.value.manage` + `config.value.approve` (SoD).
- **Notifications Triggered:** Change request/approval notices.
- **Audit Events Generated:** Approval events; blocked immutable-after-use attempts.
- **Data Created:** Approved value version.
- **Data Updated:** Config-version.
- **Data Deleted:** None.
- **Post Conditions:** High-impact change governed; locked settings protected.
- **Related Use Cases:** UC-CFG-002, UC-CFG-017, UC-INST (type lock).
- **Acceptance Criteria:**
  - Given a high-impact change, When requested, Then it requires approval by a distinct authority (SoD).
  - Given an immutable-after-use setting with dependents, When changed, Then it is blocked.
  - Given the requester attempts to approve their own change, When made, Then it is blocked.
- **Edge Case Analysis:**
  - *Invalid Input:* change without required justification blocked.
  - *Permission Failure:* self-approval blocked.
  - *Concurrent Update:* two high-impact changes → serialized; each approved independently.
  - *Duplicate Data:* idempotent.
  - *System Failure:* atomic with approval audit.
  - *Workflow Failure:* approver vacancy → escalation; not effective until approved.
- **QA Coverage:**
  - *Positive:* approved high-impact change; low-impact no-approval.
  - *Negative:* self-approval; immutable-after-use with dependents.
  - *Boundary:* change exactly when the first dependent is created (lock engages).

---

## 7. Compact Specifications (routine use cases)

- **UC-CFG-004 — Effective-Dated Change** · *High* · Rules: CFG-005, CFG-004. Set a value effective from a future date; resolution honors the date. *Edge:* non-overlapping effective periods; future activation. *QA:* future-dated; activation; overlap rejected.
- **UC-CFG-005 — Governed Rollback (forward-only)** · *High* · Rules: CFG-006. Roll back a key/scope to a prior version with a reason; forward-only (does not rewrite produced records). *Permissions:* `config.rollback`. *Audit:* `CONFIG_ROLLED_BACK`. *Edge:* dependents corrected via their own governed flow, not rollback. *QA:* rollback; forward-only effect; reason recorded.
- **UC-CFG-007 — Register / Validate Dynamic Custom-Field Schema** · *High* · Rules: CFG-010, CFG-002. Define custom-field schemas (student/admission fields) the engine validates; schema changes versioned. *Edge:* historical values resolve against their schema version. *QA:* schema register; value validation; version fidelity.
- **UC-CFG-008 — Manage Sensitive / Secret Configuration** · *Critical* · Rules: CFG-009. Store secrets encrypted, access-restricted, masked; excluded from exports/audit values. *Permissions:* `config.sensitive.manage`. *Edge:* never logged/exported; audit records change-occurred only. *QA:* encrypted store; masked read; redacted export/audit.
- **UC-CFG-009 — Apply Type Template (per-deployment seed)** · *High* · Rules: CFG-011, INST-002. Seed initial configuration from an institution-type template (editable). *Edge:* per-deployment isolation; templates seed, never hard-code. *QA:* template seed; editable; no cross-deployment.
- **UC-CFG-010 — View Configuration (resolved / by scope)** · *Medium* · Rules: CFG-003, CFG-009. View resolved config or values by scope (non-sensitive; secrets masked). *Permissions:* `config.view`. *Edge:* provenance shown; secrets masked. *QA:* resolved view; secret masking.
- **UC-CFG-011 — Deprecate Definition** · *Medium* · Rules: CFG-001, CFG-004. Deprecate a definition; stop new use; retain for historical resolution. *Audit:* `CONFIG_DEFINITION_DEPRECATED`. *Edge:* still-referenced definition retained, not deleted. *QA:* deprecate blocks new use; history kept.
- **UC-CFG-012 — Manage Cache Invalidation / Config-Version** · *High* · Rules: CFG-008. Bump config-version on change; invalidate caches; in-flight pinned versions unaffected. *Edge:* cache uncertainty → source resolution. *QA:* invalidation; fast propagation; no mid-operation flip.
- **UC-CFG-013 — Search Definitions / Values** · *Low* · Rules: AUTHZ-002, CFG-009. Scoped search (secrets masked). *QA:* scope; secret masking.
- **UC-CFG-014 — Configuration Audit & Drift Report** · *Medium* · Rules: REP-002, CFG-004. Report changes, versions, and drift from defaults/baselines. *Edge:* scoped; secrets excluded. *QA:* change history accuracy; drift detection.
- **UC-CFG-015 — Import / Bulk-Set Configuration** · *Medium* · Rules: CFG-002. Import/bulk-set validated values (per-row typed validation). *Edge:* invalid rows rejected; immutable-after-use respected. *QA:* clean import; invalid rows; lock respected.
- **UC-CFG-016 — Export Configuration (secrets redacted)** · *Medium* · Rules: CFG-009, REP-005. Export config; sensitive values redacted. *Permissions:* `config.view`/`report.export`. *Edge:* secrets never exported. *QA:* export; secret redaction.
- **UC-CFG-017 — Change-Approval Workflow** · *High* · Rules: WFL-002/004, CFG-007. Version-pinned approval for high-impact config changes. *QA:* pinning; SoD; escalation.
- **UC-CFG-018 — Unregistered-Key Use Blocked (Exception)** · *Critical* · Rules: CFG-001. Values/resolution for unregistered keys blocked. *QA:* unknown-key write/read blocked; registered allowed.
- **UC-CFG-019 — Invalid Value Blocked (Exception)** · *High* · Rules: CFG-002. Type/range/enum/required violations rejected at write. *QA:* each violation blocked; valid allowed.
- **UC-CFG-020 — Immutable-After-Use Change Blocked (Exception)** · *High* · Rules: CFG-007. Locked-after-use settings cannot change once dependents exist. *QA:* blocked with dependents; allowed during setup.

## 8. Module-level QA & Edge Themes
- **No undeclared configuration (CFG-001):** unregistered-key use is impossible — the headline integrity suite.
- **Typed validation (CFG-002):** "configurable" never means "unvalidated"; rejected at write, not discovered at consumption.
- **Deterministic resolution + reproducibility (CFG-003/004 / P6/P7):** most-specific-wins, default fallback, version stamping — the engine behind every reproducible record.
- **Governance & locks (CFG-007):** high-impact approval (SoD) and immutable-after-use enforcement (the one mechanism behind INST-004, ID schemes, etc. — Conflict C-06).
- **Secret protection (CFG-009):** sensitive values never logged, exported, or written to audit values.
- **Per-deployment isolation (CFG-011):** configuration never crosses client deployments.
