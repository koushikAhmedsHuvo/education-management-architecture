# 09 — Guardian Use Cases

Transforms the Guardian business rules (`GRD-N-001`…`GRD-N-008`) into use cases. Guardians are the legal/contact layer around minors: designation, the always-≥1-guardian invariant, sibling reuse, children-only scope, role-scoped capabilities, custody enforcement, audited changes, and consent.

## 1. Primary Actors
Guardian (portal user — accesses their children only), Registrar / Student Administrator (manages guardian links and restrictions).

## 2. Secondary Actors
System (relationship integrity, scope enforcement, custody/consent enforcement), Student module (mandatory linkage), Notification module (custody-aware routing), Finance (financial-responsible guardian), Audit service.

## 3. Goals
Designate guardians of record; guarantee every minor always has an active guardian; let one guardian serve siblings; strictly confine guardian access to their own children; scope capabilities by role; enforce custody/contact restrictions on access and communication; record guardianship changes with reason and date; manage parental consent for minors.

## 4. User Journeys
- **Onboarding:** registrar links guardian(s) at admission with roles (primary, financial-responsible, emergency) → guardian receives portal access to their children only.
- **Daily use:** guardian logs in → sees only their children's attendance, results, fees, notices → pays fees / acknowledges notices / updates own contact info.
- **Custody event:** a court restriction is recorded → the restricted party is excluded from access and all notifications; the permitted guardian continues.
- **Consent:** guardian grants/withdraws consent for optional media/communications; mandatory notices continue under a non-consent lawful basis (Conflict C-10).

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core/CRUD | UC-GRD-N-001 | Create / Designate Guardian | High |
| Core | UC-GRD-N-002 | Link Guardian to Student(s) (sibling reuse) | High |
| Core | UC-GRD-N-003 | Guardian Portal Access (children-only) | Critical |
| Core | UC-GRD-N-004 | Manage Custody / Contact Restrictions | Critical |
| Core | UC-GRD-N-005 | Manage Parental Consent | High |
| CRUD | UC-GRD-N-006 | View / Update Guardian Profile | Medium |
| CRUD | UC-GRD-N-007 | Update Guardianship (reasoned, dated) | High |
| Admin | UC-GRD-N-008 | Set Financial-Responsible & Emergency Priority | Medium |
| Approval | UC-GRD-N-009 | Approve Guardianship Change | Medium |
| Search | UC-GRD-N-010 | Search Guardians | Medium |
| Reporting | UC-GRD-N-011 | Guardian Contact & Consent Report | Medium |
| Bulk | UC-GRD-N-012 | Bulk Guardian Link / Import | Medium |
| Export | UC-GRD-N-013 | Export Guardian Data (governed) | Low |
| Workflow | UC-GRD-N-014 | Guardianship-Change Workflow | Medium |
| Exception | UC-GRD-N-015 | Last-Guardian Removal Blocked | Critical |
| Exception | UC-GRD-N-016 | Custody-Restricted Access Attempt | High |

---

## 6. Detailed Specifications (high-value use cases)

### UC-GRD-N-003 — Guardian Portal Access (children-only)
- **Module:** Guardian · **Priority:** Critical
- **Actors:** Guardian (primary), System
- **Goal:** Give a guardian access strictly limited to their own children's data, role-scoped.
- **Description:** The guardian sees only students they are linked to, with capabilities bound to their guardian role; custody restrictions and masking apply.
- **Business Rules Applied:** GRD-N-004, GRD-N-005, GRD-N-003, STU-005, AUTHZ-003.
- **Preconditions:** Authenticated guardian with ≥1 active student link.
- **Trigger:** Guardian opens the portal.
- **Main Success Scenario:**
  1. System resolves the guardian's linked children (ownership predicate, GRD-N-004).
  2. System assembles only those children's data (attendance, results, fees, notices), role-scoped (GRD-N-005).
  3. Sensitive fields are masked per need-to-know (STU-005); custody restrictions applied.
  4. Guardian interacts within their permitted capabilities.
- **Alternative Flows:** A1) A guardian of siblings sees all linked children in one view (GRD-N-003).
- **Exception Flows:** E1) Attempt to access a non-linked student → denied (UC-GRD-N-016 if restricted, else not-found). E2) Custody-restricted child → excluded.
- **Validation Rules:** Access limited to linked children; capabilities per role; custody honored (GRD-N-003/004/005/006).
- **Permissions Required:** Guardian role (children-scoped); no admin permissions.
- **Notifications Triggered:** None routine.
- **Audit Events Generated:** `GUARDIAN_PORTAL_ACCESS` (sensitive views recorded).
- **Data Created/Updated/Deleted:** None (read; self-service updates separate).
- **Post Conditions:** Guardian sees exactly their children, masked and role-scoped.
- **Related Use Cases:** UC-GRD-N-002, UC-GRD-N-004.
- **Acceptance Criteria:**
  - Given a guardian linked to two children, When they open the portal, Then they see exactly those two and no others.
  - Given an attempt to access a non-linked student, When made, Then it is denied.
  - Given a custody restriction on a child, When the restricted guardian accesses the portal, Then that child's data is excluded.
- **Edge Case Analysis:**
  - *Invalid Input:* direct id manipulation for a non-child → denied (ownership predicate).
  - *Permission Failure:* guardian attempting admin actions → 403.
  - *Concurrent Update:* child's data changing while viewed → consistent read.
  - *Duplicate Data:* sibling reuse shows one guardian, many children (no duplication).
  - *System Failure:* ownership resolution failure → fail closed (deny).
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* single child; multiple siblings; role-scoped capabilities.
  - *Negative:* non-child access; admin-action attempt; custody-restricted child.
  - *Boundary:* a guardian who just lost/gained a link (effective immediately).

### UC-GRD-N-004 — Manage Custody / Contact Restrictions
- **Module:** Guardian · **Priority:** Critical
- **Actors:** Registrar / Compliance (primary), System
- **Goal:** Record and enforce legal custody/contact restrictions across access and communication.
- **Description:** Captures a restriction (e.g., a court order barring a parent) and enforces it everywhere — portal access, notifications, file access — while preserving the permitted guardian's access.
- **Business Rules Applied:** GRD-N-006, GRD-N-007, NOT-002, FILE-004.
- **Preconditions:** A documented restriction; actor holds `student.guardian.manage` (elevated/compliance).
- **Trigger:** A custody/contact restriction is recorded.
- **Main Success Scenario:**
  1. Authorized staff record the restriction with reason/date and supporting reference (GRD-N-007).
  2. System applies it as an enforced exclusion: the restricted party loses portal access to the child, is removed from notification routing (NOT-002), and cannot access the child's files (FILE-004).
  3. The permitted guardian(s) retain full access.
- **Alternative Flows:** A1) Restriction lifted (dated, audited) → access restored.
- **Exception Flows:** E1) Restricted party attempts access/contact → blocked and audited (UC-GRD-N-016). E2) Removing a restricted party who is the last guardian → must first add a permitted guardian (UC-GRD-N-015).
- **Validation Rules:** Restriction reasoned/dated; enforced across access/notification/files; ≥1 permitted guardian remains for the minor (GRD-N-002/006).
- **Permissions Required:** `student.guardian.manage` (elevated/compliance).
- **Notifications Triggered:** Restriction recorded notice to permitted guardians/admin (never to the restricted party with the child's data).
- **Audit Events Generated:** `CUSTODY_RESTRICTION_SET/LIFTED` (reason, date, actor).
- **Data Created:** Restriction record.
- **Data Updated:** Guardian relationship flags; access/notification routing.
- **Data Deleted:** None.
- **Post Conditions:** Restriction enforced everywhere; minor still has a permitted guardian; audited.
- **Related Use Cases:** UC-GRD-N-003, UC-NOT-02 (routing), UC-FILE-02 (access).
- **Acceptance Criteria:**
  - Given a recorded restriction, When applied, Then the restricted party is excluded from portal access, notifications, and file access for that child.
  - Given a restriction, When notifications are sent, Then they route only to permitted guardians.
  - Given the restricted party is the last guardian, When restriction is applied, Then a permitted guardian must be added first.
- **Edge Case Analysis:**
  - *Invalid Input:* restriction without reason/reference rejected.
  - *Permission Failure:* non-elevated actor → 403.
  - *Concurrent Update:* restriction + notification in flight → restriction wins (recipient excluded).
  - *Duplicate Data:* duplicate restriction idempotent.
  - *System Failure:* enforcement fails closed (exclude on doubt).
  - *Workflow Failure:* approval (if required) pends; restriction can be provisional per policy.
- **QA Coverage:**
  - *Positive:* set restriction → exclusion everywhere; lift → restore.
  - *Negative:* restricted access attempt; last-guardian conflict.
  - *Boundary:* restriction effective mid-notification batch (excluded from that batch).

### UC-GRD-N-005 — Manage Parental Consent
- **Module:** Guardian · **Priority:** High
- **Actors:** Guardian (primary), Registrar, System
- **Goal:** Capture and honor consent for optional processing while keeping mandatory notices flowing.
- **Description:** Guardians grant/withdraw consent for optional items (media use, marketing-style communications, optional data processing); mandatory security/safeguarding/financial/legal notices continue under a non-consent lawful basis (Conflict C-10).
- **Business Rules Applied:** GRD-N-008, NOT-003, STU-005.
- **Preconditions:** Consent categories configured; guardian authenticated for their child.
- **Trigger:** Guardian sets consent; or a process checks consent before an optional action.
- **Main Success Scenario:**
  1. Guardian views consent categories for their child and grants/withdraws each.
  2. System records consent state (dated, versioned).
  3. Optional processing (e.g., publishing a photo) checks and honors consent.
  4. Mandatory notices continue regardless of optional-consent state (NOT-003).
- **Alternative Flows:** A1) Consent withdrawn → dependent optional processing stops going forward.
- **Exception Flows:** E1) Attempt to use a minor's media without consent → blocked (FILE/consent check). E2) Attempt to opt out of mandatory notices → not permitted (lawful basis, C-10).
- **Validation Rules:** Consent recorded per category/version; optional processing gated by consent; mandatory categories exempt (GRD-N-008, NOT-003).
- **Permissions Required:** Guardian (own children) / `student.guardian.manage` (assisted).
- **Notifications Triggered:** Consent-change confirmation.
- **Audit Events Generated:** `CONSENT_GRANTED/WITHDRAWN` (category, version).
- **Data Created:** Consent records.
- **Data Updated:** Consent state.
- **Data Deleted:** None.
- **Post Conditions:** Optional processing honors consent; mandatory notices unaffected; audited.
- **Related Use Cases:** UC-GRD-N-003, UC-FILE (media), UC-NOT (mandatory).
- **Acceptance Criteria:**
  - Given withdrawn media consent, When a photo would be published, Then publication is blocked.
  - Given withdrawn optional-comms consent, When mandatory notices arise, Then they are still delivered.
  - Given any consent change, When recorded, Then it is dated, versioned, and audited.
- **Edge Case Analysis:**
  - *Invalid Input:* consent for an unconfigured category rejected.
  - *Permission Failure:* non-guardian setting consent → 403.
  - *Concurrent Update:* rapid grant/withdraw → last state wins, versioned.
  - *Duplicate Data:* idempotent repeated consent.
  - *System Failure:* consent check fails closed for optional processing (withhold).
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* grant/withdraw; optional processing honors state.
  - *Negative:* media use without consent; opt-out of mandatory (blocked).
  - *Boundary:* consent withdrawn moments before an optional action (honored).

---

## 7. Compact Specifications (routine use cases)

- **UC-GRD-N-001 — Create / Designate Guardian** · *High* · Rules: GRD-N-001. Create a guardian record (identity, contact). *Permissions:* `student.guardian.manage`. *Audit:* `GUARDIAN_CREATED`. *Edge:* dedup to existing guardian (sibling reuse). *QA:* create; dedup link.
- **UC-GRD-N-002 — Link Guardian to Student(s)** · *High* · Rules: GRD-N-002/003. Link with role; one guardian → many students. *Edge:* minors keep ≥1 active guardian; reuse across siblings. *QA:* link; sibling reuse; invariant preserved.
- **UC-GRD-N-006 — View / Update Guardian Profile** · *Medium* · Rules: GRD-N-001, STU-005. Guardian self-updates own contact; staff view scoped. *Edge:* guardian edits own profile only. *QA:* self-update; scope respected.
- **UC-GRD-N-007 — Update Guardianship (reasoned, dated)** · *High* · Rules: GRD-N-007, GRD-N-002. Change relationships with reason/date; invariant enforced. *Audit:* `GUARDIANSHIP_CHANGED`. *Edge:* add-before-remove for sole guardian. *QA:* reasoned change; invariant; ordering.
- **UC-GRD-N-008 — Set Financial-Responsible & Emergency Priority** · *Medium* · Rules: GRD-N-005, GRD-N-007. Designate financial-responsible guardian (fees route here) and emergency priority. *Edge:* exactly the intended financial owner; priority ordering. *QA:* fee routing; emergency order.
- **UC-GRD-N-009 — Approve Guardianship Change** · *Medium* · Rules: WFL-004, GRD-N-007. Approver gates sensitive guardianship changes. *Edge:* SoD; escalation. *QA:* approval gates; self-approval blocked.
- **UC-GRD-N-010 — Search Guardians** · *Medium* · Rules: AUTHZ-002, REP-002. Scoped search. *Edge:* results scoped; guardian PII protected. *QA:* scope respected.
- **UC-GRD-N-011 — Guardian Contact & Consent Report** · *Medium* · Rules: REP-002/003, GRD-N-008. Contactability and consent status (scoped, masked). *Edge:* sensitive data masked; custody-restricted contacts handled. *QA:* accuracy; masking; restriction handling.
- **UC-GRD-N-012 — Bulk Guardian Link / Import** · *Medium* · Rules: GRD-N-002/003. Bulk link/import with dedup and invariant checks. *Edge:* duplicates reused; minors-without-guardian flagged. *QA:* clean bulk; dedup; invariant flags.
- **UC-GRD-N-013 — Export Guardian Data (governed)** · *Low* · Rules: REP-005, STU-005. Governed, scoped export; PII controlled. *QA:* scoped; PII gated.
- **UC-GRD-N-014 — Guardianship-Change Workflow** · *Medium* · Rules: WFL-002/004, GRD-N-007. Version-pinned change workflow. *QA:* pinning; SoD; escalation.
- **UC-GRD-N-015 — Last-Guardian Removal Blocked (Exception)** · *Critical* · Rules: GRD-N-002, STU-006. Cannot remove the last active guardian of a minor. *QA:* blocked; add-before-remove path.
- **UC-GRD-N-016 — Custody-Restricted Access Attempt (Exception)** · *High* · Rules: GRD-N-006, NOT-002, FILE-004. Restricted party's access/contact blocked and audited. *QA:* access blocked; notification excluded; file access denied.

## 8. Module-level QA & Edge Themes
- **Children-only scope (GRD-N-004):** the single most-tested guardian control — direct-id manipulation must never reach a non-child.
- **Always-≥1-guardian (GRD-N-002):** invariant across link/unlink/restriction/import; add-before-remove ordering.
- **Custody enforcement (GRD-N-006):** consistent exclusion across portal, notifications, and files (cross-module suite with NOT-002, FILE-004).
- **Consent vs mandatory (C-10):** optional processing honors consent; mandatory notices ride a non-consent lawful basis.
