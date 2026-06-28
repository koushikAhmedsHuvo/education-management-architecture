# 19 — Discount Business Rules

## 1. Module Purpose
Govern **discounts** — reductions applied to a student's fees. This module deliberately distinguishes three reduction concepts the rest of the catalog references:
- **Discount (this module):** a reduction of charges, rule-based (sibling, early-payment) or case-by-case (approved concession), applied to invoices.
- **Waiver:** forgiving a *specific* charge/due, usually case-by-case and approved (modeled here as a discount sub-type `WAIVER` for shared mechanics).
- **Scholarship (Doc 20):** a structured, often *funded*, criteria-and-renewal-based award.
Discounts reduce real money, so they require **defined eligibility, approval authority, separation of duties, caps, and stacking rules**, and they never silently edit issued invoices (they apply as transparent reduction lines or pre-issue adjustments).

## 2. Actors
- **Accountant / Finance Administrator** — requests/applies discounts and waivers.
- **Approver (Finance/Institute authority)** — approves discounts above authority limits (SoD).
- **Student / Guardian** — may be auto-eligible (sibling, early payment); view applied reductions (own).
- **System** — evaluates eligibility, enforces caps/stacking, applies reductions transparently, audits financial impact.

## 3. Use Cases

**Use Case ID:** UC-DSC-01 — Apply a Rule-Based Discount
**Actors:** System / Accountant
**Description:** Automatically apply an eligibility-rule discount (e.g., sibling, early-payment).
**Preconditions:** Discount rule configured; student meets eligibility.
**Main Flow:** 1) System evaluates eligibility at invoice generation (or payment for early-payment). 2) Applies the discount as a transparent reduction line within caps. 3) Net due reflects the reduction.
**Alternative Flow:** A1) Early-payment discount applied conditionally on payment before a date (reversed if condition fails).
**Exception Flow:** E1) Discount would exceed the charge → capped (DSC-004). E2) Stacking beyond policy → blocked (DSC-005).
**Post Conditions:** Discount applied transparently; audited.
**Business Rules Applied:** DSC-001, DSC-002, DSC-004, DSC-005.

**Use Case ID:** UC-DSC-02 — Approve a Case-by-Case Discount / Waiver
**Actors:** Accountant (request), Approver (decision), System
**Description:** A concession/waiver is requested and approved via workflow before applying.
**Preconditions:** Requester holds `discount.request`; amount exceeds auto-apply limits.
**Main Flow:** 1) Accountant requests a discount/waiver with reason and amount. 2) Workflow routes to an approver within authority limits (SoD: requester ≠ approver). 3) On approval, the reduction applies transparently; on rejection, it does not.
**Exception Flow:** E1) Amount beyond approver's authority → escalate to a higher authority. E2) Requester self-approval attempt → blocked (SoD).
**Post Conditions:** Approved reduction applied with provenance; audited.
**Business Rules Applied:** DSC-003, DSC-006.

**Use Case ID:** UC-DSC-03 — Reverse / Expire a Discount
**Actors:** Finance Administrator, System
**Description:** Reverse a discount whose condition failed or which was applied in error.
**Preconditions:** Discount applied; reversal condition met or error identified.
**Main Flow:** 1) System/admin reverses the discount (e.g., early-payment condition failed). 2) The reduction is removed via a transparent reversing entry; net due restored. 3) Guardian notified if material.
**Exception Flow:** E1) Reversal on a paid invoice → credit/refund handling (links FEE-008/PAY-007).
**Post Conditions:** Discount reversed transparently; audited.
**Business Rules Applied:** DSC-007.

## 4. Business Rules

**Rule ID:** DSC-001
**Rule Name:** Discount Types & Transparent Application
**Description:** Discounts are typed (percentage/fixed, per-head/overall) and always applied as visible reduction lines, never silent invoice edits.
**Priority:** Critical
**Category:** Integrity / transparency
**Preconditions:** Discount application.
**Business Rule:** A discount is percentage or fixed, scoped to a head or the overall invoice, and appears as a distinct reduction with its basis; the gross charge remains visible alongside the reduction. Issued invoices are reduced via transparent reduction/credit lines (FEE-003), not by altering originals.
**System Action:** Apply as a distinct reduction line with type/basis; preserve gross.
**Validation:** Type/scope valid; basis recorded.
**Failure Behavior:** Reject silent reductions; require transparent lines.
**Audit Requirement:** Log `DISCOUNT_APPLIED` with type, amount, basis.
**Example Scenario:** A 10% sibling discount shows as a clear reduction line, not a quietly lowered tuition figure.
**Related Rules:** FEE-003, DSC-004.

**Rule ID:** DSC-002
**Rule Name:** Eligibility-Rule Discounts
**Description:** Rule-based discounts (sibling, early-payment, merit, staff-ward where not structural) apply by defined eligibility.
**Priority:** High
**Category:** Configuration / fairness
**Preconditions:** Discount rules configured.
**Business Rule:** Eligibility criteria are defined and evaluated consistently (e.g., 2+ enrolled siblings → sibling discount). Rule discounts auto-apply within caps; conditional ones (early-payment) apply provisionally and reverse if the condition fails.
**System Action:** Evaluate eligibility at the defined point; apply within caps.
**Validation:** Eligibility criteria defined; current data evaluated.
**Failure Behavior:** No discount if ineligible; reverse conditional on failure.
**Audit Requirement:** Eligibility basis recorded.
**Example Scenario:** A family with three enrolled children receives the configured sibling discount automatically.
**Related Rules:** GRD-N-003 (siblings), DSC-007.

**Rule ID:** DSC-003
**Rule Name:** Approval Authority & Separation of Duties
**Description:** Case-by-case discounts/waivers require approval within authority limits; requester cannot approve their own.
**Priority:** Critical
**Category:** Financial control
**Preconditions:** A non-rule (concession/waiver) discount.
**Business Rule:** Discretionary reductions require approval; approval authority is tiered by amount; the requester ≠ approver (SoD, AUTHZ-009); amounts beyond an approver's limit escalate. Self-approval is impossible.
**System Action:** Route via workflow to an authorized approver; enforce SoD and limits.
**Validation:** Approver authority ≥ amount; requester ≠ approver.
**Failure Behavior:** Block self-approval/over-limit approval; escalate.
**Audit Requirement:** Log `DISCOUNT_REQUESTED/APPROVED/REJECTED` with actors and amounts.
**Example Scenario:** A large hardship concession needs the finance head's approval, not the requesting accountant's.
**Related Rules:** AUTHZ-009, WFL (Doc 28), DSC-006.

**Rule ID:** DSC-004
**Rule Name:** Discount Caps (Never Below Zero)
**Description:** A discount cannot exceed the charge it reduces; net charge floors at zero.
**Priority:** Critical
**Category:** Integrity
**Preconditions:** Discount application.
**Business Rule:** A discount is capped at the applicable charge; it never produces a negative charge or an implicit payout (any beyond-fee benefit is a scholarship stipend, Doc 20, not a discount).
**System Action:** Cap the reduction at the charge; floor net at zero.
**Validation:** Discount ≤ charge.
**Failure Behavior:** Cap excess; never go negative.
**Audit Requirement:** Capped reductions noted.
**Example Scenario:** A fixed discount larger than a small fee reduces it to zero, not below.
**Related Rules:** DSC-001, SCH (stipend distinction).

**Rule ID:** DSC-005
**Rule Name:** Stacking Rules
**Description:** Combining multiple discounts follows explicit stacking policy and an overall cap.
**Priority:** High
**Category:** Fairness / control
**Preconditions:** A student qualifies for multiple reductions.
**Business Rule:** Whether discounts stack (and in what order — additive vs sequential percentage) and the maximum combined reduction are explicitly configured; total reduction across discounts/waivers (and scholarships per policy) cannot exceed the configured ceiling.
**System Action:** Apply per the stacking order; enforce the combined cap.
**Validation:** Stacking policy + combined cap defined.
**Failure Behavior:** Block stacking beyond policy/cap.
**Audit Requirement:** Stacking computation recorded.
**Example Scenario:** Sibling + early-payment discounts combine only up to a configured 25% ceiling.
**Related Rules:** DSC-004, SCH-006 (scholarship interaction).

**Rule ID:** DSC-006
**Rule Name:** Waiver Sub-Type Semantics
**Description:** A waiver forgives a specific charge/due, governed like a discretionary discount, with clear scope.
**Priority:** High
**Category:** Clarity / control
**Preconditions:** A specific charge is to be forgiven.
**Business Rule:** A waiver targets a specific charge/invoice (often hardship), requires approval (DSC-003), and is recorded distinctly so reporting separates "discount" (often rule-based) from "waiver" (discretionary forgiveness). Mandatory/statutory charges may be non-waivable per policy.
**System Action:** Apply as a targeted, approved reduction with the `WAIVER` classification.
**Validation:** Charge waivable per policy; approval obtained.
**Failure Behavior:** Block waiver of non-waivable charges.
**Audit Requirement:** Log `WAIVER_APPROVED` with target and reason.
**Example Scenario:** A hardship case has the term's lab fee waived with finance-head approval.
**Related Rules:** DSC-003, FEE-002 (statutory heads).

**Rule ID:** DSC-007
**Rule Name:** Conditional Reversal & Expiry
**Description:** Conditional/time-bound discounts reverse when conditions fail or expire, transparently.
**Priority:** Medium
**Category:** Integrity
**Preconditions:** A conditional/expiring discount applied.
**Business Rule:** If the condition fails (e.g., early payment not made by the date) or a time-bound discount lapses, the reduction reverses via a transparent reversing entry and the due is restored; reversal on a paid invoice triggers credit/refund handling.
**System Action:** Monitor conditions; reverse transparently; restore dues.
**Validation:** Condition/expiry defined and evaluated.
**Failure Behavior:** No silent persistence of an invalid discount.
**Audit Requirement:** Log `DISCOUNT_REVERSED/EXPIRED` with reason.
**Example Scenario:** An early-payment discount reverses when the guardian misses the early-payment date.
**Related Rules:** FEE-008, PAY-007.

## 5. Validation Rules
- Discounts applied as transparent lines; gross preserved; issued invoices never silently edited.
- Eligibility-rule discounts evaluated on current data; conditional ones reversible.
- Discretionary discounts/waivers approved within authority; requester ≠ approver.
- Discount ≤ charge (cap; net floors at zero).
- Stacking and combined caps explicit and enforced.
- Reversals/expiries transparent and audited.

## 6. State Machine

**State Name:** ELIGIBLE / REQUESTED
**Description:** Auto-eligible (rule) or requested (discretionary), pending application/approval.
**Allowed Transitions:** → APPLIED (rule/approved); → REJECTED (discretionary); → EXPIRED.
**Forbidden Transitions:** apply discretionary without approval.
**System Actions:** Evaluate eligibility / route approval.

**State Name:** APPROVED
**Description:** Discretionary discount/waiver approved, awaiting/at application.
**Allowed Transitions:** → APPLIED.
**Forbidden Transitions:** apply beyond approved amount.
**System Actions:** Lock approved amount/scope.

**State Name:** APPLIED
**Description:** Reduction reflected on the invoice/due transparently.
**Allowed Transitions:** → REVERSED (condition fails/error); → EXPIRED; → ARCHIVED.
**Forbidden Transitions:** silent removal.
**System Actions:** Maintain reduction line; monitor conditions.

**State Name:** REVERSED / EXPIRED
**Description:** Discount removed transparently; due restored.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** silent reapplication.
**System Actions:** Restore due; credit/refund if paid.

**State Name:** REJECTED / ARCHIVED
**Description:** Not granted, or historical.
**Allowed Transitions:** ARCHIVED terminal.
**Forbidden Transitions:** edits.
**System Actions:** Retain decision record.

## 7. Status Definitions
`ELIGIBLE` · `REQUESTED` · `APPROVED` · `APPLIED` · `REVERSED` · `EXPIRED` · `REJECTED` · `ARCHIVED`. Classification: `DISCOUNT` (rule/discretionary) · `WAIVER` (targeted forgiveness).

## 8. Workflow Rules
- Discretionary discounts/waivers run an approval workflow with tiered authority and SoD (DSC-003).
- Over-limit requests escalate to higher authority (Workflow Engine, Doc 28).
- Conditional reversals run automatically on condition evaluation (DSC-007).
- Bulk discount campaigns (e.g., early-payment drive) are configured and audited.

## 9. Permission Rules
- `discount.rule.manage` — configure eligibility-rule discounts and stacking/caps.
- `discount.request` — request discretionary discounts/waivers.
- `discount.approve` — approve within authority (SoD; tiered).
- `discount.reverse` — reverse/expire discounts (elevated).
- `discount.view` — read applied reductions (guardians own; finance scoped).

## 10. Notification Rules
- `DISCOUNT_APPLIED`/`WAIVER_APPROVED` → notify the financial-responsible guardian (reduced due).
- `DISCOUNT_REVERSED/EXPIRED` → notify guardian (restored due).
- Approval requests/decisions → notify requester and approver.
- Never expose another family's concession data.

## 11. Audit Requirements
Mandatory: `DISCOUNT_REQUESTED/APPROVED/REJECTED`, `DISCOUNT_APPLIED` (type/amount/basis), `WAIVER_APPROVED` (target/reason), `DISCOUNT_REVERSED/EXPIRED`, stacking computations, authority/SoD checks. Discounts move real money — every grant and reversal is attributable with approver and basis.

## 12. Data Retention Rules
- Discount/waiver records retained with the financial records they affect (statutory retention).
- Approval provenance retained (who authorized what concession, when) for financial audit.
- Reporting separates discounts vs waivers vs scholarships for transparency.
- Anonymization post-retention preserves non-identifying financial integrity.

## 13. Edge Cases
- **Discount exceeding fee:** capped; never negative or an implicit payout (DSC-004).
- **Stacking beyond cap:** blocked at the combined ceiling (DSC-005).
- **Early-payment condition fails:** discount reverses, due restored (DSC-007).
- **Sibling discount when one sibling withdraws:** future eligibility recomputed; issued invoices corrected via transparent adjustment, not retroactive edit.
- **Discount on an already-paid invoice:** becomes a credit/refund (FEE-008/PAY-007).
- **Waiver of a statutory charge:** blocked if policy marks it non-waivable (DSC-006).
- **Self-approval attempt:** blocked by SoD (DSC-003).
- **Retroactive discount:** applied via transparent credit, never by editing past invoices.
- **Discount + scholarship overlap:** governed by combined caps and the interaction policy (SCH-006).

## 14. Failure Scenarios
- **Silent invoice reduction:** rejected; must be a transparent line (DSC-001).
- **Over-authority/self-approval:** blocked/escalated (DSC-003).
- **Negative net charge:** prevented by cap (DSC-004).
- **Stacking over ceiling:** blocked (DSC-005).
- **Persisting a failed conditional discount:** prevented by reversal (DSC-007).

## 15. Exception Handling Rules
- Reductions are transparent, capped, and audited; issued invoices are never silently altered.
- Discretionary reductions require approval within authority and SoD; self-approval impossible.
- Conditional discounts reverse cleanly on failure with due restoration.
- Non-waivable charges are protected.

## 16. Compliance Considerations
- **Financial control:** approval authority, SoD, caps, and audit prevent concession abuse and fraud.
- **Transparency:** visible reduction lines and discount/waiver separation support audits and family trust.
- **Fairness:** consistent eligibility rules avoid arbitrary concessions.
- **Retention:** concession records retained per financial statute.

## 17. Future Considerations
- Automated needs assessment for hardship waivers.
- Configurable concession budgets per institute (track total concessions vs a cap).
- Self-service early-payment discount campaigns.
- Analytics on concession impact (consent-bound, internal finance).
