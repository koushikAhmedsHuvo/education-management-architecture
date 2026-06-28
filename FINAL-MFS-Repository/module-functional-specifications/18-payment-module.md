# 18 — Payment Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`PAY-001…009`), and Use Case Repository (`UC-PAY-001…017`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Receive money safely: gapless immutable receipts, idempotent processing (no double-credit), deterministic allocation to dues, multi-method capture, clearance lifecycle for non-immediate methods, reconciliation against settlements, overpayment-as-credit, reversal-not-delete, and governed mis-post correction.

**Business Goal.** Ensure every payment is recorded exactly once with an immutable receipt, allocated deterministically, reconciled against actual settlements, and corrected only by reversal — never deletion.

**Scope.** Payment recording + gapless immutable receipts; idempotency; deterministic allocation; supported methods & reference capture; clearance lifecycle (cheque/transfer); reconciliation; overpayment→credit; reversal (never delete); governed mis-post correction; configuration of methods/allocation.

**Out of Scope.** Invoice/dues definition (Fee module — Payment settles invoices). Discount/scholarship reductions (Discount/Scholarship modules). Payment-gateway internals (integration adapter). Refund disbursement policy (Fee module governs refund; Payment executes reversal).

---

## 2. Actors

**Primary Actors.** Accountant / Cashier (records payments), Finance Administrator (reconciliation, reversals), Guardian (online payment), Approver (reversal/correction — SoD), System (idempotency, receipt numbering, allocation, clearance).

**Secondary Actors.** Payment Gateway (online), Fee module (invoices/dues), File/Notification (receipts), Workflow Engine (reversal/correction approval), Configuration Engine (methods/allocation), Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Gapless immutable receipts | Issue sequential, immutable receipts on payment. | Critical | FR-002 |
| FR-002 | Idempotent processing | Prevent double-credit on retries/replays. | Critical | Gateway, Config |
| FR-003 | Deterministic allocation | Allocate to dues by configured rule (e.g., oldest-first). | High | Fee module |
| FR-004 | Methods & reference capture | Support methods; capture reference/transaction id. | High | Configuration Engine |
| FR-005 | Clearance lifecycle | Non-immediate methods enter pending→cleared/bounced. | High | FR-001 |
| FR-006 | Reconciliation | Match recorded payments to settlements; flag discrepancies. | High | Gateway/Bank |
| FR-007 | Overpayment → credit | Convert overpayment to a usable credit balance. | Medium | Fee module |
| FR-008 | Reversal (never delete) | Reverse via linked transaction; never delete. | Critical | Workflow (P5) |
| FR-009 | Governed mis-post correction | Correct wrong-student/invoice posting under SoD. | High | Workflow Engine |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Idempotent payments | No double-credit. | Money safety. |
| Gapless immutable receipts | Sequential, never edited. | Audit integrity. |
| Deterministic allocation | Reproducible application to dues. | Predictable balances. |
| Clearance lifecycle | Cheque/transfer states. | Accurate dues timing. |
| Reconciliation | Settlement matching. | Discrepancy detection. |
| Overpayment credit | Usable credit balance. | Correct handling. |
| Reversal-not-delete | Linked reversal. | Full trail. |
| Governed correction | SoD mis-post fix. | Integrity. |

---

## 5. Screens

Record Payment (counter); Online Payment (guardian); Receipt View; Payment List; Payment Detail; Allocation View; Clearance Management; Reconciliation; Overpayment/Credit Balance; Reverse Payment; Mis-Post Correction; Method/Allocation Configuration; Collection & Reconciliation Report; Bulk Payment/Import; Payment Export; Reversal/Correction Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Record Payment | Enter Amount/Method/Reference, Record, Print Receipt | Bulk/Import |
| Online Payment | Pay, View Receipt | — |
| Payment List | Open, Search/Filter, Export | — |
| Payment Detail | View, Reverse, Correct (governed), Re-allocate | — |
| Clearance | Confirm Cleared, Mark Bounced | Bulk Clear |
| Reconciliation | Import Settlement, Match, Flag Discrepancy, Resolve | Auto-Match |
| Reverse Payment | Reverse (reason), Submit (→ approval) | — |
| Method Config | Configure Methods/Allocation, Save (versioned) | — |
| Reconciliation Report | Run, Filter, Export | Export |

---

## 7. Forms

**Record Payment** — `student`/`invoice`, `amount` (number, required), `method` (select), `reference` (text), `idempotencyKey` (system). Validation: idempotent (PAY-008); amount > 0; gapless immutable receipt (PAY-001); deterministic allocation (PAY-002); method/reference captured (PAY-003).

**Reverse Payment** — `payment` (required), `reason` (text, required), `amount` (full/partial). Validation: linked reversal, original preserved (PAY-007); SoD approval (AUTHZ-009); dues restored.

**Mis-Post Correction** — `payment`, `correctTarget` (student/invoice), `reason` (text, required). Validation: governed reversal + re-post (PAY-009); SoD; original preserved.

**Clearance** — `status` (cleared/bounced). Validation: dues update on clearance; bounce → reversal (PAY-004).

**Method/Allocation Config** — `methods` (multi), `allocationRule` (oldest-first/…). Validation: deterministic (PAY-002); versioned (CFG-004).

---

## 8. Search & Filter Requirements

**Payments:** by receipt number, student, method, status (recorded/cleared/bounced/reversed), date range, reconciliation status. Sorting: date/amount/receipt. Pagination: server-side, 25 default. Scope-bound; guardian sees own child's receipts.

---

## 9. Table Requirements

**Payment table:** Receipt No., Student, Amount, Method, Status, Date, Allocated, Reconciled. Sorting on Date/Receipt. Filtering as above. Export (governed — financial). Bulk: import, clear.

---

## 10. Workflow Requirements

**Trigger events:** record payment, clearance transition, reconcile, overpayment, reverse, mis-post correction. **Status changes:** payment `RECORDED → CLEARED/BOUNCED → REVERSED(governed)`. **Approvals:** reversals/corrections via Workflow Engine (SoD). **Notifications:** payment received + receipt, clearance, reversal (to financial-responsible guardian). **Audit:** payments, allocations, clearances, reversals, corrections (immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Record payment | `payment.record` |
| Reconcile | `payment.reconcile` |
| Reverse payment | `payment.reverse` (elevated) |
| Correct mis-post | `payment.correct` (elevated) |
| Approve reversal/correction | `payment.reversal.approve` (SoD) |
| Configure methods/allocation | `payment.config.manage` |
| View payment/receipt | `payment.view` (scoped) |
| Import/export | `payment.import`, `payment.export` |

Reversals/corrections are elevated + SoD; receipts immutable.

---

## 12. Business Rule References

PAY-001 (gapless immutable receipts), PAY-002 (deterministic allocation), PAY-003 (methods & reference capture), PAY-004 (clearance lifecycle), PAY-005 (reconciliation), PAY-006 (overpayment → credit), PAY-007 (reversal, never delete), PAY-008 (idempotent processing), PAY-009 (governed mis-post correction). Cross-cutting: FEE-002/007 (invoices/dues), FILE-007 (immutable receipt doc), WFL-002/004, AUTHZ-009 (SoD), GRD-N-005 (financial-responsible), AUD-001, CFG-004.

## 13. Use Case References

UC-PAY-001 (Record & Issue Receipt — idempotent), UC-PAY-002 (Allocate — deterministic), UC-PAY-003 (Clearance Lifecycle), UC-PAY-004 (Reconcile), UC-PAY-005 (Overpayment → Credit), UC-PAY-006 (Reverse — never delete), UC-PAY-007 (Correct Mis-Post), UC-PAY-008 (View — scoped), UC-PAY-009 (Configure Methods/Allocation), UC-PAY-010 (Approve Reversal/Correction — SoD), UC-PAY-011 (Search), UC-PAY-012 (Collection & Reconciliation Report), UC-PAY-013 (Bulk/Import), UC-PAY-014 (Export — governed), UC-PAY-015 (Reversal/Correction Workflow), UC-PAY-016 (Double-Payment/Replay Blocked), UC-PAY-017 (Delete Payment Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Record payment (idempotent) | POST | Cashier / Guardian |
| Get receipt (signed URL) | GET | Authorized / Guardian |
| Allocate / re-allocate to dues | POST | Accountant |
| Confirm clearance / mark bounced | POST | Accountant |
| Reconcile against settlement | POST | Finance Admin |
| Reverse payment | POST | Finance Admin (→ approval) |
| Correct mis-posted payment | POST | Finance Admin (→ approval) |
| Configure methods/allocation | GET/PUT | Finance Admin |
| Collection & reconciliation report | GET | Finance Admin |
| Bulk record / import / export | POST/GET | Finance Admin |

Idempotency keys prevent double-credit (PAY-008); receipts are gapless and immutable (PAY-001); payments are reversed, never deleted (PAY-007).

---

## 15. Database Requirements

**Entities:** `Payment` (studentId, amount, method, reference, status, idempotencyKey), `Receipt` (sequential number, immutable, file ref), `Allocation` (payment→invoice/dues), `Clearance` (status, dates), `Reversal` (paymentId, reason, approver), `CreditBalance` (student), `Reconciliation` (settlement match, discrepancy). **Relationships:** Payment 1—1 Receipt; Payment 1—* Allocation; Payment 1—* Reversal. **Indexes:** unique(Receipt.number) gapless, unique(Payment.idempotencyKey) (PAY-008), index(Payment.studentId, status), index(Reconciliation.status). Receipts immutable; reversal-not-delete (PAY-007).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Payment received + receipt; clearance (cleared/bounced); reversal — to financial-responsible guardian. |
| SMS | Payment confirmation (minimized). |
| Push | Payment/receipt notices. |
| In-App | Reconciliation discrepancies; reversal/correction approvals. |

Receipts delivered via signed URL (FILE-005); routed to financial-responsible guardian (GRD-N-005).

---

## 17. Audit Requirements

Log: payment recording (receipt no., method, amount), allocation, clearance transitions, reconciliation runs/discrepancies, reversals (reason, approver), mis-post corrections. Record who/when/before/after. Payments/reversals are first-class financial audit events. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Collections (by method/period), Reconciliation status, Bounced/reversed payments, Credit balances, Cashier-wise collection. **Exports:** governed payment export (audited). **Dashboards:** collection overview, reconciliation health, discrepancy alerts.

---

## 19. Error Handling

**Validation:** replay/double-payment, delete attempt, zero/negative amount → specific errors (UC-PAY-016/017). **Permission:** reversal/correction without authority/self-approval → blocked. **Workflow:** reversal/correction pending → not applied until approved. **System:** gateway failure mid-flow → no partial receipt; settlement file malformed → reconciliation rejected.

---

## 20. Edge Cases

**Concurrent updates:** double-click/parallel callback → idempotency yields one payment (PAY-008). **Duplicate data:** replay returns original receipt; receipt numbering gapless. **Partial failures:** bulk import partial → per-row report. **Rollback:** reversal of already-reversed payment → blocked/idempotent. **Reconciliation race:** reconciliation during new payments → period-bound snapshot matching.

---

## 21. Acceptance Criteria

**Functional.** Payments record exactly once (idempotent, no double-credit) with gapless immutable receipts; allocation to dues is deterministic; non-immediate methods follow a clearance lifecycle; reconciliation matches settlements and flags discrepancies without auto-resolving; overpayment becomes a credit balance; payments are reversed (linked, reasoned, SoD-approved), never deleted; mis-posts are corrected under governance.

**Business.** Money received is always accurately recorded and traceable; double-credit and silent deletion are impossible; balances reconcile against actual settlements; corrections preserve the full financial trail.

---

## 22. Future Enhancements

Multiple gateway integrations with smart routing; auto-reconciliation with bank feeds; QR/UPI/mobile-wallet capture; partial-payment plans; payment links via SMS/email; predictive collection reminders; multi-currency settlement.
