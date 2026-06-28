# 29 — Audit Log Business Rules

## 1. Module Purpose
Govern the **audit log** — the immutable, append-only, tamper-evident record of everything consequential the system does. Every module in this catalog emits audit events; this document specifies what they look like, how they are reliably captured (transactional outbox), how immutability and tamper-evidence are guaranteed, who may read them (restricted, segregated from operational admins), how sensitive values are handled (never store secrets/passwords), retention and legal hold, and how audit interacts with erasure (pseudonymize, never silently break the chain). The audit log is the backbone of accountability for minors' data, financial integrity, and academic integrity.

## 2. Actors
- **All Modules / System** — emit audit events as side effects of consequential actions.
- **Auditor / Compliance Officer** — reads and queries the audit log (read-only, restricted).
- **Security Administrator** — monitors security-relevant audit; configures (not alters) audit policy.
- **Regulator / Investigator (via authorized export)** — receives audit evidence through governed access.
- **System** — captures, persists immutably, protects, retains, and serves audit.

## 3. Use Cases

**Use Case ID:** UC-AUD-01 — Capture an Audit Event
**Actors:** Any Module, System
**Description:** Record a consequential action immutably and reliably.
**Preconditions:** A consequential action occurs (state change, access, decision).
**Main Flow:** 1) The action and its audit event are committed atomically via the transactional outbox. 2) The event records actor, action, target, scope, before/after (non-sensitive), timestamp, correlation/trace id, and source. 3) The event is persisted append-only and is tamper-evident.
**Alternative Flow:** A1) High-volume events partitioned for scale.
**Exception Flow:** E1) Sensitive values present → recorded as change-occurred with masked/omitted value (AUD-004). E2) Audit store unavailable → the action is handled per the fail-safe policy (AUD-007).
**Post Conditions:** Immutable audit event persisted; never lost.
**Business Rules Applied:** AUD-001, AUD-002, AUD-003, AUD-004.

**Use Case ID:** UC-AUD-02 — Query / Investigate
**Actors:** Auditor, System
**Description:** Search the audit trail for an investigation or compliance review.
**Preconditions:** Requester holds audit-read permission.
**Main Flow:** 1) Auditor queries by actor/subject/time/correlation. 2) System returns matching events, scoped to the auditor's authority. 3) The audit access itself is audited (audit-of-audit).
**Exception Flow:** E1) Operational admin without audit rights → denied (AUD-005).
**Post Conditions:** Findings retrieved; the access recorded.
**Business Rules Applied:** AUD-005, AUD-006.

**Use Case ID:** UC-AUD-03 — Erasure / Legal-Hold Interaction
**Actors:** Compliance Officer, System
**Description:** Reconcile audit retention with erasure requests and legal holds.
**Preconditions:** An erasure request or legal hold applies to a subject.
**Main Flow:** 1) On lawful erasure, the subject's PII in operational data is anonymized; audit references are pseudonymized so the chain stays intact without retaining raw PII. 2) Under legal hold, erasure is suspended and recorded. 3) Audit integrity is preserved throughout.
**Exception Flow:** E1) Erasure conflicting with statutory audit retention → audit retained/pseudonymized per law.
**Post Conditions:** Erasure/hold reconciled; audit chain intact; actions audited.
**Business Rules Applied:** AUD-008.

## 4. Business Rules

**Rule ID:** AUD-001
**Rule Name:** Immutable, Append-Only, Tamper-Evident Log
**Description:** Audit events are append-only and tamper-evident; they cannot be edited or deleted, and tampering is detectable.
**Priority:** Critical
**Category:** Integrity (non-repudiation)
**Preconditions:** Any audit event.
**Business Rule:** Once written, an audit event is immutable; there is no edit/delete path (even for admins); tamper-evidence (e.g., hash-chaining/sequencing) makes any alteration detectable. Corrections are new events referencing the original, never modifications.
**System Action:** Append-only persistence with tamper-evidence; no mutation API.
**Validation:** Sequence/hash integrity maintained.
**Failure Behavior:** Reject any edit/delete; detect/flag tampering.
**Audit Requirement:** The integrity mechanism is itself verifiable.
**Example Scenario:** No one can quietly delete the record of a grade change; a tamper attempt is detectable.
**Related Rules:** AUD-002, AUD-005, EXM-007 (immutability parallel).

**Rule ID:** AUD-002
**Rule Name:** Reliable Capture via Transactional Outbox
**Description:** Audit events are committed atomically with the action they describe, so they are never lost or orphaned.
**Priority:** Critical
**Category:** Reliability
**Preconditions:** A consequential action.
**Business Rule:** The audit event is written in the same transaction as the business change (outbox), then reliably dispatched to durable audit storage; either both the action and its audit commit, or neither does — no action without its audit, no orphan audit.
**System Action:** Atomic outbox write; reliable downstream persistence.
**Validation:** Atomicity with the source transaction; at-least-once durable delivery with dedup.
**Failure Behavior:** If audit cannot be guaranteed, fail the action per the fail-safe policy (AUD-007) for high-sensitivity operations.
**Audit Requirement:** Capture is the audit; delivery is monitored.
**Example Scenario:** A payment and its audit event commit together; a crash never leaves one without the other.
**Related Rules:** AUD-007, NOT-005 (outbox parallel), architecture D13/D17.

**Rule ID:** AUD-003
**Rule Name:** Complete, Structured Event Content
**Description:** Each event captures who/what/when/where with before/after and correlation for end-to-end traceability.
**Priority:** High
**Category:** Usefulness
**Preconditions:** Event creation.
**Business Rule:** Events record actor (and on-behalf/delegation), action, target, scope (institute/campus/session), before/after for changes (non-sensitive), timestamp, correlation/trace id, and source (IP/device/system). Correlation lets an investigator follow a workflow end-to-end across modules.
**System Action:** Populate the structured event; propagate correlation ids.
**Validation:** Required fields present; correlation consistent.
**Failure Behavior:** Reject incomplete events for consequential actions.
**Audit Requirement:** This is the audit content standard.
**Example Scenario:** An admission decision traces from submission through each approval to conversion via one correlation id.
**Related Rules:** WFL (correlation), AUD-001.

**Rule ID:** AUD-004
**Rule Name:** Sensitive Values Are Never Stored in Audit
**Description:** Secrets, passwords, and highly sensitive values are never written to audit; only the fact of change is recorded.
**Priority:** Critical
**Category:** Security / privacy
**Preconditions:** A change involves sensitive/secret data.
**Business Rule:** Passwords, tokens, encryption keys, and secret config (CFG-009) are never in audit; for highly sensitive fields (e.g., medical, national ID), audit records that a change/access occurred and by whom, with the value masked/omitted. Audit proves accountability without becoming a data-leak vector.
**System Action:** Mask/omit sensitive values; record change-occurred metadata.
**Validation:** Sensitivity classification honored in audit.
**Failure Behavior:** Block writing raw secrets/sensitive values to audit.
**Audit Requirement:** Sensitive change recorded as metadata only.
**Example Scenario:** A password change logs the event and actor, never the password.
**Related Rules:** AUTH-008, CFG-009, STU-005, STF-007.

**Rule ID:** AUD-005
**Rule Name:** Restricted, Segregated Audit Access
**Description:** Reading audit requires dedicated permission, segregated from operational administration; admins cannot alter audit.
**Priority:** Critical
**Category:** Control / SoD
**Preconditions:** Audit read.
**Business Rule:** Audit-read is a distinct permission held by auditors/compliance, segregated from the operational admins whose actions are audited (so the audited cannot edit/curate their own trail). Audit is read-only to everyone; there is no privileged edit.
**System Action:** Gate audit reads by dedicated permission; no edit path for anyone.
**Validation:** Audit-read permission distinct; no mutation capability.
**Failure Behavior:** Deny audit read without the permission; deny any edit universally.
**Audit Requirement:** Audit access is itself audited (AUD-006).
**Example Scenario:** A system admin can operate the system but cannot read or alter the audit trail of their own actions without separate auditor rights.
**Related Rules:** AUTHZ-009 (SoD), AUD-001, AUD-006.

**Rule ID:** AUD-006
**Rule Name:** Audit-of-Audit
**Description:** Access to the audit log is itself recorded.
**Priority:** High
**Category:** Accountability
**Preconditions:** Someone reads/exports audit.
**Business Rule:** Every audit query/export is recorded (who looked at what, when), so even oversight is accountable and audit misuse is detectable.
**System Action:** Record audit-read/export events.
**Validation:** Access events captured.
**Failure Behavior:** No unrecorded audit access.
**Audit Requirement:** Log `AUDIT_ACCESSED/EXPORTED`.
**Example Scenario:** An auditor reviewing a student's history leaves their own access record.
**Related Rules:** AUD-005, REP-008.

**Rule ID:** AUD-007
**Rule Name:** Fail-Safe on Audit Unavailability
**Description:** For high-sensitivity actions, if audit cannot be guaranteed, the action fails closed rather than proceed unrecorded.
**Priority:** High
**Category:** Integrity
**Preconditions:** Audit subsystem degraded/unavailable.
**Business Rule:** High-sensitivity operations (financial movements, grade changes, security/privilege changes) must be auditable to proceed; if audit cannot be guaranteed, they fail closed and alert. Lower-sensitivity actions may proceed with buffered/queued audit per policy, never silently unrecorded.
**System Action:** Block unauditable high-sensitivity actions; buffer where safe; alert.
**Validation:** Sensitivity classification; audit-availability check.
**Failure Behavior:** Fail closed for sensitive actions; never lose their audit.
**Audit Requirement:** Record degradation/fail-closed events.
**Example Scenario:** If audit storage is down, a grade change is blocked until auditing is restored.
**Related Rules:** AUD-002, EXM-007, AUTH-005.

**Rule ID:** AUD-008
**Rule Name:** Erasure-Aware Pseudonymization & Legal Hold
**Description:** Erasure pseudonymizes audit references to keep the chain intact; legal hold suspends erasure; statutory audit retention is respected.
**Priority:** High
**Category:** Compliance
**Preconditions:** Erasure request or legal hold on a subject.
**Business Rule:** On lawful erasure, operational PII is removed and audit references to the subject are pseudonymized (the integrity chain and accountability survive without retaining raw PII); legal holds suspend erasure and are recorded; where law mandates audit retention (financial/academic), audit is retained (pseudonymized) over an erasure request.
**System Action:** Pseudonymize audit subject-references on erasure; enforce legal hold; honor statutory retention.
**Validation:** Erasure lawful; hold status checked; chain integrity preserved.
**Failure Behavior:** Never break the audit chain; never delete audit to satisfy erasure where retention is mandated.
**Audit Requirement:** Log erasure/pseudonymization and hold actions.
**Example Scenario:** A right-to-erasure removes a former student's PII while audit shows pseudonymized references proving past actions occurred.
**Related Rules:** STU-007, AUD-001, Compliance.

## 5. Validation Rules
- Audit is append-only, immutable, tamper-evident; no edit/delete for anyone.
- Events committed atomically with their action (outbox); none lost/orphaned.
- Events complete (actor/action/target/scope/before-after/time/correlation/source).
- Secrets/sensitive values never stored; change-occurred metadata only.
- Audit-read is segregated/permissioned; audit access is itself audited.
- High-sensitivity actions fail closed if unauditable; erasure pseudonymizes, legal hold suspends.

## 6. State Machine
*(An audit event is fundamentally immutable; its lifecycle is minimal by design.)*

**State Name:** EMITTED
**Description:** Event written to the outbox with its source transaction.
**Allowed Transitions:** → PERSISTED.
**Forbidden Transitions:** edit/delete.
**System Actions:** Atomic commit with the action.

**State Name:** PERSISTED
**Description:** Durably stored, append-only, tamper-evident.
**Allowed Transitions:** → ARCHIVED (aged partitions); references → PSEUDONYMIZED (erasure).
**Forbidden Transitions:** mutation of event content.
**System Actions:** Maintain integrity chain; serve read-only queries.

**State Name:** PSEUDONYMIZED (references)
**Description:** Subject references pseudonymized post-erasure; the event remains.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** breaking the chain.
**System Actions:** Replace subject PII references with pseudonyms; preserve integrity.

**State Name:** ARCHIVED
**Description:** Aged audit retained per retention class (cold storage).
**Allowed Transitions:** → PURGED (only after legal retention, never under hold).
**Forbidden Transitions:** edit; purge under legal hold or before retention.
**System Actions:** Cold retention; integrity preserved.

**State Name:** PURGED
**Description:** Removed only after the full legal retention period with no hold.
**Allowed Transitions:** terminal.
**Forbidden Transitions:** premature purge.
**System Actions:** Governed, recorded removal.

## 7. Status Definitions
`EMITTED` · `PERSISTED` · `PSEUDONYMIZED` (references) · `ARCHIVED` · `PURGED`. Categories: `SECURITY` · `DATA_CHANGE` · `ACCESS` · `FINANCIAL` · `ACADEMIC` · `CONFIG` · `WORKFLOW` · `ADMIN`.

## 8. Workflow Rules
- Audit capture is automatic and non-optional for consequential actions (not a user workflow).
- Audit policy (what is audited, sensitivity classes) is configured, never used to disable required auditing.
- Erasure/legal-hold reconciliation follows a governed compliance process (AUD-008).
- Audit-store degradation triggers alerting and fail-closed for sensitive actions.

## 9. Permission Rules
- `audit.read` — query the audit log (auditors/compliance; segregated from ops).
- `audit.export` — export audit evidence (governed; for investigations/regulators).
- `audit.policy.manage` — configure audit policy/classes (cannot disable mandatory auditing or enable edits).
- No permission grants audit edit/delete — none exists.

## 10. Notification Rules
- Audit-store degradation / fail-closed events → alert security/operations.
- Tamper-evidence anomalies → alert security immediately.
- Legal-hold application/removal → notify compliance.
- Audit is generally silent (it records, not notifies) except for integrity/availability alerts.

## 11. Audit Requirements
*(This module defines the audit standard the catalog uses.)* Every consequential action across modules emits a complete, immutable event; audit access/export is itself audited (AUD-006); integrity and availability events are recorded. The catalog's per-module "Audit Requirements" sections enumerate the specific events; this module guarantees their capture, immutability, and protection.

## 12. Data Retention Rules
- Audit retained long-term per class (security, financial, academic) and statutory minimums — often the longest retention in the system.
- Legal hold suspends purge; purge only after full retention with no hold (governed, recorded).
- Pseudonymized (not deleted) on erasure where retention is mandated; raw PII not retained beyond need.
- High-volume audit partitioned and tiered (hot to cold) without losing integrity.

## 13. Edge Cases
- **Admin tries to delete an embarrassing record:** impossible; no edit/delete path; attempt itself is detectable (AUD-001/AUD-005).
- **Failed transaction:** outbox ensures no orphan audit and no unaudited action (AUD-002).
- **Erasure vs statutory audit retention:** pseudonymize and retain per law; chain intact (AUD-008).
- **Audit store outage:** sensitive actions fail closed; others buffer; nothing silently unrecorded (AUD-007).
- **Secret in a payload:** never written to audit; metadata only (AUD-004).
- **Clock skew/ordering:** sequencing/monotonic ordering preserves a coherent timeline.
- **Auditor over-reach:** audit-of-audit records their access (AUD-006).
- **High-volume burst (result publish):** partitioned capture keeps pace without loss.

## 14. Failure Scenarios
- **Edit/delete attempt:** rejected universally (AUD-001).
- **Audit unavailable for a grade/financial change:** action fails closed (AUD-007).
- **Sensitive value in audit:** blocked (AUD-004).
- **Unauthorized audit read:** denied and (the attempt) recorded (AUD-005/AUD-006).
- **Erasure breaking the chain:** prevented by pseudonymization (AUD-008).

## 15. Exception Handling Rules
- Audit is immutable and reliably captured; corrections are new events, never edits.
- Sensitive/secret values are excluded; only accountability metadata is stored.
- High-sensitivity actions are blocked if they cannot be audited.
- Erasure and legal hold are reconciled without compromising integrity.

## 16. Compliance Considerations
- **Accountability / non-repudiation:** the immutable trail underpins financial, academic, and data-protection accountability for minors' data.
- **GDPR balance:** erasure honored via pseudonymization while statutory audit retention is preserved; legal holds respected.
- **Segregation of duties:** audit access segregated from operations; the audited cannot curate their trail.
- **Investigations/regulators:** complete, correlated, exportable (governed) evidence.

## 17. Future Considerations
- Cryptographic notarization / external anchoring for stronger tamper-evidence.
- Real-time anomaly detection on the audit stream (insider-threat signals).
- Standardized audit export formats for regulators.
- Immutable retention tiers (WORM storage) for the most sensitive classes.
