# Rule Versioning Strategy

How business rules themselves are versioned over time, so that the catalog can evolve without breaking traceability, reproducibility, or the records produced under older rules. This is distinct from the *runtime* configuration versioning specified in Doc 27 (CFG) — that versions **values**; this versions **rules** (the logic in this catalog).

## 1. Principles

1. **Rule IDs are permanent and immutable.** `AUTH-001` means one thing forever. An ID is never re-pointed to a different rule and never reused after retirement.
2. **Rules are versioned, not overwritten.** A material change to a rule produces a new *rule version*, retaining the prior version's text and rationale.
3. **Records reference the rule version in force when they were produced.** Combined with the runtime config-version stamping (CFG-004), this makes any historical record fully reproducible: *which rule, at which version, with which configuration*.
4. **The catalog is itself a versioned artifact** with a semantic version (see §4).

## 2. Rule-level versioning

Each rule carries an implicit version, surfaced in a change-controlled environment as a header field:

```
Rule ID: RES-006
Rule Version: 3
Status: Active
Supersedes: RES-006 v2
Effective: 2026-09-01
```

**Change classification (per rule):**

| Class | Definition | Version effect | Example |
|---|---|---|---|
| **Editorial** | Wording/clarity only; behavior unchanged | No version bump; revision note | Fixing a typo in an Example |
| **Minor** | Tightens/clarifies behavior without changing outcomes for existing valid inputs | Minor version bump (v2 → v2.1) | Adding an explicit audit field already implied |
| **Major** | Changes system behavior or outcomes | Major version bump (v2 → v3); supersession recorded | Changing rounding direction in GRD-003 |
| **Retirement** | Rule no longer applies | Status → Retired; ID frozen, never reused | Removing a deprecated flow |

**Status lifecycle of a rule:** `Draft → Active → (Superseded by a new version | Deprecated → Retired)`. A Retired rule's ID and history are kept read-only for traceability.

## 3. Rule states and their meaning

- **Draft** — proposed, under review; not yet binding on implementation.
- **Active** — the current binding version; what development and QA implement/test against.
- **Superseded** — a prior version replaced by a newer Active version; retained for records produced under it.
- **Deprecated** — slated for retirement; still Active until the effective retirement date, but no new dependencies should be added.
- **Retired** — no longer in force; kept for historical reproducibility; ID frozen.

## 4. Catalog-level versioning (semantic)

The catalog as a whole uses `MAJOR.MINOR.PATCH`:

- **MAJOR** — one or more *Major* rule changes, rule retirements, or a new module that alters cross-module contracts. (e.g., 1.x → 2.0)
- **MINOR** — new rules or modules that are additive and backward-compatible; *Minor* rule changes. (e.g., 1.2 → 1.3)
- **PATCH** — editorial changes only. (e.g., 1.2.0 → 1.2.1)

**Current catalog version:** `1.0.0` — initial complete release (30 modules, 248 rules, all 9 phases merged, conflict register resolved).

Each release ships a **changelog** (added/changed/retired rules with class and rationale) and a **migration note** when a Major change affects already-produced records.

## 5. Reproducibility contract

For any record the system produces (marksheet, invoice, payslip, decision), the following triple must be recoverable:
1. **Rule version** — which version of the governing rules applied (this strategy).
2. **Configuration version** — which config values applied (CFG-004).
3. **Workflow version** — which process governed any approval (WFL-002).

This triple is the foundation of every "reproducible/defensible historical record" claim in the catalog. A record is never re-evaluated against newer rules; it is reproduced against the versions stamped on it.

## 6. Backward compatibility & data migration

- **Additive changes** (new rules/modules) require no data migration.
- **Major changes** that alter computation (e.g., grading) apply **forward only**; existing records keep their stamped versions. If historical recomputation is ever required, it is an explicit, governed, audited batch — never an implicit side effect of a rule change.
- **Retirements** require a dependency check (see Dependency Map): a rule with active upstream dependents cannot be retired until those are migrated.

## 7. Tooling expectations

- The catalog lives in version control; each rule change is a reviewed commit/PR (see Change Management).
- Rule IDs are validated for uniqueness and non-reuse on every change.
- The Index, Dependency Map, and Cross-Module References are **regenerated from the module text** on every release (as they were for v1.0.0) so they never drift from the rules.
