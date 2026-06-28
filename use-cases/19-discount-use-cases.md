# 19 — Discount Use Cases

Transforms the Discount business rules (`DSC-001`…`DSC-007`) into use cases. Reductions on fees: transparent typed discounts, eligibility rules, SoD-governed approval, caps (never below zero), stacking rules, **waiver semantics (the single source of truth, Conflict C-03)**, and conditional reversal/expiry.

## 1. Primary Actors
Finance Administrator (defines/applies discounts), Approver (authorizes discretionary discounts/waivers).

## 2. Secondary Actors
System (eligibility, caps, stacking, transparent application), Workflow Engine (approval/SoD), Fee module (charge reductions; non-waivable heads), Scholarship module (interaction), Audit service.

## 3. Goals
Apply transparent, typed discounts; automate eligibility-based discounts; govern discretionary discounts and waivers with SoD; never reduce a charge below zero; enforce stacking rules; own waiver semantics consistently (charges marked non-waivable in Fee cannot be waived); reverse/expire discounts conditionally.

## 4. User Journeys
- **Automatic:** an eligibility rule (sibling, merit, staff-ward) auto-applies a transparent discount line on the invoice.
- **Discretionary:** finance proposes a discretionary discount/waiver → SoD approval (requester ≠ approver) → applied transparently within caps.
- **Waiver:** a waiver (a discount sub-type owned here) forgives part/all of a waivable charge; non-waivable heads (set in Fee) are refused.
- **Lifecycle:** conditional discounts reverse if conditions lapse; time-bound discounts expire.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-DSC-001 | Apply Discount (transparent, typed) | High |
| Core | UC-DSC-002 | Auto-Apply Eligibility Discount | High |
| Core | UC-DSC-003 | Apply Waiver (sub-type, non-waivable check) | High |
| Core | UC-DSC-004 | Enforce Caps & Stacking | High |
| Core | UC-DSC-005 | Reverse / Expire Discount (conditional) | Medium |
| CRUD | UC-DSC-006 | View Discounts (scoped) | Medium |
| Admin | UC-DSC-007 | Configure Discount Types & Eligibility Rules | High |
| Approval | UC-DSC-008 | Approve Discretionary Discount / Waiver (SoD) | Critical |
| Search | UC-DSC-009 | Search Discounts | Low |
| Reporting | UC-DSC-010 | Discount / Waiver Utilization Report | Medium |
| Bulk | UC-DSC-011 | Bulk Discount Application | Medium |
| Import/Export | UC-DSC-012 | Import / Export Discount Rules | Low |
| Workflow | UC-DSC-013 | Discount / Waiver Approval Workflow | High |
| Exception | UC-DSC-014 | Below-Zero Charge Blocked | High |
| Exception | UC-DSC-015 | Non-Waivable Head Waiver Blocked | High |
| Exception | UC-DSC-016 | Self-Approval (SoD) Blocked | Critical |

---

## 6. Detailed Specifications (high-value use cases)

### UC-DSC-003 — Apply Waiver (sub-type, non-waivable check)
- **Module:** Discount · **Priority:** High
- **Actors:** Finance Administrator (primary), Approver, System
- **Goal:** Forgive part/all of a waivable charge under governance, refusing non-waivable heads.
- **Description:** A waiver is a discount sub-type owned by this module (Conflict C-03); it checks the charge's waivable classification (set in Fee, FEE-002), enforces caps (never below zero), requires SoD approval for discretionary amounts, and applies transparently.
- **Business Rules Applied:** DSC-006, DSC-003, DSC-004, FEE-002, AUTHZ-009.
- **Preconditions:** A waivable charge; actor authorized; approval where discretionary.
- **Trigger:** Admin proposes a waiver.
- **Main Success Scenario:**
  1. Admin selects the charge and waiver amount/percentage with a reason.
  2. System verifies the charge is waivable (not a non-waivable head, FEE-002).
  3. System enforces caps (resulting charge ≥ 0, DSC-004) and routes SoD approval (DSC-003).
  4. On approval, System applies the waiver as a transparent line; balances recompute.
- **Alternative Flows:** A1) Full waiver (charge to zero, not below). A2) Partial waiver.
- **Exception Flows:** E1) Non-waivable head → blocked (UC-DSC-015). E2) Waiver exceeding the charge → capped at zero (UC-DSC-014). E3) Requester = approver → blocked (UC-DSC-016).
- **Validation Rules:** Charge waivable; result ≥ 0; SoD enforced; transparent (DSC-003/004/006, FEE-002).
- **Permissions Required:** `discount.waiver.apply` + `discount.approve` (SoD).
- **Notifications Triggered:** `WAIVER_APPLIED` to financial-responsible guardian.
- **Audit Events Generated:** `WAIVER_APPLIED` (charge, amount, reason, approver).
- **Data Created:** Waiver record/line.
- **Data Updated:** Invoice/dues balance.
- **Data Deleted:** None.
- **Post Conditions:** Waiver applied transparently within caps; SoD satisfied; auditable.
- **Related Use Cases:** UC-FEE-003, UC-DSC-008, UC-SCH-006.
- **Acceptance Criteria:**
  - Given a waivable charge, When a waiver is approved under SoD, Then it applies transparently and the charge never goes below zero.
  - Given a non-waivable head, When a waiver is attempted, Then it is blocked.
  - Given the requester is the only approver, When approval is needed, Then it is blocked (SoD).
- **Edge Case Analysis:**
  - *Invalid Input:* waiver without reason rejected.
  - *Permission Failure:* self-approval blocked (AUTHZ-009).
  - *Concurrent Update:* waiver + payment racing → balance consistent, never negative.
  - *Duplicate Data:* duplicate waiver on the same charge prevented.
  - *System Failure:* atomic; transparent line recorded.
  - *Workflow Failure:* approval pends; waiver not applied until approved.
- **QA Coverage:**
  - *Positive:* partial/full waiver with approval.
  - *Negative:* non-waivable head; over-charge waiver; self-approval.
  - *Boundary:* waiver to exactly zero; cap edge.

### UC-DSC-004 — Enforce Caps & Stacking
- **Module:** Discount · **Priority:** High
- **Actors:** System (primary), Finance Administrator
- **Goal:** Ensure combined discounts respect caps and stacking rules and never drive a charge below zero.
- **Description:** When multiple discounts/waivers/scholarships apply, the system enforces configured stacking rules and an absolute floor of zero on the net charge.
- **Business Rules Applied:** DSC-004, DSC-005, SCH-006, SCH-004.
- **Preconditions:** One or more reductions applicable; stacking rules configured.
- **Trigger:** A reduction is applied or recomputed.
- **Main Success Scenario:**
  1. System gathers applicable reductions (discounts, waivers, scholarships).
  2. System applies stacking rules (e.g., non-stacking, best-of, additive-capped) (DSC-005, SCH-006).
  3. System enforces the net charge ≥ 0 (DSC-004, SCH-004).
  4. The transparent net is recorded with each reduction line visible.
- **Alternative Flows:** A1) Best-of selection when stacking is disallowed.
- **Exception Flows:** E1) Combined reductions exceeding the charge → clamped to zero, not negative (UC-DSC-014).
- **Validation Rules:** Stacking rules honored; net ≥ 0; each reduction transparent (DSC-004/005, SCH-006).
- **Permissions Required:** System; `discount.manage` for overrides.
- **Notifications Triggered:** None routine.
- **Audit Events Generated:** `DISCOUNT_STACK_RESOLVED` (where material).
- **Data Created:** None (computation).
- **Data Updated:** Net charge / reduction lines.
- **Data Deleted:** None.
- **Post Conditions:** Net charge correct, non-negative, transparent.
- **Related Use Cases:** UC-DSC-001, UC-SCH-006.
- **Acceptance Criteria:**
  - Given non-stacking rules, When multiple discounts apply, Then only the permitted one applies (e.g., best-of).
  - Given additive discounts exceeding the charge, When combined, Then the net is clamped to zero (never negative).
  - Given a scholarship plus a discount, When both apply, Then the configured interaction (SCH-006) governs the result.
- **Edge Case Analysis:**
  - *Invalid Input:* contradictory stacking config rejected at setup.
  - *Permission Failure:* override without permission → 403.
  - *Concurrent Update:* two reductions applied simultaneously → resolved deterministically.
  - *Duplicate Data:* same discount applied twice prevented.
  - *System Failure:* deterministic recompute.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* additive-capped; best-of; scholarship+discount interaction.
  - *Negative:* below-zero attempt; double-apply.
  - *Boundary:* reductions summing to exactly the charge; cap exactly reached.

---

## 7. Compact Specifications (routine use cases)

- **UC-DSC-001 — Apply Discount (transparent, typed)** · *High* · Rules: DSC-001, DSC-004. Apply a typed discount as a transparent line within caps. *Permissions:* `discount.apply`. *Audit:* `DISCOUNT_APPLIED`. *Edge:* transparent; never below zero. *QA:* apply; transparency; cap.
- **UC-DSC-002 — Auto-Apply Eligibility Discount** · *High* · Rules: DSC-002. Auto-apply rule-based discounts (sibling/merit/staff-ward). *Edge:* eligibility re-evaluated; lapses trigger reversal (UC-DSC-005). *QA:* auto-apply; eligibility change.
- **UC-DSC-005 — Reverse / Expire Discount (conditional)** · *Medium* · Rules: DSC-007. Reverse on condition lapse; expire time-bound discounts. *Audit:* `DISCOUNT_REVERSED/EXPIRED`. *Edge:* reversal recomputes dues; transparent. *QA:* condition lapse; expiry; recompute.
- **UC-DSC-006 — View Discounts (scoped)** · *Medium* · Rules: AUTHZ-003. Scoped view; transparent on invoices. *QA:* scope; transparency.
- **UC-DSC-007 — Configure Discount Types & Eligibility Rules** · *High* · Rules: DSC-001/002/005, CFG-004. Configure types, eligibility, stacking, caps (versioned). *Permissions:* `discount.config.manage`. *Edge:* stacking/caps coherent. *QA:* config; coherence.
- **UC-DSC-008 — Approve Discretionary Discount / Waiver (SoD)** · *Critical* · Rules: DSC-003, AUTHZ-009, WFL-004. Approver (≠ requester) approves discretionary reductions. *Edge:* SoD; tiered approval by amount; escalation. *QA:* approval; SoD; tier thresholds.
- **UC-DSC-009 — Search Discounts** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-DSC-010 — Discount / Waiver Utilization Report** · *Medium* · Rules: REP-002. Utilization, value forgone, by type/approver. *Edge:* scoped; supports financial governance. *QA:* totals accuracy.
- **UC-DSC-011 — Bulk Discount Application** · *Medium* · Rules: DSC-002/004. Bulk-apply eligibility discounts to a cohort. *Edge:* per-student caps; partial-failure report. *QA:* clean bulk; cap enforcement.
- **UC-DSC-012 — Import / Export Discount Rules** · *Low* · Rules: CFG-002, REP-005. Import/export rules (validated). *QA:* valid import; export scoped.
- **UC-DSC-013 — Discount / Waiver Approval Workflow** · *High* · Rules: WFL-002/004, DSC-003. Version-pinned, tiered approval. *QA:* pinning; SoD; tier escalation.
- **UC-DSC-014 — Below-Zero Charge Blocked (Exception)** · *High* · Rules: DSC-004. Net charge clamped at zero. *QA:* clamp; never negative.
- **UC-DSC-015 — Non-Waivable Head Waiver Blocked (Exception)** · *High* · Rules: DSC-006, FEE-002. Non-waivable heads cannot be waived. *QA:* blocked; waivable allowed.
- **UC-DSC-016 — Self-Approval (SoD) Blocked (Exception)** · *Critical* · Rules: DSC-003, AUTHZ-009. Requester cannot approve own discount/waiver. *QA:* self-approval blocked; different approver allowed.

## 8. Module-level QA & Edge Themes
- **Waiver single source of truth (DSC-006 / C-03):** waiver semantics live here; Fee classifies non-waivable heads; Result/Enrollment consume outcomes — no duplicate waiver logic.
- **SoD on money (DSC-003 / P9):** discretionary discounts/waivers require requester ≠ approver, tiered by amount — a headline control suite.
- **Never below zero (DSC-004):** caps and stacking always clamp the net charge at zero.
- **Transparency (DSC-001):** every reduction is a visible line, never a hidden adjustment.
