# 15 — Result Processing Business Rules

## 1. Module Purpose
Govern **result processing** — transforming locked marks (Doc 14) into finalized results using the grading scheme (Doc 16): subject results, totals/percentages, GPA/CGPA, grades, pass/fail, division/classification, and rank/position. It owns the most computation- and integrity-heavy logic in the system, the **result publishing** peak event (async, step-up MFA, version-stamped), governed post-publish **revision**, result **withholding**, **recheck/re-evaluation** appeals, and marksheet/transcript generation. Results are life-affecting records about minors, so correctness, reproducibility, fairness, and auditability are paramount.

## 2. Actors
- **Exam Controller / Academic Coordinator** — computes, verifies, publishes, revises, and withholds results.
- **System** — runs result calculation deterministically, applies grading, ranks, generates documents, and publishes asynchronously.
- **Student / Guardian** — view published results (own only); may request recheck/re-evaluation.
- **Verifier** — reviews results before publication where configured.

## 3. Use Cases

**Use Case ID:** UC-RES-01 — Compute & Verify Results
**Actors:** Exam Controller, Verifier, System
**Description:** Aggregate locked marks into results and verify before publication.
**Preconditions:** Marks locked (EXM-007); marks complete (EXM-009); grading scheme resolved (GRD-007).
**Main Flow:** 1) System aggregates component → subject → overall per the exam weightages and grading scheme. 2) Computes %, GPA, grade, pass/fail, division, and rank with tie-breaks. 3) Results enter `CALCULATED`; verifier reviews. 4) Verified results are ready to publish.
**Alternative Flow:** A1) Partial computation for the subset with complete data; incomplete students held (RES-005).
**Exception Flow:** E1) Missing marks → those students excluded/held, not zero-scored (EXM-009). E2) Computation inconsistency (e.g., undefined rounding) → block (GRD-003).
**Post Conditions:** Verified results staged; provenance (mark + scheme versions) recorded; audited.
**Business Rules Applied:** RES-001, RES-002, RES-003, RES-004, RES-005.

**Use Case ID:** UC-RES-02 — Publish Results (Peak Event)
**Actors:** Exam Controller, System
**Description:** Make verified results official and visible, at scale.
**Preconditions:** Results verified; controller holds `result.publish`; step-up MFA passed.
**Main Flow:** 1) Controller initiates publish. 2) System publishes asynchronously in batches (peak-event handling), stamping the mark + grading versions. 3) Marksheets/transcripts generate (document pipeline). 4) Students/guardians are notified per preference; held/withheld results stay hidden.
**Exception Flow:** E1) Withheld results (dues/malpractice) excluded from publication (RES-007). E2) Publish during the institute's peak window → throttled/scheduled to protect performance.
**Post Conditions:** Results official and visible; promotion eligibility derivable; fully audited.
**Business Rules Applied:** RES-006, RES-007, RES-009.

**Use Case ID:** UC-RES-03 — Recheck / Re-evaluation & Revision
**Actors:** Student/Guardian, Exam Controller, System
**Description:** Handle a result dispute and any resulting governed revision.
**Preconditions:** Result published; recheck window open; requester is the student/guardian (own).
**Main Flow:** 1) Student requests recheck/re-evaluation (possibly fee-gated). 2) Authorized staff re-verify marks/computation via a governed flow. 3) If a change is warranted, a **revision** corrects the result, re-publishes, version-history retained, and the student is notified of the change.
**Exception Flow:** E1) No change → request closed with outcome recorded. E2) Revision after session close → governed reopen (SESS-005).
**Post Conditions:** Dispute resolved; any revision audited with before/after; notifications sent.
**Business Rules Applied:** RES-006, RES-008.

## 4. Business Rules

**Rule ID:** RES-001
**Rule Name:** Deterministic, Provenance-Stamped Computation
**Description:** Result computation is a deterministic function of locked marks and the resolved grading/exam versions, and records that provenance.
**Priority:** Critical
**Category:** Integrity / reproducibility
**Preconditions:** Marks locked and complete; versions resolved.
**Business Rule:** Given the same locked marks, exam configuration version, and grading scheme version, computation always yields the same result. Each result records the mark set, exam-config version, and grading version used (provenance), so it is reproducible and auditable.
**System Action:** Compute deterministically; stamp provenance on the result.
**Validation:** Inputs finalized; versions resolved; rounding policy defined.
**Failure Behavior:** Block computation on unresolved versions or undefined rounding.
**Audit Requirement:** Result records provenance; `RESULT_CALCULATED` logged.
**Example Scenario:** Recomputing a result from the same inputs reproduces it exactly, defensible in an appeal.
**Related Rules:** EXM-007, GRD-007, RES-003.

**Rule ID:** RES-002
**Rule Name:** Layered Pass/Fail (Component → Subject → Aggregate)
**Description:** Pass/fail is evaluated at component, subject, and overall levels per defined rules; failing any required level fails accordingly.
**Priority:** Critical
**Category:** Assessment integrity
**Preconditions:** Subject and aggregate rules defined.
**Business Rule:** A student must meet component pass rules (SUB-003) to pass a subject; must pass each required subject (or within allowed-fail limits) and meet any aggregate minimum to pass overall. A student can pass every subject total yet fail a component, or pass all subjects yet miss an aggregate threshold — all handled explicitly.
**System Action:** Evaluate pass/fail bottom-up per configured rules.
**Validation:** Pass rules defined at each level; allowed-fail limits set.
**Failure Behavior:** Block result finalization on undefined pass rules.
**Audit Requirement:** Result records which rule(s) determined pass/fail.
**Example Scenario:** A student scoring high in theory but failing practical fails Science despite a high subject total.
**Related Rules:** SUB-003, EXM-002, RES-004.

**Rule ID:** RES-003
**Rule Name:** Single-Point, Defined Rounding
**Description:** Percentages and aggregates are rounded once, at the defined point, per the grading scheme's rounding policy.
**Priority:** Critical
**Category:** Fairness (dispute prevention)
**Preconditions:** Aggregation/percentage computation.
**Business Rule:** Intermediate values are carried at full precision; rounding is applied once at the policy-defined point (e.g., final percentage) using the defined method/precision (GRD-003). No double-rounding; boundary outcomes are deterministic.
**System Action:** Carry precision; round once per policy before grade/division mapping.
**Validation:** Rounding policy resolved from the grading scheme.
**Failure Behavior:** Block computation if rounding undefined.
**Audit Requirement:** Rounding policy version recorded with the result.
**Example Scenario:** 79.49 averaged components do not get rounded twice into a different grade band.
**Related Rules:** GRD-003, RES-001.

**Rule ID:** RES-004
**Rule Name:** Division / Classification & GPA Derivation
**Description:** Overall classification (division/class or CGPA band) and GPA derive from defined rules, consistent with pass/fail.
**Priority:** High
**Category:** Computation integrity
**Preconditions:** Subject results + grading scheme available.
**Business Rule:** Division/classification (First Division / Distinction / CGPA band) is computed per the scheme; a failing student is not awarded a passing division regardless of aggregate; GPA/CGPA follow GRD-004. Classification rules are version-stamped.
**System Action:** Derive classification and GPA per the scheme; reconcile with pass/fail.
**Validation:** Classification rules defined; consistent with pass/fail.
**Failure Behavior:** Block on inconsistent or undefined classification rules.
**Audit Requirement:** Classification basis recorded with the result.
**Example Scenario:** A student failing one required subject is "Fail," not "Second Division," even with a high aggregate.
**Related Rules:** GRD-004, RES-002.

**Rule ID:** RES-005
**Rule Name:** Completeness Gate (No Silent Zeros)
**Description:** A student's result is computed only when their data is complete; missing marks hold the result rather than scoring zero.
**Priority:** Critical
**Category:** Integrity
**Preconditions:** Result computation.
**Business Rule:** If any required mark is missing (not `AB`/`EX`, just absent data), that student's result is **held** (`INCOMPLETE`), not computed with implicit zeros. Absences/exemptions use their special markers (EXM-006/GRD-006). Publication excludes held results.
**System Action:** Detect incompleteness; hold affected results; allow the rest to proceed.
**Validation:** All required marks present or validly special.
**Failure Behavior:** Hold incomplete results; flag for resolution.
**Audit Requirement:** Log `RESULT_HELD_INCOMPLETE`.
**Example Scenario:** A student missing one subject's mark is held, not given a zero that would unfairly fail them.
**Related Rules:** EXM-009, RES-007.

**Rule ID:** RES-006
**Rule Name:** Governed Post-Publish Revision
**Description:** Published results are immutable except via a governed, audited revision that re-publishes and notifies.
**Priority:** Critical
**Category:** Integrity / governance
**Preconditions:** A published result needs correction (error, recheck, override).
**Business Rule:** Revision requires elevated permission (and step-up MFA), a recorded reason, and approval where configured; the prior published result is retained in version history; the revised result is re-published, version-stamped, and the student/guardian notified of the change. Silent edits are impossible.
**System Action:** Create a result revision; retain prior version; re-publish; notify.
**Validation:** Authorized; reason captured; step-up satisfied; approval if required.
**Failure Behavior:** Reject ungoverned post-publish changes.
**Audit Requirement:** Log `RESULT_REVISED` with before/after, reason, actor; notification recorded.
**Example Scenario:** A recheck reveals a totaling error; the result is revised, re-published, and the guardian is told it changed.
**Related Rules:** EXM-007, GRD-008, RES-008, SESS-005.

**Rule ID:** RES-007
**Rule Name:** Result Withholding Is Defined, Fair & Time-Bound
**Description:** Results may be withheld (dues, malpractice, incomplete) under defined, fair, time-bound rules.
**Priority:** High
**Category:** Fairness / finance integration
**Preconditions:** A withholding condition applies.
**Business Rule:** Configurable withholding (e.g., unpaid mandatory dues, pending malpractice, incompleteness) hides the result from the student until resolved; the rule, reason, and resolution path are transparent; withholding is not punitive beyond policy and is released on resolution. Statutory results may be exempt from fee-based withholding per policy/law.
**System Action:** Apply withholding flag; exclude from publication view; release on resolution.
**Validation:** Withholding condition valid; release path defined.
**Failure Behavior:** Improper/indefinite withholding blocked; releases audited.
**Audit Requirement:** Log `RESULT_WITHHELD/RELEASED` with reason.
**Example Scenario:** A result withheld for unpaid dues is released immediately upon payment/waiver.
**Related Rules:** FEE clearance, EXM-006 (malpractice), RES-005.

**Rule ID:** RES-008
**Rule Name:** Deterministic Rank/Position with Defined Tie-Breaking
**Description:** Rank/position is computed within a defined cohort using deterministic, configured tie-breaking.
**Priority:** Medium
**Category:** Fairness
**Preconditions:** Ranking enabled.
**Business Rule:** Rank is computed within a defined cohort (section/class/stream) over a defined metric (total/GPA); ties are broken by a configured deterministic rule (e.g., higher marks in a priority subject, then DOB, then roll number) or shared rank with skipped positions — the policy is explicit, not implementation-dependent. Failed/withheld/incomplete students are handled per policy (often unranked).
**System Action:** Rank deterministically per the configured metric and tie-break.
**Validation:** Cohort and tie-break policy defined.
**Failure Behavior:** Block ranking if the tie-break policy is undefined (no arbitrary ordering).
**Audit Requirement:** Ranking basis/version recorded.
**Example Scenario:** Two students with identical totals are ranked by the configured priority-subject tie-break, reproducibly.
**Related Rules:** RES-004, RES-002.

**Rule ID:** RES-009
**Rule Name:** Asynchronous, Performance-Safe Publishing
**Description:** Publishing runs asynchronously in batches to handle the peak event without degrading interactive performance.
**Priority:** High
**Category:** Performance / reliability
**Preconditions:** Verified results ready; publish initiated.
**Business Rule:** Bulk publication (e.g., a full institution's results) runs as batched background work; marksheet generation is async to object storage; interactive latency is protected; partial failures are recoverable and the operation is idempotent (re-running does not double-publish).
**System Action:** Queue batched publication + document generation; idempotent; progress-tracked.
**Validation:** Idempotency keys; batch sizing per tier.
**Failure Behavior:** Failed batches retry; never double-publish; surface progress/errors.
**Audit Requirement:** Log `RESULT_PUBLISHED` per cohort/student; publish run recorded.
**Example Scenario:** Publishing 30,000 students' results on result day runs in the background while the portal stays responsive.
**Related Rules:** RES-006, performance (Blueprint Part G).

## 5. Validation Rules
- Inputs (marks) locked and complete; exam + grading versions resolved.
- Rounding applied once per defined policy; no double-rounding.
- Pass/fail evaluated at component/subject/aggregate; classification consistent with pass/fail.
- Incomplete results held, never zero-scored.
- Ranking only with a defined cohort + deterministic tie-break.
- Publish/revision require step-up MFA; revision retains prior version.

## 6. State Machine

**State Name:** DRAFT
**Description:** Result computation initiated; not yet finalized.
**Allowed Transitions:** → CALCULATED; → INCOMPLETE (held).
**Forbidden Transitions:** publish from draft.
**System Actions:** Aggregate; detect completeness.

**State Name:** CALCULATED
**Description:** Computed, awaiting verification.
**Allowed Transitions:** → VERIFIED; → DRAFT (recompute on input change).
**Forbidden Transitions:** publish before verification.
**System Actions:** Hold provenance; present for review.

**State Name:** VERIFIED
**Description:** Reviewed and ready to publish.
**Allowed Transitions:** → PUBLISHED; → WITHHELD; back to CALCULATED on issue.
**Forbidden Transitions:** edits to marks (locked upstream).
**System Actions:** Stage for batched publish.

**State Name:** PUBLISHED
**Description:** Official and visible to students/guardians.
**Allowed Transitions:** → REVISED (governed); → WITHHELD (post-publish, exceptional); → ARCHIVED.
**Forbidden Transitions:** silent edits.
**System Actions:** Generate marksheets; notify; enable promotion eligibility.

**State Name:** REVISED
**Description:** A governed correction re-published; prior version retained.
**Allowed Transitions:** → PUBLISHED (the revision is now current); → ARCHIVED.
**Forbidden Transitions:** loss of prior version.
**System Actions:** Retain history; notify of change.

**State Name:** WITHHELD
**Description:** Computed but hidden pending resolution (dues/malpractice/incomplete).
**Allowed Transitions:** → PUBLISHED (released); → REVISED.
**Forbidden Transitions:** indefinite/unaudited withholding.
**System Actions:** Hide from student view; release on resolution.

**State Name:** INCOMPLETE
**Description:** Held for missing data; not computed with zeros.
**Allowed Transitions:** → CALCULATED (data completed).
**Forbidden Transitions:** publish.
**System Actions:** Flag for resolution.

**State Name:** ARCHIVED
**Description:** Result for a closed/archived session; immutable.
**Allowed Transitions:** governed reopen only.
**Forbidden Transitions:** edits.
**System Actions:** Read-only retention.

## 7. Status Definitions
`DRAFT` · `CALCULATED` · `VERIFIED` · `PUBLISHED` · `REVISED` · `WITHHELD` · `INCOMPLETE` · `ARCHIVED`. Per-student outcome: `PASS` · `FAIL` · `INCOMPLETE` · `WITHHELD` · `EXEMPT`.

## 8. Workflow Rules
- Verification is a configurable pre-publish step (verifier ≠ sole entering teacher under SoD).
- Publish and revision require step-up MFA (RES-006/RES-009).
- Recheck/re-evaluation runs a governed, possibly fee-gated workflow; revisions follow RES-006.
- Withholding/release is governed and audited (RES-007); closed-session changes need reopen (SESS-005).

## 9. Permission Rules
- `result.calculate` — run computation/verification.
- `result.publish` — publish results (step-up; narrowly granted).
- `result.revise` — governed post-publish revision (elevated; step-up).
- `result.withhold` — apply/release withholding.
- `result.view` — read results (students/guardians own; staff scoped).
- `result.recheck.process` — handle recheck/re-evaluation.

## 10. Notification Rules
- `RESULT_PUBLISHED` → notify students/guardians per preference (deduplicated bulk; respects custody restrictions GRD-N-006).
- `RESULT_REVISED` → explicitly notify affected students/guardians that the result changed.
- `RESULT_WITHHELD/RELEASED` → notify with reason and resolution path.
- Recheck outcome → notify the requester.
- Notifications never expose another student's result; rank notices follow policy.

## 11. Audit Requirements
Mandatory: `RESULT_CALCULATED` (provenance: mark + exam + grading versions), `RESULT_VERIFIED`, `RESULT_PUBLISHED` (per student/cohort), `RESULT_REVISED` (before/after, reason, step-up), `RESULT_WITHHELD/RELEASED`, `RESULT_HELD_INCOMPLETE`, recheck requests/outcomes, ranking basis. Results are life-affecting — every state change is attributable and reproducible.

## 12. Data Retention Rules
- Published results and transcripts retained long-term per academic-record retention (often permanent / many years).
- All result versions (original + revisions) retained — the history of what was published when.
- Provenance (mark/exam/grading versions) retained so any result is reproducible.
- Recheck records retained per appeals policy; anonymization per lawful erasure preserves aggregate integrity.

## 13. Edge Cases
- **Pass all subjects, fail aggregate (or vice versa):** handled by layered pass/fail (RES-002) — a frequent logic bug.
- **Boundary percentage:** single-point rounding makes it deterministic (RES-003).
- **Rank ties:** resolved by the configured deterministic tie-break (RES-008), never arbitrary.
- **Missing mark vs absent:** missing data holds the result (INCOMPLETE); `AB` is a scored outcome per policy — distinct (RES-005/EXM-006).
- **Supplementary/makeup merge:** a separate exam's result merges per policy into an updated result (RES-006), not an edit of the original sitting.
- **Withheld then resolved after the publish event:** released and published on resolution without a full re-run (RES-007).
- **Result of a transferred student:** partial results attributed to the period attended; combined per policy.
- **Exempted subject:** excluded from aggregates/GPA (GRD-006), not zero.
- **Re-evaluation lowers a mark:** policy decides whether re-evaluation can decrease a result; the rule is explicit and disclosed.

## 14. Failure Scenarios
- **Publishing with incomplete data:** blocked for affected students (RES-005); the rest proceed.
- **Undefined rounding/classification/tie-break:** computation/ranking blocked (no arbitrary output).
- **Publish run partial failure:** idempotent retry; never double-publish (RES-009).
- **Post-publish silent edit:** impossible; only governed revision (RES-006).
- **Projection/marksheet lag at peak:** results consistent; document generation async; portal protected.

## 15. Exception Handling Rules
- Computation requires finalized, complete inputs and resolved versions; otherwise it blocks rather than guessing.
- Special outcomes and incompleteness are explicit; never silent zeros.
- Publishing and revisions are step-up-gated, idempotent, and audited.
- Withholding is bounded, transparent, and releasable.

## 16. Compliance Considerations
- **Accuracy & reproducibility:** provenance-stamped, deterministic computation makes results defensible in disputes and audits.
- **Right to appeal:** recheck/re-evaluation with governed revision and disclosure supports fairness.
- **Transparency of revision:** students are told when a published result changes — never silently.
- **Record permanence:** transcripts retained per legal/accreditation requirements; minors' results protected and access-controlled.
- **Withholding fairness:** bounded, transparent, with statutory exemptions respected.

## 17. Future Considerations
- Predictive/at-risk analytics from results (consent-bound) for intervention.
- Verifiable digital transcripts (QR/blockchain-style verification).
- Automated recheck triage (flag likely errors for human review).
- Outcome-based result models alongside marks/GPA.
