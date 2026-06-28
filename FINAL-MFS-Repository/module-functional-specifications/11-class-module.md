# 11 — Class Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`CLS-001…007`), and Use Case Repository (`UC-CLS-001…015`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Define the class/grade level as a structure-definition element instantiated per session: configurable label/ordering, guaranteed ≥1 section, campus applicability, acyclic promotion path, configurable streams, and optional aggregate capacity.

**Business Goal.** Provide a clean, reusable academic-structure backbone that sessions instantiate, promotion paths follow, and sections/subjects attach to — adaptable to any institution type without code change.

**Scope.** Class definition (label, ordering, campus applicability); ≥1 section guarantee (default section); acyclic promotion path; streams (sub-grouping); optional aggregate class capacity; retirement/archival preserving history.

**Out of Scope.** Per-session instancing mechanics (Session module). Section internals/capacity-as-enrollment-limit (Section module). Subject mapping (Subject module). Enrollment (Enrollment module).

---

## 2. Actors

**Primary Actors.** Academic Administrator (defines class structure), Institute Administrator (campus applicability), System (default-section guarantee, acyclicity checks).

**Secondary Actors.** Session module (instances classes), Section module (children), Subject module (curriculum mapping), Enrollment/Result (consumers), Workflow Engine (structure-change approval), Configuration Engine (labels/streams), Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Define class | Define a class as a structure element (instantiated per session). | Critical | Session (instancing) |
| FR-002 | Configurable label & ordering | Configure display label and sequence. | High | Configuration Engine |
| FR-003 | ≥1 section guarantee | A class always contains at least one (default) section. | High | Section module |
| FR-004 | Campus applicability | Set which campuses a class applies to. | High | Campus |
| FR-005 | Acyclic promotion path | Define next-class promotion path with no cycles. | High | FR-001 |
| FR-006 | Streams (sub-grouping) | Configure streams (e.g., Science/Commerce) as sub-groupings. | Medium | Configuration Engine |
| FR-007 | Aggregate class capacity (optional) | Optionally cap total across sections. | Medium | Section capacity |
| FR-008 | Retire/archive (history-preserving) | Retire a class without losing historical instances. | High | Retention/Audit |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Structure definition | Reusable class backbone. | Define once, instance per session. |
| Label & ordering | Configurable naming/sequence. | Fits any institution's nomenclature. |
| Default-section guarantee | Always ≥1 section. | No empty class shells. |
| Promotion path | Acyclic next-class mapping. | Correct, safe progression. |
| Streams | Sub-grouping support. | Higher-secondary/university structures. |
| Aggregate capacity | Optional total cap. | Planning control. |
| History-preserving retirement | Retire without data loss. | Integrity across years. |

---

## 5. Screens

Class List; Class Detail; Define Class; Edit Class; Promotion Path Configuration; Stream Management; Aggregate Capacity; Label & Ordering; Retire/Archive Class; Class Structure & Capacity Report; Import/Bulk-Define Structure; Export Structure; Structure-Change Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Class List | Define, Open, Reorder, Search/Filter, Export | Bulk Define/Import |
| Class Detail | Edit, Configure Promotion, Manage Streams, Set Capacity, Retire | — |
| Promotion Path | Set Next Class, Validate Acyclic, Save (→ approval) | — |
| Stream Management | Add/Edit/Remove Stream | — |
| Label & Ordering | Edit Label, Drag-Reorder, Save | Bulk Reorder |
| Retire/Archive | Confirm with Reason | — |
| Structure Report | Run, Filter, Export | Export |

---

## 7. Forms

**Define Class** — `label` (text, required), `code` (text, optional), `ordering` (number, required), `campusApplicability` (multi-select, required), `streams` (multi, optional), `aggregateCapacity` (number, optional). Validation: ordering unique within scope (CLS-002); applies to ≥1 campus (CLS-004); creating a class provisions a default section (CLS-003).

**Promotion Path** — `nextClass` (select, optional for terminal), graduate/exit flag. Validation: no cycle in the promotion graph (CLS-005 / UC-CLS-014).

**Stream** — `name` (text, required), `subjects` (link to curriculum). Validation: stream sub-grouping consistent with subject mapping (CLS-006/SUB-002).

**Aggregate Capacity** — `capacity` (number ≥ sum of section minimums, optional). Validation: not below current enrolled total; advisory vs section hard cap (CLS-007 / C-01).

**Retire/Archive** — `reason` (text, required). Validation: history preserved; no hard delete (CLS-001 history).

---

## 8. Search & Filter Requirements

**Classes:** by label, code, campus, stream, status (active/retired), ordering. Sorting: ordering/label. Pagination: server-side, 25 default. Scope-bound to institute.

---

## 9. Table Requirements

**Class table:** Order, Label, Code, Campuses, Streams, #Sections, Aggregate Cap, Status. Sorting on Order/Label. Filtering as above. Export (governed). Bulk: define/import, reorder.

---

## 10. Workflow Requirements

**Trigger events:** define, label/order change, promotion-path change, stream change, capacity change, retire. **Status changes:** Class `ACTIVE → RETIRED`. **Approvals:** structure changes (promotion path, retirement) via Workflow Engine. **Notifications:** structure-change notices to academic admins. **Audit:** all definition/structure changes (before/after, immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| View class(es) | `class.view` |
| Define class | `class.create` |
| Update class/label/ordering | `class.update` |
| Configure promotion path | `class.promotion.manage` |
| Manage streams | `class.stream.manage` |
| Set aggregate capacity | `class.capacity.manage` |
| Retire/archive | `class.retire` |
| Approve structure change | `class.change.approve` |
| Import/export | `class.import`, `class.export` |

Class structure is institute-scoped; structural changes are governed.

---

## 12. Business Rule References

CLS-001 (structure definition, instantiated per session), CLS-002 (configurable label & ordering), CLS-003 (≥1 section / default-section guarantee), CLS-004 (campus applicability), CLS-005 (acyclic promotion path), CLS-006 (streams as sub-grouping), CLS-007 (aggregate class capacity — optional). Cross-cutting: SESS-004 (per-session instancing), SEC-001/003 (sections/leaf capacity), SUB-002 (curriculum mapping), C-01 (capacity precedence), WFL-004, AUD-001.

## 13. Use Case References

UC-CLS-001 (Define), UC-CLS-002 (View), UC-CLS-003 (Update), UC-CLS-004 (Retire/Archive), UC-CLS-005 (Configure Promotion Path), UC-CLS-006 (Manage Streams), UC-CLS-007 (Set Aggregate Capacity), UC-CLS-008 (Label & Ordering), UC-CLS-009 (Search), UC-CLS-010 (Structure & Capacity Report), UC-CLS-011 (Import/Bulk-Define), UC-CLS-012 (Export), UC-CLS-013 (Structure-Change Approval), UC-CLS-014 (Cyclic Promotion Path Blocked), UC-CLS-015 (Class Without Section Blocked).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Define class | POST | Academic Admin |
| Get / list classes | GET | Authorized roles |
| Update class (label/ordering/campus) | PUT | Academic Admin |
| Configure promotion path (acyclic-checked) | POST | Academic Admin (→ approval) |
| Manage streams | POST/PUT/DELETE | Academic Admin |
| Set aggregate capacity | PUT | Academic Admin |
| Retire / archive class | POST | Academic Admin (→ approval) |
| Class structure & capacity report | GET | Admin |
| Import / export class structure | POST/GET | Admin |

Defining a class provisions a default section transactionally (CLS-003); promotion-path edits run a cycle check (CLS-005).

---

## 15. Database Requirements

**Entities:** `Class` (label, code, ordering, campusApplicability, streams, aggregateCapacity, status), `PromotionPath` (fromClass, toClass), `Stream`, `ClassSessionInstance` (per-session instance ref). **Relationships:** Class 1—* Section; Class 1—* PromotionPath; Class 1—* Stream; Session 1—* ClassSessionInstance. **Indexes:** unique(Class.ordering, instituteId), index(Class.status), index(PromotionPath.fromClass). Acyclicity validated on promotion-path writes (CLS-005); retire-not-delete (CLS-001 history).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| In-App | Structure-change notices; pending approvals. |
| Email | Major structure changes to academic admins. |
| SMS/Push | Not used. |

---

## 17. Audit Requirements

Log: class define/update (label/ordering/campus), promotion-path changes (before/after, acyclic-check result), stream changes, capacity changes, retirement. Record who/when/before/after. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Class structure overview, Promotion-path map, Stream distribution, Aggregate vs section capacity. **Exports:** governed class-structure export. **Dashboards:** academic structure health (classes, sections-per-class, capacity).

---

## 19. Error Handling

**Validation:** cyclic promotion path, class without section, duplicate ordering → specific errors (UC-CLS-014/015). **Permission:** non-academic-admin structural change → 403. **Workflow:** structure change pending approval → clear state. **System:** Session instancing dependency unavailable → define allowed, instancing deferred.

---

## 20. Edge Cases

**Concurrent updates:** two ordering edits → versioned, last-coherent wins. **Duplicate data:** duplicate ordering/label → blocked. **Partial failures:** bulk-define partial → per-row report. **Rollback:** retirement reversed → class restored, history intact. **Default-section race:** define class while removing its only section → default-section guarantee holds (CLS-003).

---

## 21. Acceptance Criteria

**Functional.** A class is a reusable structure element instanced per session; label/ordering are configurable; every class always has ≥1 (default) section; promotion paths are acyclic; streams are configurable sub-groupings; aggregate capacity is optional and never below enrolled; classes are retired (history-preserving), never hard-deleted.

**Business.** The academic structure adapts to any institution type without code change; promotion is safe and well-defined; sections, subjects, and enrollment attach to a clean, versioned class backbone.

---

## 22. Future Enhancements

Visual structure/promotion-path designer; template class structures per institution type; multi-year structure planning; stream-prerequisite enforcement at promotion; capacity what-if planning; class-level timetable templates.
