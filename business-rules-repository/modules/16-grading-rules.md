# 16 — Grading Business Rules

## 1. Module Purpose
Govern the **grading scheme** — the configurable definition that converts marks into grades, grade points, GPA/CGPA, and pass/fail/division outcomes. Grading is *configuration* (Doc 27/Configuration Engine) consumed by Result Processing (Doc 15); it is **versioned and effective-dated** so historical results never silently change. This module owns grade scales, boundaries, **rounding rules** (the single largest source of result disputes, made explicit here), GPA computation, special grades, and the integrity rules that keep grading transparent and reproducible.

## 2. Actors
- **Exam Controller / Academic Coordinator** — defines and versions grading schemes.
- **System** — validates scheme integrity, applies rounding deterministically, computes grades/GPA, version-stamps results.
- **Auditor / Accreditor** — reads the historical grading scheme that produced any result.

## 3. Use Cases

**Use Case ID:** UC-GRD-01 — Define / Version a Grading Scheme
**Actors:** Academic Coordinator, System
**Description:** Create or revise a grading scheme with grade bands, points, GPA scale, and rounding.
**Preconditions:** Actor holds `grading.scheme.manage`.
**Main Flow:** 1) Define grade bands (mark ranges → letter grade → grade point), GPA scale, pass rules, rounding policy. 2) System validates completeness (0–max fully covered, no overlaps/gaps). 3) Publish as a new immutable version with an effective date; the prior version remains for historical results.
**Alternative Flow:** A1) Distinct schemes per class/stream/subject (GRD-005).
**Exception Flow:** E1) Bands overlap or leave gaps → reject (GRD-002). E2) Edit a published version in place → blocked; create a new version.
**Post Conditions:** Scheme version active for future results; history preserved; audited.
**Business Rules Applied:** GRD-001, GRD-002, GRD-007.

**Use Case ID:** UC-GRD-02 — Apply Grading to Compute a Grade
**Actors:** System (invoked by Result Processing)
**Description:** Convert a (possibly aggregated) mark to a grade and grade point.
**Preconditions:** A finalized mark exists; the applicable scheme version is resolved.
**Main Flow:** 1) Resolve the scheme for the student's scope/subject as of the result's effective context. 2) Apply rounding per policy. 3) Map the rounded mark to a grade band → grade + grade point. 4) Return for GPA/result aggregation.
**Exception Flow:** E1) Mark on a boundary → resolved deterministically by the rounding/boundary rule (GRD-003). E2) Special marker (AB/EX) → special grade per policy (GRD-006).
**Post Conditions:** Grade/point computed reproducibly; the scheme version recorded with the result.
**Business Rules Applied:** GRD-003, GRD-004, GRD-006.

**Use Case ID:** UC-GRD-03 — Grade Override (Exceptional)
**Actors:** Exam Controller, System
**Description:** Manually override a computed grade in a governed, audited way.
**Preconditions:** A justified exceptional case; actor holds `grading.override`.
**Main Flow:** 1) Controller overrides a grade with a reason. 2) System records the computed value and the override distinctly (computed value preserved). 3) The override is reflected with provenance.
**Exception Flow:** E1) Unauthorized/unjustified override → rejected.
**Post Conditions:** Override applied transparently; audited; original computation retained.
**Business Rules Applied:** GRD-008.

## 4. Business Rules

**Rule ID:** GRD-001
**Rule Name:** Configurable, Versioned Grade Scale
**Description:** Grade scales (bands, points, GPA scale) are configurable and published as immutable versions.
**Priority:** Critical
**Category:** Configuration / integrity
**Preconditions:** Scheme definition.
**Business Rule:** A grading scheme defines ordered grade bands (mark range → grade → grade point) and a GPA scale (e.g., 4.0/5.0/10.0). Publishing creates an immutable version with an effective date; in-place edits to a published version are forbidden.
**System Action:** Store and version schemes; freeze published versions.
**Validation:** Bands ordered; points/GPA scale consistent.
**Failure Behavior:** Reject edits to published versions; require a new version.
**Audit Requirement:** Log `GRADING_SCHEME_PUBLISHED` with version + effective date.
**Example Scenario:** A 2027 scheme change is a new version; 2025 results keep using the 2025 version.
**Related Rules:** GRD-007, CFG (versioning), SESS-005.

**Rule ID:** GRD-002
**Rule Name:** Complete, Non-Overlapping Band Coverage
**Description:** Grade bands must fully cover the valid mark range with no overlaps or gaps.
**Priority:** Critical
**Category:** Integrity
**Preconditions:** Scheme validation.
**Business Rule:** Bands must cover 0 to max contiguously; no mark may map to two grades (overlap) or to none (gap). Boundary ownership (inclusive/exclusive) is explicit.
**System Action:** Validate full contiguous coverage with explicit boundary ownership.
**Validation:** Union of bands = [0, max]; pairwise non-overlapping; boundaries defined.
**Failure Behavior:** Reject schemes with overlaps or gaps.
**Audit Requirement:** Captured in scheme publication.
**Example Scenario:** Bands 80–100 A+, 70–79 A, ... 0–32 F — every mark lands in exactly one band.
**Related Rules:** GRD-003, GRD-001.

**Rule ID:** GRD-003
**Rule Name:** Explicit, Deterministic Rounding & Boundary Rules
**Description:** Rounding precision, method, and boundary ownership are explicitly defined and applied once, deterministically.
**Priority:** Critical
**Category:** Fairness (top dispute source)
**Preconditions:** Mark-to-grade conversion / percentage computation.
**Business Rule:** The scheme defines: where rounding occurs (e.g., final percentage only, not at every intermediate step — avoid double-rounding), the precision (decimal places), the method (half-up / half-even), and boundary inclusivity (is exactly 79.5 an A?). The same input always yields the same grade.
**System Action:** Apply the single configured rounding step deterministically before band mapping.
**Validation:** Rounding policy defined; boundary ownership explicit.
**Failure Behavior:** Block computation if rounding policy is undefined.
**Audit Requirement:** Rounding policy version recorded with results.
**Example Scenario:** Policy "round final % half-up to 0 decimals; band lower bound inclusive" makes 79.5 → 80 → A+, reproducibly and defensibly.
**Related Rules:** GRD-002, RES-003.

**Rule ID:** GRD-004
**Rule Name:** GPA / CGPA Computation Rule
**Description:** GPA is credit-weighted per the defined scale; CGPA aggregates across terms/sessions per policy.
**Priority:** High
**Category:** Computation integrity
**Preconditions:** Subject grades + credits available.
**Business Rule:** GPA = sum(grade_point × credit) / sum(credit) over graded subjects, on the scheme's scale; non-graded subjects (SUB-005) are excluded from the denominator; CGPA aggregates per the configured method (credit-weighted across terms, or average of GPAs). Failed subjects count per policy (often grade-point 0 included).
**System Action:** Compute GPA/CGPA per the defined formula and scale.
**Validation:** Credits present; scale consistent; denominator > 0.
**Failure Behavior:** Block GPA on zero graded-credit or inconsistent scale.
**Audit Requirement:** GPA formula/scale version recorded with results.
**Example Scenario:** A 4-credit A (4.0) and a 2-credit B (3.0) yield GPA (4×4 + 2×3)/6 = 3.67.
**Related Rules:** SUB-005, GRD-006, RES-004.

**Rule ID:** GRD-005
**Rule Name:** Scope-Specific Schemes (Absolute vs Relative)
**Description:** Different classes/streams/subjects may use different schemes, including absolute or relative (curved) grading.
**Priority:** Medium
**Category:** Flexibility
**Preconditions:** Scheme assignment.
**Business Rule:** Schemes resolve by scope (most-specific-wins: subject → class/stream → institute). Absolute grading uses fixed bands; relative (curve/percentile) grading derives bands from the cohort distribution per defined statistics and a minimum-cohort-size guard.
**System Action:** Resolve the applicable scheme; for relative grading, compute cohort statistics with the guard.
**Validation:** Scheme exists for scope; relative grading meets minimum cohort size.
**Failure Behavior:** Relative grading below minimum cohort falls back to absolute (per policy), audited.
**Audit Requirement:** Log scheme resolution and any relative-grading fallback.
**Example Scenario:** University courses use relative grading; school classes use absolute — both configured.
**Related Rules:** GRD-001, INST-005 (config resolution).

**Rule ID:** GRD-006
**Rule Name:** Special Grades Are Defined and Excluded Correctly
**Description:** Non-numeric outcomes (Absent, Incomplete, Exempt, Withdrawn, Withheld) have defined grades and consistent aggregate treatment.
**Priority:** High
**Category:** Integrity
**Preconditions:** A non-numeric outcome reaches grading.
**Business Rule:** The scheme defines special grades (`AB`, `I`, `EX`, `W`, `WH`) and their treatment: exempt excluded from aggregates; absent/incomplete handled per policy (often blocking a final result until resolved); these never become grade-point 0 silently.
**System Action:** Map special markers to special grades; apply defined aggregate treatment.
**Validation:** Special grades defined; treatment policy set.
**Failure Behavior:** Block aggregation that would mis-treat a special grade as zero.
**Audit Requirement:** Captured in result computation provenance.
**Example Scenario:** An exempted subject is excluded from GPA rather than dragging it to zero.
**Related Rules:** EXM-006, RES-002, GRD-004.

**Rule ID:** GRD-007
**Rule Name:** Effective-Dated Resolution & Result Stamping
**Description:** Results are graded by the scheme version in effect for their context and stamp that version permanently.
**Priority:** Critical
**Category:** Historical fidelity
**Preconditions:** Result computation.
**Business Rule:** The grading scheme version resolved is the one effective for the result's session/context; the result records the scheme version used, so re-printing or recomputation reproduces the original outcome even after the scheme changes.
**System Action:** Resolve by effective date; stamp version on the result/marksheet.
**Validation:** A scheme version is effective for the context.
**Failure Behavior:** Block grading if no effective scheme resolves.
**Audit Requirement:** Result records the grading-scheme version.
**Example Scenario:** A reprinted 2025 marksheet uses the 2025 scheme even though the scheme changed in 2027.
**Related Rules:** GRD-001, RES-008, SESS-005.

**Rule ID:** GRD-008
**Rule Name:** Governed, Transparent Grade Overrides
**Description:** Manual grade overrides are exceptional, authorized, reasoned, and preserve the computed value.
**Priority:** High
**Category:** Integrity / governance
**Preconditions:** A grade override.
**Business Rule:** An override requires elevated permission and a reason; the system stores the computed grade and the override distinctly (computed value never erased); overrides are visible in provenance and may require approval (SoD: not the entering teacher).
**System Action:** Record override as a layer over the computed grade; preserve original.
**Validation:** Authorized; reason captured; SoD respected.
**Failure Behavior:** Reject unauthorized/unreasoned overrides.
**Audit Requirement:** Log `GRADE_OVERRIDDEN` with computed value, override, reason, actor.
**Example Scenario:** A controller corrects a grade for a documented exceptional reason; the original computation remains visible.
**Related Rules:** AUTHZ-009, EXM-008, RES-006.

## 5. Validation Rules
- Bands ordered, contiguous over [0, max], non-overlapping, explicit boundaries.
- Rounding policy defined (where/precision/method/boundary) and applied once.
- GPA scale and formula consistent; denominator > 0; non-graded excluded.
- Special grades defined with explicit aggregate treatment.
- Published scheme versions immutable; changes create new versions.
- Relative grading meets minimum cohort or falls back (audited).

## 6. State Machine

**State Name:** DRAFT
**Description:** Scheme being defined; not applied to results.
**Allowed Transitions:** → ACTIVE (published as a version); → DISCARDED.
**Forbidden Transitions:** application to results while draft.
**System Actions:** Validate coverage/rounding/GPA before publish.

**State Name:** ACTIVE (version)
**Description:** Published, immutable, effective for its date range.
**Allowed Transitions:** → SUPERSEDED (a newer version published); remains readable for historical results.
**Forbidden Transitions:** in-place edits.
**System Actions:** Apply to in-scope results; stamp version on outputs.

**State Name:** SUPERSEDED
**Description:** Replaced by a newer version; still used by results created under it.
**Allowed Transitions:** → ARCHIVED.
**Forbidden Transitions:** edits.
**System Actions:** Serve historical grading; never alter past results.

**State Name:** ARCHIVED
**Description:** Long-term retained scheme version.
**Allowed Transitions:** none (terminal).
**Forbidden Transitions:** edits/delete (results depend on it).
**System Actions:** Read-only retention.

## 7. Status Definitions
Scheme: `DRAFT` · `ACTIVE` (versioned) · `SUPERSEDED` · `ARCHIVED`. Grade types: numeric-derived letter grades + special grades `AB`/`I`/`EX`/`W`/`WH`/`F`/`P`.

## 8. Workflow Rules
- Scheme publication is a governed action (validated, versioned, effective-dated); may require approval given downstream impact.
- Grade overrides require authorization + reason and may require approval (SoD).
- Relative-grading fallbacks are automatic but audited.
- Scheme changes never trigger recomputation of already-published historical results (only forward).

## 9. Permission Rules
- `grading.scheme.manage` — define/publish/version grading schemes.
- `grading.override` — apply governed grade overrides (narrowly granted).
- `grading.view` — read schemes (academic staff; published scheme is transparent to students/guardians).
- Scoped to institute; scheme resolution respects scope specificity.

## 10. Notification Rules
- `GRADING_SCHEME_PUBLISHED` → notify academic staff (and, per policy, publish the scheme to students/guardians for transparency).
- `GRADE_OVERRIDDEN` → notify governance/admin (audit channel).
- Grading-scheme transparency: the scheme used is available to students/guardians with their results.

## 11. Audit Requirements
Mandatory: `GRADING_SCHEME_DEFINED/PUBLISHED` (version, effective date, bands, rounding, GPA), `GRADE_OVERRIDDEN` (computed vs override, reason, actor), relative-grading fallbacks, scheme-resolution provenance on results. The scheme version is recorded on every result it produces.

## 12. Data Retention Rules
- All grading-scheme versions retained indefinitely (or per academic-record retention) — results permanently depend on them.
- Override records retained with full provenance.
- Scheme history supports accreditation/audit ("what scale produced this grade").

## 13. Edge Cases
- **Boundary mark (79.5):** resolved deterministically by explicit boundary + rounding rules (GRD-003) — the classic dispute, now defined.
- **Double-rounding:** prevented by rounding once at the defined step, not at every intermediate (GRD-003).
- **Relative grading, tiny cohort:** falls back to absolute below the minimum cohort guard (GRD-005).
- **Non-graded subject in GPA:** excluded from the denominator (GRD-004/SUB-005), not counted as zero.
- **Scheme change mid-session:** applies forward only; current-session results use the session's effective version (GRD-007).
- **Failed subject in GPA:** counts per policy (commonly 0 grade-point included) — defined, not assumed.
- **Different schemes per stream:** Science and Commerce in one class may grade differently (GRD-005).
- **Grade override after publish:** governed revision (links RES-006), original computation preserved.

## 14. Failure Scenarios
- **Bands overlap/gap:** scheme rejected at publish (GRD-002).
- **Undefined rounding policy:** computation blocked (GRD-003).
- **Zero graded credit:** GPA computation blocked, not divided-by-zero (GRD-004).
- **No effective scheme for context:** grading blocked (GRD-007).
- **Unauthorized override:** rejected (GRD-008).

## 15. Exception Handling Rules
- Scheme integrity (coverage, rounding, GPA) validated before publish; invalid schemes never reach results.
- Special grades handled explicitly; never coerced to numeric zero.
- Overrides are layered and audited; computed values are never destroyed.
- Historical results are immune to scheme changes (forward-only).

## 16. Compliance Considerations
- **Transparency & fairness:** explicit, published, versioned schemes with defined rounding defend grade decisions against disputes and appeals.
- **Reproducibility:** version-stamping makes any historical result reproducible — essential for transcripts and accreditation.
- **Auditability:** overrides and scheme changes are fully attributable.
- **Minors:** grades are sensitive, life-affecting data; integrity and transparency are protective obligations.

## 17. Future Considerations
- Outcome-/competency-based grading alongside numeric scales.
- Configurable percentile/curve models with richer statistics.
- Grade-trend analytics (consent-bound) for early intervention.
- Cross-institution grade equivalence/transfer-credit mapping.
