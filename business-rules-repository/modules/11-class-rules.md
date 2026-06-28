# 11 — Class Business Rules

## 1. Module Purpose
Govern the **class** (also called grade / standard / year / level — the term is relabel-able per institute, D28) — a level in the academic structure that students progress through. A class is part of the timeless **structure definition** and is realized as a per-session **instance** (Doc 05/SESS-004). It groups one or more **sections** (the enrollment leaf, Doc 12) and is mapped to **subjects** via curriculum (Doc 13). The class also defines the **promotion path** (which class a student advances to) and may carry **streams** (Science/Commerce/Arts) at higher levels.

## 2. Actors
- **Institute Administrator** — defines classes, ordering, promotion path, and curriculum mapping.
- **Campus Administrator** — manages class instances/sections at their campus.
- **System** — enforces structure validity, ordering, instance generation, and promotion-path integrity.

## 3. Use Cases

**Use Case ID:** UC-CLS-01 — Define a Class in the Structure
**Actors:** Institute Administrator, System
**Description:** Add a class (level) to the institute's academic structure definition.
**Preconditions:** Institute `ACTIVE`; actor holds `academic.structure.manage`.
**Main Flow:** 1) Admin defines the class (name/label, sequence/order, optional streams, applicable campuses). 2) Maps subjects via curriculum (Doc 13). 3) Sets the promotion target (next class) or marks it a final/terminal level. 4) Class enters the definition, ready for instantiation per session.
**Alternative Flow:** A1) Class applies to multiple campuses or is campus-specific (CLS-004).
**Exception Flow:** E1) Duplicate sequence/label within the institute → reject. E2) Circular promotion path → reject (CLS-005).
**Post Conditions:** Class defined; instantiable per session; audited.
**Business Rules Applied:** CLS-001, CLS-002, CLS-005.

**Use Case ID:** UC-CLS-02 — Instantiate Classes for a Session
**Actors:** Administrator, System
**Description:** Realize the class definition as concrete instances for a session.
**Preconditions:** Session in setup (Doc 05); structure defined.
**Main Flow:** 1) Session instantiation creates a class instance from each applicable class definition. 2) Sections are created under each class instance (Doc 12). 3) Instances are session-pinned and historically faithful.
**Exception Flow:** E1) Definition incomplete (no subjects/sections) → block instantiation per policy.
**Post Conditions:** Session class instances exist; enrollment can target their sections.
**Business Rules Applied:** CLS-003, SESS-004.

**Use Case ID:** UC-CLS-03 — Reorder / Restructure Classes
**Actors:** Institute Administrator, System
**Description:** Change class ordering, streams, or promotion path in the definition.
**Preconditions:** Actor authorized; changes affect future sessions, not past instances.
**Main Flow:** 1) Admin edits sequence/streams/promotion target in the definition. 2) System validates no cycles and a coherent path. 3) Change applies to future instantiation; existing session instances are untouched.
**Exception Flow:** E1) Change would orphan a promotion path (a class promotes to a removed class) → block.
**Post Conditions:** Definition updated for future sessions; audited; history intact.
**Business Rules Applied:** CLS-005, CLS-006.

## 4. Business Rules

**Rule ID:** CLS-001
**Rule Name:** Class Is a Structure Definition Element, Instantiated per Session
**Description:** A class lives in the timeless structure definition and is instantiated per session; definition edits never rewrite past instances.
**Priority:** Critical
**Category:** Definition/instance integrity
**Preconditions:** Class defined or edited.
**Business Rule:** The class definition describes the level; each session creates an instance. Renaming or restructuring a class affects future instantiation only; closed sessions keep their original class instances and records (SESS-004/SESS-005).
**System Action:** Maintain definition; generate session instances referencing it without copying.
**Validation:** Definition complete enough to instantiate (label, sequence).
**Failure Behavior:** Block instantiation of an incomplete definition.
**Audit Requirement:** Log `CLASS_DEFINED/EDITED` and `CLASS_INSTANTIATED` (per session).
**Example Scenario:** Renaming "Class 5" to "Grade 5" for 2027 leaves 2026 records as "Class 5".
**Related Rules:** SESS-004, CLS-003.

**Rule ID:** CLS-002
**Rule Name:** Configurable Label & Ordering
**Description:** The class term and its sequence are configurable per institute and drive progression.
**Priority:** High
**Category:** Configuration
**Preconditions:** Class definition.
**Business Rule:** The class label (Class/Grade/Standard/Year/Level/Semester) is terminology-resolved (D28); each class has a unique sequence number within the institute that defines progression order and default promotion direction.
**System Action:** Resolve label via terminology; enforce unique ordering.
**Validation:** Unique sequence within institute; label key valid.
**Failure Behavior:** Reject duplicate sequence.
**Audit Requirement:** Captured in `CLASS_DEFINED`.
**Example Scenario:** A madrasa labels levels differently from a school, both ordered 1..N.
**Related Rules:** CLS-005, INST-002.

**Rule ID:** CLS-003
**Rule Name:** A Class Contains ≥1 Section (Default Section Guarantee)
**Description:** Every active class instance has at least one section (the enrollment leaf); a default section is auto-created if the institute doesn't subdivide.
**Priority:** High
**Category:** Structure invariant
**Preconditions:** Class instance activation; enrollment requires a section.
**Business Rule:** Students enroll into a section, not directly into a class. A class with no explicit sections gets an auto default section so single-section institutions work transparently.
**System Action:** Auto-create a default section when none defined; ensure ≥1 section per active class instance.
**Validation:** Active class instance has ≥1 active section.
**Failure Behavior:** Block enrollment to a class lacking a section; auto-provision default.
**Audit Requirement:** Log `DEFAULT_SECTION_CREATED`.
**Example Scenario:** A small school's "Class 3" has one default section; students attach there.
**Related Rules:** SEC-001, ENR-001.

**Rule ID:** CLS-004
**Rule Name:** Class Campus Applicability
**Description:** A class definition may apply to all campuses or be campus-specific.
**Priority:** Medium
**Category:** Scope
**Preconditions:** Class definition with campus applicability.
**Business Rule:** A class can be offered at all campuses of an institute or restricted to specific campuses; instantiation creates instances only at applicable campuses.
**System Action:** Instantiate per campus applicability.
**Validation:** Applicable campuses exist and are active.
**Failure Behavior:** Skip instantiation at non-applicable campuses; flag if none applicable.
**Audit Requirement:** Captured in `CLASS_INSTANTIATED` per campus.
**Example Scenario:** A specialized "Pre-University" class exists only at the main campus.
**Related Rules:** CAMP-001, CLS-003.

**Rule ID:** CLS-005
**Rule Name:** Acyclic Promotion Path
**Description:** The promotion path between classes must be acyclic and coherent (or terminal at the final level).
**Priority:** Critical
**Category:** Progression integrity
**Preconditions:** Promotion target defined/edited.
**Business Rule:** Each class either promotes to exactly one next class (by sequence/explicit target) or is a terminal (graduating) level; cycles are forbidden so rollover (SESS-006/ENR-007) is deterministic.
**System Action:** Validate the promotion graph is acyclic; mark terminals.
**Validation:** No cycle; every non-terminal has a valid next class.
**Failure Behavior:** Reject cyclic or dangling promotion paths.
**Audit Requirement:** Log `PROMOTION_PATH_CHANGED`.
**Example Scenario:** Class 10 promotes to Class 11; the final class is terminal (graduation).
**Related Rules:** SESS-006, ENR-007.

**Rule ID:** CLS-006
**Rule Name:** Streams as Configurable Sub-Grouping
**Description:** Classes may carry streams (e.g., Science/Commerce/Arts) realized as stream-typed sections or an intermediate grouping.
**Priority:** Medium
**Category:** Structure flexibility
**Preconditions:** A class uses streams (typically higher secondary).
**Business Rule:** Streams are a configurable attribute; a streamed class's sections belong to streams, and subject mapping/promotion can be stream-aware. Non-streamed classes ignore the concept.
**System Action:** Support stream tagging on sections and stream-aware curriculum/promotion.
**Validation:** Stream definitions valid; sections assigned to valid streams.
**Failure Behavior:** Reject sections referencing undefined streams.
**Audit Requirement:** Log stream configuration changes.
**Example Scenario:** "Class 11" has Science and Commerce streams with different subjects.
**Related Rules:** SEC-001, SUB-005.

**Rule ID:** CLS-007
**Rule Name:** Aggregate Class Capacity (Optional)
**Description:** A class may carry an aggregate capacity across its sections, informing admissions/enrollment.
**Priority:** Low
**Category:** Capacity
**Preconditions:** Capacity configured at class level.
**Business Rule:** Where set, total active enrollments across a class's sections cannot exceed the class capacity; section capacity (SEC) remains the operative per-section limit.
**System Action:** Sum section enrollments; enforce class ceiling if configured.
**Validation:** Class capacity ≥ sum of section capacities or treated as an independent ceiling per policy.
**Failure Behavior:** Block enrollment beyond class capacity.
**Audit Requirement:** Log capacity blocks at class level.
**Example Scenario:** "Class 1" capped at 120 across three 40-seat sections.
**Related Rules:** SEC-003, ADM-004, ENR-003.

## 5. Validation Rules
- Class sequence unique within institute; label key valid (terminology).
- Active class instance has ≥1 active section.
- Promotion path acyclic; terminals marked.
- Campus applicability references active campuses.
- Streams reference valid stream definitions.

## 6. State Machine

**State Name:** DRAFT (definition)
**Description:** Class being defined; not yet instantiable.
**Allowed Transitions:** → ACTIVE_DEFINITION (complete); → ARCHIVED_DEFINITION (abandoned).
**Forbidden Transitions:** instantiation while incomplete.
**System Actions:** Validate ordering/promotion before activation.

**State Name:** ACTIVE_DEFINITION
**Description:** Part of the live structure; instantiated each session.
**Allowed Transitions:** → DEPRECATED_DEFINITION (retired from future sessions).
**Forbidden Transitions:** edits that rewrite existing instances.
**System Actions:** Generate session instances; apply future edits to future instances only.

**State Name:** SESSION_INSTANCE: ACTIVE
**Description:** A class instance live in a session.
**Allowed Transitions:** → COMPLETED (session closes); promotion processed at rollover.
**Forbidden Transitions:** → DRAFT.
**System Actions:** Hold sections/enrollments for the session.

**State Name:** SESSION_INSTANCE: COMPLETED / ARCHIVED
**Description:** Class instance for a closed/archived session.
**Allowed Transitions:** COMPLETED → ARCHIVED.
**Forbidden Transitions:** edits (immutable history).
**System Actions:** Read-only retention.

## 7. Status Definitions
Definition: `DRAFT` · `ACTIVE_DEFINITION` · `DEPRECATED_DEFINITION` · `ARCHIVED_DEFINITION`. Session instance: `ACTIVE` · `COMPLETED` · `ARCHIVED`.

## 8. Workflow Rules
- Class definition/edit is a direct admin action, audited; structural changes apply to future sessions only.
- Promotion-path changes are validated (acyclic) before save.
- Instantiation runs as part of session setup (Doc 05).

## 9. Permission Rules
- `academic.structure.manage` — define/edit classes, ordering, streams, promotion path.
- `academic.structure.view` — read the structure (most academic roles).
- Scoped to institute; campus admins manage instances within their campus.

## 10. Notification Rules
- Structural changes (new class, promotion-path change) → notify institute academic admins.
- Class instantiation completion → notify admins as part of session-setup notifications.

## 11. Audit Requirements
Mandatory: `CLASS_DEFINED/EDITED`, `CLASS_INSTANTIATED` (per session/campus), `PROMOTION_PATH_CHANGED`, stream/capacity changes, `DEFAULT_SECTION_CREATED`. With actor, institute, class, before/after.

## 12. Data Retention Rules
- Class definitions retained as configuration history (structure on any past date is reconstructable).
- Class instances retained with their session (long-term academic record).
- Definition versions retained so historical instances resolve to the definition active when created.

## 13. Edge Cases
- **Final-level class:** terminal; students graduate rather than promote (CLS-005) — must never auto-promote into a non-existent next class.
- **Class with no sections:** auto default section (CLS-003); single-section schools never see the concept.
- **Streamed classes:** subject sets and promotion differ per stream (CLS-006); a stream change for a student is effectively a section transfer.
- **Reordering classes mid-structure:** affects future sessions only; past instances keep their order/records.
- **Campus-specific class:** instantiated only where applicable; reporting must not assume every class exists at every campus.
- **Repeating/retained student:** re-enters the same class's next-session instance (a new instance), not the old one (ENR-007).
- **Multi-year classes / semesters:** a "class" may map to a term or semester depending on type; terminology and ordering handle this.

## 14. Failure Scenarios
- **Cyclic promotion path:** rejected at definition time (CLS-005).
- **Instantiation with incomplete definition:** blocked; session setup flags the gap.
- **Class capacity below committed enrollments:** lowering capacity below current enrollment is blocked or flagged, never silently dropping students.
- **Orphaned promotion target:** removing a class that others promote to is blocked until paths are fixed.

## 15. Exception Handling Rules
- Structural edits are validated (ordering, acyclic path) before commit.
- Definition changes never mutate existing session instances.
- Forbidden operations (cyclic path, removing a promotion target in use) are rejected with explicit messaging.

## 16. Compliance Considerations
- **Accurate progression record:** the promotion path and per-session instances underpin truthful transcripts and progression history.
- **Structure history:** retained definition versions answer "what was the structure in year X" for audits/accreditation.

## 17. Future Considerations
- Credit-based / non-linear progression (university modules without a strict next-class).
- Elective-driven class composition.
- Stream-change workflows with subject reconciliation.
- Multi-track classes (regular + vocational) within one level.
