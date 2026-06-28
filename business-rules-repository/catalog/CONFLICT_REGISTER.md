# Conflict Register & Resolutions

A source of truth must be internally consistent. This register is the output of a deliberate cross-rule conflict hunt across all 248 rules — not an assertion that "everything is fine." Each entry is a genuine tension, overlap, or ambiguity that, left unresolved, would cause two developers (or a developer and a QA engineer) to implement contradictory behavior. Each is resolved with a **binding ruling** that the affected modules must follow.

Conflicts fall into three classes:
- **Genuine conflict** — two rules would produce contradictory behavior; one must yield.
- **Layering/boundary ambiguity** — two rules describe the same mechanism at different levels; the boundary must be fixed so it is implemented once.
- **Open policy decision** — the catalog deliberately left a choice to the institution; this register records the decision needed and the recommended default.

---

## C-01 — Admission capacity buffer vs hard section cap  *(Genuine conflict)*

**Rules in tension:** `ADM-004` (capacity counts confirmed enrollments, with a configurable buffer for expected declines) vs `SEC-003` / `ENR-003` (section capacity is the hard operative limit on *active enrollments*).

**The conflict:** ADM-004 permits approving *more* applicants than seats (e.g., approve 44 for a 40-seat section, expecting declines). But SEC-003 makes 40 a hard cap on active enrollments. If only 2 of the expected 4 decline, 42 confirmed students collide with a 40-seat hard cap. Which rule wins at the moment of enrollment?

**Resolution (binding):** The buffer applies **only at the offer/approval stage** (ADM-004). The hard cap (SEC-003/ENR-003) is enforced **at enrollment conversion** (ADM-007). When confirmed conversions would exceed section capacity, the excess does **not** enroll into that section: they are placed on the waitlist (ADM-005) or routed to another section per policy. The buffer is an admissions optimization, never an override of the enrollment leaf's capacity.

**Module actions:** ADM-004 reworded to state the buffer is offer-stage only; ENR-003/SEC-003 remain the authoritative hard cap at conversion; ADM-007 conversion explicitly re-checks SEC-003 at commit.

---

## C-02 — Can re-evaluation lower a published result?  *(Open policy decision)*

**Rules in tension:** `RES-006` (governed post-publish revision can change a result) and the recheck flow (`UC-RES-03`) vs student expectation/fairness. The catalog explicitly left open whether a re-evaluation may *decrease* a result.

**The decision needed:** When a student requests re-evaluation, may the outcome be a *lower* result than originally published, or only equal-or-higher?

**Resolution (binding default + disclosure requirement):** The institution must choose one policy and **disclose it to students before they request re-evaluation**. The recommended default is **"re-evaluation may move the result in either direction"** (it is a true re-mark, not an appeal for more marks), because a one-directional policy incentivizes frivolous requests and can entrench errors. Whichever is chosen, it is configured once (Configuration Engine), version-stamped, shown on the recheck request screen, and applied uniformly. RES-006 revisions remain fully audited and the student is always notified of any change (up or down).

**Module actions:** Add a configured, disclosed `recheck.direction_policy` setting; RES module references it; the chosen value is surfaced at request time.

---

## C-03 — "Waiver" defined in two places  *(Layering/boundary ambiguity)*

**Rules in tension:** `DSC-006` models *waiver* as a discount sub-type, while `FEE` (statutory/non-waivable heads), `RES-007` (fee-based result withholding), and `ENR-006` (withdrawal clearance) all reference waiving charges.

**The ambiguity:** If waiver logic lives in multiple modules, the rules for *who can waive what, with what approval* could drift apart.

**Resolution (binding):** **Doc 19 (Discount), rule DSC-006, is the single source of truth for the waiver concept** — its approval authority, SoD, caps, and the non-waivable-charge rule. FEE, RES, and ENR *consume* the waiver outcome (a charge is reduced/forgiven) but do not define waiver mechanics. FEE-002 contributes only the *classification* of which heads are statutory/non-waivable, which DSC-006 reads.

**Module actions:** No rule change; documented here that DSC-006 owns waiver semantics. Implementation places waiver logic in one service consumed by finance/result/enrollment.

---

## C-04 — Student leave owned by Attendance or Leave?  *(Layering/boundary ambiguity)*

**Rules in tension:** `ATT-005` (approved leave reflects as excused in attendance) and `LEV-009` (student leave is a distinct, balance-free request/approval sub-domain).

**The ambiguity:** Both rules touch student leave; without a fixed boundary, the request/approval workflow could be built twice or inconsistently.

**Resolution (binding):** **LEV-009 owns the student-leave request and approval lifecycle** (who requests, who approves, the workflow). **ATT-005 owns only the attendance consequence** — when an approved student leave exists, attendance marks those dates excused. Attendance never approves leave; Leave never writes attendance directly beyond signaling the approved period. The integration is one-directional: Leave (decision) → Attendance (reflection).

**Module actions:** Boundary documented; ATT-005 reworded to "consumes approved student-leave"; LEV-009 is the authoritative workflow.

---

## C-05 — Ownership after teacher reassignment  *(Layering/boundary ambiguity)*

**Rules in tension:** `AUTHZ-003` edge case (whether a reassigned teacher retains *read* access to historical data is "configurable policy, default no live access") vs `TCH-004` (assignment confers ownership; ending it ends live ownership immediately, historical actions remain attributed).

**The ambiguity:** "Live ownership ends immediately" vs "historical read access is configurable" could read as contradictory.

**Resolution (binding):** Two distinct things, both true: **(a) live operational ownership** (ability to mark attendance, enter/edit marks for the section/subject) **always ends immediately** on unassignment — never configurable. **(b) historical read access** (viewing past data the teacher themselves created) is the configurable part, default **no**. Authorship attribution of past actions is permanent and independent of either.

**Module actions:** AUTHZ-003 and TCH-004 cross-reference this split; the configurable flag governs only historical *read*, never live ownership.

---

## C-06 — "Immutable-after-use" lock defined as engine + instances  *(Layering/boundary ambiguity)*

**Rules in tension:** `CFG-007` (engine-level immutable-after-use flag) vs domain instances that each assert a lock: `INST-004` (institute type), `STU-001` / `STF-001` / `SCH`/ID schemes (immutable IDs), `FEE-001` (issued-invoice immutability is a related but distinct case).

**The ambiguity:** Each domain rule could re-implement the "block change if dependents exist" check, risking inconsistent behavior.

**Resolution (binding):** **CFG-007 is the single enforcement mechanism.** Domain rules (INST-004, ID schemes, etc.) are *declarations* that a given setting carries the `IMMUTABLE_AFTER_USE` flag; the Configuration Engine performs the dependency check and blocks the change uniformly. Domain modules must **not** hand-roll their own lock logic. (Note: issued-invoice immutability FEE-003 is a *record*-immutability case, not a *config*-immutability case — it stays in Finance; the two are deliberately separate mechanisms.)

**Module actions:** Implement the lock once in the engine; domain rules set flags, not bespoke checks.

---

## C-07 — Inconsistent "governed correction" patterns across modules  *(Genuine conflict — consistency)*

**Rules in tension:** Post-finalization corrections are governed in five places with slightly different language: `SESS-005`/`SESS-008` (closed-session reopen), `EXM-007` (post-lock mark correction), `RES-006` (post-publish result revision), `FEE-003` (issued-invoice credit note), `HR-003`/`HR-006` (locked-payslip adjustment), `ATT-007` (post-lock attendance correction).

**The conflict:** Each defines its own correction governance; without a unifying pattern, six subtly different correction flows emerge, confusing implementers and QA.

**Resolution (binding — unified Governed Correction Pattern):** All post-finalization corrections MUST follow one canonical pattern:
1. The original record is **never edited or deleted** (immutability preserved).
2. The correction is a **new, linked entry** (credit note / revision / reopen-scoped change / correcting attendance entry) carrying the **original value, the new value, a recorded reason, and the actor**.
3. It requires **elevated permission** and, where the source rule says so, **step-up MFA** and/or **approval (SoD)**.
4. It **cascades** to dependents (e.g., a mark correction recomputes the result; a result revision regenerates the marksheet).
5. The affected party is **notified** when a published/issued artifact changes.
6. Closed-period changes additionally require the period's **governed reopen** (SESS-008).

Each of the six rules above is a domain-specific application of this single pattern. QA tests the pattern once and the per-module specializations against it.

**Module actions:** This pattern is added to the repository README as a cross-cutting principle; the six rules reference it.

---

## C-08 — Erasure vs statutory retention vs legal hold  *(Genuine conflict — precedence)*

**Rules in tension:** `STU-007` / `STF-008` (archive then anonymize on lawful erasure) vs `FEE`/`PAY`/`HR` (financial/payroll records exempt from erasure during legal retention) vs `AUD-008` (audit pseudonymized, retained) vs every module's "legal hold overrides erasure."

**The conflict:** A single erasure request touches data with three different retention obligations. Without a precedence order, modules could erase records others are legally required to keep, or refuse erasure they should honor.

**Resolution (binding — Retention Precedence Order):** When obligations collide, apply in strict order:
1. **Legal hold** (active litigation/investigation) — suspends *all* erasure and purge until lifted. Highest.
2. **Statutory retention** (financial, payroll, academic transcripts, audit) — these records are **retained, not erased**, until their legal minimum elapses; where they contain personal identifiers, those are minimized/pseudonymized (AUD-008 for audit) but the record persists.
3. **Erasure request** — honored for all data **not** held by (1) or (2): operational PII, profile data, contact details, media, are anonymized/securely deleted (STU-007/STF-008/FILE-008).

So a lawful erasure of a former student **does** remove their profile, photos, and contact data, but **does not** remove their transcript, fee ledger, or audit trail until those reach their statutory limits; audit references are pseudonymized meanwhile. This precedence is implemented centrally and consumed by every module's anonymization path.

**Module actions:** Precedence order added as a cross-cutting principle; each module's Data Retention section conforms to it.

---

## C-09 — `GRD` vs `GRD-N` prefix proximity  *(Readability risk, not behavioral)*

**Rules in tension:** Grading uses prefix `GRD` (e.g., `GRD-003`); Guardian uses `GRD-N` (e.g., `GRD-N-004`). A reader could misread `GRD-N-004` as a Grading rule.

**Resolution (binding naming clarification):** The IDs are unambiguous to the parser (the `-N` segment distinguishes them) and are retained to avoid renumbering already-referenced rules. **Documentation convention:** always write Guardian IDs with the full `GRD-N-NNN` form; never abbreviate to `GRD-NNN` for guardian rules. Tooling treats `GRD-N` as an atomic prefix. No renumbering.

**Module actions:** Convention recorded; index and catalog render the full form.

---

## C-10 — Mandatory notifications vs consent/opt-out  *(Resolved by design, recorded for clarity)*

**Rules in tension:** `NOT-003` (mandatory categories override opt-out) and `GRD-N-008` (parental consent governs processing) and general preference/consent handling.

**The apparent conflict:** Does forcing a "mandatory" security/safeguarding/financial notification violate a withdrawn consent?

**Resolution (binding):** Consent (GRD-N-008) governs **optional/marketing-style** processing and notifications. **Transactional, security, safeguarding, and legal notifications are processed under a lawful basis other than consent** (legitimate interest / legal obligation / contract) and therefore are **not** subject to opt-out (NOT-003). Custody/contact restrictions (GRD-N-006) still apply — a *barred* recipient is excluded even from mandatory categories; the notice routes to the permitted guardian instead. No conflict: consent and mandatory-category delivery operate on different lawful bases.

**Module actions:** Lawful-basis distinction documented; NOT-003 and GRD-N-008 cross-reference it.

---

## Summary of rulings

| ID | Type | One-line resolution |
|---|---|---|
| C-01 | Genuine conflict | Capacity buffer is offer-stage only; section hard cap wins at enrollment. |
| C-02 | Open decision | Institution chooses + discloses recheck direction; default = either direction. |
| C-03 | Boundary | DSC-006 owns waiver semantics; FEE/RES/ENR consume the outcome. |
| C-04 | Boundary | LEV-009 owns student-leave workflow; ATT-005 only reflects it in attendance. |
| C-05 | Boundary | Live ownership ends immediately (non-configurable); only historical *read* is configurable. |
| C-06 | Boundary | CFG-007 is the one lock mechanism; domain rules set flags, not bespoke checks. |
| C-07 | Consistency | One Governed Correction Pattern; six modules are its specializations. |
| C-08 | Precedence | Legal hold > statutory retention > erasure request, applied centrally. |
| C-09 | Naming | Guardian IDs always written `GRD-N-NNN`; no renumbering. |
| C-10 | Lawful basis | Mandatory notices ride a non-consent lawful basis; consent governs optional only. |

**No unresolved genuine conflicts remain.** C-02 is the one item requiring an institutional decision before go-live; all others are resolved as binding rulings or fixed boundaries.
