# 27 — Configuration Engine Business Rules

## 1. Module Purpose
Govern the **Configuration Engine** — the platform capability that makes the entire system "configuration-driven with no hard-coded business rules." Every place another module says *configurable*, *version-stamped*, *effective-dated*, or *most-specific-wins* resolves here. The engine owns: a **definition registry** (every setting declared with type, default, allowed scopes, sensitivity), **typed validation** of values, **scoped storage** with deterministic **most-specific-wins resolution**, **immutable versioning** so consumers can stamp the exact version they used (the reason historical invoices, marksheets, and payslips reproduce), **effective-dating**, governed **rollback**, fast **cache propagation**, **secret protection**, and **dynamic custom-field schemas**. It is per-deployment isolated and never crosses clients.

## 2. Actors
- **Platform (build-time) / Developer** — registers configuration **definitions** (the catalog of what is configurable); never sets client values in code.
- **Organization Administrator** — sets org-level values, manages templates, performs governed rollbacks.
- **Institute / Campus Administrator** — sets scoped overrides within allowed scopes.
- **Consuming Modules** — resolve values deterministically and stamp the version used.
- **System** — validates, versions, resolves, caches, audits, and protects configuration.

## 3. Use Cases

**Use Case ID:** UC-CFG-01 — Register a Configuration Definition
**Actors:** Platform/Developer, System
**Description:** Declare a new configurable setting before any value can be set.
**Preconditions:** The setting is needed by a module.
**Main Flow:** 1) Register a definition (key, data type, allowed values/range, default, allowed scope levels, overridability, sensitivity, immutable-after-use flag). 2) System validates the definition and adds it to the registry. 3) The key becomes settable within its allowed scopes.
**Alternative Flow:** A1) Register a dynamic custom-field schema (CFG-010).
**Exception Flow:** E1) Duplicate/invalid definition → reject. E2) Setting a value for an unregistered key → blocked (CFG-001).
**Post Conditions:** Definition registered; key configurable; audited.
**Business Rules Applied:** CFG-001, CFG-002, CFG-010.

**Use Case ID:** UC-CFG-02 — Set a Scoped Value (New Version)
**Actors:** Administrator, System
**Description:** Set or change a configuration value at a scope, creating a new immutable version.
**Preconditions:** Definition registered; actor authorized for the scope; value valid.
**Main Flow:** 1) Admin sets a value at a scope (org/institute/campus/...). 2) System validates against the definition, checks approval/immutability rules, and publishes a new immutable version with an effective date. 3) Caches invalidate; the config version bumps; consumers resolve fresh.
**Alternative Flow:** A1) High-impact change routes through approval (CFG-007).
**Exception Flow:** E1) Invalid value → reject (CFG-002). E2) Immutable-after-use setting with dependent data → blocked (CFG-007). E3) Override at a disallowed scope → reject (CFG-003).
**Post Conditions:** New version effective; history retained; audited.
**Business Rules Applied:** CFG-002, CFG-003, CFG-004, CFG-005, CFG-007, CFG-008.

**Use Case ID:** UC-CFG-03 — Resolve a Value & Roll Back
**Actors:** Consuming Module / Administrator, System
**Description:** Resolve the effective value for a scope; optionally roll back to a prior version.
**Preconditions:** Definition exists; scope context available.
**Main Flow (resolve):** 1) A module requests a key for a scope and (optionally) an effective date. 2) System resolves most-specific-wins across scope hierarchy and effective-dating, returns the value and its version. 3) The consumer stamps the version on any record it produces.
**Main Flow (rollback):** 1) Admin rolls back a key/scope to a prior version with a reason. 2) System publishes the prior value as a new current version (history preserved), governed and audited.
**Exception Flow:** E1) No value at any scope → return the definition default. E2) Rollback affecting dependent data → governed handling (CFG-006).
**Post Conditions:** Deterministic value returned/stamped; rollback audited.
**Business Rules Applied:** CFG-003, CFG-004, CFG-006, CFG-008.

## 4. Business Rules

**Rule ID:** CFG-001
**Rule Name:** Definition Registry — No Undeclared Configuration
**Description:** Every configurable setting must be a registered definition before any value can be set or resolved.
**Priority:** Critical
**Category:** Governance / integrity
**Preconditions:** Configuration use.
**Business Rule:** A definition declares key, data type, allowed values/range, default, allowed scope levels, overridability, sensitivity, and an immutable-after-use flag. Values for unregistered keys are impossible; modules cannot invent ad-hoc settings the engine does not know.
**System Action:** Maintain the registry; reject values/resolutions for unknown keys.
**Validation:** Key registered; definition complete.
**Failure Behavior:** Reject unknown-key writes/reads.
**Audit Requirement:** Log `CONFIG_DEFINITION_REGISTERED/DEPRECATED`.
**Example Scenario:** A "max-attendance-backdate-days" setting must be registered before any institute can set it.
**Related Rules:** CFG-002, AUTHZ-008 (permission-registry parallel).

**Rule ID:** CFG-002
**Rule Name:** Typed, Validated Values
**Description:** Configuration values are validated against their definition; "configurable" never means "unvalidated."
**Priority:** Critical
**Category:** Data quality
**Preconditions:** Setting a value.
**Business Rule:** Each value is validated for type, range, enum membership, required-ness, and any registered cross-field/consistency rule (e.g., a hook to validate grade-band coverage). Invalid values are rejected at write time, not discovered at consumption.
**System Action:** Validate against the definition (and registered consistency hooks) before publishing.
**Validation:** Value conforms to type/range/enum/required and consistency hooks.
**Failure Behavior:** Reject invalid values with the specific violation.
**Audit Requirement:** Captured in `CONFIG_VALUE_PUBLISHED`.
**Example Scenario:** Setting backdate-days to "ten" (string) or -5 is rejected; 5 (integer, in range) is accepted.
**Related Rules:** CFG-001, GRD-002 (consumer consistency), STU-002.

**Rule ID:** CFG-003
**Rule Name:** Scoped Values & Most-Specific-Wins Resolution
**Description:** Values may be set at multiple scope levels; resolution deterministically picks the most specific, falling back to the default.
**Priority:** Critical
**Category:** Resolution integrity
**Preconditions:** Resolving a value.
**Business Rule:** Scope precedence is deterministic (e.g., section → class → campus → institute → organization → definition default); a value is set at exactly one place per key per scope; resolution returns the most specific set value, else the default. Overrides are only allowed at scopes the definition permits.
**System Action:** Resolve along the precedence chain; enforce allowed override scopes.
**Validation:** Override scope permitted; one value per key/scope.
**Failure Behavior:** Reject overrides at disallowed scopes; never ambiguous resolution.
**Audit Requirement:** Resolution provenance available; override changes logged.
**Example Scenario:** A campus start-time override wins over the institute default for that campus only.
**Related Rules:** INST-005, CAMP-005, CFG-004.

**Rule ID:** CFG-004
**Rule Name:** Versioning & Immutable Published Versions
**Description:** Every value change creates a new immutable version; consumers stamp the version they used.
**Priority:** Critical
**Category:** Reproducibility
**Preconditions:** Value change / consumption.
**Business Rule:** Published config versions are immutable; a change creates a new version (the prior remains for history). Consuming modules record the resolved version on the records they produce, so any historical record (invoice, marksheet, payslip) is reproducible to the exact configuration that produced it.
**System Action:** Version on change; expose version on resolution; never mutate published versions.
**Validation:** Versions monotonic; immutability enforced.
**Failure Behavior:** Reject in-place edits of published versions.
**Audit Requirement:** Log `CONFIG_VALUE_PUBLISHED` with version.
**Example Scenario:** A 2025 marksheet stamps grading-config v3; a later v4 never alters it.
**Related Rules:** GRD-007, FEE-001, HR-003, RES-001.

**Rule ID:** CFG-005
**Rule Name:** Effective-Dating
**Description:** Values may carry an effective date so changes apply forward, not retroactively.
**Priority:** High
**Category:** Temporal integrity
**Preconditions:** Setting a value with future effect.
**Business Rule:** A value version has an effective-from (and optional until); resolution for a given effective date returns the version in force then. This is how future-dated changes (next term's fees, next year's grading) coexist with current operations.
**System Action:** Resolve by effective date; activate future versions on schedule.
**Validation:** Effective dates non-overlapping per scope/key; coherent.
**Failure Behavior:** Reject overlapping effective periods.
**Audit Requirement:** Effective dates recorded with versions.
**Example Scenario:** A fee change effective next term is set now and applies only from that date.
**Related Rules:** FEE-001, GRD-001, SESS-002.

**Rule ID:** CFG-006
**Rule Name:** Governed Rollback
**Description:** Rolling back to a prior configuration is a governed, audited action that preserves history and handles dependents.
**Priority:** High
**Category:** Safety / governance
**Preconditions:** A bad change needs reverting.
**Business Rule:** Rollback publishes a prior value as the new current version (history never lost) with a recorded reason; rollbacks affecting already-produced dependent data (e.g., issued invoices under the new version) do not retroactively alter that data — they only change forward resolution. High-impact rollbacks may require approval.
**System Action:** Re-publish prior value as current; preserve history; forward-only effect.
**Validation:** Authorized; reason recorded; dependents handled forward-only.
**Failure Behavior:** Block rollback that would silently rewrite produced records.
**Audit Requirement:** Log `CONFIG_ROLLED_BACK` with from/to versions, reason.
**Example Scenario:** A mistaken grading change is rolled back; already-published results under the bad version are corrected via their own governed revision, not by the rollback.
**Related Rules:** CFG-004, RES-006, GRD-008.

**Rule ID:** CFG-007
**Rule Name:** Change Governance, Approval & Immutable-After-Use
**Description:** High-impact and lock-after-use settings are governed; some settings cannot change once dependent data exists.
**Priority:** Critical
**Category:** Governance / integrity
**Preconditions:** A configuration change.
**Business Rule:** High-impact settings (e.g., grading, fee, security policy) may require approval (SoD: requester ≠ approver). Definitions flagged immutable-after-use (e.g., institute type INST-004, ID schemes) cannot be changed once dependent data exists — only set during setup. Self-approval is impossible.
**System Action:** Route high-impact changes to approval; block changes to locked-after-use settings with dependents.
**Validation:** Approver authority; dependency check for locked settings.
**Failure Behavior:** Block ungoverned high-impact changes and locked-after-use mutations.
**Audit Requirement:** Log approvals and blocked lock-after-use attempts.
**Example Scenario:** Changing the grading scheme needs approval; changing an institute's type after enrollments exist is blocked.
**Related Rules:** AUTHZ-009, INST-004, GRD-001, STU-001.

**Rule ID:** CFG-008
**Rule Name:** Config-Version Cache Invalidation & Fast Propagation
**Description:** Configuration changes propagate quickly via a version stamp and cache invalidation, not at some arbitrary delay.
**Priority:** High
**Category:** Consistency / performance
**Preconditions:** A value change.
**Business Rule:** Resolved values are cached for performance; a change bumps a config-version and invalidates affected caches so the next resolution is fresh (mirrors permissions-version, AUTHZ-006). In-flight operations that pinned a version are unaffected (no mid-operation flips).
**System Action:** Bump version; invalidate caches; serve fresh on next resolve.
**Validation:** Version monotonic; cache keyed by version/scope.
**Failure Behavior:** On cache uncertainty, resolve from source (correctness over speed).
**Audit Requirement:** Log `CONFIG_VERSION_BUMPED`.
**Example Scenario:** Raising a section's capacity takes effect on the next resolution, not after a cache TTL.
**Related Rules:** AUTHZ-006, CFG-004.

**Rule ID:** CFG-009
**Rule Name:** Sensitive / Secret Configuration Protection
**Description:** Sensitive configuration (integration secrets, key references) is access-controlled, masked, and never exposed in plain reads or logs.
**Priority:** Critical
**Category:** Security
**Preconditions:** Sensitive config stored/accessed.
**Business Rule:** Definitions flagged sensitive are encrypted at rest, access-restricted to authorized roles, masked in UI/API responses, and excluded from general config exports and audit value-capture (audit records that a change occurred, not the secret value).
**System Action:** Encrypt/mask sensitive values; restrict access; redact in logs/exports.
**Validation:** Sensitivity flag honored; access authorized.
**Failure Behavior:** Deny/redact unauthorized access; never log secret values.
**Audit Requirement:** Log access to sensitive config (without the value).
**Example Scenario:** A payment-gateway secret is stored encrypted and never shown in the config UI or audit value field.
**Related Rules:** STF-007, STU-005, Compliance.

**Rule ID:** CFG-010
**Rule Name:** Dynamic Custom-Field Schemas Are Engine-Validated
**Description:** Dynamic fields (student custom fields, admission forms) are config-defined schemas the engine validates like any other configuration.
**Priority:** High
**Category:** Data quality / flexibility
**Preconditions:** Dynamic-field definition/use.
**Business Rule:** Custom fields are defined as schemas (key, type, options, required, validation) in the registry; values entered against them (STU-002, ADM-001) are validated by the engine; schema changes are versioned so historical values resolve against the version they were created under.
**System Action:** Register/validate dynamic-field schemas; version schema changes.
**Validation:** Schema valid; values conform; version-stamped.
**Failure Behavior:** Reject values violating the schema; never store unvalidated custom data.
**Audit Requirement:** Log schema definition/changes.
**Example Scenario:** An institute adds a validated "blood group" student field; entries are checked on save.
**Related Rules:** STU-002, ADM-001, CFG-002, CFG-004.

**Rule ID:** CFG-011
**Rule Name:** Per-Deployment Isolation & Type Templates
**Description:** Configuration is isolated per client deployment; institution-type templates seed initial values.
**Priority:** Critical
**Category:** Isolation / onboarding
**Preconditions:** Deployment/institute setup.
**Business Rule:** All configuration lives within one client deployment; cross-client config access is structurally impossible. Institution-type templates (INST-002) seed an institute's initial configuration, which is then editable; templates never hard-code behavior, only seed values.
**System Action:** Scope all config to the deployment; apply type templates as seed values.
**Validation:** Deployment scope enforced; template valid for type.
**Failure Behavior:** Reject cross-deployment access; flag missing templates.
**Audit Requirement:** Log template application.
**Example Scenario:** A new madrasa institute is seeded with madrasa-appropriate defaults, all overridable.
**Related Rules:** INST-002, INST-005, AUTHZ-002.

## 5. Validation Rules
- No value for an unregistered key; definitions complete before use.
- Values typed/range/enum/required-validated, plus registered consistency hooks.
- One value per key per scope; overrides only at allowed scopes; resolution deterministic.
- Published versions immutable; effective periods non-overlapping per scope/key.
- High-impact changes governed; immutable-after-use settings locked once dependents exist.
- Sensitive values protected; dynamic-field values schema-validated; all config deployment-scoped.

## 6. State Machine

**State Name:** DEFINITION: REGISTERED
**Description:** A configurable setting is declared and settable.
**Allowed Transitions:** → DEPRECATED (retired); dynamic-field schemas → versioned.
**Forbidden Transitions:** values for unregistered/deprecated-removed keys.
**System Actions:** Allow scoped value-setting within allowed scopes.

**State Name:** VALUE: DRAFT
**Description:** A value being prepared (esp. when approval is required).
**Allowed Transitions:** → PUBLISHED (validated/approved); → DISCARDED.
**Forbidden Transitions:** consumption of a draft value.
**System Actions:** Validate; route approval if high-impact.

**State Name:** VALUE: PUBLISHED (version)
**Description:** Immutable, effective value version.
**Allowed Transitions:** → SUPERSEDED (new version); → (rollback re-publishes a prior value as a new current version).
**Forbidden Transitions:** in-place edits.
**System Actions:** Serve in resolution; expose version to consumers.

**State Name:** VALUE: SUPERSEDED
**Description:** Replaced by a newer version; retained for history/effective-date resolution.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** edits.
**System Actions:** Serve historical/effective-dated resolution.

**State Name:** DEPRECATED / ARCHIVED
**Description:** Definition retired / version archived.
**Allowed Transitions:** ARCHIVED terminal.
**Forbidden Transitions:** edits/hard-delete (consumers may depend on history).
**System Actions:** Read-only retention.

## 7. Status Definitions
Definition: `REGISTERED` · `DEPRECATED` · `ARCHIVED`. Value version: `DRAFT` · `PUBLISHED` · `SUPERSEDED` · `ARCHIVED`. Flags: `OVERRIDABLE_SCOPES`, `SENSITIVE`, `IMMUTABLE_AFTER_USE`, `HIGH_IMPACT`.

## 8. Workflow Rules
- High-impact changes route through approval with SoD (CFG-007).
- Rollbacks are governed, reasoned, and may require approval (CFG-006).
- Immutable-after-use settings are settable only during setup, before dependents exist.
- Future-dated changes activate on their effective date without manual intervention (CFG-005).

## 9. Permission Rules
- `config.definition.manage` — register/deprecate definitions (platform; tightly held).
- `config.value.manage` — set values within authorized scopes.
- `config.value.approve` — approve high-impact changes (SoD).
- `config.rollback` — perform governed rollbacks (elevated).
- `config.sensitive.manage` — manage sensitive/secret config (narrowly granted).
- `config.view` — read non-sensitive resolved config (scoped).

## 10. Notification Rules
- High-impact change requests/approvals → notify requester and approver.
- `CONFIG_ROLLED_BACK` → notify org admins / governance channel.
- Future-dated change activation → notify relevant admins ahead of effect.
- Sensitive-config changes → notify a security channel (without the value).

## 11. Audit Requirements
Mandatory: `CONFIG_DEFINITION_REGISTERED/DEPRECATED`, `CONFIG_VALUE_PUBLISHED` (version, scope, before/after for non-sensitive), `CONFIG_VERSION_BUMPED`, `CONFIG_ROLLED_BACK` (from/to, reason), approvals, `SENSITIVE_CONFIG_ACCESSED` (no value), template application, schema changes. Sensitive values are never written to audit; only the fact of change.

## 12. Data Retention Rules
- Definitions and all value versions retained as long as any consumer record depends on them (often indefinitely for reproducibility).
- Configuration history answers "what was the rule on date X" for audits/accreditation/disputes.
- Sensitive config follows secret-management retention/rotation; values are not retained in audit.
- Per-deployment; anonymization of client config follows deployment offboarding policy.

## 13. Edge Cases
- **No value set anywhere:** resolution returns the definition default, never null/ambiguous (CFG-003).
- **Mid-operation config change:** in-flight operations use the version they pinned; no mid-flight flip (CFG-008/CFG-004).
- **Immutable-after-use with dependents:** change blocked (e.g., institute type after enrollments, ID scheme after IDs issued) (CFG-007).
- **Rollback after dependent records produced:** forward-only; produced records corrected via their own governed revision, not silently rewritten (CFG-006).
- **Deprecating a still-referenced definition:** deprecation stops new use but retains the definition for historical resolution (CFG-004).
- **Effective-dated overlap:** rejected; periods must be coherent (CFG-005).
- **Secret in an export/log:** prevented; sensitive values redacted (CFG-009).
- **Dynamic-field schema change after data exists:** versioned; old values resolve against their creation-version (CFG-010).
- **App-version default change vs client override:** client overrides persist; only un-overridden keys see new defaults.

## 14. Failure Scenarios
- **Unregistered-key write/read:** rejected (CFG-001).
- **Invalid value:** rejected at write with the specific violation (CFG-002).
- **Disallowed-scope override:** rejected (CFG-003).
- **In-place edit of a published version:** rejected (CFG-004).
- **Ungoverned high-impact change / locked-after-use mutation:** blocked (CFG-007).
- **Cache unavailable:** resolve from source (CFG-008); never serve a guessed value.
- **Cross-deployment access:** structurally impossible (CFG-011).

## 15. Exception Handling Rules
- Configuration is declared-before-use, validated-before-store, and versioned-before-serve.
- Resolution is always deterministic (most-specific-wins, else default); never ambiguous or null.
- Governance (approval, immutability, rollback) is enforced; self-approval impossible.
- Sensitive values fail closed and are never exposed in reads, logs, exports, or audit values.

## 16. Compliance Considerations
- **Reproducibility & auditability:** versioned, stamped configuration makes every dependent record (financial, academic) reproducible and defensible — central to audits, accreditation, and disputes.
- **Change accountability:** every configuration change is attributable, governed, and reversible-by-record.
- **Secret management:** sensitive config protected per security baseline (Blueprint Part F).
- **Tenant isolation:** per-deployment configuration upholds client data separation.

## 17. Future Considerations
- Configuration-as-code with GitOps-style review for high-impact settings.
- Policy simulation ("what changes if I set X?") before publishing.
- Configuration drift detection and compliance baselines per client.
- Self-service definition catalog browsing for admins with safe guardrails.
