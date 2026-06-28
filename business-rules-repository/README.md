# Business Rules Repository — Master Source of Truth

The consolidated, governed **functional source of truth** for the Enterprise Education ERP. All nine delivery phases merged into one repository: **30 modules · 248 business rules · 87 use cases**, every module in the fixed 17-section structure, every rule in the fixed 12-field format. This is the catalog development builds to and QA tests against.

**Catalog version:** `1.0.0` (initial complete release) · **Status:** all conflicts resolved (1 institutional decision pending, C-02).

> Scope: this is the **functional** source of truth. It does not redesign the locked Architecture Blueprint (Parts A–H) or replace the Implementation Plan; it specifies *what must be true*, which development and QA implement and verify.

## Repository structure

```
business-rules-repository/
├── README.md                          ← you are here (master entry point)
├── CROSS_CUTTING_PRINCIPLES.md        ← 14 patterns implemented once, obeyed everywhere
├── modules/                           ← the 30 module rule documents (+ catalog README)
│   ├── 01-authentication-rules.md … 30-file-management-rules.md
├── catalog/
│   ├── MASTER_RULE_CATALOG.md         ← all 248 rules, consolidated (the index of record)
│   ├── BUSINESS_RULES_INDEX.md        ← lookups by ID, priority, category
│   ├── RULE_DEPENDENCY_MAP.md         ← upstream/downstream dependencies; blast-radius ranking
│   ├── CROSS_MODULE_REFERENCES.md     ← inter-module coupling matrix
│   └── CONFLICT_REGISTER.md           ← genuine conflicts found + binding resolutions
└── governance/
    ├── RULE_VERSIONING_STRATEGY.md    ← how rules version; the reproducibility triple
    ├── RULE_CHANGE_MANAGEMENT.md      ← how a change flows (RCR pipeline)
    └── RULE_GOVERNANCE_PROCESS.md     ← who owns rules; guardrails; review cadence
```

## How to use this repository

- **Developers** — implement against the **module documents** (full 12-field specs). Use the **Master Catalog** to locate a rule, the **Dependency Map** before changing a high-blast-radius rule, and **Cross-Cutting Principles** for the patterns (correction, retention, SoD, idempotency) that span modules.
- **QA** — derive test cases from each rule's *Validation*, *Failure Scenarios*, and *Edge Cases* sections. Start with the **87 Critical-priority rules** (Index §B); they are the integrity/safety backbone. Test the cross-cutting patterns once (P5 Correction, P6 Reproducibility, P9 SoD, P10 Idempotency) plus their per-module specializations.
- **Product/BA** — the **Index** and **Master Catalog** are the navigable overview; the **Conflict Register** records every cross-module ruling and the one open decision (C-02).
- **Governance** — **Versioning**, **Change Management**, and **Governance Process** define how the catalog evolves without losing trustworthiness.

## The catalog at a glance

- **248 rules** by priority: **Critical 87 · High 121 · Medium 39 · Low 1**.
- **The two engines (27 Configuration, 28 Workflow) are the substrate.** Every "configurable/version-stamped" promise resolves through Configuration; every "approval/escalation/SoD" promise through Workflow.
- **Highest-blast-radius rules** (most depended-upon — change with extra care): `AUTHZ-003` (ownership), `SESS-005` (post-close immutability), `STU-005` (sensitive-data masking), `AUTHZ-009` (SoD), `AUTH-005` (immediate revocation).
- **Most-referenced modules** (foundational): 02 Authorization, 05 Academic Session, 22 Staff, 06 Student, 10 Attendance.

## Consistency guarantee

The four catalog artifacts (Master Catalog, Index, Dependency Map, Cross-Module References) are **generated from the module text**, not hand-maintained — so they cannot silently drift from the rules. On every release they are regenerated and validated: 30 modules present, 17 sections each, all rule IDs unique and non-reused, every cross-reference resolvable, the Conflict Register re-confirmed. (See Governance Process §5.)

## What's resolved, what's open

The **Conflict Register** resolved 10 genuine tensions across the 248 rules — capacity-buffer-vs-hard-cap (C-01), waiver ownership (C-03), student-leave boundary (C-04), ownership-after-reassignment (C-05), one lock mechanism (C-06), the unified Governed Correction Pattern (C-07), retention precedence (C-08), and more. **One item (C-02 — whether re-evaluation may lower a published result) is an institutional policy decision** that must be made and disclosed before go-live; the recommended default is documented.

---

*Generated from the approved Phase 1–9 modules. This repository supersedes the per-phase deliverables as the single, governed source of truth.*
