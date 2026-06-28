# 19 — Discount Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`DSC-001…007`), and Use Case Repository (`UC-DSC-001…016`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Reduce fees transparently and under control: typed discounts, eligibility-rule automation, SoD-governed discretionary approval, caps (never below zero), stacking rules, **waiver semantics (the single source of truth — C-03)**, and conditional reversal/expiry.

**Business Goal.** Apply fee reductions that are always transparent, capped, governed, and consistent — waivers honor non-waivable head classification, and discretionary reductions require separation of duties.

**Scope.** Discount types & transparent application; eligibility-rule (auto) discounts; discretionary discount/waiver approval (SoD); caps (never below zero); stacking rules; **waiver sub-type semantics (owned here)**; conditional reversal & expiry. Non-waivable heads are classified in Fee (FEE-002) and honored here.

**Out of Scope.** Fee structure/non-waivable classification (Fee module — Discount consumes it). Scholarship awards (Scholarship module — distinct; interaction via SCH-006). Payment (Payment module). Config versioning (Configuration Engine).

---

## 2. Actors

**Primary Actors.** Finance Administrator (defines/applies discounts), Approver (discretionary discounts/waivers — SoD), System (eligibility evaluation, caps, stacking, transparent application).

**Secondary Actors.** Fee module (charge reductions; non-waivable heads), Scholarship module (interaction), Workflow Engine (approval/SoD), Audit, Reporting, Notification.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Typed transparent discounts | Apply typed discounts as transparent line items. | High | Fee |
| FR-002 | Eligibility-rule discounts | Auto-apply rule-based discounts (sibling/merit/staff-ward). | High | Configuration Engine |
| FR-003 | Waiver semantics (non-waivable check) | Apply waivers (a discount sub-type) honoring non-waivable heads. | High | Fee (FEE-002), C-03 |
| FR-004 | Caps (never below zero) | Net charge never goes below zero. | High | FR-001 |
| FR-005 | Stacking rules | Enforce stacking (non-stacking/best-of/additive-capped). | High | Scholarship (SCH-006) |
| FR-006 | Discretionary approval (SoD) | Discretionary discounts/waivers require requester ≠ approver. | Critical | Workflow, AUTHZ-009 |
| FR-007 | Conditional reversal & expiry | Reverse on condition lapse; expire time-bound discounts. | Medium | Fee |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Transparent discounts | Visible line items. | No hidden adjustments. |
| Eligibility automation | Rule-based auto-apply. | Efficiency; consistency. |
| Waiver (single source) | Owned semantics; non-waivable check. | C-03 integrity. |
| Caps | Never below zero. | Financial integrity. |
| Stacking rules | Controlled combination. | Predictable net. |
| SoD approval | Requester ≠ approver. | Fraud prevention. |
| Reversal/expiry | Conditional lifecycle. | Accurate, current reductions. |

---

## 5. Screens

Discount List; Apply Discount; Auto-Eligibility Discounts; Apply Waiver (non-waivable check); Caps & Stacking; Reverse/Expire Discount; Discount Type & Eligibility Configuration; Discount/Waiver Utilization Report; Bulk Discount Application; Import/Export Discount Rules; Discount/Waiver Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Discount List | Apply, Open, Search/Filter, Export | Bulk Apply |
| Apply Discount | Select Type/Amount, Validate Cap, Submit | — |
| Apply Waiver | Select Charge, Check Non-Waivable, Amount, Submit (→ approval) | — |
| Caps & Stacking | View Net, Resolve Stack | — |
| Reverse/Expire | Reverse (condition), Set Expiry | — |
| Configuration | Set Types/Eligibility/Stacking/Caps, Save (versioned) | — |
| Utilization Report | Run, Filter, Export | Export |

---

## 7. Forms

**Apply Discount** — `type` (select), `amount/percentage`, `student/charge`. Validation: transparent line (DSC-001); within caps, net ≥ 0 (DSC-004); stacking honored (DSC-005).

**Apply Waiver** — `charge` (required), `amount/percentage`, `reason` (text, required). Validation: charge is waivable (not a non-waivable head — FEE-002/DSC-006/C-03); net ≥ 0 (DSC-004); SoD approval (DSC-003/AUTHZ-009).

**Configuration** — `discountTypes`, `eligibilityRules`, `stackingPolicy`, `caps`, `approvalTiers`. Validation: typed/versioned (CFG-002/004); coherent stacking/caps.

**Discretionary Approval** — routed by amount tier. Validation: requester ≠ approver (DSC-003 / UC-DSC-016); tiered authority.

**Reverse/Expire** — `condition`/`expiryDate`. Validation: conditional reversal recomputes dues, transparent (DSC-007).

---

## 8. Search & Filter Requirements

**Discounts:** by student, type, status (active/reversed/expired), approver, eligibility rule, waiver vs discount. Sorting: date/amount/type. Pagination: server-side, 25 default. Scope-bound.

---

## 9. Table Requirements

**Discount table:** Student, Type, Amount, Waiver?, Status, Applied By, Approver, Date. Sorting on Date/Amount. Filtering as above. Export (governed). Bulk: apply.

---

## 10. Workflow Requirements

**Trigger events:** apply discount/waiver, eligibility auto-apply, discretionary approval, reversal/expiry. **Status changes:** discount `PROPOSED → APPROVED → ACTIVE → REVERSED/EXPIRED`. **Approvals:** discretionary discounts/waivers via Workflow Engine (SoD, tiered). **Notifications:** discount/waiver applied (to financial-responsible guardian). **Audit:** applications, approvals, reversals/expiry (before/after, immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Apply discount | `discount.apply` |
| Apply waiver | `discount.waiver.apply` |
| Configure types/eligibility | `discount.config.manage` |
| Approve discretionary discount/waiver | `discount.approve` (SoD) |
| Reverse/expire | `discount.reverse` |
| View discounts | `discount.view` (scoped) |
| Import/export | `discount.import`, `discount.export` |

Discretionary reductions enforce SoD (DSC-003); waivers honor non-waivable heads (C-03).

---

## 12. Business Rule References

DSC-001 (types & transparent application), DSC-002 (eligibility-rule discounts), DSC-003 (approval authority & SoD), DSC-004 (caps, never below zero), DSC-005 (stacking rules), DSC-006 (waiver sub-type semantics — owns waivers, C-03), DSC-007 (conditional reversal & expiry). Cross-cutting: FEE-002 (non-waivable heads), SCH-004/006 (scholarship interaction/no-negative-fee), WFL-002/004, AUTHZ-009 (SoD), GRD-N-005, AUD-001, CFG-004.

## 13. Use Case References

UC-DSC-001 (Apply Discount), UC-DSC-002 (Auto-Apply Eligibility), UC-DSC-003 (Apply Waiver — non-waivable check), UC-DSC-004 (Enforce Caps & Stacking), UC-DSC-005 (Reverse/Expire), UC-DSC-006 (View — scoped), UC-DSC-007 (Configure Types/Eligibility), UC-DSC-008 (Approve Discretionary — SoD), UC-DSC-009 (Search), UC-DSC-010 (Utilization Report), UC-DSC-011 (Bulk Application), UC-DSC-012 (Import/Export Rules), UC-DSC-013 (Approval Workflow), UC-DSC-014 (Below-Zero Charge Blocked), UC-DSC-015 (Non-Waivable Head Waiver Blocked), UC-DSC-016 (Self-Approval Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Apply discount | POST | Finance Admin |
| Apply waiver (non-waivable check) | POST | Finance Admin (→ approval) |
| Auto-apply eligibility discounts | POST | System / Finance |
| Resolve caps & stacking (net) | GET | System |
| Approve discretionary discount/waiver | POST | Approver |
| Reverse / expire discount | POST | Finance Admin |
| Configure types/eligibility/stacking/caps | GET/PUT | Finance Admin |
| Utilization report | GET | Finance Admin |
| Bulk apply / import / export | POST/GET | Finance Admin |

Waiver applies only to waivable heads (C-03); net charge is clamped at zero (DSC-004); discretionary requires SoD (DSC-003).

---

## 15. Database Requirements

**Entities:** `Discount` (student, type, amount, status, isWaiver, approver), `DiscountType`/`EligibilityRule` (via Config), `StackingPolicy`, `WaiverRecord` (charge, non-waivable-checked). **Relationships:** Discount *—1 Student; Discount *—1 Charge/Invoice. **Indexes:** index(Discount.studentId, status), index(Discount.type), index(WaiverRecord.chargeId). Net-charge ≥ 0 enforced (DSC-004); waiver checks FeeHead.nonWaivable (C-03).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Discount/waiver applied; reversal/expiry — to financial-responsible guardian. |
| In-App | Discretionary approvals; eligibility changes. |
| SMS/Push | Optional discount confirmation. |

Routed to financial-responsible guardian (GRD-N-005).

---

## 17. Audit Requirements

Log: discount/waiver applications (type, amount, transparent line), eligibility auto-applications, approvals (approver, tier), reversals/expiry, config changes. Record who/when/before/after. Waivers and discretionary discounts are first-class audit events. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Discount/waiver utilization, Value forgone (by type/approver), Eligibility coverage, Reversal/expiry log. **Exports:** governed export. **Dashboards:** discount governance (value forgone, pending approvals, stacking exceptions).

---

## 19. Error Handling

**Validation:** below-zero charge, non-waivable head waiver, self-approval → specific errors (UC-DSC-014/015/016). **Permission:** discretionary without authority/self-approval → blocked. **Workflow:** approval pending → not applied. **System:** Fee non-waivable classification unavailable → block waiver (fail safe).

---

## 20. Edge Cases

**Concurrent updates:** waiver + payment racing → net consistent, never negative. **Duplicate data:** duplicate discount on same charge prevented. **Partial failures:** bulk apply partial → per-student report; per-student caps enforced. **Rollback:** discount reversed → dues recompute transparently. **Stacking race:** two reductions applied simultaneously → resolved deterministically, net ≥ 0.

---

## 21. Acceptance Criteria

**Functional.** Discounts apply as transparent line items within caps (net never below zero); eligibility discounts auto-apply and reverse on lapse; waivers apply only to waivable heads (non-waivable blocked) and require SoD approval; stacking rules govern combined reductions; discretionary discounts/waivers cannot be self-approved; conditional/time-bound discounts reverse/expire correctly.

**Business.** Fee reductions are always transparent, capped, and governed; waiver semantics are consistent (single source of truth honoring non-waivable classification); discretionary reductions are fraud-resistant via SoD.

---

## 22. Future Enhancements

Discount-campaign management; rule-builder UI for eligibility; approval-tier analytics; promo-code discounts; auto-expiry reminders; net-fee simulator combining discounts/scholarships; discount-budget caps per institute.
