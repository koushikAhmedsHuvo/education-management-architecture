# 29 — Audit Log Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint (D13 outbox, D17 immutable audit), Business Rules Catalog (`AUD-001…008`), and Use Case Repository (`UC-AUD-001…014`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Provide the system-wide immutable, append-only, tamper-evident audit trail: reliable transactional-outbox capture, complete structured event content, no sensitive values stored, restricted segregated access, audit-of-audit, fail-safe behavior when audit is unavailable, and erasure-aware pseudonymization with legal hold.

**Business Goal.** Guarantee that everything of consequence is recorded immutably and verifiably, that the audit itself cannot be edited or quietly read, and that the system fails closed on sensitive actions rather than proceeding unaudited.

**Scope.** Immutable append-only tamper-evident log; transactional-outbox capture; complete structured events; no sensitive values; restricted segregated access; audit-of-audit (access logging); fail-safe on unavailability; erasure pseudonymization + legal hold (C-08); integrity verification; governed regulator export. Cross-cutting service every module writes to.

**Out of Scope.** Domain event meaning (modules emit; audit records). Report rendering (Reporting module — reads audit via authorized projections). Secret storage (Configuration Engine — audit never stores secrets). Erasure decision/retention policy authority (Student/Compliance own the decision; audit applies pseudonymization).

---

## 2. Actors

**Primary Actors.** System (captures events via outbox), Auditor / Compliance Officer (queries/investigates — segregated), Data-Protection Officer (erasure/legal hold), Security Administrator (integrity verification, policy).

**Secondary Actors.** All modules (event emitters), Workflow/Outbox (capture path), Authorization (segregated access), Reporting (compliance reports), Legal (holds).

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Immutable append-only tamper-evident | No edit/delete path; tamper-evident chaining. | Critical | D17 |
| FR-002 | Transactional-outbox capture | Capture reliably with the originating transaction. | Critical | Outbox (D13) |
| FR-003 | Complete structured events | Record who/what/when/before/after, correlation id. | High | — |
| FR-004 | No sensitive values stored | Never store secrets/raw sensitive data in audit. | Critical | CFG-009 |
| FR-005 | Restricted segregated access | Audit access is restricted and segregated from operations. | Critical | Authorization |
| FR-006 | Audit-of-audit | Log all audit access/queries/exports. | High | FR-001 |
| FR-007 | Fail-safe on unavailability | Sensitive actions fail closed if audit cannot record. | Critical | consuming modules |
| FR-008 | Erasure pseudonymization + legal hold | Pseudonymize on erasure; legal hold overrides (C-08). | High | Compliance |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Immutable trail | Append-only, tamper-evident. | Trustworthy evidence. |
| Outbox capture | Reliable, transactional. | No lost events. |
| Structured events | Complete content. | Investigable. |
| No sensitive values | Secrets excluded. | Security. |
| Segregated access | Restricted readers. | Insider-risk control. |
| Audit-of-audit | Access logged. | Watch the watchers. |
| Fail-safe | Fail closed on sensitive. | Integrity over availability. |
| Erasure-aware | Pseudonymize + hold. | Compliance (C-08). |

---

## 5. Screens

Audit Investigation (query/correlate); Audit Event Detail; Integrity Verification; Audit-of-Audit (access log); Erasure / Pseudonymization; Legal-Hold Management; Audit Policy & Retention; Compliance / Accountability Report; Governed Audit Export (regulator); Audit Search.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Audit Investigation | Query, Correlate, Filter, Drill | — |
| Event Detail | View (who/what/when/before/after), Verify | — |
| Integrity Verification | Verify Chain/Tamper-Evidence | Bulk Verify |
| Audit-of-Audit | View Access Log | — |
| Erasure | Pseudonymize (subject), Confirm | — |
| Legal Hold | Apply, Remove (governed) | — |
| Policy & Retention | Configure Retention/Policy | — |
| Compliance Report | Run, Filter, Export (governed) | Export |

---

## 7. Forms

**Audit Query** — `filters` (actor/entity/event/date/correlation). Validation: segregated-access permission (AUD-005); query itself audited (AUD-006).

**Integrity Verification** — `range`. Validation: tamper-evidence/chain verified; discrepancy flagged (AUD-001 / UC-AUD-008).

**Erasure / Pseudonymization** — `subject`, `reason`. Validation: pseudonymize references, retain structural event (AUD-008); legal hold blocks erasure (C-08 / UC-AUD-012).

**Legal Hold** — `subject/scope`, `reason`. Validation: hold overrides erasure/retention; governed apply/remove (C-08).

**Policy & Retention** — `retention`, `accessPolicy`. Validation: statutory minimums respected; precedence legal-hold > statutory > erasure (C-08).

---

## 8. Search & Filter Requirements

**Audit:** by actor, entity/type, event, date range, correlation id, severity. Sorting: timestamp. Pagination: server-side, 50 default (volume). Access restricted to segregated audit roles; every search audited (AUD-006).

---

## 9. Table Requirements

**Audit table:** Timestamp, Actor, Action, Entity, Correlation, Before/After (structured, non-sensitive). Read-only — no edit/delete actions exist (AUD-001). Sorting on Timestamp. Filtering as above. Export (governed — regulator only, audited). No bulk mutation.

---

## 10. Workflow Requirements

**Trigger events:** event capture (outbox), audit access/query, integrity verification, erasure, legal-hold apply/remove, export. **Status changes:** (events are immutable — no lifecycle); legal hold `APPLIED/REMOVED`; erasure `PSEUDONYMIZED`. **Approvals:** regulator export and legal-hold removal governed. **Notifications:** integrity-failure/security alerts to security admins. **Audit:** audit access and exports are themselves logged (AUD-006).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Query/investigate audit | `audit.read` (segregated) |
| Verify integrity | `audit.verify` |
| View audit-of-audit | `audit.meta.read` |
| Erasure pseudonymization | `audit.erasure.manage` |
| Manage legal hold | `audit.legal_hold.manage` |
| Manage policy/retention | `audit.policy.manage` |
| Governed export (regulator) | `audit.export` |

Audit access is segregated from operational roles (AUD-005); there is no write/edit/delete capability (AUD-001).

---

## 12. Business Rule References

AUD-001 (immutable, append-only, tamper-evident), AUD-002 (reliable capture via transactional outbox), AUD-003 (complete structured event content), AUD-004 (sensitive values never stored), AUD-005 (restricted, segregated access), AUD-006 (audit-of-audit), AUD-007 (fail-safe on unavailability), AUD-008 (erasure-aware pseudonymization & legal hold). Cross-cutting: D13/D17 (outbox/immutable audit), CFG-009 (secret protection), AUTHZ (segregated access), C-08 (retention precedence), and every module as an emitter.

## 13. Use Case References

UC-AUD-001 (Capture Event — immutable, outbox), UC-AUD-002 (Query/Investigate), UC-AUD-003 (Protect Sensitive Values), UC-AUD-004 (Fail-Safe on Unavailability), UC-AUD-005 (Erasure Pseudonymization & Legal Hold), UC-AUD-006 (Audit-of-Audit), UC-AUD-007 (Manage Policy & Retention), UC-AUD-008 (Verify Tamper-Evidence/Integrity), UC-AUD-009 (Search — correlated), UC-AUD-010 (Compliance/Accountability Report), UC-AUD-011 (Governed Export — regulator), UC-AUD-012 (Legal-Hold Application/Removal), UC-AUD-013 (Audit Edit/Delete Blocked), UC-AUD-014 (Unauthorized Audit Access Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Capture audit event (outbox) | POST (internal) | System |
| Query / investigate audit | GET | Auditor (segregated) |
| Verify integrity / tamper-evidence | POST | Security Admin |
| Get audit-of-audit log | GET | Auditor |
| Erasure pseudonymization | POST | DPO |
| Apply / remove legal hold | POST | DPO / Legal |
| Manage policy & retention | PUT | Security Admin |
| Compliance / accountability report | GET | Compliance |
| Governed audit export (regulator) | POST | Compliance (→ approval) |

There is no update or delete endpoint by design (AUD-001/UC-AUD-013); capture is via transactional outbox (AUD-002); every read is itself audited (AUD-006).

---

## 15. Database Requirements

**Entities:** `AuditEvent` (actor, action, entity, before/after-structured, correlationId, timestamp, hash/chain), `AuditAccessLog` (audit-of-audit), `LegalHold` (subject, reason, dates), `ErasureRecord` (pseudonymization map ref), `AuditPolicy` (retention). **Relationships:** AuditEvent chained (tamper-evidence); Subject 1—* LegalHold. **Indexes:** index(AuditEvent.entityId, ts), index(AuditEvent.actorId, ts), index(AuditEvent.correlationId), append-only storage (no UPDATE/DELETE grants — AUD-001). Capture via outbox (AUD-002); no sensitive values (AUD-004); pseudonymization preserves structure (AUD-008).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| In-App | Integrity-verification failures; unauthorized-access attempts; legal-hold changes — to security/compliance. |
| Email | Critical tamper-evidence alerts. |
| SMS/Push | Not used. |

Alerts contain no sensitive payloads (AUD-004/NOT-004).

---

## 17. Audit Requirements

This module IS the audit. It records every system event (who/what/when/before/after) and additionally logs all access to itself (audit-of-audit, AUD-006). Integrity is tamper-evident and verifiable (AUD-001). Erasure pseudonymizes rather than deletes (AUD-008). Capture reliability is guaranteed by the transactional outbox (AUD-002); sensitive actions fail closed when audit is unavailable (AUD-007).

---

## 18. Reporting Requirements

**Reports:** Compliance/accountability, Access patterns (audit-of-audit), Integrity-verification results, Legal-hold register, Erasure log. **Exports:** governed regulator export (approved, audited). **Dashboards:** audit health (capture rate, integrity status, unauthorized-access attempts, fail-safe triggers).

---

## 19. Error Handling

**Validation:** edit/delete attempt → blocked by design (UC-AUD-013); unauthorized access → blocked + logged (UC-AUD-014). **Permission:** non-segregated read → 403 + audit-of-audit. **Workflow:** regulator export/legal-hold removal pending → governed. **System:** audit store unavailable → sensitive actions fail closed (AUD-007); outbox retries capture (no lost events).

---

## 20. Edge Cases

**Concurrent updates:** none — events are immutable/append-only. **Duplicate data:** outbox dedup → one event per occurrence. **Partial failures:** capture failure on sensitive action → action fails closed (AUD-007); on non-sensitive → outbox retries. **Rollback:** originating txn rolls back → outbox event not committed (no phantom audit). **Erasure race:** erasure vs legal hold → hold wins (C-08).

---

## 21. Acceptance Criteria

**Functional.** The audit log is immutable, append-only, and tamper-evident with no edit/delete path; events capture via transactional outbox with complete who/what/when/before/after; sensitive values are never stored; access is restricted/segregated and every access is itself audited; sensitive actions fail closed when audit cannot record; erasure pseudonymizes (never destroys structure) and legal hold overrides erasure/retention per precedence.

**Business.** Everything of consequence is provably recorded and verifiable; the audit cannot be tampered with or quietly read; the system never performs a sensitive action it cannot account for; privacy erasure and legal hold are correctly reconciled.

---

## 22. Future Enhancements

Cryptographic notarization/external anchoring; real-time SIEM streaming; anomaly/insider-threat detection on audit-of-audit; configurable correlation/trace visualization; automated compliance-evidence packs; tamper-evidence dashboards; retention-policy simulation.
