# 27 — Configuration Engine Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint (Platform Core; D2/D3/D36), Business Rules Catalog (`CFG-001…011`), and Use Case Repository (`UC-CFG-001…020`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Provide the meta-engine that makes the system configuration-driven with no hard-coded business rules: a definition registry, typed validation, scoped most-specific-wins resolution, immutable versioning with consumer stamping, effective-dating, governed rollback, change governance with immutable-after-use locks, fast cache propagation, secret protection, dynamic custom-field schemas, and per-deployment isolation.

**Business Goal.** Let every other module declare, validate, version, and resolve configuration so behavior is per-deployment configurable, reproducible, and governed — without code branches per client.

**Scope.** Definition registry; typed/validated values; scoped resolution (most-specific-wins, default fallback); immutable versioning + consumer stamping; effective-dating; governed rollback; high-impact approval + immutable-after-use; config-version cache invalidation; sensitive/secret protection; dynamic custom-field schemas; per-deployment isolation + type templates.

**Out of Scope.** Domain logic of any consuming module (they own meaning; the engine validates/versions/resolves). Workflow execution (Workflow Engine — invoked for high-impact change approval). Audit storage (Audit module — receives change events). Report-definition semantics (Reporting — though report defs are configuration, D42).

---

## 2. Actors

**Primary Actors.** Platform Engineer / Developer (registers definitions at build time), Organization Administrator (org-level values, rollback, templates), Institute/Campus Administrator (scoped overrides), System (resolution, cache invalidation).

**Secondary Actors.** All 29 other modules (consumers of resolution; stampers of versions), Workflow Engine (high-impact approval), Audit, Authorization (config permissions).

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Definition registry | Declare every configurable setting before any value is set/resolved. | Critical | — |
| FR-002 | Typed/validated values | Validate values by type/range/enum at write. | Critical | FR-001 |
| FR-003 | Scoped resolution | Resolve most-specific-wins across scopes, default fallback. | Critical | Institute/Campus scope |
| FR-004 | Immutable versioning + stamping | Each publish creates an immutable version; expose version for consumer stamping. | Critical | FR-002 |
| FR-005 | Effective-dating | Apply changes effective from a future date. | High | FR-004 |
| FR-006 | Governed rollback (forward-only) | Roll back to a prior version with reason; forward-only. | High | FR-004 |
| FR-007 | High-impact approval & immutable-after-use | Govern high-impact changes (SoD); lock settings once dependents exist. | Critical | Workflow, AUTHZ-009 |
| FR-008 | Cache invalidation / config-version | Bump config-version on change; invalidate caches; fast propagation. | High | Redis |
| FR-009 | Sensitive/secret protection | Encrypt/mask secrets; exclude from exports/audit values. | Critical | Audit (AUD-004) |
| FR-010 | Dynamic custom-field schemas | Register/validate custom-field schemas (engine-validated). | High | Student/Admission |
| FR-011 | Per-deployment isolation + templates | Isolate config per deployment; seed from type templates. | High | Institute (INST-002) |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Definition registry | No undeclared config. | Discipline; safety. |
| Typed validation | Configurable ≠ unvalidated. | Correctness at write. |
| Scoped resolution | Most-specific-wins + default. | Consistent yet flexible. |
| Versioning + stamping | Immutable, reproducible. | Defensible records. |
| Effective-dating | Future-dated changes. | Planned rollouts. |
| Governed rollback | Forward-only, reasoned. | Safe recovery. |
| Immutable-after-use locks | One lock mechanism (C-06). | Integrity. |
| Secret protection | Never logged/exported. | Security. |
| Dynamic schemas | Engine-validated custom fields. | Flexible data models. |
| Per-deployment isolation | No cross-client leakage. | Multi-tenant safety. |

---

## 5. Screens

Definition Registry (admin); Configuration Browser (resolved / by scope); Set Value (scoped); Effective-Dated Change; Governed Rollback; High-Impact Change & Immutable-After-Use; Dynamic Custom-Field Schema; Sensitive/Secret Configuration; Apply Type Template (per-deployment seed); Deprecate Definition; Cache/Config-Version Management; Configuration Audit & Drift Report; Import/Bulk-Set; Export (secrets redacted); Change Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Definition Registry | Register, Deprecate, View References | — |
| Configuration Browser | View Resolved, View by Scope, Search | — |
| Set Value | Set (validated), Publish Version, Future-Date | Bulk Set/Import |
| Rollback | Roll Back to Version (reason) | — |
| High-Impact Change | Submit (→ approval), View Lock Status | — |
| Dynamic Schema | Register Schema, Validate, Version | — |
| Sensitive Config | Set (encrypted), Mask, Restrict | — |
| Type Template | Preview, Apply to Draft, Discard | — |
| Audit & Drift Report | Run, Filter, Export | Export |

---

## 7. Forms

**Register Definition** — `key` (text, required, namespaced), `dataType` (select), `allowedValues/range` (typed), `default`, `allowedScopes` (multi), `sensitivity` (toggle), `immutableAfterUse` (toggle), `validation`. Validation: complete, unique key (CFG-001/002); immutable once referenced.

**Set Value** — `definitionKey` (select), `value` (typed per definition), `scope` (picker), `effectiveDate` (optional). Validation: registered key (CFG-001 / UC-CFG-018); typed/valid (CFG-002 / UC-CFG-019); allowed scope (CFG-003); immutable-after-use respected (CFG-007 / UC-CFG-020); new immutable version on publish (CFG-004).

**Dynamic Custom-Field Schema** — `entity`, fields (`name`, `type`, `required`, `validation`). Validation: engine-validated; versioned; historical values resolve against their schema version (CFG-010).

**Sensitive Config** — `key`, `secretValue` (encrypted). Validation: encrypted at rest, masked on read, excluded from export/audit values (CFG-009 / AUD-004).

**Rollback** — `key/scope`, `targetVersion`, `reason` (text, required). Validation: forward-only; does not rewrite produced records (CFG-006).

---

## 8. Search & Filter Requirements

**Definitions/Values:** by key, category, scope, sensitivity, status (active/deprecated), drift-from-default. Sorting: key/scope/version. Pagination: server-side, 25 default. Scope-bound; secrets masked.

---

## 9. Table Requirements

**Definition table:** Key, Type, Default, Scopes, Sensitive?, Immutable-After-Use, Status. **Value table:** Key, Scope, Value (masked if secret), Version, Effective Date. Sorting on Key/Version. Filtering as above. Export (governed, secrets redacted). Bulk: set/import.

---

## 10. Workflow Requirements

**Trigger events:** register, set value, effective-dated change, rollback, high-impact change, schema register, deprecate, cache bump. **Status changes:** value version `DRAFT → PUBLISHED → SUPERSEDED/ROLLED-BACK`; definition `ACTIVE → DEPRECATED`. **Approvals:** high-impact changes via Workflow Engine (SoD). **Notifications:** high-impact change/approval; future-dated activation. **Audit:** definition/value changes (before/after for non-sensitive; change-occurred only for secrets), rollbacks, lock-blocked attempts (immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Register/deprecate definition | `config.definition.manage` (platform) |
| Set scoped value | `config.value.manage` (scoped) |
| Approve high-impact change | `config.value.approve` (SoD) |
| Rollback | `config.rollback` |
| Manage sensitive config | `config.sensitive.manage` |
| Manage dynamic schema | `config.schema.manage` |
| View configuration | `config.view` |
| Import/export | `config.import`, `config.export` |

Definition registration is tightly held (platform); value management is scope-bound; secrets gated.

---

## 12. Business Rule References

CFG-001 (definition registry — no undeclared config), CFG-002 (typed/validated values), CFG-003 (scoped values & most-specific-wins), CFG-004 (versioning & immutable published versions), CFG-005 (effective-dating), CFG-006 (governed rollback), CFG-007 (change governance, approval & immutable-after-use), CFG-008 (config-version cache invalidation & fast propagation), CFG-009 (sensitive/secret protection), CFG-010 (dynamic custom-field schemas engine-validated), CFG-011 (per-deployment isolation & type templates). Cross-cutting: INST-002/004 (type templates/locks — C-06), AUTHZ-009 (SoD), AUD-004 (no secrets in audit), WFL-002/004 (approval), AUD-001.

## 13. Use Case References

UC-CFG-001 (Register Definition), UC-CFG-002 (Set Scoped Value), UC-CFG-003 (Resolve — most-specific-wins), UC-CFG-004 (Effective-Dated Change), UC-CFG-005 (Governed Rollback), UC-CFG-006 (High-Impact Approval & Immutable-After-Use), UC-CFG-007 (Dynamic Custom-Field Schema), UC-CFG-008 (Sensitive/Secret Config), UC-CFG-009 (Apply Type Template), UC-CFG-010 (View Configuration), UC-CFG-011 (Deprecate Definition), UC-CFG-012 (Cache/Config-Version), UC-CFG-013 (Search), UC-CFG-014 (Audit & Drift Report), UC-CFG-015 (Import/Bulk-Set), UC-CFG-016 (Export — secrets redacted), UC-CFG-017 (Change-Approval Workflow), UC-CFG-018 (Unregistered-Key Use Blocked), UC-CFG-019 (Invalid Value Blocked), UC-CFG-020 (Immutable-After-Use Change Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Register / deprecate definition | POST/DELETE | Platform Engineer |
| Set scoped value (new version) | POST | Admin |
| Resolve value (most-specific-wins) | GET (internal) | System (any module) |
| Effective-dated change | POST | Admin |
| Rollback (forward-only) | POST | Admin |
| High-impact change (approval) | POST | Admin (→ approval) |
| Register/validate dynamic schema | POST | Admin |
| Manage sensitive/secret config | POST/PUT | Security Admin |
| Apply type template (draft seed) | POST | Admin |
| View configuration (resolved/by scope) | GET | Admin |
| Cache invalidation / config-version | POST | Admin/System |
| Audit & drift report | GET | Admin |
| Import / export (secrets redacted) | POST/GET | Admin |

Resolution is the hot path consumed by all modules; it returns value + version for stamping (CFG-003/004). Unregistered-key use and invalid values are rejected (CFG-001/002).

---

## 15. Database Requirements

**Entities:** `ConfigDefinition` (key, type, default, allowedScopes, sensitivity, immutableAfterUse, validation), `ConfigValueVersion` (key, scope, value, version, effectiveDate, immutable), `ConfigVersionPointer` (active per key/scope), `SecretConfig` (encrypted), `DynamicSchema` (entity, fields, version), `ConfigChangeLog`. **Relationships:** Definition 1—* ValueVersion; Definition 1—1 (active pointer per scope). **Indexes:** unique(ConfigDefinition.key), index(ConfigValueVersion.key, scope, version), index(ConfigValueVersion.effectiveDate), index(DynamicSchema.entity, version). Resolution cached in Redis keyed by config-version (CFG-008); published versions immutable (CFG-004); secrets encrypted (CFG-009).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| In-App | High-impact change requests/approvals; future-dated activation; lock-blocked attempts. |
| Email | High-impact change approval requests/decisions. |
| SMS/Push | Not used. |

---

## 17. Audit Requirements

Log: definition registration/deprecation, value publishes (before/after for non-sensitive; change-occurred metadata only for secrets — AUD-004), effective-dated changes, rollbacks, high-impact approvals, immutable-after-use blocked attempts, schema changes. Record who/when/before/after. Secrets never written to audit values. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Configuration audit (changes/versions), Drift from defaults/baselines, Override coverage by scope, Deprecated-but-referenced definitions. **Exports:** governed config export (secrets redacted — CFG-009). **Dashboards:** configuration health (drift, pending high-impact changes, secret usage).

---

## 19. Error Handling

**Validation:** unregistered key, invalid value, immutable-after-use with dependents → specific errors (UC-CFG-018/019/020). **Permission:** definition registration without platform rights → 403; self-approval of high-impact → SoD block. **Workflow:** high-impact change pending → not effective until approved. **System:** cache unavailable → resolve from source (correctness over speed); secret store unavailable → fail closed for secret reads.

---

## 20. Edge Cases

**Concurrent updates:** two value sets → last-write publishes a new version (no lost history). **Duplicate data:** duplicate definition/value version prevented. **Partial failures:** bulk import partial → per-row report; immutable-after-use respected. **Rollback:** rollback is forward-only — does not rewrite already-produced records. **Resolution race:** value change mid-resolution → in-flight operations use the version pinned at start; next resolve is fresh.

---

## 21. Acceptance Criteria

**Functional.** No value can be set/resolved for an unregistered key; values are typed/validated at write; resolution is most-specific-wins with default fallback and never null; each publish creates an immutable version and resolution exposes the version for stamping; changes can be effective-dated; rollback is forward-only; high-impact changes require SoD approval and immutable-after-use settings lock once dependents exist; secrets are encrypted, masked, and never written to exports or audit values; custom-field schemas are engine-validated; configuration is isolated per deployment and seedable from type templates.

**Business.** The system is genuinely configuration-driven with no hard-coded rules; every reproducible record across the platform stamps the exact config version it used; high-impact and locked settings are protected; secrets never leak.

---

## 22. Future Enhancements

Visual configuration designer with live validation; change-set staging/preview across scopes; config diff/compare between deployments; automated drift remediation; policy-as-config simulation ("what changes if…"); secret rotation workflows; configuration marketplace of type templates.
