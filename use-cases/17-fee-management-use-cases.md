# 17 — Fee Management Use Cases

Transforms the Fee Management business rules (`FEE-001`…`FEE-008`) into use cases. Defines what students owe: versioned fee structures, immutable issued invoices, duplicate prevention, pro-rata, category differentiation, dues/fines, and governed arrears/refunds.

## 1. Primary Actors
Finance Administrator (defines structures, issues invoices), Accountant (manages dues, fines, refunds).

## 2. Secondary Actors
System (versioning, immutability, duplicate prevention, pro-rata, fine accrual), Configuration Engine (fee structure versions), Payment module (settlement consumer), Discount/Scholarship modules (reductions), Notification & File modules (invoices), Audit service.

## 3. Goals
Define configurable, effective-dated, versioned fee structures by category; issue immutable invoices with no duplicates; pro-rate mid-period join/leave; track dues, due dates, and late fines fairly; carry forward arrears and govern refunds — all reproducible and auditable.

## 4. User Journeys
- **Define:** finance admin sets fee heads and schedules (versioned, effective-dated) per category.
- **Invoice:** the system generates invoices from the effective structure (idempotent, no duplicates) → immutable once issued → corrections via credit notes.
- **Collect:** dues accrue against due dates; late fines apply per policy; payments settle invoices (Payment module).
- **Adjust:** pro-rata for mid-period join/leave; governed refunds and arrears carry-forward at rollover.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-FEE-001 | Define Fee Structure (versioned, effective-dated) | High |
| Core | UC-FEE-002 | Generate / Issue Invoice (idempotent, immutable) | Critical |
| Core | UC-FEE-003 | Correct Invoice via Credit Note (governed) | High |
| Core | UC-FEE-004 | Apply Pro-Rata (mid-period join/leave) | High |
| Core | UC-FEE-005 | Accrue Dues, Due Dates & Late Fines | High |
| Core | UC-FEE-006 | Carry Forward Arrears (rollover) | Medium |
| Core | UC-FEE-007 | Process Refund (governed) | High |
| CRUD | UC-FEE-008 | View Fee Structure / Invoice (scoped) | Medium |
| Admin | UC-FEE-009 | Configure Fee Heads, Schedules & Categories | High |
| Approval | UC-FEE-010 | Approve Refund / Fine Waiver (SoD) | High |
| Search | UC-FEE-011 | Search Invoices / Dues | Medium |
| Reporting | UC-FEE-012 | Dues, Collection & Aging Report | High |
| Bulk | UC-FEE-013 | Bulk Invoice Generation | Critical |
| Import | UC-FEE-014 | Import Fee Structure / Opening Balances | Low |
| Export | UC-FEE-015 | Export Fee / Dues Data (governed) | Medium |
| Workflow | UC-FEE-016 | Refund / Waiver Workflow | High |
| Exception | UC-FEE-017 | Duplicate Invoice Blocked | High |
| Exception | UC-FEE-018 | Edit Issued Invoice Blocked | Critical |

---

## 6. Detailed Specifications (high-value use cases)

### UC-FEE-002 — Generate / Issue Invoice (idempotent, immutable)
- **Module:** Fee Management · **Priority:** Critical
- **Actors:** Finance Administrator / System (primary)
- **Goal:** Issue a correct invoice exactly once, immutable thereafter, stamped to the fee-structure version.
- **Description:** Generates an invoice from the effective fee structure for a student/period; idempotent generation prevents duplicates; once issued it is immutable and reproducible (provenance-stamped).
- **Business Rules Applied:** FEE-001, FEE-002, FEE-003, FEE-004, FILE-007.
- **Preconditions:** Active enrollment; effective fee structure; billing period open.
- **Trigger:** Invoice run (scheduled/bulk) or manual issue.
- **Main Success Scenario:**
  1. System resolves the effective, versioned fee structure for the student's category/period (FEE-001).
  2. System computes line items from fee heads/schedules (FEE-002), applying any discounts/scholarships.
  3. System performs an idempotency check (no existing invoice for the same student/period/run, FEE-004) and issues the invoice.
  4. System stamps the fee-structure/config version and generates an immutable invoice document (FEE-003, FILE-007).
- **Alternative Flows:** A1) Pro-rata for a mid-period join (UC-FEE-004). A2) Bulk run for a cohort (UC-FEE-013).
- **Exception Flows:** E1) Duplicate invoice for the same student/period → blocked (UC-FEE-017). E2) Post-issue change needed → credit note, not edit (UC-FEE-003/018).
- **Validation Rules:** Effective structure resolved; idempotent generation; immutable once issued; provenance stamped (FEE-001/003/004).
- **Permissions Required:** `fee.invoice.issue`.
- **Notifications Triggered:** `INVOICE_ISSUED` to financial-responsible guardian (GRD-N-005).
- **Audit Events Generated:** `INVOICE_ISSUED` (student, period, version, amount).
- **Data Created:** Invoice; line items; invoice document.
- **Data Updated:** Student dues balance.
- **Data Deleted:** None.
- **Post Conditions:** Immutable, reproducible invoice exists; exactly one per student/period; dues updated.
- **Related Use Cases:** UC-FEE-003, UC-FEE-013, UC-PAY-001.
- **Acceptance Criteria:**
  - Given an effective fee structure and an active enrollment, When an invoice is issued, Then it is generated from the stamped version and is immutable.
  - Given a re-run for the same student/period, When generated, Then no duplicate invoice is created (idempotent).
  - Given a needed change after issue, When attempted as an edit, Then it is blocked and a credit note is required.
- **Edge Case Analysis:**
  - *Invalid Input:* invoice for an inactive enrollment blocked.
  - *Permission Failure:* lacks `fee.invoice.issue` → 403.
  - *Concurrent Update:* two bulk runs overlapping → idempotency yields one invoice per student/period.
  - *Duplicate Data:* duplicate prevented (FEE-004).
  - *System Failure:* atomic issue with document + audit (outbox); no half-issued invoice.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* single issue; bulk issue; with discount/scholarship.
  - *Negative:* duplicate run; inactive enrollment; post-issue edit.
  - *Boundary:* invoice at period boundary; idempotency-window edge; zero-net invoice (full waiver).

### UC-FEE-003 — Correct Invoice via Credit Note (governed)
- **Module:** Fee Management · **Priority:** High
- **Actors:** Finance Administrator (primary), Approver, System
- **Goal:** Correct an issued invoice without editing it, preserving the financial trail.
- **Description:** Applies the Governed Correction Pattern (P5) to finance: the issued invoice is never edited or deleted; a linked credit note (and, if needed, a new invoice) records the correction with reason, actor, and approval where required.
- **Business Rules Applied:** FEE-003, FEE-008, AUTHZ-009 (Cross-Cutting P5).
- **Preconditions:** An issued invoice; a justified correction; approval if configured.
- **Trigger:** An error or change requires correcting an issued invoice.
- **Main Success Scenario:**
  1. Admin initiates a correction with a reason.
  2. System issues a linked credit note (full/partial) against the original invoice (FEE-003).
  3. If a corrected charge is needed, a new invoice is issued; the original remains intact.
  4. Approval is obtained where required (SoD); balances recompute.
- **Alternative Flows:** A1) Credit note offsets to a credit balance (PAY-006).
- **Exception Flows:** E1) Direct edit/delete of the invoice → blocked (UC-FEE-018). E2) Closed financial period → governed reopen (SESS-006) before correction.
- **Validation Rules:** Original preserved; credit note linked/reasoned; approved where required (FEE-003, P5).
- **Permissions Required:** `fee.creditnote.issue` (+ approval).
- **Notifications Triggered:** `CREDIT_NOTE_ISSUED` to financial-responsible guardian.
- **Audit Events Generated:** `CREDIT_NOTE_ISSUED` (original invoice, amount, reason, approver).
- **Data Created:** Credit note; (optional) corrected invoice.
- **Data Updated:** Dues/credit balance.
- **Data Deleted:** None (original retained).
- **Post Conditions:** Correction transparent; original invoice intact; balances accurate.
- **Related Use Cases:** UC-FEE-002, UC-PAY-006, UC-FEE-010.
- **Acceptance Criteria:**
  - Given an issued invoice needing correction, When corrected, Then a linked credit note records it and the original is preserved.
  - Given a direct edit attempt, When made, Then it is blocked.
  - Given a closed period, When correction is needed, Then a governed reopen is required first.
- **Edge Case Analysis:**
  - *Invalid Input:* credit note exceeding the invoice amount blocked (unless policy allows).
  - *Permission Failure:* lacks `fee.creditnote.issue` → 403; self-approval blocked.
  - *Concurrent Update:* credit note + payment racing → balances consistent.
  - *Duplicate Data:* duplicate credit note prevented.
  - *System Failure:* atomic; original never altered.
  - *Workflow Failure:* approval pends.
- **QA Coverage:**
  - *Positive:* full/partial credit note; corrected re-invoice.
  - *Negative:* invoice edit; over-amount credit; self-approval.
  - *Boundary:* credit note equal to invoice (net zero); closed-period correction.

### UC-FEE-005 — Accrue Dues, Due Dates & Late Fines
- **Module:** Fee Management · **Priority:** High
- **Actors:** System (primary), Accountant (oversight)
- **Goal:** Track outstanding dues against due dates and apply late fines fairly per policy.
- **Description:** Computes dues from issued invoices net of payments/discounts; applies configured late fines after due dates with grace, transparently and reversibly.
- **Business Rules Applied:** FEE-007, FEE-002, PAY-002.
- **Preconditions:** Invoices issued with due dates; fine policy configured.
- **Trigger:** Due-date passage / dues evaluation.
- **Main Success Scenario:**
  1. System computes outstanding dues = issued − paid − reductions (FEE-002/007).
  2. After a due date (plus any grace), System accrues the configured late fine (FEE-007).
  3. Fines are recorded transparently as line items (reversible via credit note if waived).
- **Alternative Flows:** A1) Fine waiver (governed, UC-FEE-010). A2) Installment schedules with per-installment due dates.
- **Exception Flows:** E1) Disputed dues → flagged; collection actions held pending resolution.
- **Validation Rules:** Dues computed deterministically; fines per policy with grace; transparent and reversible (FEE-007).
- **Permissions Required:** System; `fee.fine.manage` for waivers.
- **Notifications Triggered:** `DUE_REMINDER`, `LATE_FINE_APPLIED` to financial-responsible guardian.
- **Audit Events Generated:** `LATE_FINE_APPLIED/WAIVED`.
- **Data Created:** Fine line items.
- **Data Updated:** Dues balance.
- **Data Deleted:** None.
- **Post Conditions:** Accurate dues with transparent, reversible fines.
- **Related Use Cases:** UC-FEE-007, UC-PAY-002, UC-RES-006 (withholding).
- **Acceptance Criteria:**
  - Given an unpaid invoice past its due date and grace, When evaluated, Then the configured late fine is applied transparently.
  - Given a waiver, When approved, Then the fine is reversed via a governed action.
  - Given partial payment, When dues are computed, Then the remaining balance and any fine reflect the net.
- **Edge Case Analysis:**
  - *Invalid Input:* negative dues impossible (clamped/credit balance).
  - *Permission Failure:* fine waiver without permission → 403.
  - *Concurrent Update:* payment at the fine-accrual moment → consistent net (payment-before-fine ordering per policy).
  - *Duplicate Data:* one fine per overdue period (no double-fining).
  - *System Failure:* deterministic recompute.
  - *Workflow Failure:* waiver approval pends.
- **QA Coverage:**
  - *Positive:* fine after grace; waiver; partial-payment net.
  - *Negative:* double-fine; negative balance.
  - *Boundary:* payment exactly on the due date / end of grace.

---

## 7. Compact Specifications (routine use cases)

- **UC-FEE-001 — Define Fee Structure (versioned, effective-dated)** · *High* · Rules: FEE-001, FEE-006, CFG-004. Define heads/amounts per category, versioned, effective forward. *Permissions:* `fee.config.manage`. *Audit:* `FEE_STRUCTURE_PUBLISHED`. *Edge:* future-dated change applies forward; historical invoices keep their version. *QA:* versioned define; effective-dating; category differentiation.
- **UC-FEE-004 — Apply Pro-Rata (mid-period join/leave)** · *High* · Rules: FEE-005. Pro-rate charges for mid-period join/leave per policy. *Edge:* fair fraction; leave triggers credit/refund. *QA:* join pro-rata; leave pro-rata; boundary dates.
- **UC-FEE-006 — Carry Forward Arrears (rollover)** · *Medium* · Rules: FEE-008, SESS-006. Carry outstanding dues to the next session at rollover. *Edge:* arrears visible/linked; not silently dropped. *QA:* carry-forward; linkage; no loss.
- **UC-FEE-007 — Process Refund (governed)** · *High* · Rules: FEE-008, PAY-007, AUTHZ-009. Governed refund (approval, reason); never a silent reversal. *Audit:* `REFUND_PROCESSED`. *Edge:* refund ≤ paid; SoD; reversal-not-delete. *QA:* approved refund; over-refund blocked; SoD.
- **UC-FEE-008 — View Fee Structure / Invoice (scoped)** · *Medium* · Rules: AUTHZ-003, GRD-N-005. Scoped view; financial-responsible guardian sees their child's invoices. *Edge:* only financial-responsible guardian for finance. *QA:* scope; financial-guardian routing.
- **UC-FEE-009 — Configure Fee Heads, Schedules & Categories** · *High* · Rules: FEE-002, FEE-006. Configure heads (tuition/transport/lab), schedules, categories (incl. non-waivable heads). *Edge:* non-waivable classification feeds DSC-006 (C-03). *QA:* head/schedule config; non-waivable flag.
- **UC-FEE-010 — Approve Refund / Fine Waiver (SoD)** · *High* · Rules: AUTHZ-009, WFL-004. Approver (≠ requester) approves refunds/waivers. *Edge:* SoD; escalation. *QA:* approval gates; self-approval blocked.
- **UC-FEE-011 — Search Invoices / Dues** · *Medium* · Rules: AUTHZ-002, REP-002. Scoped search. *QA:* scope respected.
- **UC-FEE-012 — Dues, Collection & Aging Report** · *High* · Rules: REP-002. Dues aging, collection rates, defaulters. *Edge:* scoped; consistent with dues computation. *QA:* aging accuracy; collection totals.
- **UC-FEE-013 — Bulk Invoice Generation** · *Critical* · Rules: FEE-002/004, RES-009-style async. Generate invoices for a cohort (idempotent, async, no duplicates). *Edge:* partial-failure resumable; duplicates prevented. *QA:* large run; idempotent re-run; partial recovery.
- **UC-FEE-014 — Import Fee Structure / Opening Balances** · *Low* · Rules: FEE-001/008. Import structures/opening arrears (validated). *Edge:* balances reconciled; invalid rows reported. *QA:* clean import; reconciliation.
- **UC-FEE-015 — Export Fee / Dues Data (governed)** · *Medium* · Rules: REP-005. Governed, scoped financial export. *Permissions:* `report.export` (elevated). *Edge:* financial data export audited. *QA:* scoped; gated; audited.
- **UC-FEE-016 — Refund / Waiver Workflow** · *High* · Rules: WFL-002/004, FEE-008. Version-pinned refund/waiver approval. *QA:* pinning; SoD; escalation.
- **UC-FEE-017 — Duplicate Invoice Blocked (Exception)** · *High* · Rules: FEE-004. Duplicate invoice for the same student/period prevented. *QA:* duplicate blocked; idempotent run.
- **UC-FEE-018 — Edit Issued Invoice Blocked (Exception)** · *Critical* · Rules: FEE-003. Direct edit/delete of an issued invoice blocked; credit note only. *QA:* edit blocked; credit-note path works.

## 8. Module-level QA & Edge Themes
- **Invoice immutability (FEE-003 / P5):** issued invoices never edited/deleted; corrections via credit notes — the headline financial-integrity suite.
- **Idempotent generation (FEE-004 / P10):** bulk and re-run invoice generation never duplicates.
- **Reproducibility (FEE-001 / P6):** invoices stamped to the fee-structure version reproduce exactly.
- **Non-waivable heads (FEE-002 / C-03):** statutory heads classified here; waiver semantics owned by Discount (DSC-006).
- **Transparent, reversible fines (FEE-007):** fines are line items, never silent, always reversible via governed waiver.
