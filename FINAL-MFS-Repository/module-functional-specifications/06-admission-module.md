# 06 — Admission Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`ADM-001…008`), and Use Case Repository (`UC-ADM-001…020`). No new architecture or business rules introduced.

---

## 1. Module Overview

**Purpose.** Manage the applicant journey from application to enrolled student: configurable application forms, idempotent submission, version-pinned admission decisions, capacity/seat and waitlist control, admission-fee gating, and integrity-guaranteed conversion to Student + Enrollment.

**Business Goal.** Run a fair, auditable, capacity-respecting intake that converts approved applicants into students without data loss or double-allocation of seats.

**Scope.** Application form configuration; application submission (unique number, idempotent); evaluation; decision via version-pinned workflow with SoD; capacity & seat control; waitlist; admission-fee gating; conversion (approved → student + enrollment) atomically; terminal-state immutability and re-application rules.

**Out of Scope.** Student record internals (Student module). Enrollment placement mechanics (Enrollment module — conversion calls it). Fee structure definitions (Fee module — admission gating consumes it). Workflow execution internals (Workflow Engine). Form-schema storage mechanics (Configuration Engine).

---

## 2. Actors

**Primary Actors.** Applicant / Guardian (submits, accepts offer, pays), Admission Officer (evaluates, corrects), Admission Approver (decides — SoD), Admission Administrator (configures form/capacity/criteria/fee), System (idempotency, capacity, conversion).

**Secondary Actors.** Configuration Engine (form schema, capacity/criteria config), Workflow Engine (decision workflow), Student/Enrollment modules (conversion targets), Fee/Payment (admission fee), Notification, Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Configurable application form | Define the application schema (fields, required docs) per intake. | High | Configuration Engine |
| FR-002 | Idempotent submission | Assign a unique application number; identical resubmits create no duplicate. | Critical | FR-001 |
| FR-003 | Application evaluation | Score/evaluate against configured criteria/merit. | High | FR-001 |
| FR-004 | Version-pinned decision | Decide via a workflow pinned to the definition version at submission (fairness). | Critical | Workflow Engine |
| FR-005 | Capacity & seat control | Enforce intake capacity; offers honor seat limits. | Critical | Config (capacity) |
| FR-006 | Waitlist management | Maintain and promote a waitlist as seats free up. | High | FR-005 |
| FR-007 | Admission-fee gating | Gate offer acceptance/conversion on admission-fee payment where configured. | High | Fee/Payment |
| FR-008 | Conversion integrity | Convert approved applicant → student + enrollment atomically (no partial). | Critical | Student, Enrollment |
| FR-009 | Terminal-state immutability | Lock terminal applications; re-application is a new application. | High | FR-002 |
| FR-010 | Decision SoD & return-for-correction | Enforce requester ≠ approver; bounded return-for-correction loop. | High | Workflow Engine, AUTHZ-009 |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Configurable intake forms | Schema-driven applications per program/intake. | New intakes without code change. |
| Idempotent application | Unique number; no duplicate on resubmit. | Clean applicant data; no double-processing. |
| Fair version-pinned decisions | Rules frozen at submission. | Defensible, equitable admissions. |
| Capacity, seats & waitlist | Hard seat control + waitlist promotion. | No over-admission; orderly intake. |
| Admission-fee gating | Pay-to-confirm where configured. | Revenue assurance; commitment signal. |
| Atomic conversion | Approved → student + enrollment in one transaction. | No half-converted applicants. |
| Terminal immutability + re-apply | Locked outcomes; clean re-application. | Integrity and audit clarity. |

---

## 5. Screens

Application Form (public/guardian); Application Tracker (status); Officer — Application List; Officer — Application Detail (evaluate); Decision Review (approver); Offer & Admission-Fee Payment; Conversion (approved → student/enrollment); Waitlist Management; Admission Form Configuration; Capacity/Criteria/Fee Configuration; Admission Funnel & Intake Report; Application Import; Applicant Data Export; Decision Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Application Form | Save Draft, Submit, Upload Documents | — |
| Application Tracker | View Status, Accept Offer, Pay Fee, Withdraw | — |
| Officer Application List | Open, Evaluate, Search/Filter, Export | Bulk Decision, Bulk Convert |
| Application Detail | Score, Recommend, Return for Correction, Forward to Approver | — |
| Decision Review | Approve, Reject, Return, Comment | Bulk Approve/Reject |
| Offer & Payment | Accept Offer, Pay Admission Fee, Decline | — |
| Conversion | Convert to Student, Assign Enrollment, Confirm | Bulk Convert |
| Waitlist | Promote, Remove, Reorder | Bulk Promote |
| Form / Capacity Config | Edit Schema, Set Capacity/Criteria/Fee, Save (versioned) | — |
| Funnel Report | Run, Filter, Export | Export |

---

## 7. Forms

**Application Form (configurable)** — schema-driven fields from ADM-001 config (e.g., applicant name, DOB, prior school, guardian details, required documents). Validation: dynamic per schema (CFG-010), required fields enforced, document checklist complete; submission assigns a unique application number (ADM-002).

**Evaluation** — `score`/`criteriaResult` (per configured rubric), `recommendation` (select), `officerNotes` (text). Validation: officer owns/handles the application; criteria per configured version.

**Decision** — `decision` (approve/reject/return/waitlist), `reason` (text, required for reject/return). Validation: approver ≠ submitter/officer (SoD, AUTHZ-009); version-pinned (ADM-003); return increments bounded round counter (WFL-011).

**Offer & Admission Fee** — `offerAccept` (action), payment via Payment module. Validation: offer valid/unexpired; fee gating satisfied before conversion (ADM-006).

**Form/Capacity Configuration** — `formSchema` (structured), `capacity` (number per program/section), `criteria` (structured), `feeGating` (toggle + amount ref). Validation: typed/versioned (CFG-002/004); capacity ≥ 0.

---

## 8. Search & Filter Requirements

**Applications:** by application number, applicant name, program/intake, status (draft/submitted/under-review/offered/accepted/waitlisted/rejected/withdrawn/converted), score range, submitted date, decision date. Sorting: submitted date/score/status. Pagination: server-side, 25 default. Scoped to institute/campus + officer ownership where applicable.

---

## 9. Table Requirements

**Application table:** App No., Applicant, Program/Intake, Status, Score, Submitted, Decision, Seat/Waitlist. Sorting on Submitted/Score/Status. Filtering as above. Export (governed — applicant PII gated, UC-ADM-016). Bulk actions: Decision, Convert, Promote-from-waitlist.

---

## 10. Workflow Requirements

**Trigger events:** submit, evaluate, decision (approve/reject/return/waitlist), offer accept, fee paid, convert, waitlist promote, withdraw. **Status changes:** `DRAFT → SUBMITTED → UNDER_REVIEW → (OFFERED/WAITLISTED/REJECTED) → ACCEPTED → CONVERTED` (terminal: REJECTED/WITHDRAWN/CONVERTED). **Approvals:** decision via version-pinned workflow with SoD and bounded return-for-correction. **Notifications:** submission ack, decision, offer, fee reminder, conversion, waitlist movement. **Audit:** all transitions, decisions (with version), conversions (immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| Submit/track application | applicant/guardian (own application) |
| View/evaluate applications | `admission.application.view`, `admission.application.evaluate` |
| Decide (approve/reject) | `admission.decision.approve` (SoD) |
| Convert to student | `admission.convert` |
| Manage waitlist | `admission.waitlist.manage` |
| Configure form/capacity/fee | `admission.config.manage` |
| Import / export applicants | `admission.import`, `admission.export` |

All officer/approver capabilities are scope-bound (institute/campus); SoD enforced on decisions.

---

## 12. Business Rule References

ADM-001 (configurable form), ADM-002 (unique app number & idempotent submission), ADM-003 (decision via version-pinned workflow), ADM-004 (capacity & seat control), ADM-005 (waitlist), ADM-006 (admission-fee gating), ADM-007 (conversion integrity), ADM-008 (terminal-state immutability & re-application). Cross-cutting: CFG-010 (dynamic form schema), WFL-002/004/011 (version-pinning, approver SoD, bounded return), AUTHZ-009 (SoD), FEE/PAY (admission fee), STU-001/ENR-001 (conversion targets), AUD-001, NOT-002 (custody-aware notices).

## 13. Use Case References

UC-ADM-001 (Submit — idempotent), UC-ADM-002 (Evaluate), UC-ADM-003 (Decision — version-pinned), UC-ADM-004 (Accept Offer & Pay), UC-ADM-005 (Convert → Student + Enrollment), UC-ADM-006 (Waitlist), UC-ADM-007 (Configure Form), UC-ADM-008/009/010 (Track/Update/Withdraw), UC-ADM-011 (Approve/Reject — SoD), UC-ADM-012 (Search), UC-ADM-013 (Funnel & Intake Report), UC-ADM-014 (Bulk Decision/Convert), UC-ADM-015/016 (Import/Export), UC-ADM-017 (Decision Workflow — return-for-correction), UC-ADM-018 (Configure Capacity/Criteria/Fee), UC-ADM-019 (Capacity Exceeded at Conversion), UC-ADM-020 (Duplicate/Re-Application on Terminal State).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Get application form schema | GET | Applicant/Guardian |
| Submit application (idempotent) | POST | Applicant/Guardian |
| Track application status | GET | Applicant/Guardian |
| Update application (pre-submit/officer) | PUT | Applicant/Officer |
| Evaluate application | POST | Officer |
| Record decision (approve/reject/return/waitlist) | POST | Approver |
| Accept offer / pay admission fee | POST | Applicant/Guardian |
| Convert approved → student + enrollment | POST | Officer (→ atomic) |
| Manage waitlist (promote/remove) | POST | Admin |
| Configure form/capacity/criteria/fee | GET/PUT | Admin |
| Search applications | GET | Officer/Admin |
| Funnel & intake report | GET | Admin |
| Import / export applications | POST/GET | Admin |

Conversion is transactional: student creation + enrollment + seat decrement commit together or not at all (ADM-007).

---

## 15. Database Requirements

**Entities:** `Application` (number, schema-version, status, score, intake), `ApplicationDocument`, `Decision` (outcome, reason, approver, workflow-version), `Waitlist` (position), `AdmissionConfig` (capacity/criteria/fee via Config), `ConversionRecord` (links application → student → enrollment). **Relationships:** Application 1—* Document; Application 1—1 Decision; Application 1—1 ConversionRecord; Application *—1 Intake/Program. **Indexes:** unique(Application.number), index(Application.status, intakeId), index(Waitlist.intakeId, position), unique(ConversionRecord.applicationId). Idempotency key on submission prevents duplicates (ADM-002).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Submission ack, decision, offer, admission-fee reminder/receipt, conversion confirmation, waitlist movement. |
| SMS | Decision/offer alerts (minimized content). |
| Push | Status updates (if app installed). |
| In-App | Officer/approver task items; applicant status. |

Notifications are custody-aware where a guardian is the recipient (NOT-002).

---

## 17. Audit Requirements

Log: submission (number, schema version), evaluation, every decision (outcome, reason, approver, workflow version), offer/acceptance, fee gating result, conversion (application → student → enrollment ids), waitlist movements, config changes. Record who/when/before/after. Conversions and decisions are first-class audit events. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Admission funnel (submitted → offered → accepted → converted), Intake/capacity utilization, Waitlist status, Decision turnaround/SLA, Demographics of intake. **Exports:** governed applicant-data export (PII-gated, audited). **Dashboards:** admission control room (open seats, pending decisions, conversion rate).

---

## 19. Error Handling

**Validation:** incomplete form/documents, duplicate submission, decision without reason → specific errors. **Permission:** approver = submitter → SoD block; out-of-scope application → 403/not-found. **Workflow:** return-for-correction over bound → escalate/terminate (WFL-011); conversion when capacity exceeded → blocked (UC-ADM-019). **System:** Fee/Payment unavailable for gating → block conversion; partial conversion failure → full rollback (no orphan student/enrollment).

---

## 20. Edge Cases

**Concurrent updates:** two approvers acting on one application → idempotent single decision. **Duplicate data:** resubmission with same idempotency key → original application returned; re-application on terminal state → new application (UC-ADM-020). **Partial failures:** conversion failing mid-step → atomic rollback (ADM-007). **Rollback:** offer accepted but fee fails → remains offered, not converted. **Capacity race:** last seat contested by two conversions → one succeeds, other waitlisted (UC-ADM-019).

---

## 21. Acceptance Criteria

**Functional.** Applications carry a unique number and resubmits never duplicate; decisions run on the version pinned at submission with requester ≠ approver; capacity/seat limits are never exceeded and freed seats promote the waitlist; admission-fee gating blocks conversion until satisfied; approved applicants convert to student + enrollment atomically; terminal applications are immutable and re-application is a new application.

**Business.** Intake is fair, capacity-respecting, and fully auditable; no applicant is double-processed and no seat is double-allocated; approved applicants become students with correct enrollment and no partial/orphaned records.

---

## 22. Future Enhancements

Online entrance-test integration; configurable scoring rubrics with auto-shortlisting; payment-gateway-native offer acceptance; multi-round/rolling admissions; applicant self-service document re-upload; predictive yield/seat planning; SMS/WhatsApp application via templates.
