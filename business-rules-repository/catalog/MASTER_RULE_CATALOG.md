# Master Rule Catalog

**The single source of truth for development and QA.** Every business rule across all 30 modules, consolidated. Each row links the rule to its module document (under `/modules/`) where the full 12-field specification lives (Description, Preconditions, Business Rule, System Action, Validation, Failure Behavior, Audit, Example, Related).

**Totals:** 248 rules · 30 modules · Priorities — Critical 87, High 121, Medium 39, Low 1.

> Rule IDs are stable and permanent. A retired rule's ID is never reused (see Versioning Strategy).

### 01 — Authentication

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `AUTH-001` | Short-Lived Access Token Issuance | Critical | Credential issuance |
| `AUTH-002` | Rotating Refresh Tokens with Family Reuse Detection | Critical | Session security |
| `AUTH-003` | Concurrent-Refresh Grace Window | High | Session security / reliability |
| `AUTH-004` | Progressive Failed-Attempt Lockout | Critical | Brute-force defense |
| `AUTH-005` | Immediate Revocation on State Change | Critical | Session control |
| `AUTH-006` | Session and Device Visibility & Control | High | Session management |
| `AUTH-007` | MFA Policy and Step-Up | High | Strong authentication |
| `AUTH-008` | Password Policy & Breached-Password Rejection | High | Credential strength |
| `AUTH-009` | Non-Enumerating, Rate-Limited Recovery | Critical | Account privacy |
| `AUTH-010` | Session Invalidation on Credential Change | High | Session security |
| `AUTH-011` | Invitation-Based Activation | High | Onboarding |

### 02 — Authorization

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `AUTHZ-001` | Deny-by-Default Permission Check | Critical | Access control |
| `AUTHZ-002` | Mandatory Scope Filter | Critical | Multi-institute isolation |
| `AUTHZ-003` | Ownership / Relationship Predicates | Critical | Row-level access |
| `AUTHZ-004` | Deny Overrides | Critical | Evaluation policy |
| `AUTHZ-005` | No Privilege Escalation via Role/Membership Management | Critical | Escalation prevention |
| `AUTHZ-006` | Permissions-Version Cache Invalidation | High | Consistency |
| `AUTHZ-007` | Scoped Role Assignment Authority | High | Membership management |
| `AUTHZ-008` | Permission Registry Integrity | High | Governance |
| `AUTHZ-009` | Separation of Duties (SoD) Hooks | Medium | Controls / fraud prevention |

### 03 — Institute

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `INST-001` | Institute Code Uniqueness | High | Identity |
| `INST-002` | Institution Type Drives Templates, Not Code | Critical | Configuration |
| `INST-003` | Mandatory Setup Before Activation | High | Lifecycle gating |
| `INST-004` | Institute Type Change Is Restricted | High | Integrity |
| `INST-005` | Configuration Inheritance & Override | Medium | Configuration resolution |
| `INST-006` | Institute Is a Mandatory Scope | Critical | Isolation |
| `INST-007` | Suspension Cascade | High | Lifecycle / access |
| `INST-008` | Archive, Never Hard-Delete | Critical | Data integrity / compliance |

### 04 — Campus

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `CAMP-001` | Campus Belongs to Exactly One Institute | Critical | Scope integrity |
| `CAMP-002` | Default Campus Guarantee | High | Lifecycle invariant |
| `CAMP-003` | Campus Code Uniqueness Within Institute | Medium | Identity |
| `CAMP-004` | Campus Scope Filtering | Critical | Isolation |
| `CAMP-005` | Campus-Level Configuration Override | Medium | Configuration resolution |
| `CAMP-006` | Inter-Campus Movement Is a Transfer | High | Data integrity |
| `CAMP-007` | Deactivate/Archive Requires No Active Dependents | High | Lifecycle integrity |

### 05 — Academic Session

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `SESS-001` | Session Date Validity | High | Temporal integrity |
| `SESS-002` | Controlled Session Overlap (Current + Admission-Open) | Critical | Lifecycle (commonly mis-modeled) |
| `SESS-003` | Single Current Session per Institute | Critical | Lifecycle invariant |
| `SESS-004` | Structure Instances Are Per-Session, From the Definition | Critical | Definition/instance separation (D38) |
| `SESS-005` | Post-Close Immutability | Critical | Record integrity |
| `SESS-006` | Promotion Honors Outcomes (Advance / Retain / Graduate / Exit) | High | Promotion logic |
| `SESS-007` | Close-Readiness Checks | High | Lifecycle gating |
| `SESS-008` | Reopen Is Exceptional and Audited | High | Integrity / governance |

### 06 — Student

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `STU-001` | Unique Student Identifier | Critical | Identity |
| `STU-002` | Core vs Dynamic Profile Fields | High | Configuration / data quality |
| `STU-003` | Duplicate-Student Detection | High | Data integrity |
| `STU-004` | Official-Field Edit Control | High | Integrity / governance |
| `STU-005` | Sensitive-Data Minimization & Masking | Critical | Privacy (minors) |
| `STU-006` | Mandatory Guardian Linkage for Minors | Critical | Child safety / integrity |
| `STU-007` | Archive/Anonymize, Never Hard-Delete | Critical | Data integrity / compliance |
| `STU-008` | Status Reflects Lifecycle, Driven by Enrollment & Outcomes | High | Lifecycle integrity |

### 07 — Admission

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `ADM-001` | Configurable Application Form | High | Configuration |
| `ADM-002` | Unique Application Number & Idempotent Submission | High | Identity / integrity |
| `ADM-003` | Admission Decision via Version-Pinned Workflow | Critical | Workflow / fairness |
| `ADM-004` | Capacity & Seat Control | High | Capacity |
| `ADM-005` | Waitlist Management | Medium | Fairness |
| `ADM-006` | Admission Fee Gating | High | Finance integration |
| `ADM-007` | Conversion Integrity (Approved → Student + Enrollment) | Critical | Integrity |
| `ADM-008` | Terminal-State Immutability & Re-Application | Medium | Integrity |

### 08 — Enrollment

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `ENR-001` | Enrollment Binds Student to Session/Level/Section Instance | Critical | Integrity (definition/instance) |
| `ENR-002` | One Active Enrollment per Student per Session | Critical | Integrity |
| `ENR-003` | Section Capacity Enforcement | High | Capacity |
| `ENR-004` | Roll Number Assignment | Medium | Identity |
| `ENR-005` | Transfer Preserves Period-Accurate History | High | Integrity |
| `ENR-006` | Cross-Institution Exit Is Withdrawal + Transfer Certificate | High | Lifecycle |
| `ENR-007` | Promotion Creates Next-Session Enrollment by Outcome | High | Promotion |
| `ENR-008` | Enrollment Status Drives Operational Eligibility | High | Lifecycle integrity |

### 09 — Guardian

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `GRD-N-001` | Guardian-of-Record Designation | High | Identity / responsibility |
| `GRD-N-002` | A Minor Must Always Have ≥1 Active Guardian | Critical | Child safety |
| `GRD-N-003` | One Guardian, Many Students (Sibling Reuse) | Medium | Data quality |
| `GRD-N-004` | Strict Children-Only Ownership Scope | Critical | Access control (minors) |
| `GRD-N-005` | Role-Scoped Guardian Capabilities | Medium | Authorization |
| `GRD-N-006` | Custody / Contact Restrictions Are Enforced | Critical | Child safety / legal |
| `GRD-N-007` | Guardianship Changes Are Reasoned, Dated, and Audited | High | Governance |
| `GRD-N-008` | Parental Consent for Minors | Critical | Compliance (minors / GDPR) |

### 10 — Attendance

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `ATT-001` | Configurable Attendance Mode & Statuses | High | Configuration |
| `ATT-002` | Mode Determines Owner & Granularity | High | Authorization (ownership) |
| `ATT-003` | Attendance Only for Active Enrollments | High | Integrity |
| `ATT-004` | Marking Window & Backdating Control | High | Integrity / governance |
| `ATT-005` | Approved Leave Reflects as Excused, Not Absent | High | Integration / fairness |
| `ATT-006` | Holidays & Non-Instructional Days Excluded | Medium | Calendar integrity |
| `ATT-007` | Corrections Are Governed & Audited | High | Integrity |
| `ATT-008` | Absence Notification & Safeguarding Signal | High | Child safety / communication |
| `ATT-009` | Attendance Summaries & Exam-Eligibility Threshold | Medium | Assessment integration |

### 11 — Class

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `CLS-001` | Class Is a Structure Definition Element, Instantiated per Session | Critical | Definition/instance integrity |
| `CLS-002` | Configurable Label & Ordering | High | Configuration |
| `CLS-003` | A Class Contains ≥1 Section (Default Section Guarantee) | High | Structure invariant |
| `CLS-004` | Class Campus Applicability | Medium | Scope |
| `CLS-005` | Acyclic Promotion Path | Critical | Progression integrity |
| `CLS-006` | Streams as Configurable Sub-Grouping | Medium | Structure flexibility |
| `CLS-007` | Aggregate Class Capacity (Optional) | Low | Capacity |

### 12 — Section

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `SEC-001` | Section Is the Enrollment Leaf | Critical | Structure integrity |
| `SEC-002` | Section Name Uniqueness Within Class+Campus+Shift | Medium | Identity |
| `SEC-003` | Section Capacity Is the Operative Enrollment Limit | High | Capacity |
| `SEC-004` | Class-Teacher Assignment Confers Ownership | High | Authorization (ownership) |
| `SEC-005` | Shift Is a First-Class Section Attribute | High | Structure (market-specific) |
| `SEC-006` | Merge/Split/Close Preserves History via Transfers | High | Integrity |

### 13 — Subject

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `SUB-001` | Subject Identity & Code | High | Identity |
| `SUB-002` | Curriculum Mapping (Subject ↔ Class/Stream) | Critical | Structure integrity |
| `SUB-003` | Subject Components & Aggregation | High | Assessment integrity |
| `SUB-004` | Subject-Teacher Assignment Confers Marks Ownership | High | Authorization (ownership) |
| `SUB-005` | Subject Type, Credit & Weightage | High | Assessment integrity |
| `SUB-006` | Elective Selection Rules & Prerequisites | Medium | Curriculum flexibility |
| `SUB-007` | Subject Retirement Preserves History | Medium | Integrity |

### 14 — Examination

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `EXM-001` | Configurable Exam Definition | High | Configuration |
| `EXM-002` | Per-Subject Maxima, Pass Marks & Components | Critical | Assessment integrity |
| `EXM-003` | Exam Eligibility & Admit Card Gating | High | Eligibility / fairness |
| `EXM-004` | Mark Validation Against Maxima | Critical | Data integrity |
| `EXM-005` | Ownership-Scoped Mark Entry | Critical | Authorization (ownership) |
| `EXM-006` | Absence, Exemption & Malpractice Are Distinct Outcomes | High | Assessment integrity |
| `EXM-007` | Submit-and-Lock Immutability | Critical | Integrity (tamper-evidence) |
| `EXM-008` | Bounded, Transparent Moderation & Grace | High | Fairness / integrity |
| `EXM-009` | Mark-Entry Window & Completeness | High | Process integrity |

### 15 — Result Processing

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `RES-001` | Deterministic, Provenance-Stamped Computation | Critical | Integrity / reproducibility |
| `RES-002` | Layered Pass/Fail (Component → Subject → Aggregate) | Critical | Assessment integrity |
| `RES-003` | Single-Point, Defined Rounding | Critical | Fairness (dispute prevention) |
| `RES-004` | Division / Classification & GPA Derivation | High | Computation integrity |
| `RES-005` | Completeness Gate (No Silent Zeros) | Critical | Integrity |
| `RES-006` | Governed Post-Publish Revision | Critical | Integrity / governance |
| `RES-007` | Result Withholding Is Defined, Fair & Time-Bound | High | Fairness / finance integration |
| `RES-008` | Deterministic Rank/Position with Defined Tie-Breaking | Medium | Fairness |
| `RES-009` | Asynchronous, Performance-Safe Publishing | High | Performance / reliability |

### 16 — Grading

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `GRD-001` | Configurable, Versioned Grade Scale | Critical | Configuration / integrity |
| `GRD-002` | Complete, Non-Overlapping Band Coverage | Critical | Integrity |
| `GRD-003` | Explicit, Deterministic Rounding & Boundary Rules | Critical | Fairness (top dispute source) |
| `GRD-004` | GPA / CGPA Computation Rule | High | Computation integrity |
| `GRD-005` | Scope-Specific Schemes (Absolute vs Relative) | Medium | Flexibility |
| `GRD-006` | Special Grades Are Defined and Excluded Correctly | High | Integrity |
| `GRD-007` | Effective-Dated Resolution & Result Stamping | Critical | Historical fidelity |
| `GRD-008` | Governed, Transparent Grade Overrides | High | Integrity / governance |

### 17 — Fee Management

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `FEE-001` | Configurable, Effective-Dated, Versioned Fee Structure | Critical | Configuration / integrity |
| `FEE-002` | Fee Heads & Schedules | High | Structure |
| `FEE-003` | Issued Invoices Are Immutable | Critical | Financial integrity |
| `FEE-004` | Duplicate-Invoice Prevention | High | Integrity |
| `FEE-005` | Pro-Rata for Mid-Period Join/Leave | Medium | Fairness |
| `FEE-006` | Category-Based Fee Differentiation | Medium | Configuration |
| `FEE-007` | Dues, Due Dates & Late Fines | High | Computation integrity |
| `FEE-008` | Arrears Carry-Forward & Refunds Are Governed | High | Lifecycle / integrity |

### 18 — Payment

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `PAY-001` | Gapless, Immutable Receipts | Critical | Financial integrity |
| `PAY-002` | Deterministic Payment Allocation | Critical | Integrity |
| `PAY-003` | Supported Methods & Reference Capture | Medium | Data quality |
| `PAY-004` | Clearance Lifecycle for Non-Immediate Methods | High | Integrity |
| `PAY-005` | Reconciliation Against Settlements | High | Control |
| `PAY-006` | Overpayment Becomes Credit Balance | Medium | Integrity |
| `PAY-007` | Payments Are Reversed, Never Deleted | Critical | Financial integrity |
| `PAY-008` | Idempotent Payment Processing (No Double-Credit) | Critical | Money correctness |
| `PAY-009` | Mis-Posted Payment Correction Is Governed | High | Integrity |

### 19 — Discount

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `DSC-001` | Discount Types & Transparent Application | Critical | Integrity / transparency |
| `DSC-002` | Eligibility-Rule Discounts | High | Configuration / fairness |
| `DSC-003` | Approval Authority & Separation of Duties | Critical | Financial control |
| `DSC-004` | Discount Caps (Never Below Zero) | Critical | Integrity |
| `DSC-005` | Stacking Rules | High | Fairness / control |
| `DSC-006` | Waiver Sub-Type Semantics | High | Clarity / control |
| `DSC-007` | Conditional Reversal & Expiry | Medium | Integrity |

### 20 — Scholarship

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `SCH-001` | Scholarship Is a Structured, Funded Award (Distinct from Discount/Waiver) | Critical | Domain clarity / integrity |
| `SCH-002` | Defined Eligibility & Fair, Slot-Bounded Selection | High | Fairness |
| `SCH-003` | Governed Selection & Conflict-of-Interest Controls | High | Governance / integrity |
| `SCH-004` | Coverage, Stipend & No-Negative-Fee Rule | High | Integrity |
| `SCH-005` | Fund Tracking & Sponsor Accountability | High | Fund integrity |
| `SCH-006` | Scholarship + Discount/Waiver Interaction | Medium | Fairness / control |
| `SCH-007` | Renewal & Revocation Are Condition-Based and Governed | High | Lifecycle / fairness |
| `SCH-008` | Award Changes Never Rewrite History | Medium | Integrity |

### 21 — Teacher

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `TCH-001` | Teacher Specializes Staff | High | Data model integrity |
| `TCH-002` | Qualification-Aware Subject Assignment | High | Assignment integrity |
| `TCH-003` | Workload Limits | Medium | Fairness / feasibility |
| `TCH-004` | Assignment Confers Ownership (Re-Resolves on Change) | Critical | Authorization (ownership) |
| `TCH-005` | Substitute = Temporary, Attributed Ownership Transfer | High | Continuity / integrity |
| `TCH-006` | Mandatory Handover on Departure/Reassignment | High | Continuity |
| `TCH-007` | Class-Teacher (Homeroom) Singularity | Medium | Ownership clarity |

### 22 — Staff

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `STF-001` | Unique Employee Identifier | Critical | Identity |
| `STF-002` | Staff–User Account Linkage | High | Access integration |
| `STF-003` | Duplicate-Staff Detection | High | Data integrity |
| `STF-004` | Employment-Status Lifecycle Drives Access | High | Lifecycle integrity |
| `STF-005` | Institute/Campus Scoping & Multi-Assignment | High | Scope |
| `STF-006` | Reporting Hierarchy | High | Organization |
| `STF-007` | Sensitive Staff Data Protection | Critical | Privacy |
| `STF-008` | Archive/Anonymize, Never Hard-Delete; Handover on Separation | Critical | Integrity / continuity |

### 23 — HR

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `HR-001` | Configurable Department/Designation Structure | Medium | Configuration |
| `HR-002` | Probation-to-Confirmation Lifecycle | High | Lifecycle |
| `HR-003` | Versioned Salary Structure | High | Payroll integrity |
| `HR-004` | Staff Attendance Is Distinct and Payroll-Relevant | High | Integrity / integration |
| `HR-005` | Payroll Computation Integrity | High | Payroll integrity |
| `HR-006` | Payroll Separation of Duties | Critical | Financial control / fraud prevention |
| `HR-007` | Governed Separation, Clearance & Final Settlement | High | Lifecycle / integrity |
| `HR-008` | Rehire Is New Employment, History-Linked | Medium | Integrity |

### 24 — Leave Management

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `LEV-001` | Configurable Leave Types & Policies | High | Configuration |
| `LEV-002` | Balance Accrual, Carry-Forward & Encashment | High | Entitlement integrity |
| `LEV-003` | Holiday/Weekend & Sandwich-Leave Computation | High | Computation (commonly disputed) |
| `LEV-004` | Manager-Routed Approval | High | Workflow |
| `LEV-005` | Escalation, Delegation & Timeout | High | Workflow reliability |
| `LEV-006` | Attendance & Payroll Integration | High | Integration |
| `LEV-007` | Backdated & Exceptional Leave Are Governed | Medium | Governance |
| `LEV-008` | Coverage / Substitution Coordination | Medium | Continuity |
| `LEV-009` | Student Leave Is a Distinct, Balance-Free Sub-Domain | High | Domain separation |

### 25 — Notification

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `NOT-001` | Event-Driven, Configurable, Templated Notifications | High | Configuration / decoupling |
| `NOT-002` | Custody-Aware, Privacy-Preserving Recipient Resolution | Critical | Child safety / privacy |
| `NOT-003` | Preferences with Mandatory-Category Override | High | Compliance / safety |
| `NOT-004` | Data Minimization in Message Content | High | Privacy (minors) |
| `NOT-005` | Reliable Delivery via Outbox (Non-Blocking) | Critical | Reliability |
| `NOT-006` | Bulk Batching & Deduplication for Peak Events | High | Performance |
| `NOT-007` | Quiet Hours & Timing Respect | Medium | User respect |
| `NOT-008` | Delivery Status Tracking & Failure Visibility | Medium | Observability |

### 26 — Reporting

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `REP-001` | Configurable, Parameterized Report Definitions | High | Configuration / correctness |
| `REP-002` | Reports Never Bypass Authorization (Scope + Ownership) | Critical | Security (primary exposure vector) |
| `REP-003` | Sensitive-Data Masking in Reports | High | Privacy (minors/financial/staff) |
| `REP-004` | Projection Freshness & As-Of Transparency | Medium | Correctness / transparency |
| `REP-005` | Export Governance & Bulk-PII Controls | Critical | Data protection |
| `REP-006` | Cross-Institute Reporting Is Org-Only and Deployment-Bound | High | Isolation |
| `REP-007` | Asynchronous, Performance-Safe Large Reports | Medium | Performance |
| `REP-008` | Report Access & Export Auditing | High | Accountability |

### 27 — Configuration Engine

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `CFG-001` | Definition Registry — No Undeclared Configuration | Critical | Governance / integrity |
| `CFG-002` | Typed, Validated Values | Critical | Data quality |
| `CFG-003` | Scoped Values & Most-Specific-Wins Resolution | Critical | Resolution integrity |
| `CFG-004` | Versioning & Immutable Published Versions | Critical | Reproducibility |
| `CFG-005` | Effective-Dating | High | Temporal integrity |
| `CFG-006` | Governed Rollback | High | Safety / governance |
| `CFG-007` | Change Governance, Approval & Immutable-After-Use | Critical | Governance / integrity |
| `CFG-008` | Config-Version Cache Invalidation & Fast Propagation | High | Consistency / performance |
| `CFG-009` | Sensitive / Secret Configuration Protection | Critical | Security |
| `CFG-010` | Dynamic Custom-Field Schemas Are Engine-Validated | High | Data quality / flexibility |
| `CFG-011` | Per-Deployment Isolation & Type Templates | Critical | Isolation / onboarding |

### 28 — Workflow Engine

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `WFL-001` | Declarative Workflow Definitions | Critical | Governance / flexibility |
| `WFL-002` | Version-Pinning at Initiation (Fairness) | Critical | Fairness / integrity |
| `WFL-003` | Deterministic FSM Execution | Critical | Integrity |
| `WFL-004` | Dynamic Approver Resolution with Separation of Duties | Critical | Authorization / control |
| `WFL-005` | Escalation on Timeout | High | Reliability |
| `WFL-006` | Bounded, Audited Delegation | High | Reliability / accountability |
| `WFL-007` | Safe Timeout Policies — No Silent Auto-Approval of Sensitive Actions | Critical | Safety |
| `WFL-008` | Sequential, Parallel & Conditional Routing | High | Expressiveness / integrity |
| `WFL-009` | Withdrawal, Cancellation & Reassignment | Medium | Lifecycle |
| `WFL-010` | Reliability, Idempotency & No Orphaned Instances | Critical | Reliability |
| `WFL-011` | Bounded Return-for-Correction Loops | Medium | Integrity |

### 29 — Audit Log

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `AUD-001` | Immutable, Append-Only, Tamper-Evident Log | Critical | Integrity (non-repudiation) |
| `AUD-002` | Reliable Capture via Transactional Outbox | Critical | Reliability |
| `AUD-003` | Complete, Structured Event Content | High | Usefulness |
| `AUD-004` | Sensitive Values Are Never Stored in Audit | Critical | Security / privacy |
| `AUD-005` | Restricted, Segregated Audit Access | Critical | Control / SoD |
| `AUD-006` | Audit-of-Audit | High | Accountability |
| `AUD-007` | Fail-Safe on Audit Unavailability | High | Integrity |
| `AUD-008` | Erasure-Aware Pseudonymization & Legal Hold | High | Compliance |

### 30 — File Management

| Rule ID | Name | Priority | Category |
|---|---|---|---|
| `FILE-001` | Private Storage, No Public Access | Critical | Security (minors/privacy) |
| `FILE-002` | Upload Validation (Type, Size) | High | Integrity / safety |
| `FILE-003` | Malware Scan Before Availability | Critical | Security |
| `FILE-004` | Subject-Scoped, Custody-Aware Access | Critical | Authorization (minors) |
| `FILE-005` | Short-Lived, Scoped Signed URLs | High | Security |
| `FILE-006` | Document Versioning & Metadata Hygiene | Medium | Integrity / privacy |
| `FILE-007` | Generated Official Documents Are Immutable & Reproducible | High | Integrity / reproducibility |
| `FILE-008` | Retention, Secure Deletion & Quota | Medium | Lifecycle / cost |
