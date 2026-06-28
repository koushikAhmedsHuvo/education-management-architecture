# 25 — Notification Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`NOT-001…008`), and Use Case Repository (`UC-NOT-001…017`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Deliver the right message to the right person on the right channel: event-driven templated notifications, custody-aware privacy-preserving recipient resolution, preferences with mandatory-category override, data minimization, reliable non-blocking outbox delivery, bulk batching/dedup for peak events, quiet hours, and delivery-status tracking.

**Business Goal.** Communicate reliably without leaking minors' data or over-notifying — restricted parties are never contacted, mandatory safeguarding/financial notices always reach permitted recipients, and peak events (results/invoices) never overwhelm channels or the platform.

**Scope.** Event-driven templated notifications; custody-aware recipient resolution; preferences + mandatory-category override; per-channel content minimization; outbox-based reliable non-blocking delivery; bulk batching/deduplication; quiet hours; delivery tracking + dead-letter handling. Cross-cutting service consumed by all modules.

**Out of Scope.** Domain event semantics (consuming modules raise events; this module routes/delivers). Custody/restriction source of truth (Guardian module — NOT consumes GRD-N-006). Template content storage mechanics (Configuration Engine). Audit storage (Audit module).

---

## 2. Actors

**Primary Actors.** System (event-driven delivery, resolution, batching), Notification Administrator (templates/channels/quiet hours), Recipient (guardian/staff/student — manages own preferences).

**Secondary Actors.** All consuming modules (raise events), Guardian module (custody/restrictions), Configuration Engine (templates/channels), External providers (email/SMS/push gateways), Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Event-driven templated delivery | Deliver notifications from events using configurable templates. | High | Configuration Engine |
| FR-002 | Custody-aware recipient resolution | Resolve recipients honoring custody/contact restrictions. | Critical | Guardian (GRD-N-006) |
| FR-003 | Preferences + mandatory override | Honor preferences; mandatory categories cannot be opted out. | High | FR-001 |
| FR-004 | Data minimization | Minimize sensitive content per channel. | Critical | — |
| FR-005 | Reliable non-blocking outbox | Deliver via outbox; never block the originating transaction. | Critical | Outbox (D13) |
| FR-006 | Bulk batching & dedup | Batch and deduplicate for peak events. | High | FR-005 |
| FR-007 | Quiet hours | Respect quiet hours/timing (except urgent/mandatory). | Medium | Config |
| FR-008 | Delivery tracking & failure | Track delivery status; surface failures; dead-letter. | High | Providers |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Templated events | Config-driven messages. | Consistency; no code per notice. |
| Custody-aware routing | Restricted parties excluded. | Safeguarding; privacy. |
| Mandatory override | Critical notices always sent. | Compliance. |
| Data minimization | Channel-appropriate content. | Privacy. |
| Outbox delivery | Reliable, non-blocking. | No lost notices; fast txns. |
| Batching/dedup | Peak-event safe. | Scale; no spam. |
| Quiet hours | Timing respect. | Recipient experience. |
| Delivery tracking | Status + dead-letter. | Operational visibility. |

---

## 5. Screens

Notification Center (in-app inbox); Template Management; Channel & Quiet-Hours Configuration; Recipient Preferences; Delivery Status / Tracking; Notification Search; Delivery & Engagement Report; Delivery-Log Export; Dead-Letter Handling.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Notification Center | View, Mark Read, Filter | Mark All Read |
| Template Management | Create/Edit Template, Version, Preview, Set Mandatory | — |
| Channel/Quiet-Hours Config | Configure Channels/Quiet Hours, Save | — |
| Recipient Preferences | Set Channel Prefs (mandatory locked) | — |
| Delivery Status | View Status, Retry, Inspect Failure | Bulk Retry |
| Dead-Letter | View, Requeue, Discard (governed) | Bulk Requeue |
| Delivery Report | Run, Filter, Export | Export |

---

## 7. Forms

**Template** — `event`, `channel(s)`, `body` (with safe placeholders), `category` (standard/mandatory). Validation: minimized placeholders (NOT-004); mandatory categories flagged (NOT-003); versioned (CFG-004).

**Channel/Quiet-Hours Config** — `channels` (email/SMS/push/in-app), `quietHours`, `urgentOverride`. Validation: quiet hours exclude urgent/mandatory (NOT-007); typed/versioned (CFG-002/004).

**Recipient Preferences** — per-category channel toggles. Validation: mandatory categories cannot be disabled (NOT-003 / UC-NOT-017).

**Recipient Resolution** (system) — from event subject. Validation: custody/contact restrictions exclude restricted parties (NOT-002/GRD-N-006 / UC-NOT-016); financial notices → financial-responsible (GRD-N-005).

---

## 8. Search & Filter Requirements

**Notifications:** by recipient, event/category, channel, status (queued/sent/delivered/failed/dead-letter), date. Sorting: date/status. Pagination: server-side, 50 default (volume). Scope-bound; recipients see own.

---

## 9. Table Requirements

**Notification table:** Recipient, Event, Channel, Category, Status, Sent, Delivered. Sorting on Date/Status. Filtering as above. Export (governed — delivery log). Bulk: retry, requeue.

---

## 10. Workflow Requirements

**Trigger events:** domain event raised, recipient resolution, preference/quiet-hours application, batch, send, delivery callback, failure/dead-letter. **Status changes:** notification `QUEUED → SENT → DELIVERED/FAILED → DEAD_LETTER`. **Approvals:** governed dead-letter discard. **Notifications:** (self) admin alerts on systemic failure. **Audit:** sends (event, recipients-resolved metadata), exclusions (restricted), failures, dead-letter actions (immutable; no sensitive content — AUD-004).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Manage templates | `notification.template.manage` |
| Configure channels/quiet hours | `notification.config.manage` |
| Manage own preferences | recipient (self) |
| View delivery status | `notification.status.view` |
| Retry/requeue/dead-letter | `notification.delivery.manage` |
| Delivery report/export | `notification.report.view`, `notification.export` |

Recipient resolution is custody-aware (NOT-002); mandatory categories override opt-out (NOT-003).

---

## 12. Business Rule References

NOT-001 (event-driven, configurable, templated), NOT-002 (custody-aware, privacy-preserving recipient resolution), NOT-003 (preferences with mandatory-category override), NOT-004 (data minimization in content), NOT-005 (reliable delivery via outbox, non-blocking), NOT-006 (bulk batching & deduplication), NOT-007 (quiet hours & timing), NOT-008 (delivery status tracking & failure visibility). Cross-cutting: GRD-N-005/006 (financial-responsible/custody restrictions), CFG-004 (template versioning), AUD-004 (no sensitive content in audit), D13 (outbox), AUD-001, and all event-raising modules.

## 13. Use Case References

UC-NOT-001 (Raise & Deliver), UC-NOT-002 (Resolve Recipients — custody-aware), UC-NOT-003 (Preferences & Mandatory Override), UC-NOT-004 (Minimize Content), UC-NOT-005 (Reliable Outbox Delivery), UC-NOT-006 (Bulk Batch & Dedup), UC-NOT-007 (Quiet Hours), UC-NOT-008 (Track Delivery & Failure), UC-NOT-009 (Manage Templates), UC-NOT-010 (Configure Channels & Quiet Hours), UC-NOT-011 (Manage Preferences), UC-NOT-012 (Search), UC-NOT-013 (Delivery & Engagement Report), UC-NOT-014 (Export Delivery Log), UC-NOT-015 (Dead-Letter Handling), UC-NOT-016 (Custody-Restricted Recipient Excluded), UC-NOT-017 (Mandatory-Category Opt-Out Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Raise notification (from event) | POST (internal) | System (consuming module) |
| Resolve recipients (custody-aware) | GET (internal) | System |
| Get in-app notifications | GET | Recipient |
| Manage templates | POST/PUT | Admin |
| Configure channels/quiet hours | GET/PUT | Admin |
| Manage recipient preferences | GET/PUT | Recipient |
| Delivery status / retry / requeue | GET/POST | Admin |
| Delivery & engagement report | GET | Admin |
| Export delivery log (governed) | GET | Admin |

Raising is non-blocking via outbox (NOT-005); resolution excludes restricted parties (NOT-002); mandatory categories bypass opt-out (NOT-003).

---

## 15. Database Requirements

**Entities:** `NotificationTemplate` (event, channels, body, category, version), `NotificationOutbox` (event, payload-ref, status), `Notification` (recipient, channel, status, sentAt, deliveredAt), `RecipientPreference` (category/channel), `ChannelConfig` (quiet hours), `DeadLetter`. **Relationships:** Template 1—* Notification; Outbox 1—1 dispatch. **Indexes:** index(Notification.recipientId, status), index(Notification.status, createdAt), index(NotificationOutbox.status), unique(Notification dedup key per event/recipient/channel) (NOT-006). Outbox guarantees delivery (NOT-005/D13); no sensitive content persisted beyond minimization (NOT-004).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email/SMS/Push/In-App | This IS the delivery module; it dispatches on behalf of all modules per their templates and categories. |
| In-App | Admin alerts on systemic delivery failure / dead-letter growth. |

Self-referential: the module honors its own rules (custody-aware, minimized, mandatory-override) for every dispatch.

---

## 17. Audit Requirements

Log: notification dispatch (event, template/version, channels, resolved-recipient metadata — not content), restricted-party exclusions (UC-NOT-016), mandatory-override sends, failures/retries, dead-letter actions, template/config changes. Record who/when. Sensitive message content is never stored in audit (AUD-004). Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Delivery success/failure, Engagement (open/read where available), Channel performance, Dead-letter trends, Mandatory-notice coverage. **Exports:** governed delivery-log export (no sensitive content). **Dashboards:** delivery health (queue depth, failure rate, dead-letter, peak throughput).

---

## 19. Error Handling

**Validation:** mandatory-category opt-out attempt → blocked (UC-NOT-017); template with over-exposed placeholders → rejected (NOT-004). **Permission:** restricted recipient → excluded silently from that send (UC-NOT-016). **Workflow:** provider failure → retry then dead-letter (NOT-008). **System:** provider outage → outbox holds and retries (non-blocking, NOT-005); never blocks the originating transaction.

---

## 20. Edge Cases

**Concurrent updates:** same event delivered twice → dedup yields one per recipient/channel (NOT-006). **Duplicate data:** batch overlap → deduplicated. **Partial failures:** partial batch failure → per-recipient status, failed entries dead-lettered. **Rollback:** originating transaction rolls back → outbox entry not committed (no phantom notice). **Restriction race:** restriction applied mid-batch → restricted party excluded from that batch (NOT-002).

---

## 21. Acceptance Criteria

**Functional.** Notifications are templated and event-driven; recipient resolution excludes custody/contact-restricted parties; mandatory categories cannot be opted out; content is minimized per channel; delivery is reliable via outbox and never blocks the originating transaction; peak events are batched and deduplicated; quiet hours are respected except for urgent/mandatory; delivery status is tracked with dead-letter handling.

**Business.** The right permitted recipient is reliably reached without leaking minors' data or over-notifying; critical safeguarding and financial notices always get through; result/invoice peaks never overwhelm channels or the platform.

---

## 22. Future Enhancements

WhatsApp/RCS channels; per-recipient send-time optimization; rich interactive notifications; localization/translation of templates; A/B template testing; provider failover routing; engagement-based channel selection; in-app notification preferences center.
