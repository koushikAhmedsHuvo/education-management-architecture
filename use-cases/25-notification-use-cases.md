# 25 — Notification Use Cases

Transforms the Notification business rules (`NOT-001`…`NOT-008`) into use cases. The multi-channel delivery engine every module raises events to: event-driven templated messages, custody-aware recipient resolution, preferences with mandatory overrides, data minimization, reliable non-blocking delivery, bulk batching/dedup, quiet hours, and delivery-status tracking.

## 1. Primary Actors
Consuming Modules (raise events), Recipient (staff/guardian/student), Notification Administrator (channels, templates, policy).

## 2. Secondary Actors
System (recipient resolution, rendering, delivery, retry, status), Guardian module (custody/financial-responsible), Configuration Engine (templates), Audit service, channel providers (email/SMS/push).

## 3. Goals
Render configurable templated messages from domain events; resolve recipients custody-aware and privacy-preserving; honor preferences while forcing mandatory categories; minimize sensitive content per channel; deliver reliably via the outbox without blocking the source action; batch/dedup peak bursts; respect quiet hours for non-critical messages; track delivery status and surface failures.

## 4. User Journeys
- **Raise & deliver:** a module raises an event → the engine resolves custody-appropriate recipients → renders the localized template → selects channels per preference/policy → delivers reliably → tracks status.
- **Preferences:** a recipient mutes optional categories; security/safeguarding/financial notices still arrive (mandatory override).
- **Peak event:** result publish raises 30k notifications → batched, throttled, deduplicated.
- **Failure:** a channel is down → retry/fallback/dead-letter, surfaced to admins — the originating business action is never blocked.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-NOT-001 | Raise & Deliver Notification | Critical |
| Core | UC-NOT-002 | Resolve Recipients (custody-aware) | Critical |
| Core | UC-NOT-003 | Apply Preferences & Mandatory Override | High |
| Core | UC-NOT-004 | Minimize Sensitive Content (per channel) | High |
| Core | UC-NOT-005 | Reliable Delivery via Outbox (non-blocking) | Critical |
| Core | UC-NOT-006 | Bulk Batch & Deduplicate (peak) | High |
| Core | UC-NOT-007 | Apply Quiet Hours | Medium |
| Core | UC-NOT-008 | Track Delivery & Handle Failure | High |
| CRUD | UC-NOT-009 | Manage Templates | High |
| Admin | UC-NOT-010 | Configure Channels & Quiet Hours | Medium |
| CRUD | UC-NOT-011 | Manage Recipient Preferences | Medium |
| Search | UC-NOT-012 | Search Notifications / Status | Low |
| Reporting | UC-NOT-013 | Delivery & Engagement Report | Medium |
| Export | UC-NOT-014 | Export Delivery Log (governed) | Low |
| Workflow | UC-NOT-015 | Dead-Letter Handling | Medium |
| Exception | UC-NOT-016 | Custody-Restricted Recipient Excluded | Critical |
| Exception | UC-NOT-017 | Mandatory-Category Opt-Out Blocked | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-NOT-001 — Raise & Deliver Notification
- **Module:** Notification · **Priority:** Critical
- **Actors:** Consuming Module (primary), System, Recipient
- **Goal:** Turn a domain event into a delivered, custody-appropriate, channel-correct notification.
- **Description:** A module raises a typed event; the engine resolves recipients, renders the localized template, selects channels per preference/policy, and delivers reliably — without blocking the source action.
- **Business Rules Applied:** NOT-001, NOT-002, NOT-003, NOT-005, NOT-008.
- **Preconditions:** Notification type/template configured; channels available.
- **Trigger:** A module raises an event (e.g., absence, invoice, result publish).
- **Main Success Scenario:**
  1. A module raises an event with subject and context via the outbox (NOT-005).
  2. The engine resolves recipients (relationship/role, custody-aware, NOT-002).
  3. The engine renders the template per recipient locale (NOT-001) and selects channels per preference/policy (NOT-003).
  4. The engine delivers with at-least-once semantics and dedup; tracks status (NOT-008).
- **Alternative Flows:** A1) Bulk peak event batched/deduped (UC-NOT-006). A2) Quiet hours defer non-critical (UC-NOT-007).
- **Exception Flows:** E1) Custody-restricted recipient → excluded (UC-NOT-016). E2) Channel down → fallback/retry (UC-NOT-008). E3) Opt-out on a non-mandatory type → suppressed (NOT-003).
- **Validation Rules:** Template exists per type/channel/locale; recipients authorized/custody-aware; delivery idempotent/non-blocking (NOT-001/002/005).
- **Permissions Required:** System (raised by modules); admin for templates/policy.
- **Notifications Triggered:** This is the notification mechanism.
- **Audit Events Generated:** `NOTIFICATION_RAISED` (type, template version); delivery status.
- **Data Created:** Notification record(s).
- **Data Updated:** Delivery status.
- **Data Deleted:** None.
- **Post Conditions:** Notification delivered or safely failed-and-recorded; source action unaffected.
- **Related Use Cases:** UC-NOT-002, UC-NOT-005, UC-NOT-006.
- **Acceptance Criteria:**
  - Given a domain event, When raised, Then custody-appropriate recipients receive a correctly rendered message.
  - Given a delivery failure, When it occurs, Then the originating business action is unaffected.
  - Given a duplicate event, When raised, Then recipients are not double-notified (dedup).
- **Edge Case Analysis:**
  - *Invalid Input:* missing required template variable → render blocked (no malformed send).
  - *Permission Failure:* status visible only to authorized roles.
  - *Concurrent Update:* duplicate events → deduped (NOT-006).
  - *Duplicate Data:* one notification per recipient/event.
  - *System Failure:* channel/down → retry/fallback; outbox guarantees no lost event.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* event→delivery; localized render; multi-channel.
  - *Negative:* custody-restricted; opt-out (non-mandatory); malformed template.
  - *Boundary:* delivery at quiet-hours edge; dedup window.

### UC-NOT-002 — Resolve Recipients (custody-aware)
- **Module:** Notification · **Priority:** Critical
- **Actors:** System (primary)
- **Goal:** Resolve the right recipients from relationships/roles while enforcing custody/contact restrictions and privacy.
- **Description:** Recipients derive from the subject's relationships (guardian-of-record, financial-responsible, assigned staff) and roles; custody/contact restrictions exclude barred parties; a notification about one student never reaches an unrelated recipient or carries another family's data.
- **Business Rules Applied:** NOT-002, GRD-N-005, GRD-N-006.
- **Preconditions:** A notification subject and type.
- **Trigger:** Recipient resolution during delivery.
- **Main Success Scenario:**
  1. The engine derives candidate recipients from the subject's relationships/roles (NOT-002).
  2. The engine applies custody/contact restrictions, excluding barred parties (GRD-N-006).
  3. The engine routes financial notices to the financial-responsible guardian (GRD-N-005).
  4. Content is scoped to the subject only.
- **Alternative Flows:** A1) Multiple guardians → all permitted recipients.
- **Exception Flows:** E1) Restricted party → excluded; notice goes to the permitted guardian (UC-NOT-016).
- **Validation Rules:** Recipient authorized for the subject; not restricted; content subject-scoped (NOT-002, GRD-N-006).
- **Permissions Required:** System.
- **Notifications Triggered:** Downstream delivery.
- **Audit Events Generated:** Resolved recipients + restriction exclusions.
- **Data Created/Updated/Deleted:** None.
- **Post Conditions:** Correct, custody-safe recipient set.
- **Related Use Cases:** UC-GRD-N-004, UC-NOT-001.
- **Acceptance Criteria:**
  - Given a custody restriction, When resolving, Then the restricted party is excluded and the permitted guardian receives the notice.
  - Given a financial notice, When resolving, Then it routes to the financial-responsible guardian.
  - Given a notification, When resolved, Then it never carries another family's data.
- **Edge Case Analysis:**
  - *Invalid Input:* unknown subject → no recipients.
  - *Permission Failure:* N/A (system).
  - *Concurrent Update:* restriction set mid-batch → restricted party excluded from that batch.
  - *Duplicate Data:* sibling reuse → one guardian, correct per-child routing.
  - *System Failure:* resolution fails closed (exclude on doubt).
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* relationship/role routing; financial-guardian routing.
  - *Negative:* custody-restricted exclusion; cross-family leakage (must not occur).
  - *Boundary:* restriction effective mid-delivery.

### UC-NOT-005 — Reliable Delivery via Outbox (non-blocking)
- **Module:** Notification · **Priority:** Critical
- **Actors:** System (primary)
- **Goal:** Deliver notifications reliably without ever blocking or rolling back the originating business action.
- **Description:** Events are enqueued transactionally with the source change (outbox), then delivered asynchronously with at-least-once semantics, dedup, retry-with-backoff, channel fallback, and dead-lettering; a failed notification never affects the business action.
- **Business Rules Applied:** NOT-005, NOT-008, NOT-006.
- **Preconditions:** A committed notification event.
- **Trigger:** A source change commits with a notification event.
- **Main Success Scenario:**
  1. The notification event is written in the same transaction as the source change (outbox).
  2. The engine delivers asynchronously (at-least-once, dedup).
  3. On failure, it retries with backoff, falls back to another channel, or dead-letters (NOT-008).
  4. The source business action is never blocked or rolled back by delivery outcomes.
- **Alternative Flows:** A1) Bulk delivery batched (UC-NOT-006).
- **Exception Flows:** E1) Persistent failure → dead-letter + admin alert (UC-NOT-015).
- **Validation Rules:** Outbox commit atomic with source; idempotent delivery; non-blocking (NOT-005).
- **Permissions Required:** System.
- **Notifications Triggered:** Delivery itself.
- **Audit Events Generated:** Delivery attempts/status; dead-letters.
- **Data Created:** Delivery records.
- **Data Updated:** Status.
- **Data Deleted:** None.
- **Post Conditions:** Reliable delivery or recorded failure; source action intact.
- **Related Use Cases:** UC-NOT-008, UC-AUD-001 (outbox parallel).
- **Acceptance Criteria:**
  - Given a source change, When committed, Then the notification event is enqueued atomically with it.
  - Given a channel outage, When delivering, Then the source action is unaffected and the message retries/falls back.
  - Given persistent failure, When exhausted, Then it dead-letters and alerts admins (never silently lost).
- **Edge Case Analysis:**
  - *Invalid Input:* N/A.
  - *Permission Failure:* N/A.
  - *Concurrent Update:* duplicate delivery attempts → idempotent.
  - *Duplicate Data:* deduped.
  - *System Failure:* outbox guarantees no lost event; resumable.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* reliable delivery; retry/fallback.
  - *Negative:* source rollback on notification failure (must not occur); lost event.
  - *Boundary:* retry exhaustion → dead-letter.

---

## 7. Compact Specifications (routine use cases)

- **UC-NOT-003 — Apply Preferences & Mandatory Override** · *High* · Rules: NOT-003. Honor preferences for optional types; force security/safeguarding/financial/legal categories. *Edge:* mandatory categories non-suppressible (UC-NOT-017). *QA:* optional honored; mandatory forced.
- **UC-NOT-004 — Minimize Sensitive Content (per channel)** · *High* · Rules: NOT-004, STU-005. Minimize sensitive data on low-assurance channels; secure-view link for details. *Edge:* no full results/secrets in SMS. *QA:* minimization; secure link.
- **UC-NOT-006 — Bulk Batch & Deduplicate (peak)** · *High* · Rules: NOT-006. Batch/throttle/dedup peak bursts (result publish). *Edge:* one notice per recipient/event; channel limits. *QA:* batch; dedup; throttle.
- **UC-NOT-007 — Apply Quiet Hours** · *Medium* · Rules: NOT-007. Defer non-critical within quiet hours; critical overrides. *Edge:* locale-aware; critical immediate. *QA:* defer non-critical; critical override.
- **UC-NOT-008 — Track Delivery & Handle Failure** · *High* · Rules: NOT-008. Track sent/delivered/failed/read; flag bad addresses; surface failures. *Audit:* status. *Edge:* bounce flagged for correction. *QA:* status accuracy; failure visibility.
- **UC-NOT-009 — Manage Templates** · *High* · Rules: NOT-001, CFG-010. Configure typed/channel/locale templates (versioned, validated substitution). *Permissions:* `notification.template.manage`. *Edge:* missing-variable render blocked; versioned. *QA:* template CRUD; validation; versioning.
- **UC-NOT-010 — Configure Channels & Quiet Hours** · *Medium* · Rules: NOT-007, CFG-004. Configure channels and quiet-hours policy. *Permissions:* `notification.channel.manage`. *QA:* config applies.
- **UC-NOT-011 — Manage Recipient Preferences** · *Medium* · Rules: NOT-003. Recipients set channel/type preferences (within mandatory bounds). *Edge:* cannot opt out of mandatory. *QA:* preference set; mandatory bound.
- **UC-NOT-012 — Search Notifications / Status** · *Low* · Rules: AUTHZ-002. Scoped search (no content to unauthorized). *QA:* scope respected.
- **UC-NOT-013 — Delivery & Engagement Report** · *Medium* · Rules: REP-002. Delivery rates, failures, engagement (consent-bound). *Edge:* scoped; privacy-preserving. *QA:* metrics accuracy.
- **UC-NOT-014 — Export Delivery Log (governed)** · *Low* · Rules: REP-005. Governed, scoped export (no sensitive content). *QA:* scoped; content-safe.
- **UC-NOT-015 — Dead-Letter Handling** · *Medium* · Rules: NOT-005/008. Surface/handle dead-lettered notifications for admin action. *Audit:* dead-letter. *Edge:* never silently dropped. *QA:* dead-letter surfaced; retried.
- **UC-NOT-016 — Custody-Restricted Recipient Excluded (Exception)** · *Critical* · Rules: NOT-002, GRD-N-006. Restricted party excluded from all related notifications. *QA:* excluded; permitted guardian receives.
- **UC-NOT-017 — Mandatory-Category Opt-Out Blocked (Exception)** · *High* · Rules: NOT-003. Cannot opt out of security/safeguarding/financial/legal notices. *QA:* opt-out blocked; optional allowed.

## 8. Module-level QA & Edge Themes
- **Custody-aware routing (NOT-002 / GRD-N-006):** the child-safety headline suite — restricted parties excluded across all notifications; no cross-family leakage.
- **Mandatory override (NOT-003):** security/safeguarding/financial/legal notices always delivered despite opt-out.
- **Reliable non-blocking (NOT-005):** delivery never blocks/rolls back the source action; outbox guarantees no lost event.
- **Data minimization (NOT-004):** sensitive content kept off low-assurance channels.
- **Peak handling (NOT-006):** 30k-cohort bursts batched/deduped/throttled.
