# 18 — Payment Use Cases

Transforms the Payment business rules (`PAY-001`…`PAY-009`) into use cases. Money in: gapless immutable receipts, deterministic allocation, multi-method capture, clearance for non-immediate methods, reconciliation, overpayment-as-credit, reversal-not-delete, idempotency, and governed mis-post correction.

## 1. Primary Actors
Accountant / Cashier (records payments), Finance Administrator (reconciliation, reversals), Guardian (online payment).

## 2. Secondary Actors
System (receipt numbering, allocation, idempotency, clearance, reconciliation), Payment Gateway (online), Fee module (invoices/dues), Notification & File modules (receipts), Audit service.

## 3. Goals
Record every payment once (idempotent, no double-credit); issue gapless, immutable receipts; allocate deterministically to dues; capture method/reference; handle clearance for cheques/transfers; reconcile against settlements; convert overpayment to credit; reverse (never delete); correct mis-posts under governance.

## 4. User Journeys
- **Pay:** guardian pays online or at the counter → idempotent processing → gapless immutable receipt → deterministic allocation to oldest/first dues → guardian notified.
- **Clear:** cheque/transfer enters a clearance lifecycle → confirmed/bounced → dues update on clearance.
- **Reconcile:** finance reconciles recorded payments against gateway/bank settlements → discrepancies flagged.
- **Fix:** a mis-posted payment is corrected via governed reversal + re-post; overpayment becomes a credit balance.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-PAY-001 | Record Payment & Issue Receipt (idempotent) | Critical |
| Core | UC-PAY-002 | Allocate Payment to Dues (deterministic) | High |
| Core | UC-PAY-003 | Clearance Lifecycle (cheque/transfer) | High |
| Core | UC-PAY-004 | Reconcile Against Settlements | High |
| Core | UC-PAY-005 | Handle Overpayment (credit balance) | Medium |
| Core | UC-PAY-006 | Reverse Payment (never delete) | Critical |
| Core | UC-PAY-007 | Correct Mis-Posted Payment (governed) | High |
| CRUD | UC-PAY-008 | View Payment / Receipt (scoped) | Medium |
| Admin | UC-PAY-009 | Configure Methods & Allocation Rules | Medium |
| Approval | UC-PAY-010 | Approve Reversal / Correction (SoD) | High |
| Search | UC-PAY-011 | Search Payments / Receipts | Medium |
| Reporting | UC-PAY-012 | Collection & Reconciliation Report | High |
| Bulk | UC-PAY-013 | Bulk Payment Recording / Import | Medium |
| Export | UC-PAY-014 | Export Payment Data (governed) | Medium |
| Workflow | UC-PAY-015 | Reversal / Correction Workflow | High |
| Exception | UC-PAY-016 | Double-Payment / Replay Blocked | Critical |
| Exception | UC-PAY-017 | Delete Payment Blocked | Critical |

---

## 6. Detailed Specifications (high-value use cases)

### UC-PAY-001 — Record Payment & Issue Receipt (idempotent)
- **Module:** Payment · **Priority:** Critical
- **Actors:** Accountant / Guardian (primary), System
- **Goal:** Record a payment exactly once and issue a gapless, immutable receipt.
- **Description:** Processes a payment with an idempotency key (preventing double-credit on retries/double-clicks), issues a gapless immutable receipt, captures method/reference, and allocates to dues.
- **Business Rules Applied:** PAY-001, PAY-003, PAY-008, PAY-002, FILE-007.
- **Preconditions:** Payable dues exist (or advance/credit allowed); method available.
- **Trigger:** Counter payment or online gateway callback.
- **Main Success Scenario:**
  1. System receives the payment with an idempotency key (PAY-008).
  2. System verifies the key is new (not a replay); records the payment once.
  3. System issues a gapless, immutable receipt with a sequential number (PAY-001) and captures method/reference (PAY-003).
  4. System allocates the payment to dues deterministically (PAY-002) and generates the receipt document (FILE-007).
- **Alternative Flows:** A1) Online gateway → clearance immediate; offline cheque → clearance lifecycle (UC-PAY-003). A2) Overpayment → credit balance (UC-PAY-005).
- **Exception Flows:** E1) Replayed idempotency key → return the original receipt, no double-credit (UC-PAY-016). E2) Gateway failure mid-flow → no partial receipt; reconciled later.
- **Validation Rules:** Idempotency enforced; receipt gapless/immutable; method/reference captured; allocation deterministic (PAY-001/002/003/008).
- **Permissions Required:** `payment.record` (counter) / authenticated guardian (online).
- **Notifications Triggered:** `PAYMENT_RECEIVED` + receipt to financial-responsible guardian.
- **Audit Events Generated:** `PAYMENT_RECORDED` (receipt no., method, amount).
- **Data Created:** Payment; receipt; receipt document.
- **Data Updated:** Dues balance; allocation.
- **Data Deleted:** None.
- **Post Conditions:** Payment recorded once; immutable receipt issued; dues reduced.
- **Related Use Cases:** UC-PAY-002, UC-PAY-006, UC-FEE-005.
- **Acceptance Criteria:**
  - Given a valid payment, When recorded, Then a gapless immutable receipt is issued and dues are reduced.
  - Given a replayed request (same idempotency key), When processed, Then the original receipt is returned and no double-credit occurs.
  - Given a gateway failure mid-flow, When it occurs, Then no partial receipt is created.
- **Edge Case Analysis:**
  - *Invalid Input:* zero/negative amount rejected.
  - *Permission Failure:* counter recording without permission → 403.
  - *Concurrent Update:* double-click/parallel callback → idempotency yields one payment (PAY-008).
  - *Duplicate Data:* replay prevented; receipt numbering gapless (PAY-001).
  - *System Failure:* atomic payment+receipt+allocation; reconciled if interrupted.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* counter cash; online gateway; allocation to dues.
  - *Negative:* replay; zero amount; unauthorized counter.
  - *Boundary:* exact-dues payment (zero balance); receipt-number sequence continuity.

### UC-PAY-006 — Reverse Payment (never delete)
- **Module:** Payment · **Priority:** Critical
- **Actors:** Finance Administrator (primary), Approver, System
- **Goal:** Reverse a payment (bounced cheque, error) without deleting it, preserving the trail.
- **Description:** A reversal is a new, linked transaction (not a deletion) that backs out the payment's effect; governed and audited; the original receipt and payment remain.
- **Business Rules Applied:** PAY-007, PAY-002, AUTHZ-009 (Cross-Cutting P5).
- **Preconditions:** A recorded payment; valid reversal reason; approval if configured.
- **Trigger:** Bounced cheque, gateway chargeback, or recorded error.
- **Main Success Scenario:**
  1. Admin initiates a reversal with a reason, referencing the original payment.
  2. Approval is obtained where required (SoD).
  3. System records a reversal transaction that backs out the allocation (PAY-007); the original payment/receipt remain intact.
  4. Dues are restored; balances recompute.
- **Alternative Flows:** A1) Partial reversal per policy.
- **Exception Flows:** E1) Delete attempt → blocked (UC-PAY-017). E2) Reversal of an already-reversed payment → blocked/idempotent.
- **Validation Rules:** Reversal linked/reasoned; original preserved; approved; dues restored (PAY-007, P5).
- **Permissions Required:** `payment.reverse` (elevated) + approval.
- **Notifications Triggered:** `PAYMENT_REVERSED` to financial-responsible guardian.
- **Audit Events Generated:** `PAYMENT_REVERSED` (original, reason, approver).
- **Data Created:** Reversal transaction.
- **Data Updated:** Dues balance restored.
- **Data Deleted:** None (original retained).
- **Post Conditions:** Payment effect backed out transparently; full trail preserved.
- **Related Use Cases:** UC-PAY-001, UC-PAY-007, UC-FEE-007.
- **Acceptance Criteria:**
  - Given a recorded payment, When reversed under approval, Then a linked reversal backs out its effect and the original is preserved.
  - Given a delete attempt, When made, Then it is blocked.
  - Given an already-reversed payment, When reversed again, Then it is blocked/idempotent.
- **Edge Case Analysis:**
  - *Invalid Input:* reversal without reason rejected.
  - *Permission Failure:* self-approval blocked (SoD).
  - *Concurrent Update:* reversal + new payment racing → balances consistent.
  - *Duplicate Data:* double reversal prevented.
  - *System Failure:* atomic; original never lost.
  - *Workflow Failure:* approval pends.
- **QA Coverage:**
  - *Positive:* full reversal; partial reversal; bounced cheque.
  - *Negative:* delete; double reversal; self-approval.
  - *Boundary:* reversal restoring exact prior dues; reversal after partial allocation.

### UC-PAY-004 — Reconcile Against Settlements
- **Module:** Payment · **Priority:** High
- **Actors:** Finance Administrator (primary), System
- **Goal:** Match recorded payments to actual gateway/bank settlements and flag discrepancies.
- **Description:** Compares recorded payments against settlement files; confirms cleared amounts; flags missing, extra, or mismatched entries for investigation.
- **Business Rules Applied:** PAY-005, PAY-004, PAY-008.
- **Preconditions:** Settlement data available; payments recorded.
- **Trigger:** Reconciliation run (e.g., daily).
- **Main Success Scenario:**
  1. System imports settlement data.
  2. System matches each settlement line to a recorded payment (PAY-005).
  3. Matched payments are confirmed cleared (PAY-004); discrepancies are flagged.
  4. Finance investigates and resolves flagged items.
- **Alternative Flows:** A1) Auto-match with manual review of exceptions.
- **Exception Flows:** E1) Settlement without a recorded payment → flag (possible missed record). E2) Recorded payment without settlement → flag (possible failure).
- **Validation Rules:** Deterministic matching; discrepancies surfaced, not auto-resolved (PAY-005).
- **Permissions Required:** `payment.reconcile`.
- **Notifications Triggered:** Reconciliation summary/discrepancy alerts to finance.
- **Audit Events Generated:** `RECONCILIATION_RUN`, `DISCREPANCY_FLAGGED`.
- **Data Created:** Reconciliation records.
- **Data Updated:** Payment clearance status.
- **Data Deleted:** None.
- **Post Conditions:** Payments reconciled; discrepancies visible for action.
- **Related Use Cases:** UC-PAY-003, UC-PAY-012.
- **Acceptance Criteria:**
  - Given settlement data, When reconciled, Then matched payments are confirmed cleared and discrepancies are flagged.
  - Given a settlement with no recorded payment, When reconciled, Then it is flagged for investigation.
  - Given a recorded payment with no settlement, When reconciled, Then it is flagged.
- **Edge Case Analysis:**
  - *Invalid Input:* malformed settlement file rejected.
  - *Permission Failure:* lacks `payment.reconcile` → 403.
  - *Concurrent Update:* reconciliation during new payments → snapshot/period-bound matching.
  - *Duplicate Data:* duplicate settlement lines deduped.
  - *System Failure:* re-runnable; idempotent matching.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* clean match; exception flagging.
  - *Negative:* missing/extra settlement; mismatched amount.
  - *Boundary:* partial settlement; timing across the cut-off.

---

## 7. Compact Specifications (routine use cases)

- **UC-PAY-002 — Allocate Payment to Dues (deterministic)** · *High* · Rules: PAY-002. Allocate to dues by the configured rule (e.g., oldest-first). *Edge:* deterministic, reproducible allocation; partial allocation. *QA:* allocation order; partial; remainder to credit.
- **UC-PAY-003 — Clearance Lifecycle (cheque/transfer)** · *High* · Rules: PAY-004. Non-immediate methods enter pending→cleared/bounced; dues update on clearance. *Audit:* clearance transitions. *Edge:* bounced → reversal (UC-PAY-006). *QA:* clear; bounce; pending dues handling.
- **UC-PAY-005 — Handle Overpayment (credit balance)** · *Medium* · Rules: PAY-006. Overpayment becomes a usable credit balance. *Edge:* credit applied to future dues; refundable (governed). *QA:* overpay → credit; credit applied; refund path.
- **UC-PAY-007 — Correct Mis-Posted Payment (governed)** · *High* · Rules: PAY-009, PAY-007, AUTHZ-009 (P5). Correct a payment posted to the wrong student/invoice via governed reversal + re-post. *Audit:* `PAYMENT_CORRECTED`. *Edge:* original preserved; SoD. *QA:* governed correction; silent edit blocked.
- **UC-PAY-008 — View Payment / Receipt (scoped)** · *Medium* · Rules: AUTHZ-003, GRD-N-005, FILE-005. Scoped view; financial-responsible guardian sees own child's receipts via signed URL. *QA:* scope; financial-guardian routing.
- **UC-PAY-009 — Configure Methods & Allocation Rules** · *Medium* · Rules: PAY-003, PAY-002, CFG-004. Configure methods and allocation policy (versioned). *Permissions:* `payment.config.manage`. *Edge:* allocation rule deterministic. *QA:* config applies; allocation determinism.
- **UC-PAY-010 — Approve Reversal / Correction (SoD)** · *High* · Rules: AUTHZ-009, WFL-004, PAY-007/009. Approver (≠ requester) approves reversals/corrections. *Edge:* SoD; escalation. *QA:* approval gates; self-approval blocked.
- **UC-PAY-011 — Search Payments / Receipts** · *Medium* · Rules: AUTHZ-002, REP-002. Scoped search. *QA:* scope respected.
- **UC-PAY-012 — Collection & Reconciliation Report** · *High* · Rules: REP-002, PAY-005. Collections, method mix, reconciliation status. *Edge:* scoped; consistent with reconciliation. *QA:* totals accuracy; reconciliation status.
- **UC-PAY-013 — Bulk Payment Recording / Import** · *Medium* · Rules: PAY-001/008. Bulk/import payments (idempotent, gapless receipts). *Edge:* duplicates prevented; per-row validation. *QA:* clean import; replay; invalid rows.
- **UC-PAY-014 — Export Payment Data (governed)** · *Medium* · Rules: REP-005. Governed, scoped financial export. *Permissions:* `report.export` (elevated). *Edge:* audited. *QA:* scoped; gated.
- **UC-PAY-015 — Reversal / Correction Workflow** · *High* · Rules: WFL-002/004, PAY-007/009. Version-pinned reversal/correction approval. *QA:* pinning; SoD; escalation.
- **UC-PAY-016 — Double-Payment / Replay Blocked (Exception)** · *Critical* · Rules: PAY-008. Idempotency prevents double-credit. *QA:* replay → one credit; original receipt returned.
- **UC-PAY-017 — Delete Payment Blocked (Exception)** · *Critical* · Rules: PAY-007. Payments are reversed, never deleted. *QA:* delete blocked; reversal path works.

## 8. Module-level QA & Edge Themes
- **Idempotency (PAY-008 / P10):** the headline money-safety suite — replays, double-clicks, and duplicate gateway callbacks never double-credit.
- **Gapless immutable receipts (PAY-001):** receipt numbering has no gaps and receipts are never edited.
- **Reversal-not-delete (PAY-007 / P5):** every correction is a linked reversal; nothing is destroyed.
- **Deterministic allocation (PAY-002):** allocation is reproducible and consistent with dues.
- **Reconciliation (PAY-005):** discrepancies are surfaced, never auto-resolved away.
