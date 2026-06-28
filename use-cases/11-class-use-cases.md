# 11 — Class Use Cases

Transforms the Class business rules (`CLS-001`…`CLS-007`) into use cases. A class is a structure-definition element (grade/level) instantiated per session; it contains ≥1 section and defines the promotion path.

## 1. Primary Actors
Academic Administrator (defines the class structure), Institute Administrator (oversees structure per institute).

## 2. Secondary Actors
System (definition/instance separation, default-section guarantee, acyclic promotion validation), Section module (children), Session module (instantiation), Configuration Engine (labels/streams), Audit service.

## 3. Goals
Model the academic ladder as reusable definitions; instantiate per session without rewriting history; guarantee every class has a section; configure labels, ordering, streams; keep the promotion path acyclic; optionally cap aggregate class capacity.

## 4. User Journeys
- **Define the ladder:** academic admin defines classes (label, ordering, campus applicability, promotion target) → each gets a default section → the structure is ready to instantiate.
- **Instantiate:** session creation generates class instances from the definition (UC-SESS-002); editing the definition never alters past instances.
- **Refine:** add streams (science/commerce) as configurable sub-groupings; set optional aggregate capacity; adjust ordering/labels.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core/CRUD | UC-CLS-001 | Define Class (structure definition) | High |
| CRUD | UC-CLS-002 | View Class(es) | Medium |
| CRUD | UC-CLS-003 | Update Class Definition | Medium |
| CRUD | UC-CLS-004 | Retire / Archive Class | Medium |
| Core | UC-CLS-005 | Configure Promotion Path (acyclic) | High |
| Core | UC-CLS-006 | Manage Streams (sub-grouping) | Medium |
| Admin | UC-CLS-007 | Set Aggregate Class Capacity | Low |
| Admin | UC-CLS-008 | Configure Label & Ordering | Low |
| Search | UC-CLS-009 | Search Classes | Low |
| Reporting | UC-CLS-010 | Class Structure & Capacity Report | Medium |
| Bulk/Import | UC-CLS-011 | Import / Bulk-Define Class Structure | Medium |
| Export | UC-CLS-012 | Export Class Structure | Low |
| Workflow | UC-CLS-013 | Structure-Change Approval | Low |
| Exception | UC-CLS-014 | Cyclic Promotion Path Blocked | High |
| Exception | UC-CLS-015 | Class Without Section Blocked | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-CLS-001 — Define Class (structure definition)
- **Module:** Class · **Priority:** High
- **Actors:** Academic Administrator (primary), System
- **Goal:** Define a reusable class/level with a guaranteed default section.
- **Description:** Creates a class definition (label, ordering, campus applicability) that will be instantiated per session; the system guarantees at least one section exists.
- **Business Rules Applied:** CLS-001, CLS-002, CLS-003, CLS-004.
- **Preconditions:** Institute active; actor holds `academic.structure.manage`.
- **Trigger:** Admin defines a new class.
- **Main Success Scenario:**
  1. Admin enters label, ordering position, and campus applicability.
  2. System creates the class as a definition element (not session-bound).
  3. System guarantees a default section (CLS-003), creating one if none specified.
  4. The class is available for per-session instantiation.
- **Alternative Flows:** A1) Multiple sections defined upfront. A2) Streams attached (UC-CLS-006).
- **Exception Flows:** E1) Creating a class with zero sections and blocking the default → blocked (UC-CLS-015). E2) Duplicate label/ordering conflict → reject/resolve.
- **Validation Rules:** Label/ordering valid; ≥1 section guaranteed; campus applicability valid (CLS-001/002/003/004).
- **Permissions Required:** `academic.structure.manage`.
- **Notifications Triggered:** None routine.
- **Audit Events Generated:** `CLASS_DEFINED`, `DEFAULT_SECTION_CREATED`.
- **Data Created:** Class definition; default section.
- **Data Updated:** None.
- **Data Deleted:** None.
- **Post Conditions:** Class definition exists with ≥1 section, ready to instantiate.
- **Related Use Cases:** UC-CLS-005, UC-SEC-001, UC-SESS-002.
- **Acceptance Criteria:**
  - Given valid inputs, When a class is defined, Then it exists as a definition with at least one section.
  - Given a class created with no section, When saved, Then a default section is auto-created.
- **Edge Case Analysis:**
  - *Invalid Input:* invalid ordering/label rejected.
  - *Permission Failure:* lacks `academic.structure.manage` → 403.
  - *Concurrent Update:* duplicate label created twice → uniqueness/ordering resolved.
  - *Duplicate Data:* duplicate class label in scope flagged.
  - *System Failure:* atomic class+default-section creation.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* define with explicit sections; auto default section.
  - *Negative:* zero-section block; duplicate label.
  - *Boundary:* first/last ordering position.

### UC-CLS-005 — Configure Promotion Path (acyclic)
- **Module:** Class · **Priority:** High
- **Actors:** Academic Administrator (primary), System
- **Goal:** Define which class a level promotes into, with no cycles.
- **Description:** Sets each class's promotion target; the system validates the overall path is acyclic so rollover can advance students deterministically and terminal levels graduate.
- **Business Rules Applied:** CLS-005, SESS-006, ENR-007.
- **Preconditions:** Classes defined; actor authorized.
- **Trigger:** Admin sets/edits a promotion target.
- **Main Success Scenario:**
  1. Admin sets class X's promotion target to class Y (or marks X terminal/graduating).
  2. System validates the resulting graph is acyclic (CLS-005).
  3. The path is saved and used by rollover (UC-SESS-004).
- **Alternative Flows:** A1) Stream-specific promotion targets.
- **Exception Flows:** E1) A target that would create a cycle → blocked (UC-CLS-014).
- **Validation Rules:** Promotion graph acyclic; terminal levels marked graduating (CLS-005).
- **Permissions Required:** `academic.structure.manage`.
- **Notifications Triggered:** None.
- **Audit Events Generated:** `PROMOTION_PATH_UPDATED`.
- **Data Created:** None.
- **Data Updated:** Class promotion targets.
- **Data Deleted:** None.
- **Post Conditions:** Deterministic, acyclic promotion path drives rollover.
- **Related Use Cases:** UC-SESS-004, UC-ENR-006, UC-CLS-014.
- **Acceptance Criteria:**
  - Given promotion targets forming no cycle, When saved, Then the path is accepted.
  - Given a target that would create a cycle, When set, Then it is blocked.
  - Given a terminal class, When marked graduating, Then rollover graduates its students rather than promoting.
- **Edge Case Analysis:**
  - *Invalid Input:* target to a non-existent class rejected.
  - *Permission Failure:* unauthorized → 403.
  - *Concurrent Update:* two edits forming a cycle → second validated against the first, blocked.
  - *Duplicate Data:* N/A.
  - *System Failure:* validation atomic; partial path not saved.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* linear ladder; branch/merge acyclic paths; terminal graduation.
  - *Negative:* direct cycle (X→Y→X); self-loop; target missing.
  - *Boundary:* single-class institute; longest chain.

---

## 7. Compact Specifications (routine use cases)

- **UC-CLS-002 — View Class(es)** · *Medium* · Rules: AUTHZ-002. Scoped read of class structure. *QA:* scope respected.
- **UC-CLS-003 — Update Class Definition** · *Medium* · Rules: CLS-001/002. Edit label/ordering/applicability; edits don't alter past session instances. *Audit:* `CLASS_UPDATED`. *Edge:* historical instances unchanged. *QA:* edit applies forward; history fidelity.
- **UC-CLS-004 — Retire / Archive Class** · *Medium* · Rules: CLS-001, SUB-007-style retention. Retire from future instantiation; history preserved. *Edge:* cannot delete with historical instances. *QA:* retire blocks future use; history kept.
- **UC-CLS-006 — Manage Streams (sub-grouping)** · *Medium* · Rules: CLS-006. Define streams (science/commerce) as configurable sub-groups. *Edge:* stream-specific subjects/promotion. *QA:* stream config; subject mapping per stream.
- **UC-CLS-007 — Set Aggregate Class Capacity** · *Low* · Rules: CLS-007, SEC-003. Optional class-level cap above section caps. *Edge:* section hard caps still the operative leaf (C-01). *QA:* aggregate cap advisory vs section hard cap.
- **UC-CLS-008 — Configure Label & Ordering** · *Low* · Rules: CLS-002. Localize labels (Grade/Class/Year), set order. *QA:* label/locale; ordering.
- **UC-CLS-009 — Search Classes** · *Low* · Rules: AUTHZ-002. Scoped search. *QA:* scope respected.
- **UC-CLS-010 — Class Structure & Capacity Report** · *Medium* · Rules: REP-002, CLS-007, SEC-003. Structure and fill report. *Edge:* counts consistent with section caps. *QA:* structure accuracy; capacity rollup.
- **UC-CLS-011 — Import / Bulk-Define Class Structure** · *Medium* · Rules: CLS-001/003/005. Import ladder incl. promotion path (acyclic-validated). *Edge:* cyclic import rejected; default sections ensured. *QA:* clean import; cycle rejected.
- **UC-CLS-012 — Export Class Structure** · *Low* · Rules: REP-005. Export structure definition. *QA:* scoped export.
- **UC-CLS-013 — Structure-Change Approval** · *Low* · Rules: WFL-004. Optional approval for structural changes. *QA:* approval gates; SoD.
- **UC-CLS-014 — Cyclic Promotion Path Blocked (Exception)** · *High* · Rules: CLS-005. Cycle-creating target rejected. *QA:* cycle blocked; acyclic allowed.
- **UC-CLS-015 — Class Without Section Blocked (Exception)** · *High* · Rules: CLS-003. A class must always have ≥1 section. *QA:* zero-section blocked; default ensured.

## 8. Module-level QA & Edge Themes
- **Definition vs instance (CLS-001):** editing a class definition must never rewrite historical session instances — a key fidelity suite.
- **Default-section guarantee (CLS-003):** every class always has ≥1 section.
- **Acyclic promotion (CLS-005):** cycle detection is the headline negative suite; terminal levels graduate.
