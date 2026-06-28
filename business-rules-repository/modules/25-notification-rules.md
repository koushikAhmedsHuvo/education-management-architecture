# 25 — Notification Business Rules

## 1. Module Purpose
Govern **notifications** — the multi-channel delivery of messages the whole system generates (absence alerts ATT-008, result publication RES-009, fee dues FEE, security events AUTH, workflow tasks WFL). The engine is generic and configurable: domain modules raise notification *events*; this module resolves recipients (respecting custody/contact restrictions and financial-responsibility roles), applies preferences (while honoring mandatory overrides for security/safeguarding/transactional messages), renders templates, and delivers reliably via the transactional outbox with retries — **never blocking the originating action on delivery**, and **never leaking one family's data to another**.

## 2. Actors
- **Consuming Modules** — raise notification events with context.
- **Recipient** — staff, guardian, student (resolved per relationship/role, custody-restriction-aware).
- **Administrator** — configures channels, templates, quiet hours, and policy.
- **System** — resolves recipients, renders, delivers, retries, tracks status, and audits.

## 3. Use Cases

**Use Case ID:** UC-NOT-01 — Raise & Deliver a Notification
**Actors:** Consuming Module, System, Recipient
**Description:** A domain event produces a delivered notification.
**Preconditions:** Notification type/template configured; channels available.
**Main Flow:** 1) A module raises an event (type, subject, context) via the outbox. 2) The engine resolves recipients (relationship/role, excluding custody-restricted parties). 3) Renders the template per recipient locale; selects channels per preference and policy. 4) Delivers with at-least-once semantics and dedup; tracks status.
**Alternative Flow:** A1) Bulk peak event (result publish) batched and deduplicated (NOT-006). A2) Quiet hours defer non-critical messages (NOT-007).
**Exception Flow:** E1) Recipient custody-restricted → excluded (NOT-002). E2) Channel down → fallback/retry (NOT-005). E3) Opt-out on a non-mandatory type → suppressed (NOT-003).
**Post Conditions:** Notification delivered or safely failed-and-recorded; source action unaffected; audited.
**Business Rules Applied:** NOT-001, NOT-002, NOT-003, NOT-005, NOT-008.

**Use Case ID:** UC-NOT-02 — Manage Preferences & Mandatory Overrides
**Actors:** Recipient, Administrator, System
**Description:** Recipients set channel/type preferences; mandatory categories override opt-out.
**Preconditions:** Preference model configured.
**Main Flow:** 1) Recipient sets preferences (channels, types, frequency). 2) System honors them for optional categories. 3) Security/safeguarding/transactional categories deliver regardless of opt-out (NOT-003).
**Exception Flow:** E1) Attempt to opt out of a mandatory category → not permitted.
**Post Conditions:** Preferences applied within mandatory bounds; audited.
**Business Rules Applied:** NOT-003, NOT-004.

**Use Case ID:** UC-NOT-03 — Track Delivery & Handle Failure
**Actors:** System, Administrator
**Description:** Track status and recover from delivery failures.
**Preconditions:** A notification was attempted.
**Main Flow:** 1) System records sent/delivered/failed/read per channel. 2) On failure, retries with backoff, then falls back or dead-letters. 3) Persistent failures surface to admins without affecting the source action.
**Exception Flow:** E1) Invalid channel address → mark undeliverable; try alternate; flag for correction.
**Post Conditions:** Status tracked; failures handled/visible; audited.
**Business Rules Applied:** NOT-005, NOT-008.

## 4. Business Rules

**Rule ID:** NOT-001
**Rule Name:** Event-Driven, Configurable, Templated Notifications
**Description:** Notifications are raised as events and rendered from configurable, versioned, localized templates.
**Priority:** High
**Category:** Configuration / decoupling
**Preconditions:** A notification type exists.
**Business Rule:** Modules raise typed events with context; templates (per type/channel/locale) are configurable and versioned via the Configuration Engine; variable substitution is validated so no template renders with missing/unsafe data. Terminology/i18n is applied.
**System Action:** Resolve and render the versioned template; validate substitution.
**Validation:** Template exists for type/channel/locale; variables present.
**Failure Behavior:** Block rendering with missing required variables; never send a malformed message.
**Audit Requirement:** Log `NOTIFICATION_RAISED` with type and template version.
**Example Scenario:** A fee-due notice renders in the institute's locale with the correct amount and date.
**Related Rules:** CFG-010, NOT-006.

**Rule ID:** NOT-002
**Rule Name:** Custody-Aware, Privacy-Preserving Recipient Resolution
**Description:** Recipients resolve from relationships/roles, excluding custody/contact-restricted parties, and never expose another subject's data.
**Priority:** Critical
**Category:** Child safety / privacy
**Preconditions:** Recipient resolution.
**Business Rule:** Recipients derive from the subject's relationships (guardian-of-record, financial-responsible, assigned staff) and roles; custody/contact restrictions (GRD-N-006) exclude barred parties; a notification about one student never goes to an unrelated recipient and never contains another family's data.
**System Action:** Resolve recipients per relationship/role; enforce restrictions; scope content.
**Validation:** Recipient authorized for the subject; not restricted.
**Failure Behavior:** Exclude restricted/unauthorized recipients; never broaden.
**Audit Requirement:** Log resolved recipients (and restriction-based exclusions).
**Example Scenario:** An absence alert goes to the custodial guardian, not the court-barred parent.
**Related Rules:** GRD-N-005, GRD-N-006, ATT-008.

**Rule ID:** NOT-003
**Rule Name:** Preferences with Mandatory-Category Override
**Description:** Recipients control optional notifications, but security, safeguarding, and transactional categories always deliver.
**Priority:** High
**Category:** Compliance / safety
**Preconditions:** Preference evaluation.
**Business Rule:** Optional/marketing-style notifications honor opt-out and channel preferences; mandatory categories (security alerts, safeguarding/absence, financial/transactional, legal notices) cannot be opted out of and use a reliable channel.
**System Action:** Apply preferences to optional types; force-deliver mandatory types.
**Validation:** Category classification correct; mandatory not suppressible.
**Failure Behavior:** Block opt-out of mandatory categories.
**Audit Requirement:** Log preference changes and mandatory-override deliveries.
**Example Scenario:** A guardian who muted general announcements still receives a security and an unexplained-absence alert.
**Related Rules:** AUTH (security notices), ATT-008, Compliance.

**Rule ID:** NOT-004
**Rule Name:** Data Minimization in Message Content
**Description:** Notification content minimizes sensitive data, especially on low-assurance channels.
**Priority:** High
**Category:** Privacy (minors)
**Preconditions:** Rendering content.
**Business Rule:** Messages include only what is needed; sensitive details (full results, financial specifics, minors' identifiers) are minimized on low-assurance channels (e.g., SMS), favoring a secure prompt to view in-app. Never put secrets or full sensitive records in a notification body.
**System Action:** Apply per-channel content minimization; link to secure view for details.
**Validation:** Content respects channel sensitivity classification.
**Failure Behavior:** Strip/withhold sensitive content on low-assurance channels.
**Audit Requirement:** Captured in delivery records (content class, not full content of sensitive items).
**Example Scenario:** An SMS says results are published with a secure link, not the grades themselves.
**Related Rules:** STU-005, CFG-009, NOT-002.

**Rule ID:** NOT-005
**Rule Name:** Reliable Delivery via Outbox (Non-Blocking)
**Description:** Notifications deliver via the transactional outbox with retries; delivery never blocks the originating action.
**Priority:** Critical
**Category:** Reliability
**Preconditions:** A notification event is committed.
**Business Rule:** Events are enqueued transactionally with the source change (outbox), then delivered asynchronously with at-least-once semantics, dedup, retry-with-backoff, channel fallback, and dead-lettering. A failed notification never rolls back or blocks the business action that raised it.
**System Action:** Enqueue with the transaction; deliver async; retry/fallback/dead-letter.
**Validation:** Outbox commit atomic with source; idempotent delivery.
**Failure Behavior:** Retry/fallback; dead-letter and alert on persistent failure; source action unaffected.
**Audit Requirement:** Log delivery attempts/status; dead-letters.
**Example Scenario:** Result publication succeeds even if SMS is temporarily down; messages send when it recovers.
**Related Rules:** AUD (outbox), RES-009, NOT-008.

**Rule ID:** NOT-006
**Rule Name:** Bulk Batching & Deduplication for Peak Events
**Description:** High-volume notification bursts are batched, throttled, and deduplicated.
**Priority:** High
**Category:** Performance
**Preconditions:** A bulk event (e.g., result publish, mass fee issue).
**Business Rule:** Bulk notifications are batched and rate-limited to protect channels and downstream systems; duplicates to the same recipient for the same event are collapsed; per-recipient digests may be used per policy.
**System Action:** Batch, throttle, dedup; optionally digest.
**Validation:** Dedup keys per recipient/event; throttle limits per channel.
**Failure Behavior:** Backpressure rather than channel overload; never double-spam.
**Audit Requirement:** Log bulk runs and dedup counts.
**Example Scenario:** Publishing 30,000 results notifies each guardian once, throttled to respect SMS limits.
**Related Rules:** RES-009, NOT-005.

**Rule ID:** NOT-007
**Rule Name:** Quiet Hours & Timing Respect
**Description:** Non-critical notifications respect quiet hours and locale timing; critical ones may override.
**Priority:** Medium
**Category:** User respect
**Preconditions:** Timing-sensitive delivery.
**Business Rule:** Configurable quiet hours defer non-critical messages to an appropriate time in the recipient's locale; critical/safety/security messages may override quiet hours.
**System Action:** Defer non-critical within quiet hours; deliver critical immediately.
**Validation:** Quiet-hours policy + locale resolved.
**Failure Behavior:** Defer rather than disturb for non-critical; never delay critical.
**Audit Requirement:** Log deferrals.
**Example Scenario:** A general announcement queued at 2am sends at 8am; a security alert sends immediately.
**Related Rules:** NOT-003, NOT-004.

**Rule ID:** NOT-008
**Rule Name:** Delivery Status Tracking & Failure Visibility
**Description:** Per-recipient, per-channel delivery status is tracked; failures are visible and actionable.
**Priority:** Medium
**Category:** Observability
**Preconditions:** A notification is attempted.
**Business Rule:** Status (queued/sent/delivered/failed/read where supported) is tracked; bounced/invalid addresses are flagged for correction; persistent failures surface to admins; status never exposes content to unauthorized viewers.
**System Action:** Track and expose status to authorized roles; flag bad addresses.
**Validation:** Status model complete; access-scoped.
**Failure Behavior:** Surface failures; flag invalid channels; never silently drop.
**Audit Requirement:** Delivery status recorded.
**Example Scenario:** A bounced guardian email is flagged so the contact can be corrected.
**Related Rules:** NOT-005, GRD-N (contact data).

## 5. Validation Rules
- Templates exist per type/channel/locale; substitution validated; no malformed sends.
- Recipients authorized for the subject; custody/contact-restricted parties excluded.
- Mandatory categories non-suppressible; optional categories honor preferences.
- Content minimized per channel sensitivity; no secrets/full sensitive records in bodies.
- Delivery via outbox, idempotent, non-blocking; bulk batched/deduped; quiet hours respected for non-critical.

## 6. State Machine

**State Name:** QUEUED
**Description:** Event enqueued via outbox.
**Allowed Transitions:** → SENDING; → SUPPRESSED (opt-out/restricted/quiet-defer).
**Forbidden Transitions:** delivery before recipient resolution.
**System Actions:** Resolve recipients; render; select channels.

**State Name:** SENDING
**Description:** Delivery in progress.
**Allowed Transitions:** → SENT; → FAILED.
**Forbidden Transitions:** blocking the source action.
**System Actions:** Dispatch per channel; idempotent.

**State Name:** SENT / DELIVERED / READ
**Description:** Successful delivery (read where supported).
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** content exposure to unauthorized.
**System Actions:** Track status.

**State Name:** FAILED → RETRYING → DEAD_LETTER
**Description:** Delivery failed; retried; then dead-lettered.
**Allowed Transitions:** RETRYING → SENT; → DEAD_LETTER (exhausted) → ARCHIVED.
**Forbidden Transitions:** silent drop.
**System Actions:** Backoff retry; fallback channel; alert on dead-letter.

**State Name:** SUPPRESSED
**Description:** Intentionally not sent (opt-out/restricted) or deferred.
**Allowed Transitions:** → ARCHIVED; deferred → QUEUED at the right time.
**Forbidden Transitions:** suppression of mandatory categories.
**System Actions:** Record suppression reason.

**State Name:** ARCHIVED
**Description:** Historical notification record.
**Allowed Transitions:** none.
**Forbidden Transitions:** edits.
**System Actions:** Retain per policy.

## 7. Status Definitions
`QUEUED` · `SENDING` · `SENT` · `DELIVERED` · `READ` · `FAILED` · `RETRYING` · `DEAD_LETTER` · `SUPPRESSED` · `ARCHIVED`. Categories: `SECURITY` · `SAFEGUARDING` · `TRANSACTIONAL` · `ACADEMIC` · `FINANCIAL` · `ANNOUNCEMENT` (mandatory vs optional per policy).

## 8. Workflow Rules
- Notifications are emitted by domain workflows/events; the engine handles delivery, not domain logic.
- Mandatory categories bypass preference workflows.
- Bulk peak-event notifications are throttled and deduped (NOT-006).
- Dead-letters trigger an admin-visible handling path.

## 9. Permission Rules
- `notification.template.manage` — manage templates/types/policy.
- `notification.channel.manage` — configure channels and quiet hours.
- `notification.status.view` — view delivery status (scoped; not content beyond authorization).
- `notification.preferences.manage` — recipients manage their own preferences (within mandatory bounds).

## 10. Notification Rules
*(Self-referential: this module is the notification mechanism; cross-cutting policies above govern it.)*
- Every consequential domain event has a defined notification policy (who, which channel, mandatory or optional).
- Security/safeguarding/financial notifications are mandatory and reliable.
- All notifications respect custody restrictions and data minimization.

## 11. Audit Requirements
Mandatory: `NOTIFICATION_RAISED` (type, template version), recipient resolution + restriction exclusions, delivery attempts/status, suppressions (reason), dead-letters, preference changes, mandatory-override deliveries. Audit records metadata and content class, not full sensitive content.

## 12. Data Retention Rules
- Notification records/status retained per policy (operational + compliance evidence, e.g., that a mandatory notice was sent).
- Content of sensitive notifications minimized in storage; full sensitive data lives in source records, not notification logs.
- Retention aligns with the subject's record; anonymization strips recipient PII post-retention.

## 13. Edge Cases
- **Custody-restricted recipient:** excluded from all related notifications (NOT-002).
- **Opt-out vs mandatory:** mandatory categories deliver regardless (NOT-003).
- **No valid channel/address:** flagged for correction; alternate tried (NOT-008).
- **Peak burst:** batched/deduped/throttled (NOT-006).
- **Quiet hours vs critical:** critical overrides; non-critical defers (NOT-007).
- **Subject anonymized:** stop new notifications; historical records retained per policy.
- **Bounced email/invalid phone:** flagged; contact correction prompted.
- **Duplicate event:** deduped per recipient/event (NOT-006).
- **Low-assurance channel + sensitive content:** content minimized, secure-view link used (NOT-004).

## 14. Failure Scenarios
- **Channel outage:** retry/fallback; source action unaffected (NOT-005).
- **Malformed template:** send blocked; never partial/garbled message (NOT-001).
- **Persistent failure:** dead-letter + admin alert; never silent drop (NOT-008).
- **Restricted recipient leak:** prevented by resolution rules (NOT-002).
- **Mandatory suppression attempt:** blocked (NOT-003).

## 15. Exception Handling Rules
- Delivery never blocks or rolls back the originating business action.
- Recipient resolution fails closed on restrictions (exclude on doubt).
- Sensitive content is minimized on low-assurance channels.
- Failures are retried, fallen back, dead-lettered, and surfaced; never silently lost.

## 16. Compliance Considerations
- **Child safety:** custody-aware routing and mandatory safeguarding/absence alerts.
- **Consent:** marketing/optional notifications honor consent/opt-out; transactional/security/legal are mandatory.
- **Data minimization:** sensitive data kept out of low-assurance channels and notification logs.
- **Evidence:** records prove mandatory notices (e.g., fee due, safety) were sent.

## 17. Future Considerations
- Richer channels (WhatsApp, app push) with the same policy guarantees.
- Per-recipient digests and smart bundling.
- Delivery analytics and engagement (consent-bound, privacy-preserving).
- Two-way notifications (guardian replies feeding leave/queries).
