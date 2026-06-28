# 03 — Institute Use Cases

Transforms the Institute business rules (`INST-001`…`INST-008`) into use cases. The institute is the primary scope boundary; type seeds templates but never hard-codes behavior.

## 1. Primary Actors
Organization Administrator (manages all institutes in the deployment), Institute Administrator (completes setup, manages one institute).

## 2. Secondary Actors
System (uniqueness, lifecycle, scope cascade, template seeding), Configuration Engine (type templates, overrides), Workflow Engine (suspension/archival approvals), Audit Service.

## 3. Goals
Stand up institutes with correct type-seeded templates; gate activation on mandatory setup; isolate each institute as a scope; suspend/archive without data loss; preserve academic/financial history permanently.

## 4. User Journeys
- **Create & launch:** org admin creates an institute (name, unique code, type, locale) → type template seeds structure/terminology/defaults → institute admin completes mandatory setup → activate.
- **Operate:** institute scopes all its data; config overrides resolve most-specific-wins.
- **Wind down:** suspend (access blocked, data intact) or archive (read-only history) — never hard-delete a data-bearing institute.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core/CRUD | UC-INST-001 | Create Institute | High |
| Core | UC-INST-002 | Complete Setup & Activate | High |
| CRUD | UC-INST-003 | View Institute(s) | Medium |
| CRUD | UC-INST-004 | Update Institute Details | Medium |
| Core | UC-INST-005 | Suspend Institute | High |
| Core | UC-INST-006 | Archive Institute | Critical |
| Approval | UC-INST-007 | Approve Suspension/Archival | High |
| Config | UC-INST-008 | Manage Institute Config Overrides | High |
| Config | UC-INST-009 | Apply / Re-seed Type Template (draft only) | Medium |
| Search | UC-INST-010 | Search Institutes | Medium |
| Reporting | UC-INST-011 | Institute Status & Cross-Institute Summary | Medium |
| Import | UC-INST-012 | Import Institute Setup/Config Template | Low |
| Export | UC-INST-013 | Export Institute Configuration | Low |
| Workflow | UC-INST-014 | Reactivate Archived Institute (governed) | Medium |
| Exception | UC-INST-015 | Activation Blocked (incomplete setup) | High |
| Exception | UC-INST-016 | Type-Change-With-Data Blocked | High |

*Bulk:* institute creation is rarely bulk; bulk applies to config seeding (UC-INST-012). 

---

## 6. Detailed Specifications (high-value use cases)

### UC-INST-001 — Create Institute
- **Module:** Institute · **Priority:** High
- **Actors:** Organization Administrator (primary), System
- **Goal:** Create a new institute with a type-seeded configuration baseline.
- **Description:** Org admin creates an institute (name, unique code, type, locale); the system seeds type-based templates as editable configuration.
- **Business Rules Applied:** INST-001, INST-002.
- **Preconditions:** Actor holds `org.institute.manage`; deployment provisioned.
- **Trigger:** Org admin submits a new institute.
- **Main Success Scenario:**
  1. Admin enters name, unique code, type (school/college/university/madrasa/coaching/training), locale.
  2. System enforces code uniqueness (deployment-wide) and locks the code.
  3. System applies the type template, seeding structure/terminology/grading/fee defaults as editable config.
  4. Institute is created in `DRAFT`.
- **Alternative Flows:** A1) Single-institute deployment → a default institute may be auto-created at provisioning.
- **Exception Flows:** E1) Duplicate code → reject. E2) Unknown type / missing template → reject and flag.
- **Validation Rules:** Code unique, format-valid, immutable after creation; type ∈ catalog; template exists (INST-001/002).
- **Permissions Required:** `org.institute.manage`.
- **Notifications Triggered:** None at draft; `INSTITUTE_ACTIVATED` later.
- **Audit Events Generated:** `INSTITUTE_CREATED` (code), `INSTITUTE_TYPE_TEMPLATE_APPLIED`.
- **Data Created:** Institute (DRAFT); seeded config values (versioned).
- **Data Updated:** None.
- **Data Deleted:** None.
- **Post Conditions:** Institute exists in `DRAFT`, ready for setup.
- **Related Use Cases:** UC-INST-002, UC-INST-009.
- **Acceptance Criteria:**
  - Given a unique code and valid type, When the institute is created, Then type templates are seeded as editable config and the code is locked.
  - Given a duplicate code, When creation is attempted, Then it is rejected.
- **Edge Case Analysis:**
  - *Invalid Input:* malformed code/locale rejected.
  - *Permission Failure:* non-org-admin → 403.
  - *Concurrent Update:* two admins creating the same code → one wins, the other rejected (uniqueness).
  - *Duplicate Data:* duplicate code blocked (INST-001).
  - *System Failure:* template service down → block creation (cannot seed).
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* create each type; defaults seeded and editable.
  - *Negative:* duplicate code; unknown type; missing template.
  - *Boundary:* code at format limits; single-institute default-creation path.

### UC-INST-002 — Complete Setup & Activate
- **Module:** Institute · **Priority:** High
- **Actors:** Institute Administrator (primary), System
- **Goal:** Make an institute operational only after mandatory setup.
- **Description:** Admin completes the required checklist (≥1 campus or default campus, locale/terminology, required config) and activates; operational modules unlock only when `ACTIVE`.
- **Business Rules Applied:** INST-003, INST-005, CAMP-002, SESS-001.
- **Preconditions:** Institute in `DRAFT`; actor holds `institute.setup.manage`.
- **Trigger:** Admin requests activation.
- **Main Success Scenario:**
  1. Admin completes the setup checklist.
  2. System validates the checklist (campus present, locale set, required config).
  3. System activates the institute (`DRAFT → ACTIVE`).
  4. Operational modules become available within the institute scope.
- **Alternative Flows:** A1) Auto default campus created if none defined (CAMP-002).
- **Exception Flows:** E1) Activation with incomplete setup → blocked with the precise missing list (UC-INST-015).
- **Validation Rules:** All mandatory checklist items satisfied (INST-003).
- **Permissions Required:** `institute.setup.manage`.
- **Notifications Triggered:** `INSTITUTE_ACTIVATED` to org/institute admins.
- **Audit Events Generated:** `INSTITUTE_ACTIVATED` (checklist snapshot).
- **Data Created:** Default campus (if auto-created).
- **Data Updated:** Institute state `DRAFT → ACTIVE`.
- **Data Deleted:** None.
- **Post Conditions:** Institute `ACTIVE`; scope usable downstream.
- **Related Use Cases:** UC-INST-001, UC-CAMP-001, UC-SESS-001.
- **Acceptance Criteria:**
  - Given a complete setup, When activation is requested, Then the institute becomes `ACTIVE` and modules unlock.
  - Given incomplete setup, When activation is attempted, Then it is blocked listing exactly what's missing.
- **Edge Case Analysis:**
  - *Invalid Input:* invalid config values block activation.
  - *Permission Failure:* non-setup role → 403.
  - *Concurrent Update:* two admins activating → idempotent; one wins.
  - *Duplicate Data:* N/A.
  - *System Failure:* checklist evaluation error → activation blocked, not partially active.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* activate with full checklist; auto default-campus path.
  - *Negative:* missing campus/locale/config; non-draft activation.
  - *Boundary:* exactly-minimum checklist satisfied.

### UC-INST-006 — Archive Institute
- **Module:** Institute · **Priority:** Critical
- **Actors:** Organization Administrator (primary), System, Approver
- **Goal:** Permanently close an institute without destroying its history.
- **Description:** A data-bearing institute is archived (read-only, preserved), never hard-deleted; active dependencies must be handled first; may require approval.
- **Business Rules Applied:** INST-008, INST-004, SESS-007.
- **Preconditions:** Actor holds `org.institute.manage`; approval obtained if configured.
- **Trigger:** Admin requests archival.
- **Main Success Scenario:**
  1. Admin requests archival with a reason.
  2. System checks for blocking dependencies (active sessions/enrollments) and guides resolution.
  3. (If configured) approval is obtained via the Workflow Engine.
  4. System transitions the institute to `ARCHIVED`, making data read-only.
- **Alternative Flows:** A1) An empty `DRAFT` institute with no data may be hard-deleted instead.
- **Exception Flows:** E1) Active session/enrollments present → block until handled (SESS-007). E2) Hard-delete of a data-bearing institute → forbidden.
- **Validation Rules:** No blocking dependencies (or handled); hard-delete only for never-used drafts (INST-008).
- **Permissions Required:** `org.institute.manage` (+ approval).
- **Notifications Triggered:** `INSTITUTE_ARCHIVED` to org admins.
- **Audit Events Generated:** `INSTITUTE_ARCHIVED` (reason).
- **Data Created:** None.
- **Data Updated:** Institute state → `ARCHIVED`; data read-only.
- **Data Deleted:** None (archive, not delete).
- **Post Conditions:** Institute archived; history retrievable; new operations blocked.
- **Related Use Cases:** UC-INST-005, UC-INST-014, UC-SESS-006.
- **Acceptance Criteria:**
  - Given a data-bearing institute, When archival is requested and dependencies are resolved, Then it becomes read-only and history is preserved.
  - Given a hard-delete attempt on a data-bearing institute, When requested, Then it is rejected.
- **Edge Case Analysis:**
  - *Invalid Input:* archival without reason (if required) rejected.
  - *Permission Failure:* non-org-admin → 403.
  - *Concurrent Update:* archival during active operations → blocked until clear.
  - *Duplicate Data:* N/A.
  - *System Failure:* dependency check fails → block (no partial archive).
  - *Workflow Failure:* approval unresolved → archival pending, escalates.
- **QA Coverage:**
  - *Positive:* archive a cleanly-closed institute; hard-delete an empty draft.
  - *Negative:* archive with active session; hard-delete with data.
  - *Boundary:* archive immediately after the last session closes.

---

## 7. Compact Specifications (routine use cases)

- **UC-INST-003 — View Institute(s)** · *Medium* · Rules: AUTHZ-002, INST-006. Scoped list/detail; institute admin sees only their institute(s). *Permissions:* `institute.setup.manage`/`org.institute.manage`. *QA:* scoped visibility; org-admin sees all.
- **UC-INST-004 — Update Institute Details** · *Medium* · Rules: INST-001 (code immutable), INST-004. Edit mutable fields (name, contact, locale); code & (post-data) type are locked. *Audit:* `INSTITUTE_UPDATED`. *Edge:* code/type edits rejected appropriately. *QA:* mutable edit; locked-field edit rejected.
- **UC-INST-005 — Suspend Institute** · *High* · Rules: INST-007, AUTH-005. Suspend → scoped users lose operational access (cascade), org admin retains; data intact. *Permissions:* `org.institute.manage`. *Audit:* `INSTITUTE_SUSPENDED/REINSTATED`. *Edge:* in-flight workflows pause, resume on reinstate. *QA:* cascade blocks scoped users; reinstate restores; data untouched.
- **UC-INST-007 — Approve Suspension/Archival** · *High* · Rules: WFL-004. Approver approves high-impact lifecycle change. *Edge:* SoD/escalation. *QA:* approval gates the action; timeout escalation.
- **UC-INST-008 — Manage Institute Config Overrides** · *High* · Rules: INST-005, CFG-002/004. Set institute-level overrides (most-specific-wins). *Permissions:* `institute.config.manage`. *Audit:* config change. *Edge:* invalid override rejected, inherits. *QA:* override wins for institute; invalid rejected.
- **UC-INST-009 — Apply/Re-seed Type Template (draft only)** · *Medium* · Rules: INST-002, INST-004, CFG-007. Re-seed templates only in `DRAFT` with no data. *Edge:* blocked once data exists (immutable-after-use). *QA:* re-seed in draft; blocked with data.
- **UC-INST-010 — Search Institutes** · *Medium* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-INST-011 — Institute Status & Cross-Institute Summary** · *Medium* · Rules: REP-002, REP-006. Status report; cross-institute summary is org-only, deployment-bound. *Permissions:* `report.run`/`report.cross_institute`. *Edge:* never cross-deployment. *QA:* org-only cross-institute; institute-admin denied cross view.
- **UC-INST-012 — Import Institute Setup/Config Template** · *Low* · Rules: CFG-002. Import a config template (validated). *Edge:* invalid values rejected. *QA:* valid import; invalid rejected.
- **UC-INST-013 — Export Institute Configuration** · *Low* · Rules: REP-005, CFG-009. Export config (sensitive values redacted). *Permissions:* `institute.config.manage`. *Edge:* secrets never exported. *QA:* config exported; secrets redacted.
- **UC-INST-014 — Reactivate Archived Institute (governed)** · *Medium* · Rules: INST-008. Explicit, audited, policy-gated reactivation. *Permissions:* elevated. *Audit:* reactivation. *Edge:* rare; not a casual toggle. *QA:* governed reactivation; casual toggle blocked.
- **UC-INST-015 — Activation Blocked (incomplete setup) (Exception)** · *High* · Rules: INST-003. Activation lists missing items. *QA:* each missing item surfaced; activation blocked.
- **UC-INST-016 — Type-Change-With-Data Blocked (Exception)** · *High* · Rules: INST-004, CFG-007. Type change after data exists is rejected; remedy is a new institute. *QA:* draft type-change allowed; data-bearing blocked.

## 8. Module-level QA & Edge Themes
- **Archive-not-delete:** assert no code path hard-deletes a data-bearing institute.
- **Scope cascade:** suspension must block exactly the right users (scoped-only) while org admins retain control.
- **Type lock:** the irreversible-after-data behavior is a key negative suite.
