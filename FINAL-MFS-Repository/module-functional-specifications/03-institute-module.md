# 03 — Institute Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`INST-001…008`), and Use Case Repository (`UC-INST-001…016`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Manage the institute — the mandatory top-level organizational scope within a deployment: creation, type-driven template seeding, mandatory setup before activation, configuration inheritance/override, suspension cascade, and archival.

**Business Goal.** Provide a clean, isolated organizational root that every other module scopes to, so a single deployment can host multiple institutes with correct data boundaries and per-institute configuration.

**Scope.** Institute CRUD; unique institute code; institution-type selection driving templates (not behavior-by-code); setup completion + activation gating; config inheritance/override (org → institute); suspension (with cascade) and archival (never hard-delete); restricted type change; cross-institute summary (org-only).

**Out of Scope.** Campus specifics (Campus module). Per-deployment provisioning/infrastructure (DevOps/Implementation Plan). Configuration mechanics (Configuration Engine — this module consumes it). Academic structure (Session/Class modules).

---

## 2. Actors

**Primary Actors.** Organization Administrator (creates/activates/suspends/archives institutes), Institute Administrator (manages own institute config), System (template seeding, cascade, scope enforcement).

**Secondary Actors.** Configuration Engine (templates, inheritance/override), Authorization (institute as mandatory scope), Campus/Session/all modules (scoped children), Workflow Engine (suspension/archival approval), Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Create institute | Create an institute with a unique code and an institution type. | Critical | Config (templates) |
| FR-002 | Type-driven template seeding | Institution type seeds default configuration/templates; type does not branch code. | High | Configuration Engine |
| FR-003 | Mandatory setup before activation | Block activation until required setup is complete. | Critical | Config, Campus (default), Session |
| FR-004 | Activate institute | Transition to active once setup is complete. | Critical | FR-003 |
| FR-005 | Config inheritance & override | Inherit org defaults; allow institute-level overrides (most-specific-wins). | High | Configuration Engine |
| FR-006 | Institute as mandatory scope | Every domain record is scoped to an institute; no global records. | Critical | Authorization |
| FR-007 | Restricted type change | Disallow type change once dependent data exists (immutable-after-use). | High | CFG-007 lock |
| FR-008 | Suspension cascade | Suspending an institute suspends access to its scoped operations. | High | Authorization, Auth (revocation) |
| FR-009 | Archive, never hard-delete | Archive institutes; retain data per retention. | High | Retention/Audit |
| FR-010 | Cross-institute summary | Org-only aggregate view across institutes (deployment-bound). | Medium | Reporting (REP-006) |
| FR-011 | Governed reactivation | Reactivate an archived institute under approval. | Medium | Workflow Engine |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Institute lifecycle | Create → setup → activate → suspend/archive → (governed) reactivate. | Clean organizational root management. |
| Type templates | Type seeds sensible defaults editable afterward. | Fast onboarding; no hard-coded behavior. |
| Setup gating | Activation blocked until complete. | Prevents half-configured live institutes. |
| Inheritance/override | Org defaults with institute overrides. | Consistent yet flexible configuration. |
| Suspension cascade | One switch halts scoped operations. | Fast, safe operational control. |
| Archival + retention | No hard delete; statutory retention. | Compliance; recoverability. |
| Org-only cross-institute view | Aggregate insight, deployment-bound. | Multi-institute oversight without leakage. |

---

## 5. Screens

Institute List; Institute Detail (overview/status/setup checklist); Institute Create; Institute Setup Wizard (multi-step); Institute Edit; Config Overrides (institute scope); Suspend Institute (confirm + reason); Archive Institute (confirm + reason); Reactivate Institute (governed); Apply/Re-seed Type Template (draft only); Cross-Institute Summary (org); Institute Config Import/Export; Approvals (suspension/archival).

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Institute List | Create, Open, Search/Filter, Export, Cross-Institute Summary | — |
| Institute Detail | Edit, Activate (if ready), Suspend, Archive, Manage Overrides, View Setup Checklist | — |
| Setup Wizard | Save Step, Validate Setup, Complete & Activate | — |
| Config Overrides | Add Override, Edit, Reset to Inherited, Save (versioned) | — |
| Suspend / Archive | Confirm with Reason, Submit (→ approval) | — |
| Reactivate | Request Reactivation (→ approval) | — |
| Apply Template (draft) | Preview, Apply to Draft, Discard | — |
| Cross-Institute Summary | Run, Filter, Export | Export |

---

## 7. Forms

**Create Institute** — `name` (text, required), `code` (text, required, unique within deployment), `institutionType` (select, required: school/college/university/madrasa/coaching/training), `address`/`contact` (text). Defaults: status `DRAFT`. Validation: code unique (INST-001); type valid; type seeds template (INST-002).

**Setup Wizard** — required completeness items: at least one campus (default guaranteed — CAMP-002), at least one academic session defined, core configuration present, an institute administrator assigned. Validation: all required items complete before activation (INST-003).

**Config Override** — `definitionKey` (select from registered config), `value` (typed per definition), `scope` = this institute. Validation: key registered (CFG-001), typed (CFG-002), override allowed at institute level (CFG-003), versioned on publish (CFG-004).

**Suspend / Archive** — `reason` (text, required), `effectiveDate` (date, default now). Validation: archival requires no blocking active dependents per policy; both routed to approval (INST-007).

---

## 8. Search & Filter Requirements

**Institutes:** by name, code, type, status (draft/active/suspended/archived), created range. Sorting: name/code/status/created. Pagination: server-side, 25 default. Org admins see all institutes in the deployment; institute admins see only their own.

---

## 9. Table Requirements

**Institute table:** Name, Code, Type, Status, #Campuses, Current Session, Created. Sorting on Name/Code/Status/Created. Filtering as above. Export (governed). No bulk destructive actions (archival is per-institute, governed). Cross-institute summary table: Institute, Students, Staff, Active Session, Status (org-only).

---

## 10. Workflow Requirements

**Trigger events:** create, setup-complete, activate, suspend, archive, reactivate, type-change attempt, override publish. **Status changes:** `DRAFT → ACTIVE → SUSPENDED ⇄ ACTIVE → ARCHIVED → (governed) ACTIVE`. **Approvals:** suspension, archival, reactivation via Workflow Engine. **Notifications:** activation, suspension (to institute admins), archival, reactivation. **Audit:** all lifecycle transitions with reason and actor (immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| View institute(s) | `institute.view` |
| Create institute | `institute.create` (org) |
| Update institute | `institute.update` |
| Activate institute | `institute.activate` |
| Suspend / archive | `institute.suspend`, `institute.archive` |
| Reactivate (governed) | `institute.reactivate` |
| Manage config overrides | `institute.config.manage` |
| Cross-institute summary | `institute.summary.view` (org) |
| Import/Export config | `institute.config.export` |

Institute creation and cross-institute views are org-level; everything else is scope-bound to the institute (INST-006).

---

## 12. Business Rule References

INST-001 (code uniqueness), INST-002 (type drives templates not code), INST-003 (mandatory setup before activation), INST-004 (restricted type change), INST-005 (config inheritance & override), INST-006 (institute is a mandatory scope), INST-007 (suspension cascade), INST-008 (archive never hard-delete). Cross-cutting: CFG-001/002/003/004/007 (config registry/typing/scoping/versioning/locks), AUTHZ-002 (scope), REP-006 (org-only cross-institute), AUD-001.

## 13. Use Case References

UC-INST-001 (Create), UC-INST-002 (Complete Setup & Activate), UC-INST-003 (View), UC-INST-004 (Update), UC-INST-005 (Suspend), UC-INST-006 (Archive), UC-INST-007 (Approve Suspension/Archival), UC-INST-008 (Manage Config Overrides), UC-INST-009 (Apply/Re-seed Template draft), UC-INST-010 (Search), UC-INST-011 (Status & Cross-Institute Summary), UC-INST-012/013 (Import/Export config), UC-INST-014 (Reactivate), UC-INST-015 (Activation Blocked), UC-INST-016 (Type-Change-With-Data Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Create institute | POST | Org Admin |
| List / get institute(s) | GET | Admin |
| Update institute | PUT | Admin |
| Validate setup / activate | POST | Admin |
| Suspend / archive | POST | Admin (→ approval) |
| Reactivate (governed) | POST | Org Admin (→ approval) |
| Get/set config overrides | GET/PUT | Institute Admin |
| Apply type template (draft) | POST | Admin |
| Cross-institute summary | GET | Org Admin |
| Import / export config | POST/GET | Admin |

---

## 15. Database Requirements

**Entities:** `Institute` (code, type, status, setup-state), `InstituteConfigOverride` (via Config versioning), `InstituteSetupChecklist`. **Relationships:** Institute 1—* Campus; Institute 1—* AcademicSession; Institute is the scope root referenced by all domain entities. **Indexes:** unique(Institute.code) per deployment, index(Institute.status), index(Institute.type). Type is immutable-after-use (enforced via CFG-007 lock once dependents exist).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Activation confirmation; suspension/archival notice to institute admins; reactivation. |
| In-App | Setup-incomplete reminders; status changes; pending approvals. |
| SMS/Push | Not used by default. |

---

## 17. Audit Requirements

Log: create, setup completion, activation, suspension, archival, reactivation, type-change attempts (allowed/blocked), config override publishes (before/after value, version). Record who/when/before/after. Lifecycle transitions carry the mandatory reason. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Institute status overview, Setup-completion, Config-override drift from org defaults, Cross-institute summary (org-only — REP-006). **Exports:** institute configuration export (governed). **Dashboards:** org admin multi-institute health (status, sessions, headcounts).

---

## 19. Error Handling

**Validation:** duplicate code, invalid type, incomplete setup on activation → specific errors (UC-INST-015). **Permission:** non-org create attempt → 403; institute admin acting outside own institute → 403/not-found. **Workflow:** suspension/archival pending approval → clear state. **System:** Config unavailable → activation blocked (cannot seed/resolve); type-change with dependents → blocked (UC-INST-016).

---

## 20. Edge Cases

**Concurrent updates:** two admins editing config → versioned, last-coherent wins. **Duplicate data:** duplicate code attempt → blocked. **Partial failures:** template seed partially applied → atomic seed or rollback to draft. **Rollback:** activation failing a late check → remains draft (no half-active institute). **Cascade race:** suspend during an in-flight scoped operation → operation denied at authorization (cascade authoritative).

---

## 21. Acceptance Criteria

**Functional.** An institute cannot activate until required setup is complete; institute code is unique within the deployment; institution type seeds templates and becomes locked once dependent data exists; institute-level config overrides resolve most-specific-wins; suspension immediately halts scoped operations; institutes are archived, never hard-deleted; cross-institute summaries are available only to org admins and never cross deployments.

**Business.** Every domain record is correctly isolated under exactly one institute; a multi-institute deployment maintains clean boundaries; configuration is consistent (inherited) yet flexible (overridable); lifecycle changes are governed, reasoned, and auditable.

---

## 22. Future Enhancements

Institute-level branding/white-label presets surfaced at setup; guided onboarding scorecards; institute cloning (config-only) for fast new-branch setup; configurable setup-checklist templates per institution type; delegated org-admin roles per institute group.
