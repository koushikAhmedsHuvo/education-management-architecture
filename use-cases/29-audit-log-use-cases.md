# 29 — Audit Log Use Cases

Transforms the Audit Log business rules (`AUD-001`…`AUD-008`) into use cases. The immutable, append-only, tamper-evident record every module emits to: reliable outbox capture, complete structured events, no stored secrets, restricted segregated access, audit-of-audit, fail-safe on unavailability, and erasure-aware pseudonymization with legal hold.

## 1. Primary Actors
All Modules / System (emit events), Auditor / Compliance Officer (read/investigate), Security Administrator (monitor), Regulator/Investigator (via governed export).

## 2. Secondary Actors
System (capture, immutability, tamper-evidence, retention), Configuration Engine (audit policy), all 29 modules (emitters), legal-hold/compliance process.

## 3. Goals
Capture every consequential action immutably and reliably (outbox); record complete structured events with correlation; never store secrets/sensitive values; restrict audit access (segregated from operations, no edit path); audit the auditors; fail closed for sensitive actions when audit is unavailable; reconcile erasure with retention via pseudonymization and legal hold.

## 4. User Journeys
- **Capture:** a module performs a consequential action → the audit event commits atomically with it (outbox) → persisted append-only and tamper-evident.
- **Investigate:** an auditor queries by actor/subject/time/correlation → the access itself is audited.
- **Fail-safe:** if audit storage is degraded, a sensitive action (grade change, payment) fails closed rather than proceeding unrecorded.
- **Erasure/hold:** a lawful erasure pseudonymizes audit references while retaining statutory records; a legal hold suspends erasure.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-AUD-001 | Capture Audit Event (immutable, outbox) | Critical |
| Core | UC-AUD-002 | Query / Investigate Audit | High |
| Core | UC-AUD-003 | Protect Sensitive Values in Audit | Critical |
| Core | UC-AUD-004 | Fail-Safe on Audit Unavailability | High |
| Core | UC-AUD-005 | Erasure Pseudonymization & Legal Hold | High |
| Core | UC-AUD-006 | Audit-of-Audit (access logging) | High |
| Admin | UC-AUD-007 | Manage Audit Policy & Retention | Medium |
| Admin | UC-AUD-008 | Verify Tamper-Evidence / Integrity | High |
| Search | UC-AUD-009 | Search Audit (correlated) | Medium |
| Reporting | UC-AUD-010 | Compliance / Accountability Report | Medium |
| Export | UC-AUD-011 | Governed Audit Export (regulator) | High |
| Workflow | UC-AUD-012 | Legal-Hold Application / Removal | Medium |
| Exception | UC-AUD-013 | Audit Edit / Delete Blocked | Critical |
| Exception | UC-AUD-014 | Unauthorized Audit Access Blocked | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-AUD-001 — Capture Audit Event (immutable, outbox)
- **Module:** Audit Log · **Priority:** Critical
- **Actors:** Any Module / System (primary)
- **Goal:** Record a consequential action immutably and reliably, with no action left unaudited and no orphan audit.
- **Description:** The action and its audit event commit atomically via the transactional outbox; the event (actor, action, target, scope, before/after, timestamp, correlation, source) is persisted append-only and tamper-evident.
- **Business Rules Applied:** AUD-001, AUD-002, AUD-003, AUD-004.
- **Preconditions:** A consequential action occurs (state change, access, decision).
- **Trigger:** Any consequential action across modules.
- **Main Success Scenario:**
  1. The action and its audit event are committed in the same transaction (outbox, AUD-002).
  2. The event records complete structured content with correlation (AUD-003).
  3. Sensitive values are masked/omitted (AUD-004); only change-occurred metadata stored for secrets.
  4. The event is persisted append-only and tamper-evident (AUD-001).
- **Alternative Flows:** A1) High-volume events partitioned for scale.
- **Exception Flows:** E1) Sensitive values present → recorded as change-occurred, value masked (UC-AUD-003). E2) Audit store unavailable → sensitive actions fail closed (UC-AUD-004).
- **Validation Rules:** Atomic with source; complete content; immutable/tamper-evident; no secrets (AUD-001/002/003/004).
- **Permissions Required:** System (emitted by modules).
- **Notifications Triggered:** Integrity/availability alerts only.
- **Audit Events Generated:** The event itself (this is the audit mechanism).
- **Data Created:** Immutable audit event.
- **Data Updated:** None (append-only).
- **Data Deleted:** None.
- **Post Conditions:** Consequential action permanently, immutably recorded.
- **Related Use Cases:** UC-AUD-003, UC-AUD-004, UC-NOT-005 (outbox parallel).
- **Acceptance Criteria:**
  - Given a consequential action, When it commits, Then its audit event commits atomically (no action without audit, no orphan audit).
  - Given a secret in the payload, When audited, Then only change-occurred metadata is stored (never the value).
  - Given a persisted event, When any edit is attempted, Then it is rejected (append-only).
- **Edge Case Analysis:**
  - *Invalid Input:* incomplete event for a consequential action rejected.
  - *Permission Failure:* N/A (system).
  - *Concurrent Update:* high-volume burst → partitioned capture keeps pace.
  - *Duplicate Data:* idempotent capture (dedup).
  - *System Failure:* outbox guarantees no orphan/lost; sensitive actions fail closed (AUD-004).
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* capture state-change/access/decision events.
  - *Negative:* secret stored (must not occur); orphan audit; edit attempt.
  - *Boundary:* burst at result-publish scale; clock-skew ordering.

### UC-AUD-004 — Fail-Safe on Audit Unavailability
- **Module:** Audit Log · **Priority:** High
- **Actors:** System (primary), Administrator
- **Goal:** Ensure high-sensitivity actions never proceed unrecorded when audit is degraded.
- **Description:** For high-sensitivity operations (financial movements, grade changes, security/privilege changes), if audit cannot be guaranteed, the action fails closed and alerts; lower-sensitivity actions may proceed with buffered/queued audit, never silently unrecorded.
- **Business Rules Applied:** AUD-007, AUD-002, EXM-007, AUTH-005.
- **Preconditions:** Audit subsystem degraded/unavailable.
- **Trigger:** A consequential action while audit is impaired.
- **Main Success Scenario:**
  1. The system detects audit unavailability.
  2. For high-sensitivity actions, it fails closed (blocks) and alerts (AUD-007).
  3. For lower-sensitivity actions, it buffers/queues audit and proceeds (never silently unrecorded).
  4. On recovery, buffered audit is flushed.
- **Alternative Flows:** A1) Degradation event recorded on recovery.
- **Exception Flows:** E1) Grade/financial change during outage → blocked until audit restored.
- **Validation Rules:** Sensitivity classification; high-sensitivity fail-closed; no silent loss (AUD-007).
- **Permissions Required:** System.
- **Notifications Triggered:** Degradation/fail-closed alerts to security/ops.
- **Audit Events Generated:** Degradation/fail-closed events (on recovery).
- **Data Created:** Buffered audit (lower-sensitivity).
- **Data Updated:** None.
- **Data Deleted:** None.
- **Post Conditions:** Sensitive actions never unaudited; system integrity preserved.
- **Related Use Cases:** UC-AUD-001, UC-EXM-004, UC-HR-006.
- **Acceptance Criteria:**
  - Given audit is down, When a grade/financial change is attempted, Then it is blocked until audit is restored.
  - Given audit is down, When a low-sensitivity action occurs, Then it proceeds with buffered audit (never silently unrecorded).
  - Given recovery, When restored, Then buffered audit is flushed and degradation is recorded.
- **Edge Case Analysis:**
  - *Invalid Input:* N/A.
  - *Permission Failure:* N/A.
  - *Concurrent Update:* many actions during outage → buffered/blocked per sensitivity.
  - *Duplicate Data:* dedup on flush.
  - *System Failure:* the scenario itself; fail closed for sensitive.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* low-sensitivity buffered; recovery flush.
  - *Negative:* sensitive action unrecorded (must not occur).
  - *Boundary:* action exactly at outage onset/recovery.

### UC-AUD-005 — Erasure Pseudonymization & Legal Hold
- **Module:** Audit Log · **Priority:** High
- **Actors:** Compliance Officer (primary), System
- **Goal:** Reconcile audit retention with erasure requests and legal holds without breaking the chain.
- **Description:** On lawful erasure, operational PII is anonymized and audit references to the subject are pseudonymized (chain intact, no raw PII retained); legal holds suspend erasure; statutory audit retention is honored over an erasure request (Conflict C-08 / P4).
- **Business Rules Applied:** AUD-008, STU-007, STF-008 (Cross-Cutting P4).
- **Preconditions:** An erasure request or legal hold on a subject.
- **Trigger:** Lawful erasure or legal hold.
- **Main Success Scenario:**
  1. On lawful erasure, the system anonymizes operational PII and pseudonymizes audit subject-references (AUD-008).
  2. The audit chain/integrity is preserved without retaining raw PII.
  3. Under legal hold, erasure is suspended and recorded.
  4. Where law mandates audit retention, audit is retained (pseudonymized) over the erasure request.
- **Alternative Flows:** A1) Hold lifted → normal erasure precedence resumes.
- **Exception Flows:** E1) Erasure conflicting with statutory retention → audit retained/pseudonymized per law.
- **Validation Rules:** Erasure lawful; hold honored; chain integrity preserved; precedence applied (AUD-008, P4).
- **Permissions Required:** `compliance.erasure.manage` / `legal_hold.manage` (elevated).
- **Notifications Triggered:** Erasure/hold confirmation to compliance.
- **Audit Events Generated:** `ERASURE_PSEUDONYMIZED`, `LEGAL_HOLD_APPLIED/REMOVED`.
- **Data Created:** Hold records.
- **Data Updated:** Pseudonymized references.
- **Data Deleted:** Raw PII (where lawful); audit events retained (pseudonymized).
- **Post Conditions:** Erasure/hold reconciled; audit chain intact.
- **Related Use Cases:** UC-STU-007, UC-STF-008, UC-AUD-012.
- **Acceptance Criteria:**
  - Given a lawful erasure, When processed, Then audit references are pseudonymized while the chain stays intact.
  - Given a legal hold, When applied, Then erasure is suspended and recorded.
  - Given statutory audit retention, When erasure is requested, Then audit is retained (pseudonymized).
- **Edge Case Analysis:**
  - *Invalid Input:* unverified erasure rejected.
  - *Permission Failure:* non-elevated actor → 403.
  - *Concurrent Update:* erasure + hold racing → hold wins (suspends erasure).
  - *Duplicate Data:* idempotent.
  - *System Failure:* chain never broken; recoverable.
  - *Workflow Failure:* compliance approval pends.
- **QA Coverage:**
  - *Positive:* erasure pseudonymization; hold suspends; statutory retention.
  - *Negative:* chain break (must not occur); audit deleted to satisfy erasure (must not occur).
  - *Boundary:* erasure at retention expiry.

---

## 7. Compact Specifications (routine use cases)

- **UC-AUD-002 — Query / Investigate Audit** · *High* · Rules: AUD-005, AUD-006. Auditor queries by actor/subject/time/correlation; access audited. *Permissions:* `audit.read` (segregated). *Edge:* operational admin without audit rights denied. *QA:* query; correlation; access-audited.
- **UC-AUD-003 — Protect Sensitive Values in Audit** · *Critical* · Rules: AUD-004, CFG-009. Secrets/sensitive values masked/omitted; change-occurred metadata only. *Edge:* never logs secret values. *QA:* secret masked; metadata present.
- **UC-AUD-006 — Audit-of-Audit (access logging)** · *High* · Rules: AUD-006. Every audit query/export recorded. *Audit:* `AUDIT_ACCESSED/EXPORTED`. *Edge:* even oversight accountable. *QA:* access recorded.
- **UC-AUD-007 — Manage Audit Policy & Retention** · *Medium* · Rules: AUD-007, CFG-004. Configure audited events/sensitivity classes/retention (cannot disable mandatory auditing or enable edits). *Permissions:* `audit.policy.manage`. *Edge:* mandatory auditing non-disableable. *QA:* policy config; no edit-enable.
- **UC-AUD-008 — Verify Tamper-Evidence / Integrity** · *High* · Rules: AUD-001. Verify the integrity chain; detect tampering. *Edge:* anomaly → security alert. *QA:* integrity verify; tamper detected.
- **UC-AUD-009 — Search Audit (correlated)** · *Medium* · Rules: AUD-003, AUD-005. Correlated end-to-end search (follow a workflow across modules). *Edge:* scoped to auditor authority. *QA:* correlation traversal; scope.
- **UC-AUD-010 — Compliance / Accountability Report** · *Medium* · Rules: REP-002, AUD. Accountability reports (who did what, when). *Edge:* scoped; supports audits. *QA:* report accuracy.
- **UC-AUD-011 — Governed Audit Export (regulator)** · *High* · Rules: AUD-006, REP-005. Governed audit export for investigations/regulators; export audited. *Permissions:* `audit.export`. *Edge:* no secrets; access recorded. *QA:* governed export; audit-of-export.
- **UC-AUD-012 — Legal-Hold Application / Removal** · *Medium* · Rules: AUD-008. Apply/remove legal hold (suspends erasure/purge). *Audit:* hold events. *Edge:* hold overrides erasure precedence. *QA:* hold suspends; removal resumes.
- **UC-AUD-013 — Audit Edit / Delete Blocked (Exception)** · *Critical* · Rules: AUD-001. No edit/delete path for anyone (incl. admins). *QA:* edit/delete blocked universally.
- **UC-AUD-014 — Unauthorized Audit Access Blocked (Exception)** · *High* · Rules: AUD-005. Audit read requires dedicated permission, segregated from ops; attempt recorded. *QA:* unauthorized denied; attempt audited.

## 8. Module-level QA & Edge Themes
- **Immutability & tamper-evidence (AUD-001 / AUD-013):** no edit/delete path for anyone; tampering detectable — the headline integrity suite.
- **Reliable capture (AUD-002 / fail-safe AUD-007):** outbox guarantees no orphan/lost audit; sensitive actions fail closed when audit is down.
- **No stored secrets (AUD-004):** accountability without becoming a leak vector.
- **Segregated access + audit-of-audit (AUD-005/006):** the audited cannot curate their trail; even oversight is accountable.
- **Erasure precedence (AUD-008 / C-08):** pseudonymize-not-break; legal hold > statutory retention > erasure.
