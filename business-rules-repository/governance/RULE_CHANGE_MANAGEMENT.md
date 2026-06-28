# Rule Change Management Strategy

The controlled process for proposing, reviewing, approving, and releasing changes to the Business Rules Catalog. A business rule is a contract between product, development, and QA; changing one can ripple across modules and alter records. This process keeps changes deliberate, traceable, and safe.

## 1. When this process applies

Any change to a rule's **behavior, scope, priority, or relationships**, the addition or retirement of a rule, or a new module. Editorial fixes use a lightweight fast-track (§6). Runtime *configuration* changes by clients (values, not rules) are governed by the Configuration Engine (CFG) and its own approval rules — not this process.

## 2. Roles

| Role | Responsibility |
|---|---|
| **Requestor** | Anyone (product, dev, QA, support, client-facing) who raises a Rule Change Request (RCR). |
| **Rule Owner** | The accountable owner of the affected module(s) (see Governance Process). Triages and shepherds the RCR. |
| **Domain Authority** | The functional expert (e.g., Head of Examinations for grading/results) who validates correctness. |
| **Change Board** | The cross-functional body that approves Major changes and resolves cross-module impact. |
| **Implementation Lead** | Dev lead who assesses technical impact and sequences the change. |
| **QA Lead** | Confirms test impact and coverage before release. |

## 3. The Rule Change Request (RCR)

Every change starts as an RCR capturing:
- **Rule(s) affected** (IDs) and **change class** (Editorial / Minor / Major / Retirement / New).
- **Motivation** — the problem, regulation, or improvement driving it.
- **Proposed new text** in the standard 12-field format.
- **Impact analysis** — downstream and upstream dependencies (from the Dependency Map), cross-module references (from Cross-Module References), and any conflicts (checked against the Conflict Register).
- **Records impact** — does it affect already-produced records? (If yes → forward-only by default; recomputation is a separate governed decision.)
- **Test impact** — which test cases change or are added (QA Lead).

## 4. Workflow (the change pipeline)

```
Raise RCR → Triage (Rule Owner) → Impact Analysis → Review → Approve → Implement → Test → Release → Communicate
```

1. **Raise** — Requestor files the RCR.
2. **Triage** — Rule Owner classifies it and checks it is not a duplicate or already-resolved conflict.
3. **Impact Analysis** — mandatory dependency/cross-reference/conflict check. **High-blast-radius rules** (the most-depended-on, per the Dependency Map — e.g., `AUTHZ-003`, `SESS-005`, `STU-005`, `AUTHZ-009`, `AUTH-005`) automatically escalate to the Change Board regardless of class.
4. **Review** — Domain Authority validates correctness; Implementation Lead and QA Lead assess feasibility/coverage.
5. **Approve** —
   - *Editorial/Minor:* Rule Owner approves (Domain Authority informed).
   - *Major/Retirement/New module:* Change Board approves (with Domain Authority sign-off).
   - **Separation of duties:** the Requestor cannot be the sole Approver of their own Major change (mirrors AUTHZ-009).
6. **Implement** — code and rule text updated together; rule version bumped per the Versioning Strategy.
7. **Test** — QA updates/adds test cases derived from the rule's Validation/Failure/Edge sections; Critical-priority rules require passing coverage before release.
8. **Release** — bundled into a catalog version (§ Versioning), with changelog and migration note.
9. **Communicate** — affected teams and (if behavior-facing) clients are notified ahead of the effective date.

## 5. Impact-analysis checklist (mandatory for Minor+)

- [ ] Downstream dependencies reviewed (rule's Related Rules).
- [ ] Upstream dependents reviewed (Dependency Map reverse edges) — who breaks if this changes?
- [ ] Cross-module references reviewed (does another module cite this rule's behavior?).
- [ ] Conflict Register checked — does this re-open a resolved conflict (C-01…C-10) or create a new one?
- [ ] Records impact decided — forward-only (default) or governed recomputation?
- [ ] Audit/permission/notification implications reviewed (the rule's own sections).
- [ ] Test cases identified (QA).
- [ ] Cross-cutting patterns honored — Governed Correction Pattern (C-07), Retention Precedence (C-08).

## 6. Fast-track (Editorial only)

Wording/clarity changes with **no behavioral effect** may be merged by the Rule Owner with a single peer review, no Change Board, no version-major bump (PATCH only). If review reveals any behavioral effect, it exits the fast-track into the full pipeline.

## 7. Emergency changes

A rule that is actively causing harm (e.g., a safeguarding gap, a financial-integrity defect) may be hot-fixed under an **expedited path**: Rule Owner + Domain Authority + one Change Board member approve; full Change Board ratifies retroactively within a defined window. The change is still fully documented, versioned, and audited — speed never bypasses traceability.

## 8. Traceability

Every released change records: RCR id, change class, approvers, rule version before/after, catalog version, effective date, and rationale. This history is itself a governed, retained record — the audit trail of the rules that govern the system.
