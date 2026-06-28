# 09 — Guardian Module — Functional Specification

> Implementation-ready specification. Derives from the approved Architecture Blueprint, Business Rules Catalog (`GRD-N-001…008`), and Use Case Repository (`UC-GRD-N-001…016`). No new architecture or business rules introduced.
>
> **ID convention:** Guardian rules/use cases use the `GRD-N` prefix (Grading uses `GRD`) — they never collide.

---

## 1. Module Overview

**Purpose.** Manage guardians and the custody/consent model that governs who may access a minor's data and receive their communications: guardian-of-record designation, always-≥1-active-guardian for minors, sibling reuse, strict children-only scope, role-scoped capabilities, custody/contact restrictions, reasoned/dated guardianship changes, and parental consent.

**Business Goal.** Protect minors by ensuring only authorized, custody-permitted guardians can see data, act, and be contacted — the relationship backbone for ownership predicates and notification routing across the system.

**Scope.** Guardian creation/designation; link to one or many students (sibling reuse); strict children-only ownership; role-scoped capabilities (financial-responsible, emergency priority, portal access); custody/contact restriction enforcement; reasoned/dated/audited guardianship changes; parental consent management; last-guardian-removal protection.

**Out of Scope.** Student master record (Student module — links here). Authentication of guardian accounts (Authentication). Notification delivery mechanics (Notification — this module supplies custody-aware recipient rules). Financial responsibility execution (Fee/Payment — consumes the financial-responsible flag).

---

## 2. Actors

**Primary Actors.** Guardian (portal access to own children), Registrar / Student-Records Administrator (designates/links/restricts), System (children-only scoping, restriction enforcement, consent gating).

**Secondary Actors.** Student module (linkage subject), Authentication (guardian accounts), Notification (custody-aware routing), Fee/Payment (financial-responsible), Workflow Engine (guardianship-change approval), Audit, Reporting.

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Guardian-of-record designation | Designate the official guardian(s) of record. | Critical | Student |
| FR-002 | Minor always ≥1 active guardian | A minor must never be left without an active guardian. | Critical | Student (STU-006) |
| FR-003 | Sibling reuse | One guardian links to many students. | High | FR-001 |
| FR-004 | Children-only ownership scope | A guardian can access only their own linked children's data. | Critical | Authorization (AUTHZ-003) |
| FR-005 | Role-scoped capabilities | Financial-responsible, emergency priority, portal access per role. | High | Fee/Payment, Notification |
| FR-006 | Custody/contact restrictions | Enforce restrictions (no-access/no-contact) across data and notifications. | Critical | Notification, File |
| FR-007 | Reasoned/dated guardianship changes | All changes reasoned, dated, audited, governed. | High | Workflow Engine, Audit |
| FR-008 | Parental consent | Capture/enforce consent for minors (activities, data, media). | High | Configuration (consent types) |
| FR-009 | Last-guardian protection | Block removal of a minor's last active guardian. | Critical | FR-002 |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Guardian-of-record | Official designation. | Authoritative responsibility. |
| Sibling reuse | One guardian, many children. | No duplicate guardian records. |
| Children-only scope | Strict per-child access. | Minor-data protection. |
| Role capabilities | Financial/emergency/portal roles. | Correct routing and rights. |
| Custody/contact restrictions | Enforced everywhere. | Safeguarding; legal compliance. |
| Consent management | Captured and enforced. | Lawful processing of minor data. |
| Governed changes | Reasoned, dated, audited. | Trustworthy custody records. |

---

## 5. Screens

Guardian List; Guardian Detail; Create/Designate Guardian; Link Guardian to Student(s); Guardian Portal (children-only); Custody/Contact Restrictions; Parental Consent; Financial-Responsible & Emergency Priority; Update Guardianship (reasoned/dated); Guardian Contact & Consent Report; Bulk Guardian Link/Import; Guardian Data Export; Guardianship-Change Approvals.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Guardian List | Create, Open, Search/Filter, Export | Bulk Link, Bulk Import |
| Guardian Detail | Edit, Link Student, Set Roles, Set Restrictions, Manage Consent, Update Guardianship | — |
| Link to Student | Select Student(s), Relationship, Save (sibling reuse) | Bulk Link |
| Guardian Portal | View Child, Pay Fees, View Results/Attendance (per role) | — |
| Custody/Restrictions | Add Restriction, Edit, Remove (governed) | — |
| Parental Consent | Grant, Revoke, View History | — |
| Update Guardianship | Change with Reason/Date, Submit (→ approval) | — |
| Contact & Consent Report | Run, Filter, Export | Export |

---

## 7. Forms

**Create/Designate Guardian** — `name` (text, required), `relationship` (select, required), `contact` (phone/email, required), `nationalId` (text, optional, sensitive). Validation: dedup against existing guardians (sibling reuse — GRD-N-003); guardian-of-record designation (GRD-N-001).

**Link to Student** — `student(s)` (multi-search-select, required), `relationship` (select), `isFinancialResponsible` (toggle), `emergencyPriority` (number), `portalAccess` (toggle). Validation: minor retains ≥1 active guardian (GRD-N-002); children-only scope established (GRD-N-004).

**Custody/Contact Restriction** — `type` (no-access / no-contact / supervised), `student` (required), `effectiveDate` (date), `reason` (text, required), `documentRef` (optional). Validation: enforced across data + notifications + files (GRD-N-006); governed/audited.

**Parental Consent** — `consentType` (config-defined: data/media/activity), `status` (granted/revoked), `date`. Validation: required where mandated for minors (GRD-N-008); mandatory notices ride non-consent lawful basis (C-10).

**Update Guardianship** — `change` (add/remove/replace), `reason` (text, required), `effectiveDate` (date, required). Validation: cannot remove last active guardian of a minor (GRD-N-002 / UC-GRD-N-015); reasoned/dated/approved (GRD-N-007).

---

## 8. Search & Filter Requirements

**Guardians:** by name, contact, linked student, relationship, financial-responsible, has-restriction, consent-status. Sorting: name/relationship. Pagination: server-side, 25 default. Scope-bound; guardian portal sees only own children.

---

## 9. Table Requirements

**Guardian table:** Name, Relationship, Linked Children (count), Financial-Responsible, Restrictions, Consent, Updated. Sensitive fields (national ID) masked (STU-005-style). Sorting on Name. Filtering as above. Export (governed — UC-GRD-N-013). Bulk: Link, Import.

---

## 10. Workflow Requirements

**Trigger events:** create/designate, link/unlink, restriction add/remove, consent grant/revoke, guardianship change. **Status changes:** Guardian link `ACTIVE → INACTIVE`; restriction `ACTIVE/LIFTED`; consent `GRANTED/REVOKED`. **Approvals:** guardianship changes and restriction removals via Workflow Engine. **Notifications:** linkage, restriction applied/lifted, consent changes, guardianship updates (to permitted parties only). **Audit:** all designation/linkage/restriction/consent/guardianship changes (reasoned, dated, immutable).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| View guardian(s) | `guardian.view` |
| Create/designate guardian | `guardian.create` |
| Link/unlink to student | `guardian.link.manage` |
| Set roles (financial/emergency/portal) | `guardian.role.manage` |
| Manage custody/contact restrictions | `guardian.restriction.manage` |
| Manage consent | `guardian.consent.manage` |
| Update guardianship (governed) | `guardian.guardianship.update` |
| Approve guardianship change | `guardian.change.approve` |
| Import/export | `guardian.import`, `guardian.export` |

Guardian portal access is implicitly children-only (GRD-N-004); restricted parties are excluded everywhere (GRD-N-006).

---

## 12. Business Rule References

GRD-N-001 (guardian-of-record), GRD-N-002 (minor always ≥1 active guardian), GRD-N-003 (one guardian many students), GRD-N-004 (strict children-only scope), GRD-N-005 (role-scoped capabilities), GRD-N-006 (custody/contact restrictions enforced), GRD-N-007 (changes reasoned/dated/audited), GRD-N-008 (parental consent). Cross-cutting: STU-006 (mandatory guardian for minors), AUTHZ-003 (ownership predicates), NOT-002 (custody-aware routing), FILE-004 (custody-aware file access), C-10 (mandatory notices lawful basis), AUD-001.

## 13. Use Case References

UC-GRD-N-001 (Create/Designate), UC-GRD-N-002 (Link — sibling reuse), UC-GRD-N-003 (Portal — children-only), UC-GRD-N-004 (Custody/Contact Restrictions), UC-GRD-N-005 (Parental Consent), UC-GRD-N-006 (View/Update Profile), UC-GRD-N-007 (Update Guardianship), UC-GRD-N-008 (Financial-Responsible & Emergency Priority), UC-GRD-N-009 (Approve Guardianship Change), UC-GRD-N-010 (Search), UC-GRD-N-011 (Contact & Consent Report), UC-GRD-N-012 (Bulk Link/Import), UC-GRD-N-013 (Export — governed), UC-GRD-N-014 (Guardianship-Change Workflow), UC-GRD-N-015 (Last-Guardian Removal Blocked), UC-GRD-N-016 (Custody-Restricted Access Attempt).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Create / designate guardian | POST | Registrar |
| Get / list guardians | GET | Authorized roles |
| Link / unlink guardian to student | POST/DELETE | Registrar |
| Set roles (financial/emergency/portal) | PUT | Registrar |
| Manage custody/contact restrictions | POST/PUT | Registrar (governed) |
| Manage parental consent | POST | Registrar/Guardian |
| Update guardianship (reasoned/dated) | POST | Registrar (→ approval) |
| Guardian portal — children data | GET | Guardian (children-only) |
| Contact & consent report | GET | Admin |
| Import / export guardians | POST/GET | Admin |

Every guardian data access is filtered children-only and custody-aware at the data layer (GRD-N-004/006).

---

## 15. Database Requirements

**Entities:** `Guardian` (identity, contact, sensitive ids), `GuardianStudentLink` (relationship, financial-responsible, emergency priority, portal access, active), `CustodyRestriction` (type, student, reason, dates), `ParentalConsent` (type, status, dates), `GuardianshipChangeLog` (reason, date, approver). **Relationships:** Guardian *—* Student (via link); Guardian 1—* CustodyRestriction; Guardian/Student 1—* ParentalConsent. **Indexes:** index(GuardianStudentLink.guardianId), index(GuardianStudentLink.studentId, active), unique-ish dedup(Guardian.contact/nationalId), index(CustodyRestriction.studentId, active), index(ParentalConsent.studentId, type). Last-active-guardian invariant enforced at data layer (GRD-N-002).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Linkage, restriction applied/lifted, consent changes, guardianship updates — permitted parties only. |
| SMS | Emergency/financial alerts to financial-responsible/emergency-priority guardian (minimized). |
| Push | Portal updates for linked children. |
| In-App | Portal notices; pending guardianship approvals. |

Custody-restricted parties are excluded from all notifications (NOT-002/GRD-N-006); mandatory safeguarding/financial notices override opt-out (NOT-003/C-10).

---

## 17. Audit Requirements

Log: guardian create/designate, link/unlink (student, relationship), role changes (financial/emergency/portal), restriction add/remove (reason, dates, approver), consent grant/revoke, guardianship changes (reason, date, approver), and every custody-restricted access attempt (UC-GRD-N-016). Record who/when/before/after. Sensitive guardian data access is auditable. Immutable via outbox.

---

## 18. Reporting Requirements

**Reports:** Guardian contact directory (permitted), Consent status/coverage, Custody-restriction register, Minors-without-guardian (with Student module), Financial-responsible mapping. **Exports:** governed guardian-data export (PII-gated). **Dashboards:** safeguarding overview (active restrictions, consent gaps, guardianship changes pending).

---

## 19. Error Handling

**Validation:** removing last guardian of a minor, linking beyond children scope, missing required consent → specific errors (UC-GRD-N-015). **Permission:** custody-restricted access attempt → denied + audited (UC-GRD-N-016); non-children data access → 403/not-found. **Workflow:** guardianship change pending approval → clear state. **System:** consent service unavailable → block consent-gated actions (fail safe).

---

## 20. Edge Cases

**Concurrent updates:** simultaneous guardianship changes → serialized; last-guardian invariant always held. **Duplicate data:** duplicate guardian → linked (sibling reuse, GRD-N-003), not duplicated. **Partial failures:** bulk link partial → per-row report. **Rollback:** guardianship change reversed → prior custody state restored, history intact. **Restriction race:** restriction applied mid-notification batch → restricted party excluded from that batch (NOT-002).

---

## 21. Acceptance Criteria

**Functional.** A minor always retains ≥1 active guardian and the last guardian cannot be removed; one guardian links to many students (sibling reuse); a guardian accesses only their own children's data; custody/contact restrictions are enforced across data, files, and notifications; guardianship changes are reasoned, dated, approved, and audited; parental consent is captured and enforced where mandated.

**Business.** Minors' data is structurally protected by custody-aware scope and restrictions; the right guardian is contacted for the right purpose; custody and consent records are trustworthy, governed, and auditable across the system.

---

## 22. Future Enhancements

Guardian self-service onboarding with identity verification; configurable consent catalogs per jurisdiction; court-order document workflow for custody changes; multi-guardian approval for sensitive actions; emergency-contact cascade; guardian communication preferences center; relationship-graph visualization for complex families.
