# 17 — Fee Management Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`FEE-001…008`), and Use Case Repository (`UC-FEE-001…018`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Define what students owe and bill it correctly: configurable effective-dated versioned fee structures, fee heads/schedules, immutable issued invoices, duplicate prevention, pro-rata, category differentiation, dues/due-dates/late-fines, and governed arrears/refunds.

**Business Goal.** Produce accurate, immutable, reproducible invoices and dues — corrected only via credit notes — so the financial trail is defensible and auditable end to end.

**Scope.** Fee structure (versioned, effective-dated, category-based); fee heads & schedules (incl. non-waivable classification); invoice generation (idempotent, immutable) + credit-note correction; pro-rata; dues/due-dates/late-fines (transparent, reversible); arrears carry-forward; governed refunds.

**Out of Scope.** Payment recording/receipts (Payment module). Waiver semantics (Discount module owns DSC-006 — this module only classifies non-waivable heads, C-03). Scholarship coverage (Scholarship module). Fee structure value storage mechanics (Configuration Engine). Session rollover (Session module — arrears carry on rollover).

---

## 2. Actors

**Primary Actors.** Finance Administrator (structures, invoices), Accountant (dues, fines, refunds), Approver (refunds/waivers — SoD), System (idempotency, immutability, fine accrual, pro-rata).

**Secondary Actors.** Configuration Engine (fee versions), Payment module (settlement), Discount/Scholarship modules (reductions), Session module (arrears carry-forward), File/Notification (invoices), Workflow Engine (refund/waiver approval), Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Versioned fee structure | Define configurable, effective-dated, versioned fee structures by category. | High | Configuration Engine |
| FR-002 | Fee heads & schedules | Configure heads (tuition/transport/lab), schedules, and non-waivable classification. | High | FR-001 |
| FR-003 | Idempotent invoice generation | Generate invoices from the effective structure; no duplicates. | Critical | FR-001 |
| FR-004 | Issued-invoice immutability | Issued invoices are immutable; corrections via credit note only. | Critical | FR-003 (P5) |
| FR-005 | Pro-rata | Pro-rate charges for mid-period join/leave. | High | Enrollment |
| FR-006 | Dues, due dates & late fines | Track dues; apply transparent, reversible late fines after due dates. | High | Payment |
| FR-007 | Arrears carry-forward | Carry outstanding dues to next session at rollover. | Medium | Session |
| FR-008 | Governed refunds | Process refunds under approval (reason, never silent). | High | Payment, Workflow |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Versioned fee structures | Effective-dated, category-based. | Reproducible billing. |
| Non-waivable classification | Heads flagged non-waivable. | Statutory integrity (feeds C-03). |
| Idempotent invoices | No duplicate billing. | Clean financials. |
| Immutable invoices + credit notes | Correct without editing. | Auditable trail. |
| Pro-rata | Fair mid-period charges. | Accurate billing. |
| Transparent fines | Reversible line items. | Fairness; no silent charges. |
| Arrears carry-forward | No lost dues at rollover. | Continuity. |
| Governed refunds | Approved, reasoned. | Financial control. |

---

## 5. Screens

Fee Structure List; Fee Structure Definition (versioned); Fee Heads/Schedules/Categories; Invoice List; Invoice Detail; Generate Invoice; Bulk Invoice Generation; Credit Note (correction); Pro-Rata; Dues & Aging; Late-Fine Management; Refund (governed); Arrears Carry-Forward; Dues/Collection/Aging Report; Fee Import/Export; Refund/Waiver Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Fee Structure | Define, Edit, Publish Version, View History | — |
| Heads/Schedules | Add/Edit Head, Flag Non-Waivable, Set Schedule, Save | — |
| Invoice List | Generate, Open, Search/Filter, Export | Bulk Generate |
| Invoice Detail | View, Issue Credit Note, Download (signed URL) | — |
| Credit Note | Create (reason), Submit (→ approval if required) | — |
| Dues & Aging | View, Apply Fine, Waive Fine (→ approval) | Bulk Reminder |
| Refund | Process (reason), Submit (→ approval) | — |
| Aging Report | Run, Filter, Export | Export |

---

## 7. Forms

**Fee Structure** — `category` (select), heads with `amount`, `schedule`, `effectiveDate`, `version`. Validation: typed/versioned (CFG-002/004); effective forward; historical invoices keep version (FEE-001); category differentiation (FEE-006).

**Fee Head** — `name` (text, required), `type` (tuition/transport/lab/…), `nonWaivable` (toggle). Validation: non-waivable heads feed Discount's waiver check (FEE-002 / C-03).

**Generate Invoice** — `student`/`cohort`, `period`. Validation: idempotency key prevents duplicate (FEE-004); resolves effective structure; immutable on issue (FEE-003); stamps version (P6); pro-rata for mid-period (FEE-005).

**Credit Note** — `invoice` (required), `amount`, `reason` (text, required). Validation: original invoice preserved (FEE-003); linked; approved where required (FEE-008/P5).

**Refund** — `amount`, `reason` (text, required). Validation: ≤ paid; SoD approval (FEE-008/AUTHZ-009); reversal-not-delete (PAY-007).

**Late Fine** — `policy` (amount, grace). Validation: transparent line item, reversible via waiver (FEE-007).

---

## 8. Search & Filter Requirements

**Invoices/Dues:** by student, invoice number, period, status (issued/paid/partial/overdue/credited), category, due-date range, aging bucket. Sorting: due date/amount/status. Pagination: server-side, 25 default. Scope-bound; financial-responsible guardian sees own child's.

---

## 9. Table Requirements

**Invoice table:** Invoice No., Student, Period, Amount, Paid, Due, Status, Due Date. **Dues/Aging:** Student, Outstanding, Aging bucket, Fine. Sorting on Due Date/Amount. Filtering as above. Export (governed — financial). Bulk: generate, reminder.

---

## 10. Workflow Requirements

**Trigger events:** structure publish, invoice generate/issue, credit note, fine accrual/waiver, refund, arrears carry-forward. **Status changes:** invoice `ISSUED → PAID/PARTIAL/OVERDUE → CREDITED`. **Approvals:** refunds and fine waivers via Workflow Engine (SoD). **Notifications:** invoice issued, due reminder, late fine, credit note, refund (to financial-responsible guardian). **Audit:** invoices, credit notes, fines, refunds (before/after, immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Configure fee structure/heads | `fee.config.manage` |
| Issue invoice | `fee.invoice.issue` |
| Issue credit note | `fee.creditnote.issue` |
| Manage fines | `fee.fine.manage` |
| Process refund | `fee.refund.process` |
| Approve refund/waiver | `fee.refund.approve` (SoD) |
| View fee/invoice | `fee.view` (scoped) |
| Import/export | `fee.import`, `fee.export` |

Financial views are scoped to the financial-responsible guardian (GRD-N-005); refunds/waivers enforce SoD.

---

## 12. Business Rule References

FEE-001 (versioned, effective-dated structure), FEE-002 (fee heads & schedules; non-waivable classification), FEE-003 (issued invoices immutable), FEE-004 (duplicate-invoice prevention), FEE-005 (pro-rata), FEE-006 (category-based differentiation), FEE-007 (dues, due dates & late fines), FEE-008 (arrears carry-forward & governed refunds). Cross-cutting: CFG-002/004 (typed/versioned), DSC-006 (waiver semantics — C-03 boundary), PAY-007 (reversal-not-delete), SESS-006 (arrears at rollover), FILE-007 (immutable invoice doc), WFL-002/004, AUTHZ-009, AUD-001, GRD-N-005 (financial-responsible).

## 13. Use Case References

UC-FEE-001 (Define Structure), UC-FEE-002 (Generate/Issue Invoice — idempotent, immutable), UC-FEE-003 (Credit Note), UC-FEE-004 (Pro-Rata), UC-FEE-005 (Dues/Due-Dates/Fines), UC-FEE-006 (Carry Forward Arrears), UC-FEE-007 (Process Refund), UC-FEE-008 (View — scoped), UC-FEE-009 (Configure Heads/Schedules/Categories), UC-FEE-010 (Approve Refund/Waiver — SoD), UC-FEE-011 (Search), UC-FEE-012 (Dues/Collection/Aging Report), UC-FEE-013 (Bulk Invoice Generation), UC-FEE-014 (Import Structure/Opening Balances), UC-FEE-015 (Export — governed), UC-FEE-016 (Refund/Waiver Workflow), UC-FEE-017 (Duplicate Invoice Blocked), UC-FEE-018 (Edit Issued Invoice Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Define / version fee structure | POST/PUT | Finance Admin |
| Configure heads/schedules/categories | POST/PUT | Finance Admin |
| Generate / issue invoice (single/bulk) | POST | Finance Admin |
| Get invoice (signed URL doc) | GET | Authorized / Guardian |
| Issue credit note | POST | Finance Admin (→ approval) |
| Accrue / waive late fine | POST | Accountant (→ approval) |
| Process refund | POST | Finance Admin (→ approval) |
| Carry forward arrears | POST | System / Finance |
| Dues / collection / aging report | GET | Finance Admin |
| Import / export fee data | POST/GET | Finance Admin |

Invoice generation is idempotent and transactional (FEE-004); issued invoices are immutable — corrections via credit note (FEE-003).

---

## 15. Database Requirements

**Entities:** `FeeStructure` (category, version, effectiveDate), `FeeHead` (name, type, nonWaivable), `Invoice` (number, studentId, period, lineItems, amount, status, structureVersion, immutable), `CreditNote` (invoiceId, amount, reason, approver), `Dues` (balance, dueDate), `LateFine` (line item, reversible), `Refund` (amount, reason, approver). **Relationships:** Invoice 1—* LineItem; Invoice 1—* CreditNote; Student 1—* Invoice. **Indexes:** unique(Invoice.number), unique(Invoice.studentId, period, run) for idempotency (FEE-004), index(Invoice.status, dueDate), index(Dues.studentId). Issued invoices write-locked (FEE-003); structure version stamped (P6).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Invoice issued, due reminder, late fine applied, credit note, refund processed — to financial-responsible guardian. |
| SMS | Due/overdue alerts (minimized). |
| Push | Invoice/payment reminders. |
| In-App | Dues status; refund/waiver approvals. |

Financial notices route to the financial-responsible guardian (GRD-N-005); mandatory-category (NOT-003).

---

## 17. Audit Requirements

Log: structure publish (version), invoice issue (immutable, version), credit notes (reason, approver), fine accrual/waiver, refunds (reason, approver), arrears carry-forward. Record who/when/before/after. Invoices/credit notes/refunds are first-class financial audit events. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Dues aging, Collection, Defaulters, Fee-structure drift, Refund/credit-note register. **Exports:** governed financial export (audited). **Dashboards:** finance overview (billed, collected, outstanding, overdue).

---

## 19. Error Handling

**Validation:** duplicate invoice, edit issued invoice, over-refund → specific errors (UC-FEE-017/018). **Permission:** refund/waiver without authority/self-approval → blocked. **Workflow:** refund/waiver pending → not applied until approved. **System:** Config unavailable → invoice generation blocked; closed period correction → governed reopen first (SESS-006).

---

## 20. Edge Cases

**Concurrent updates:** two bulk runs overlapping → idempotency yields one invoice per student/period. **Duplicate data:** duplicate invoice/credit note prevented. **Partial failures:** bulk generation partial → resumable, per-row report. **Rollback:** refund reversed before disbursement → dues restored. **Fine race:** payment at fine-accrual moment → payment-before-fine ordering, consistent net.

---

## 21. Acceptance Criteria

**Functional.** Fee structures are versioned and effective-dated; invoices generate idempotently (no duplicates) from the stamped structure and are immutable once issued; corrections flow through linked credit notes; pro-rata applies for mid-period join/leave; late fines are transparent and reversible; arrears carry forward at rollover; refunds are governed (approved, reasoned), never silent; non-waivable heads are classified here for Discount to honor.

**Business.** Billing is accurate, immutable, and reproducible; the financial trail is defensible end to end; statutory (non-waivable) charges are protected; corrections never destroy the original record.

---

## 22. Future Enhancements

Installment/EMI plans; auto fee-structure templates per category; gateway-native invoice presentment; multi-currency; predictive collection/defaulter scoring; configurable fine-escalation ladders; scholarship/discount-aware net-fee preview.
