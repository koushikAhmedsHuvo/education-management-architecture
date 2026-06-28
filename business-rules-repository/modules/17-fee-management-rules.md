# 17 — Fee Management Business Rules

## 1. Module Purpose
Govern **fee management** — defining fee structures, generating invoices, and tracking dues. Fees are *configuration* (effective-dated, version-stamped) consumed to produce **immutable invoices**. The dominant concern is **financial integrity**: an issued invoice is never edited (corrections are credit notes/adjustments), historical invoices are unaffected by future fee changes, and every figure is auditable. This module is the source of charges; Payment (Doc 18) settles them, Discount (Doc 19) and Scholarship (Doc 20) reduce them.

## 2. Actors
- **Accountant / Finance Administrator** — defines fee structures, generates invoices, issues adjustments/credit notes, manages arrears.
- **Institute Administrator** — approves high-impact fee configuration where required.
- **Student / Guardian** — view invoices and dues (own only).
- **System** — resolves effective fee structures, generates immutable version-stamped invoices, computes dues/fines, prevents duplicates.

## 3. Use Cases

**Use Case ID:** UC-FEE-01 — Define a Fee Structure
**Actors:** Finance Administrator, System
**Description:** Configure fee heads, amounts, schedules, and applicability for a session.
**Preconditions:** Session active; actor holds `fee.structure.manage`.
**Main Flow:** 1) Define fee heads (tuition, transport, lab, exam, etc.) with amounts mapped to structure nodes (class/section/category). 2) Set schedule (one-time, monthly, term, installment), due dates, and late-fine rules. 3) Set an effective-from date. 4) Publish as an immutable version.
**Alternative Flow:** A1) Category-specific structures (staff-ward, sibling) defined (FEE-006).
**Exception Flow:** E1) Overlapping effective periods for the same scope/head → reject. E2) Negative/invalid amounts → reject.
**Post Conditions:** Fee structure versioned and effective; audited.
**Business Rules Applied:** FEE-001, FEE-002, FEE-006.

**Use Case ID:** UC-FEE-02 — Generate Invoices (Bulk, Peak Event)
**Actors:** Finance Administrator, System
**Description:** Produce invoices for a cohort per the effective structure and schedule.
**Preconditions:** Effective fee structure resolved; students actively enrolled (ENR-008).
**Main Flow:** 1) System resolves the effective structure for each student's scope/category. 2) Computes invoice lines per head, applies pro-rata where configured, stamps the fee-config version. 3) Issues immutable, uniquely-numbered invoices asynchronously in batches. 4) Discounts/scholarships apply per Docs 19–20.
**Alternative Flow:** A1) Single ad-hoc invoice for a specific charge.
**Exception Flow:** E1) Duplicate invoice for the same student/period/head → prevented (FEE-004). E2) No effective structure → block with a flag.
**Post Conditions:** Invoices issued and immutable; dues tracked; audited.
**Business Rules Applied:** FEE-003, FEE-004, FEE-005, FEE-007.

**Use Case ID:** UC-FEE-03 — Adjust an Issued Invoice (Credit Note)
**Actors:** Finance Administrator, System
**Description:** Correct or reduce an issued invoice without editing it.
**Preconditions:** Invoice issued; actor authorized; reason recorded.
**Main Flow:** 1) Admin issues a credit note/adjustment against the invoice with a reason. 2) System records the adjustment as a distinct, linked document (original invoice unchanged). 3) Net due recomputed; provenance retained.
**Exception Flow:** E1) Direct edit of the issued invoice attempted → blocked (FEE-003). E2) Adjustment exceeding the invoice → blocked/capped.
**Post Conditions:** Adjustment recorded; original immutable; audited.
**Business Rules Applied:** FEE-003, FEE-008.

## 4. Business Rules

**Rule ID:** FEE-001
**Rule Name:** Configurable, Effective-Dated, Versioned Fee Structure
**Description:** Fee structures are configurable, carry an effective-from date, and are published as immutable versions.
**Priority:** Critical
**Category:** Configuration / integrity
**Preconditions:** Fee structure definition.
**Business Rule:** A fee structure maps fee heads to amounts for a scope (institute/campus/class/section/category) with a schedule and effective date; publishing creates an immutable version. A change creates a new version effective from a future date and never alters already-issued invoices.
**System Action:** Version and effective-date structures; resolve most-specific-wins (campus → institute → org).
**Validation:** Amounts ≥ 0; no overlapping effective periods per scope/head; schedule valid.
**Failure Behavior:** Reject overlaps/invalid amounts; require a new version for changes.
**Audit Requirement:** Log `FEE_STRUCTURE_PUBLISHED` with version + effective date.
**Example Scenario:** A tuition increase effective next term leaves this term's issued invoices unchanged.
**Related Rules:** FEE-003, INST-005, SESS-005.

**Rule ID:** FEE-002
**Rule Name:** Fee Heads & Schedules
**Description:** Fees are composed of named heads with defined schedules (one-time, monthly, term, installment).
**Priority:** High
**Category:** Structure
**Preconditions:** Fee structure definition.
**Business Rule:** Each fee head has a type (recurring/one-time), a schedule, due dates, and optional late-fine rules. Statutory/mandatory heads are distinguished from optional ones (affects withholding and waiver rules).
**System Action:** Drive invoice line generation and due-date/fine computation from heads/schedules.
**Validation:** Schedule consistent; due dates valid; fine rules bounded.
**Failure Behavior:** Reject inconsistent schedules.
**Audit Requirement:** Captured in structure publication.
**Example Scenario:** Tuition is monthly; admission is one-time; exam fee is per-term.
**Related Rules:** FEE-005, FEE-007.

**Rule ID:** FEE-003
**Rule Name:** Issued Invoices Are Immutable
**Description:** Once issued, an invoice is never edited; corrections are credit notes/adjustments.
**Priority:** Critical
**Category:** Financial integrity
**Preconditions:** Invoice issued.
**Business Rule:** An issued invoice's line items and amounts are read-only. Reductions/corrections are made via linked credit notes or adjustment documents; cancellation voids (not deletes) the invoice with a reason. The original always remains.
**System Action:** Lock issued invoices; record adjustments/voids as distinct linked documents.
**Validation:** No write path edits an issued invoice; adjustments authorized + reasoned.
**Failure Behavior:** Reject any direct edit of an issued invoice.
**Audit Requirement:** Log `INVOICE_ISSUED`, `INVOICE_ADJUSTED` (credit note), `INVOICE_VOIDED` (reason).
**Example Scenario:** An over-billed invoice is corrected by a credit note, not by changing the original figure.
**Related Rules:** FEE-008, PAY-007 (reversal parallel).

**Rule ID:** FEE-004
**Rule Name:** Duplicate-Invoice Prevention
**Description:** The system prevents issuing duplicate invoices for the same student/period/head.
**Priority:** High
**Category:** Integrity
**Preconditions:** Invoice generation.
**Business Rule:** Idempotent generation prevents charging a student twice for the same head and period; re-running a batch does not duplicate already-issued invoices.
**System Action:** Check existing invoices per student/period/head; skip/idempotent on re-run.
**Validation:** No existing invoice for the same key.
**Failure Behavior:** Skip duplicates; flag anomalies.
**Audit Requirement:** Log skipped duplicates in batch runs.
**Example Scenario:** Re-running the monthly batch after a partial failure does not double-charge already-invoiced students.
**Related Rules:** FEE-007, PAY-008 (idempotency parallel).

**Rule ID:** FEE-005
**Rule Name:** Pro-Rata for Mid-Period Join/Leave
**Description:** Fees are pro-rated for students who join or leave mid-period where configured.
**Priority:** Medium
**Category:** Fairness
**Preconditions:** Mid-period admission/withdrawal/transfer.
**Business Rule:** Where pro-rata is enabled, recurring fees are apportioned for the part of the period the student is enrolled; a transferred student's fees split by period across sections/campuses (ENR-005). Pro-rata method (daily/weekly) is configurable.
**System Action:** Compute apportioned amounts per the method and enrollment dates.
**Validation:** Pro-rata method defined; enrollment dates valid.
**Failure Behavior:** Default to full-period or flag if pro-rata undefined.
**Audit Requirement:** Pro-rata basis recorded on the invoice.
**Example Scenario:** A student admitted mid-month is billed a partial tuition for that month.
**Related Rules:** ENR-005, FEE-002.

**Rule ID:** FEE-006
**Rule Name:** Category-Based Fee Differentiation
**Description:** Different student categories may have different fee structures (sibling, staff-ward, sponsored).
**Priority:** Medium
**Category:** Configuration
**Preconditions:** Categories defined.
**Business Rule:** A student's category resolves the applicable structure (e.g., staff-ward concessional tuition); category-based differences are structural, distinct from discounts (Doc 19) and scholarships (Doc 20).
**System Action:** Resolve structure by student category within scope specificity.
**Validation:** Category valid; structure exists for it.
**Failure Behavior:** Fall back to the default structure if no category-specific one.
**Audit Requirement:** Category basis recorded on the invoice.
**Example Scenario:** A staff member's child is billed the staff-ward tuition structure.
**Related Rules:** FEE-001, DSC (distinct), SCH (distinct).

**Rule ID:** FEE-007
**Rule Name:** Dues, Due Dates & Late Fines
**Description:** Outstanding dues, due dates, and configurable late fines are computed and tracked deterministically.
**Priority:** High
**Category:** Computation integrity
**Preconditions:** Invoices issued.
**Business Rule:** Net due = issued − adjustments − payments − approved reductions. Late fines apply per configured rules after due dates (bounded, possibly capped, configurable grace). Fines are added as distinct charges, not silent invoice edits (FEE-003).
**System Action:** Compute dues and fines; add fines as new charges; track aging.
**Validation:** Fine rules bounded; grace defined; due dates valid.
**Failure Behavior:** Block fine application beyond configured caps.
**Audit Requirement:** Log `LATE_FINE_APPLIED` with basis.
**Example Scenario:** A fine is added as a new line after a 7-day grace, capped at the configured maximum.
**Related Rules:** FEE-003, PAY-002 (allocation).

**Rule ID:** FEE-008
**Rule Name:** Arrears Carry-Forward & Refunds Are Governed
**Description:** Unpaid dues carry forward explicitly across periods/sessions; refunds and credit balances are governed.
**Priority:** High
**Category:** Lifecycle / integrity
**Preconditions:** Session/period transition or overpayment/withdrawal.
**Business Rule:** Carry-forward of arrears to the next period/session is an explicit, recorded decision (SESS-007), not silent. Overpayments become a credit balance; refunds (e.g., on withdrawal) follow a governed, approved, audited process (links PAY-007 refunds).
**System Action:** Record carry-forward; maintain credit balances; route refunds through approval.
**Validation:** Carry-forward decision recorded; refund authorized.
**Failure Behavior:** Block silent arrears loss or unapproved refunds.
**Audit Requirement:** Log `ARREARS_CARRIED_FORWARD`, `REFUND_APPROVED/ISSUED`.
**Example Scenario:** A withdrawing student's paid advance is refunded via an approved, audited process.
**Related Rules:** SESS-007, PAY-007, ENR-006 (withdrawal clearance).

## 5. Validation Rules
- Fee amounts ≥ 0; no overlapping effective periods per scope/head.
- Invoices immutable once issued; corrections via credit notes only.
- No duplicate invoice per student/period/head (idempotent generation).
- Pro-rata and fine rules defined and bounded.
- Net due = issued − adjustments − payments − reductions (deterministic).
- Refunds and arrears carry-forward are explicit and authorized.

## 6. State Machine

**State Name:** DRAFT
**Description:** Invoice computed, not yet issued.
**Allowed Transitions:** → ISSUED; → DISCARDED.
**Forbidden Transitions:** payment against a draft.
**System Actions:** Compute lines; await issue.

**State Name:** ISSUED
**Description:** Official, immutable invoice with dues outstanding.
**Allowed Transitions:** → PARTIALLY_PAID; → PAID; → ADJUSTED (credit note); → VOIDED.
**Forbidden Transitions:** edits to lines/amounts.
**System Actions:** Track dues/aging; accept payments/adjustments.

**State Name:** PARTIALLY_PAID
**Description:** Some payment allocated; balance remains.
**Allowed Transitions:** → PAID; → ADJUSTED; → VOIDED (governed, with refund of paid part).
**Forbidden Transitions:** line edits.
**System Actions:** Update balance; apply fines on overdue.

**State Name:** PAID
**Description:** Fully settled.
**Allowed Transitions:** → ADJUSTED (post-payment credit/refund); → ARCHIVED.
**Forbidden Transitions:** line edits.
**System Actions:** Mark settled; enable receipts.

**State Name:** ADJUSTED
**Description:** A credit note/adjustment is linked; net due recomputed.
**Allowed Transitions:** → PAID/PARTIALLY_PAID/VOIDED per net; → ARCHIVED.
**Forbidden Transitions:** altering the original invoice.
**System Actions:** Maintain linked documents; recompute net.

**State Name:** VOIDED
**Description:** Cancelled with reason; not deleted.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** reactivation (re-issue is a new invoice).
**System Actions:** Preserve void record; handle any paid-part refund.

**State Name:** ARCHIVED
**Description:** Historical financial record, immutable.
**Allowed Transitions:** none.
**Forbidden Transitions:** edits/hard-delete.
**System Actions:** Long-term retention.

## 7. Status Definitions
Invoice: `DRAFT` · `ISSUED` · `PARTIALLY_PAID` · `PAID` · `ADJUSTED` · `VOIDED` · `ARCHIVED`. Derived: `OVERDUE` (past due, unpaid). Structure: `DRAFT` · `ACTIVE` (versioned) · `SUPERSEDED` · `ARCHIVED`.

## 8. Workflow Rules
- High-impact fee-structure changes may require approval (configurable).
- Credit notes/adjustments above a threshold and refunds require approval (SoD: requester ≠ approver, AUTHZ-009).
- Void of a paid/partially-paid invoice triggers a governed refund flow.
- Arrears carry-forward at session close is a recorded decision (SESS-007).

## 9. Permission Rules
- `fee.structure.manage` — define/version fee structures.
- `fee.invoice.generate` — generate/issue invoices.
- `fee.invoice.adjust` — issue credit notes/adjustments/voids (elevated; SoD on large amounts).
- `fee.refund.approve` — approve refunds.
- `fee.view` — read invoices/dues (students/guardians own; finance staff scoped).

## 10. Notification Rules
- `INVOICE_ISSUED` → notify guardian (financial-responsible, GRD-N-005) with due date.
- Upcoming/overdue due reminders → guardian per preference (respecting custody restrictions).
- `LATE_FINE_APPLIED`, `INVOICE_ADJUSTED/VOIDED`, `REFUND_ISSUED` → notify the financial-responsible guardian.
- Notifications never expose another family's financial data.

## 11. Audit Requirements
Mandatory: `FEE_STRUCTURE_PUBLISHED` (version/effective date), `INVOICE_ISSUED` (config version, amounts), `INVOICE_ADJUSTED/VOIDED` (reason), `LATE_FINE_APPLIED`, `ARREARS_CARRIED_FORWARD`, `REFUND_APPROVED/ISSUED`. Every figure traceable to its structure version and the actor. Financial audit is non-negotiable.

## 12. Data Retention Rules
- Invoices, adjustments, and structures retained long-term per financial/legal retention (often many years, statutory).
- Fee-structure versions retained so any invoice is reproducible to its source figures.
- Financial records are typically exempt from erasure during their legal retention period (legal-hold overrides erasure).
- Anonymization (post-retention) preserves non-identifying ledger integrity.

## 13. Edge Cases
- **Mid-session fee change:** effective-dated; issued invoices unchanged; only future invoices use the new version (FEE-001).
- **Student withdrawn with paid advance:** governed refund of the unused portion (FEE-008/PAY-007).
- **Transferred student fees:** split by period across sections/campuses (FEE-005/ENR-005).
- **Overpayment:** becomes a credit balance applied to future dues or refunded (FEE-008).
- **Duplicate batch run:** idempotent, no double invoices (FEE-004).
- **Statutory result vs fee withholding:** statutory exams/results may be exempt from fee-based withholding per law (links RES-007).
- **Rounding of amounts:** defined precision per currency; never silent drift.
- **Category change mid-session:** affects future invoices; issued ones stand (use adjustment if correction needed).
- **Sibling discount when one sibling withdraws:** recompute future eligibility (links DSC), not retroactively edit issued invoices.

## 14. Failure Scenarios
- **Direct edit of an issued invoice:** rejected (FEE-003); use credit note.
- **No effective structure:** generation blocked and flagged, never a guessed amount.
- **Duplicate generation:** prevented idempotently (FEE-004).
- **Fine beyond cap:** blocked (FEE-007).
- **Unapproved refund/large adjustment:** rejected (workflow/SoD).
- **Batch partial failure:** idempotent retry; no double-charging.

## 15. Exception Handling Rules
- Issued invoices are immutable; all corrections are linked credit notes/voids with reasons.
- Generation blocks on missing/ambiguous structure rather than guessing.
- Refunds, large adjustments, and arrears decisions are governed and audited.
- Fines and pro-rata are bounded and deterministic.

## 16. Compliance Considerations
- **Financial integrity & audit:** immutable invoices, gapless adjustment trails, and version provenance support statutory financial audit.
- **Consumer fairness:** transparent dues, bounded fines, pro-rata, and governed refunds protect families.
- **Legal retention:** financial records retained per statute; legal-hold overrides erasure.
- **Minors' financial data:** scoped to the financial-responsible guardian; custody restrictions respected.

## 17. Future Considerations
- Multi-currency for international clients (per-deployment locale today).
- Payment-gateway-driven invoicing and auto-reconciliation (Doc 18, R4).
- Installment-plan automation with reminders.
- Sponsor/third-party billing (corporate-sponsored students).
