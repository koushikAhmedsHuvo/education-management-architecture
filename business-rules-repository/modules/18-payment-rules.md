# 18 — Payment Business Rules

## 1. Module Purpose
Govern **payments** — recording money received against invoices, allocating it, issuing receipts, and reconciling. Payment is where **money correctness** is absolute: receipts are gapless-numbered and immutable, payments are **reversed (never deleted)**, gateway callbacks are **idempotent** (no double-credit), and recorded payments **reconcile** against bank/gateway settlements. Payment settles the charges raised by Fee Management (Doc 17); Discounts (Doc 19) and Scholarships (Doc 20) reduce charges before/around payment.

## 2. Actors
- **Accountant / Cashier** — records payments, issues receipts, handles reversals and reconciliation.
- **Finance Administrator** — approves reversals/refunds, oversees reconciliation.
- **Payment Gateway (future)** — sends settlement callbacks (bKash/Nagad/cards).
- **Student / Guardian** — make/initiate payments; view receipts (own only).
- **System** — allocates payments, enforces idempotency, numbers receipts, tracks clearance, reconciles.

## 3. Use Cases

**Use Case ID:** UC-PAY-01 — Record a Payment & Issue a Receipt
**Actors:** Cashier / Guardian (online) / System
**Description:** Capture money received and produce an immutable receipt.
**Preconditions:** Outstanding dues exist; payer/student identified.
**Main Flow:** 1) Record payment (amount, method, reference). 2) System allocates per the allocation policy (PAY-002). 3) Issues a gapless, uniquely-numbered receipt; updates invoice/due status. 4) Notifies the guardian.
**Alternative Flow:** A1) Cheque/bank transfer recorded as `PENDING_CLEARANCE` until cleared (PAY-004). A2) Online gateway payment confirmed via idempotent callback (PAY-008).
**Exception Flow:** E1) Overpayment → credit balance (PAY-006). E2) Payment for the wrong student → governed correction/transfer (PAY-009).
**Post Conditions:** Payment recorded/cleared; receipt immutable; dues updated; audited.
**Business Rules Applied:** PAY-001, PAY-002, PAY-003, PAY-004.

**Use Case ID:** UC-PAY-02 — Reverse a Payment (Bounced Cheque / Error)
**Actors:** Finance Administrator, System
**Description:** Reverse a payment that failed or was recorded in error, without deleting it.
**Preconditions:** Payment exists; reversal authorized; reason recorded.
**Main Flow:** 1) Admin reverses with reason (e.g., bounced cheque). 2) System creates a reversal entry (original retained); restores the due; may add a bounce fine. 3) Receipt marked reversed; guardian notified.
**Exception Flow:** E1) Reversal of a reconciled/settled payment → governed, may need adjustment with the bank.
**Post Conditions:** Reversal recorded; dues restored; audit complete.
**Business Rules Applied:** PAY-007.

**Use Case ID:** UC-PAY-03 — Reconcile Recorded Payments to Settlements
**Actors:** Finance Administrator, System
**Description:** Match recorded payments against bank/gateway settlement reports.
**Preconditions:** Settlement data available.
**Main Flow:** 1) System matches recorded payments to settled transactions by reference. 2) Flags mismatches (missing, extra, amount differences). 3) Admin resolves exceptions via governed adjustments.
**Exception Flow:** E1) Unmatched gateway credit → investigate/allocate or refund. E2) Recorded-but-not-settled → flag for follow-up.
**Post Conditions:** Reconciliation status set; exceptions tracked; audited.
**Business Rules Applied:** PAY-005, PAY-008.

## 4. Business Rules

**Rule ID:** PAY-001
**Rule Name:** Gapless, Immutable Receipts
**Description:** Every cleared payment yields a uniquely-numbered, gapless, immutable receipt.
**Priority:** Critical
**Category:** Financial integrity
**Preconditions:** Payment recorded/cleared.
**Business Rule:** Receipt numbers follow a gapless, unique sequence (configurable scheme) per the required statutory granularity; receipts are immutable; a reversal does not delete or renumber — it marks the receipt reversed and links the reversal.
**System Action:** Assign the next sequence atomically; lock the receipt.
**Validation:** Sequence gapless and unique; no reuse.
**Failure Behavior:** Block on numbering conflict; never reuse a number.
**Audit Requirement:** Log `RECEIPT_ISSUED` with number; reversals link to it.
**Example Scenario:** Receipts 1001, 1002, 1003 with no gaps; a reversal of 1002 keeps the number and marks it reversed.
**Related Rules:** PAY-007, FEE-003 (invoice immutability).

**Rule ID:** PAY-002
**Rule Name:** Deterministic Payment Allocation
**Description:** Payments are allocated to dues by a defined, deterministic policy.
**Priority:** Critical
**Category:** Integrity
**Preconditions:** A payment is recorded against a payer with dues.
**Business Rule:** Allocation follows a configured policy (e.g., oldest-due-first / specific-invoice / by-head priority); partial payments allocate per policy; the policy is explicit so the same payment always allocates the same way. Unallocated remainder becomes a credit balance.
**System Action:** Allocate per policy; record allocation lines; carry remainder to credit.
**Validation:** Allocation policy defined; sum of allocations ≤ payment.
**Failure Behavior:** Block on undefined allocation policy.
**Audit Requirement:** Record allocation breakdown per payment.
**Example Scenario:** A partial payment clears the oldest overdue invoice first, then the next.
**Related Rules:** FEE-007, PAY-006.

**Rule ID:** PAY-003
**Rule Name:** Supported Methods & Reference Capture
**Description:** Payment methods are configurable; each requires appropriate reference data.
**Priority:** Medium
**Category:** Data quality
**Preconditions:** Payment recording.
**Business Rule:** Methods (cash, cheque, bank transfer, online gateway) are configurable; cheque/transfer/gateway payments capture reference numbers needed for clearance and reconciliation. Cash is immediate; others may be pending.
**System Action:** Capture method-specific references; set initial clearance state.
**Validation:** Required reference present per method.
**Failure Behavior:** Block recording without required references.
**Audit Requirement:** Captured in `PAYMENT_RECORDED`.
**Example Scenario:** A cheque payment records the cheque number and bank.
**Related Rules:** PAY-004, PAY-005.

**Rule ID:** PAY-004
**Rule Name:** Clearance Lifecycle for Non-Immediate Methods
**Description:** Cheque/transfer/gateway payments track a clearance lifecycle before counting as settled.
**Priority:** High
**Category:** Integrity
**Preconditions:** Non-cash payment.
**Business Rule:** Such payments enter `PENDING_CLEARANCE`; dues are provisionally marked but only fully settled on `CLEARED`. A `BOUNCED`/failed payment reverses provisionally-cleared dues (PAY-007). Receipt issuance policy for pending payments is configurable (provisional vs on-clearance).
**System Action:** Track clearance; settle on clearance; reverse on failure.
**Validation:** Clearance state transitions valid; references present.
**Failure Behavior:** Do not treat pending as fully settled; reverse on bounce.
**Audit Requirement:** Log clearance transitions (`PAYMENT_CLEARED/BOUNCED`).
**Example Scenario:** A cheque shows as pending until the bank clears it three days later.
**Related Rules:** PAY-003, PAY-007.

**Rule ID:** PAY-005
**Rule Name:** Reconciliation Against Settlements
**Description:** Recorded payments are reconciled against bank/gateway settlement data; mismatches are flagged and governed.
**Priority:** High
**Category:** Control
**Preconditions:** Settlement data available.
**Business Rule:** Reconciliation matches recorded payments to settled transactions by reference/amount; discrepancies (unmatched, duplicate, amount mismatch) are flagged and resolved via governed adjustments — never silent edits.
**System Action:** Match; flag exceptions; track resolution.
**Validation:** References/amounts compared; exception states defined.
**Failure Behavior:** Unresolved discrepancies remain flagged, not auto-cleared.
**Audit Requirement:** Log `RECONCILED`/`RECONCILIATION_EXCEPTION` with resolution.
**Example Scenario:** A gateway settlement missing a recorded payment is flagged for investigation.
**Related Rules:** PAY-008, FEE-008.

**Rule ID:** PAY-006
**Rule Name:** Overpayment Becomes Credit Balance
**Description:** Amounts beyond dues are held as a credit balance, applied to future dues or refunded.
**Priority:** Medium
**Category:** Integrity
**Preconditions:** Payment exceeds outstanding dues.
**Business Rule:** Overpayment is never lost or silently absorbed; it becomes a per-student credit balance auto-applied to future dues or refundable via the governed refund flow (FEE-008/PAY-007).
**System Action:** Record credit balance; auto-apply or refund per policy.
**Validation:** Credit tracked per student; application audited.
**Failure Behavior:** Block silent absorption of overpayment.
**Audit Requirement:** Log `CREDIT_BALANCE_CREATED/APPLIED`.
**Example Scenario:** A guardian overpays; the surplus offsets next month's tuition automatically.
**Related Rules:** FEE-008, PAY-002.

**Rule ID:** PAY-007
**Rule Name:** Payments Are Reversed, Never Deleted
**Description:** Erroneous/failed payments are reversed with a linked entry; the original is retained.
**Priority:** Critical
**Category:** Financial integrity
**Preconditions:** A payment must be undone (bounce, error, refund).
**Business Rule:** A reversal creates a distinct linked entry, restores affected dues, may add a bounce fine, and is authorized + reasoned; the original payment and receipt are never deleted. Refunds (returning money) are a governed sub-type with approval.
**System Action:** Create reversal; restore dues; preserve originals; route refunds through approval.
**Validation:** Reversal authorized; reason recorded; SoD on refunds.
**Failure Behavior:** Reject deletion or unauthorized reversal.
**Audit Requirement:** Log `PAYMENT_REVERSED`/`REFUND_ISSUED` (reason, actor, link to original).
**Example Scenario:** A bounced cheque is reversed; the due is restored and a bounce fine added — the original record stays.
**Related Rules:** PAY-001, FEE-008, FEE-003.

**Rule ID:** PAY-008
**Rule Name:** Idempotent Payment Processing (No Double-Credit)
**Description:** Repeated callbacks/submissions for the same payment never credit twice.
**Priority:** Critical
**Category:** Money correctness
**Preconditions:** Payment recording, especially gateway callbacks or retried submissions.
**Business Rule:** Each payment is keyed by a unique idempotency reference (gateway transaction id / client token); duplicate callbacks or double-clicks are recognized and ignored after the first, so a single payment is credited exactly once.
**System Action:** Deduplicate by idempotency key; first wins, duplicates acknowledged without re-crediting.
**Validation:** Idempotency key present and unique per payment.
**Failure Behavior:** Reject/ignore duplicates safely; never double-credit.
**Audit Requirement:** Log duplicate-callback detection.
**Example Scenario:** A gateway sends the success callback twice; the student is credited once.
**Related Rules:** FEE-004, PAY-005.

**Rule ID:** PAY-009
**Rule Name:** Mis-Posted Payment Correction Is Governed
**Description:** A payment recorded against the wrong student/invoice is corrected via a governed transfer, not a silent edit.
**Priority:** High
**Category:** Integrity
**Preconditions:** A payment posted to the wrong account.
**Business Rule:** Correction reverses/transfers the payment to the correct account via a linked, authorized, reasoned entry; both source and target ledgers reflect the move transparently.
**System Action:** Execute a governed transfer; update both ledgers; retain history.
**Validation:** Authorized; both accounts identified; reason recorded.
**Failure Behavior:** Reject silent re-pointing of a payment.
**Audit Requirement:** Log `PAYMENT_TRANSFERRED` (from/to, reason, actor).
**Example Scenario:** A payment posted to the wrong sibling is transferred to the correct one, transparently.
**Related Rules:** PAY-007, GRD-N-003 (siblings).

## 5. Validation Rules
- Receipt numbers gapless, unique, immutable; never reused.
- Allocation per a defined policy; allocations ≤ payment.
- Method-specific references required; clearance tracked for non-cash.
- Idempotency key required; duplicates never double-credit.
- Reversals/refunds authorized + reasoned; originals retained.
- Reconciliation exceptions flagged, not auto-cleared.

## 6. State Machine

**State Name:** RECORDED
**Description:** Payment captured (cash immediate; others pending).
**Allowed Transitions:** → CLEARED; → PENDING_CLEARANCE; → REVERSED.
**Forbidden Transitions:** deletion.
**System Actions:** Allocate; provisional due update.

**State Name:** PENDING_CLEARANCE
**Description:** Non-cash payment awaiting clearance.
**Allowed Transitions:** → CLEARED; → BOUNCED.
**Forbidden Transitions:** treat as fully settled.
**System Actions:** Provisional allocation; await clearance.

**State Name:** CLEARED
**Description:** Settled; dues reduced; receipt valid.
**Allowed Transitions:** → REVERSED (governed); → ARCHIVED.
**Forbidden Transitions:** silent edits.
**System Actions:** Finalize allocation; enable reconciliation.

**State Name:** BOUNCED
**Description:** Non-cash payment failed at clearance.
**Allowed Transitions:** → REVERSED (restore dues + fine).
**Forbidden Transitions:** counting as settled.
**System Actions:** Trigger reversal; restore dues.

**State Name:** REVERSED
**Description:** Payment undone via a linked entry; original retained.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** deletion of the original.
**System Actions:** Restore dues; preserve history.

**State Name:** ARCHIVED
**Description:** Historical payment record, immutable.
**Allowed Transitions:** none.
**Forbidden Transitions:** edits/delete.
**System Actions:** Long-term retention.

## 7. Status Definitions
Payment: `RECORDED` · `PENDING_CLEARANCE` · `CLEARED` · `BOUNCED` · `REVERSED` · `ARCHIVED`. Reconciliation: `UNRECONCILED` · `RECONCILED` · `EXCEPTION`. Receipt: `VALID` · `REVERSED`.

## 8. Workflow Rules
- Reversals and refunds require authorization (SoD: cashier records, administrator approves refunds, AUTHZ-009).
- Reconciliation exceptions follow a governed resolution flow.
- Mis-posted payment corrections are governed transfers (PAY-009).
- Bounce handling (reversal + fine) is policy-driven and audited.

## 9. Permission Rules
- `payment.record` — record payments, issue receipts.
- `payment.reverse` — reverse payments (elevated).
- `payment.refund.approve` — approve refunds (SoD).
- `payment.reconcile` — perform reconciliation.
- `payment.view` — read payments/receipts (guardians own; finance staff scoped).

## 10. Notification Rules
- `RECEIPT_ISSUED` → receipt to the paying guardian.
- `PAYMENT_CLEARED/BOUNCED` → notify guardian (bounce includes restored due + fine).
- `REFUND_ISSUED` → notify guardian.
- Reconciliation exceptions → notify finance staff (internal).
- Never expose another family's payment data.

## 11. Audit Requirements
Mandatory: `PAYMENT_RECORDED` (method, reference, idempotency key), `RECEIPT_ISSUED` (number), allocation breakdown, `PAYMENT_CLEARED/BOUNCED`, `PAYMENT_REVERSED`/`REFUND_ISSUED` (reason), `PAYMENT_TRANSFERRED`, `CREDIT_BALANCE_*`, `RECONCILED/EXCEPTION`, duplicate-callback detection. Every money movement is attributable and reversible-by-record.

## 12. Data Retention Rules
- Payments, receipts, reversals, and reconciliation retained long-term per financial/statutory retention.
- Receipt sequences retained intact (gaplessness is itself an audit property).
- Financial records exempt from erasure during legal retention; legal-hold overrides.
- Anonymization post-retention preserves non-identifying ledger integrity.

## 13. Edge Cases
- **Bounced cheque after receipt:** reversal restores dues + adds fine; receipt marked reversed, not deleted (PAY-007).
- **Gateway double-callback:** idempotency prevents double-credit (PAY-008).
- **Overpayment:** credit balance, auto-applied or refunded (PAY-006).
- **Payment for a withdrawn student:** allocated to dues or refunded; never lost.
- **Mis-posted to wrong sibling:** governed transfer (PAY-009/GRD-N-003).
- **Concurrent payment of the same invoice (two guardians):** idempotency/allocation prevent over-allocation; surplus becomes credit.
- **Partial allocation across heads/invoices:** per the defined policy (PAY-002), deterministic.
- **Reconciliation mismatch:** flagged exception, governed resolution, never silent (PAY-005).
- **Refund of a partially-used scholarship/discount-affected invoice:** computed on net paid, governed.

## 14. Failure Scenarios
- **Duplicate callback/submission:** ignored after first (PAY-008); no double-credit.
- **Payment deletion attempt:** rejected; reverse instead (PAY-007).
- **Receipt numbering conflict:** blocked; sequence integrity preserved (PAY-001).
- **Undefined allocation policy:** blocked (PAY-002).
- **Unapproved refund/reversal:** rejected (workflow/SoD).
- **Settlement mismatch:** flagged, not auto-cleared (PAY-005).

## 15. Exception Handling Rules
- Idempotency guards every recording path; the same payment credits exactly once.
- Non-cash payments never count as settled until cleared; bounces reverse.
- Reversals/refunds/transfers are governed, authorized, reasoned, and preserve originals.
- Reconciliation exceptions are surfaced and resolved, never silently absorbed.

## 16. Compliance Considerations
- **Financial controls & audit:** gapless receipts, idempotency, reconciliation, and reversal-not-delete meet statutory financial-control expectations.
- **Separation of duties:** recording vs reversing/refunding split by SoD to prevent fraud.
- **Consumer protection:** transparent receipts, governed refunds, and bounce handling protect families.
- **Legal retention:** financial records retained per statute; legal-hold overrides erasure.

## 17. Future Considerations
- Live gateway integration (bKash/Nagad/cards) with auto-reconciliation (R4).
- Auto-reconciliation from bank statement feeds.
- Installment auto-debit / standing instructions.
- Multi-currency and FX handling for international clients.
