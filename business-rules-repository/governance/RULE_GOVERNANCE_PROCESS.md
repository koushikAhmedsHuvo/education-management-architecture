# Rule Governance Process

Who owns the business rules, how their quality and consistency are maintained over time, and how the catalog stays the trustworthy source of truth for development and QA. Versioning (how rules change form) and Change Management (how a single change flows) are the *mechanics*; this document is the *organization* around them.

## 1. Ownership model

The catalog is owned collectively but accountably. Every module has a single **Rule Owner** (accountable) and a **Domain Authority** (functional expert). Ownership is grouped to match the catalog's structure:

| Domain group | Modules | Typical Rule Owner | Domain Authority |
|---|---|---|---|
| Platform & Access | 01 Auth, 02 Authz, 27 Config, 28 Workflow, 29 Audit | Platform Lead | Security/Architecture |
| Organization | 03 Institute, 04 Campus, 05 Session | Product (Core) | Operations |
| Student Lifecycle | 06 Student, 07 Admission, 08 Enrollment, 09 Guardian | Product (Student) | Admissions/Registrar |
| Academic | 10 Attendance, 11 Class, 12 Section, 13 Subject | Product (Academic) | Academic Coordinator |
| Examination | 14 Exam, 15 Results, 16 Grading | Product (Assessment) | Head of Examinations |
| Finance | 17 Fee, 18 Payment, 19 Discount, 20 Scholarship | Product (Finance) | Finance Controller |
| HR | 21 Teacher, 22 Staff, 23 HR, 24 Leave | Product (HR) | HR Head |
| Platform Services | 25 Notification, 26 Reporting, 30 File | Platform Lead | Data Protection Officer |

The **Change Board** is the cross-functional body (Rule Owners' representatives + Architecture + QA Lead + Data Protection Officer) that approves Major changes and arbitrates cross-module impact.

## 2. Responsibilities

**Rule Owner** — keeps their module's rules correct, consistent, and current; triages RCRs; ensures cross-references stay valid; signs off Minor changes.

**Domain Authority** — guarantees functional correctness against real-world practice and regulation; signs off Major changes in their domain.

**Change Board** — approves Major changes, new modules, and retirements; owns the Conflict Register; sets release cadence; the final arbiter when modules disagree.

**Data Protection Officer (standing seat)** — reviews any change touching minors' data, consent, retention, or audit; holds a **veto on child-safety and data-protection regressions** (e.g., a change must never weaken custody enforcement GRD-N-006, sensitive-data masking STU-005, or audit immutability AUD-001).

**QA Lead** — maintains the rule-to-test traceability; blocks release of Critical-rule changes without passing coverage.

## 3. Standing guardrails (non-negotiable)

These invariants cannot be weakened by any single change, only by an explicit, minuted Change Board decision with DPO sign-off:

1. **Child safety** — mandatory guardianship (STU-006), custody enforcement (GRD-N-006), and safeguarding signals (ATT-008) cannot be weakened.
2. **Authorization integrity** — three-layer access (AUTHZ-001/002/003), no privilege escalation (AUTHZ-005), and SoD (AUTHZ-009) cannot be bypassed.
3. **Financial integrity** — invoice/payment immutability and reversal-not-delete (FEE-003, PAY-007), idempotency (PAY-008), and SoD on money (HR-006, DSC-003) cannot be relaxed.
4. **Academic integrity** — mark/result immutability and governed revision (EXM-007, RES-006) and grading reproducibility (GRD-007) cannot be circumvented.
5. **Auditability** — audit immutability (AUD-001) and reliable capture (AUD-002) cannot be disabled.
6. **Reproducibility triple** — rule + config + workflow version stamping cannot be dropped.

## 4. Review cadence

- **Per-change** — every RCR (Change Management process).
- **Quarterly module review** — each Rule Owner reviews their module for drift, dead cross-references, and new edge cases surfaced in production/support.
- **Semi-annual conflict sweep** — the Change Board re-runs the consistency checks (regenerate Index/Dependency/Cross-Ref from text; re-examine the Conflict Register) and confirms no new conflicts have crept in.
- **Annual regulatory review** — the DPO and Domain Authorities review against current law (data protection, education, labor, financial) per jurisdiction.

## 5. Consistency enforcement

The structural artifacts (Index, Dependency Map, Cross-Module References, Master Catalog) are **generated from the module text**, never hand-maintained, so they cannot silently drift from the rules. On every release:
1. Regenerate all four artifacts from `/modules`.
2. Validate: all 30 modules present, all 17 sections each, all rule IDs unique and non-reused, every `Related Rules` reference resolves to a real rule.
3. Re-confirm the Conflict Register: no resolved conflict re-opened, no new conflict introduced.
4. Confirm cross-cutting patterns intact: Governed Correction Pattern (C-07), Retention Precedence (C-08).

## 6. Relationship to architecture & implementation

The catalog is the **functional** source of truth. It does not redesign the locked architecture (Blueprint Parts A–H) and does not replace technical artifacts. The intended flow:

```
Architecture Blueprint (how) ──┐
                               ├─→ Business Rules Catalog (what must be true)
Implementation Plan (when) ────┘            │
                                            ├─→ Development (build to rules)
                                            └─→ QA (test against rules)
```

Where a rule and the architecture appear to disagree, the Change Board and Architecture jointly reconcile — neither silently overrides the other.

## 7. Decision log

The Change Board maintains a decision log (the Conflict Register C-01…C-10 is its seed). Every cross-module ruling, every standing-guardrail exception, and every open-decision resolution (e.g., C-02 recheck direction) is recorded there with date, rationale, and approvers — so future teams inherit not just the rules but the *reasoning* behind them.
