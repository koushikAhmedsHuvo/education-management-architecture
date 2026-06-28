# Business Rules Index

Multiple lookups into the 248-rule catalog: alphabetical by ID, by priority, and by category. For full specifications see the Master Catalog and the module documents.

## A. Index by Rule ID (alphabetical)

| Rule ID | Module | Name | Priority |
|---|---|---|---|
| `ADM-001` | 07 Admission | Configurable Application Form | High |
| `ADM-002` | 07 Admission | Unique Application Number & Idempotent Submission | High |
| `ADM-003` | 07 Admission | Admission Decision via Version-Pinned Workflow | Critical |
| `ADM-004` | 07 Admission | Capacity & Seat Control | High |
| `ADM-005` | 07 Admission | Waitlist Management | Medium |
| `ADM-006` | 07 Admission | Admission Fee Gating | High |
| `ADM-007` | 07 Admission | Conversion Integrity (Approved → Student + Enrollment) | Critical |
| `ADM-008` | 07 Admission | Terminal-State Immutability & Re-Application | Medium |
| `ATT-001` | 10 Attendance | Configurable Attendance Mode & Statuses | High |
| `ATT-002` | 10 Attendance | Mode Determines Owner & Granularity | High |
| `ATT-003` | 10 Attendance | Attendance Only for Active Enrollments | High |
| `ATT-004` | 10 Attendance | Marking Window & Backdating Control | High |
| `ATT-005` | 10 Attendance | Approved Leave Reflects as Excused, Not Absent | High |
| `ATT-006` | 10 Attendance | Holidays & Non-Instructional Days Excluded | Medium |
| `ATT-007` | 10 Attendance | Corrections Are Governed & Audited | High |
| `ATT-008` | 10 Attendance | Absence Notification & Safeguarding Signal | High |
| `ATT-009` | 10 Attendance | Attendance Summaries & Exam-Eligibility Threshold | Medium |
| `AUD-001` | 29 Audit Log | Immutable, Append-Only, Tamper-Evident Log | Critical |
| `AUD-002` | 29 Audit Log | Reliable Capture via Transactional Outbox | Critical |
| `AUD-003` | 29 Audit Log | Complete, Structured Event Content | High |
| `AUD-004` | 29 Audit Log | Sensitive Values Are Never Stored in Audit | Critical |
| `AUD-005` | 29 Audit Log | Restricted, Segregated Audit Access | Critical |
| `AUD-006` | 29 Audit Log | Audit-of-Audit | High |
| `AUD-007` | 29 Audit Log | Fail-Safe on Audit Unavailability | High |
| `AUD-008` | 29 Audit Log | Erasure-Aware Pseudonymization & Legal Hold | High |
| `AUTH-001` | 01 Authentication | Short-Lived Access Token Issuance | Critical |
| `AUTH-002` | 01 Authentication | Rotating Refresh Tokens with Family Reuse Detection | Critical |
| `AUTH-003` | 01 Authentication | Concurrent-Refresh Grace Window | High |
| `AUTH-004` | 01 Authentication | Progressive Failed-Attempt Lockout | Critical |
| `AUTH-005` | 01 Authentication | Immediate Revocation on State Change | Critical |
| `AUTH-006` | 01 Authentication | Session and Device Visibility & Control | High |
| `AUTH-007` | 01 Authentication | MFA Policy and Step-Up | High |
| `AUTH-008` | 01 Authentication | Password Policy & Breached-Password Rejection | High |
| `AUTH-009` | 01 Authentication | Non-Enumerating, Rate-Limited Recovery | Critical |
| `AUTH-010` | 01 Authentication | Session Invalidation on Credential Change | High |
| `AUTH-011` | 01 Authentication | Invitation-Based Activation | High |
| `AUTHZ-001` | 02 Authorization | Deny-by-Default Permission Check | Critical |
| `AUTHZ-002` | 02 Authorization | Mandatory Scope Filter | Critical |
| `AUTHZ-003` | 02 Authorization | Ownership / Relationship Predicates | Critical |
| `AUTHZ-004` | 02 Authorization | Deny Overrides | Critical |
| `AUTHZ-005` | 02 Authorization | No Privilege Escalation via Role/Membership Management | Critical |
| `AUTHZ-006` | 02 Authorization | Permissions-Version Cache Invalidation | High |
| `AUTHZ-007` | 02 Authorization | Scoped Role Assignment Authority | High |
| `AUTHZ-008` | 02 Authorization | Permission Registry Integrity | High |
| `AUTHZ-009` | 02 Authorization | Separation of Duties (SoD) Hooks | Medium |
| `CAMP-001` | 04 Campus | Campus Belongs to Exactly One Institute | Critical |
| `CAMP-002` | 04 Campus | Default Campus Guarantee | High |
| `CAMP-003` | 04 Campus | Campus Code Uniqueness Within Institute | Medium |
| `CAMP-004` | 04 Campus | Campus Scope Filtering | Critical |
| `CAMP-005` | 04 Campus | Campus-Level Configuration Override | Medium |
| `CAMP-006` | 04 Campus | Inter-Campus Movement Is a Transfer | High |
| `CAMP-007` | 04 Campus | Deactivate/Archive Requires No Active Dependents | High |
| `CFG-001` | 27 Configuration Engine | Definition Registry — No Undeclared Configuration | Critical |
| `CFG-002` | 27 Configuration Engine | Typed, Validated Values | Critical |
| `CFG-003` | 27 Configuration Engine | Scoped Values & Most-Specific-Wins Resolution | Critical |
| `CFG-004` | 27 Configuration Engine | Versioning & Immutable Published Versions | Critical |
| `CFG-005` | 27 Configuration Engine | Effective-Dating | High |
| `CFG-006` | 27 Configuration Engine | Governed Rollback | High |
| `CFG-007` | 27 Configuration Engine | Change Governance, Approval & Immutable-After-Use | Critical |
| `CFG-008` | 27 Configuration Engine | Config-Version Cache Invalidation & Fast Propagation | High |
| `CFG-009` | 27 Configuration Engine | Sensitive / Secret Configuration Protection | Critical |
| `CFG-010` | 27 Configuration Engine | Dynamic Custom-Field Schemas Are Engine-Validated | High |
| `CFG-011` | 27 Configuration Engine | Per-Deployment Isolation & Type Templates | Critical |
| `CLS-001` | 11 Class | Class Is a Structure Definition Element, Instantiated per Session | Critical |
| `CLS-002` | 11 Class | Configurable Label & Ordering | High |
| `CLS-003` | 11 Class | A Class Contains ≥1 Section (Default Section Guarantee) | High |
| `CLS-004` | 11 Class | Class Campus Applicability | Medium |
| `CLS-005` | 11 Class | Acyclic Promotion Path | Critical |
| `CLS-006` | 11 Class | Streams as Configurable Sub-Grouping | Medium |
| `CLS-007` | 11 Class | Aggregate Class Capacity (Optional) | Low |
| `DSC-001` | 19 Discount | Discount Types & Transparent Application | Critical |
| `DSC-002` | 19 Discount | Eligibility-Rule Discounts | High |
| `DSC-003` | 19 Discount | Approval Authority & Separation of Duties | Critical |
| `DSC-004` | 19 Discount | Discount Caps (Never Below Zero) | Critical |
| `DSC-005` | 19 Discount | Stacking Rules | High |
| `DSC-006` | 19 Discount | Waiver Sub-Type Semantics | High |
| `DSC-007` | 19 Discount | Conditional Reversal & Expiry | Medium |
| `ENR-001` | 08 Enrollment | Enrollment Binds Student to Session/Level/Section Instance | Critical |
| `ENR-002` | 08 Enrollment | One Active Enrollment per Student per Session | Critical |
| `ENR-003` | 08 Enrollment | Section Capacity Enforcement | High |
| `ENR-004` | 08 Enrollment | Roll Number Assignment | Medium |
| `ENR-005` | 08 Enrollment | Transfer Preserves Period-Accurate History | High |
| `ENR-006` | 08 Enrollment | Cross-Institution Exit Is Withdrawal + Transfer Certificate | High |
| `ENR-007` | 08 Enrollment | Promotion Creates Next-Session Enrollment by Outcome | High |
| `ENR-008` | 08 Enrollment | Enrollment Status Drives Operational Eligibility | High |
| `EXM-001` | 14 Examination | Configurable Exam Definition | High |
| `EXM-002` | 14 Examination | Per-Subject Maxima, Pass Marks & Components | Critical |
| `EXM-003` | 14 Examination | Exam Eligibility & Admit Card Gating | High |
| `EXM-004` | 14 Examination | Mark Validation Against Maxima | Critical |
| `EXM-005` | 14 Examination | Ownership-Scoped Mark Entry | Critical |
| `EXM-006` | 14 Examination | Absence, Exemption & Malpractice Are Distinct Outcomes | High |
| `EXM-007` | 14 Examination | Submit-and-Lock Immutability | Critical |
| `EXM-008` | 14 Examination | Bounded, Transparent Moderation & Grace | High |
| `EXM-009` | 14 Examination | Mark-Entry Window & Completeness | High |
| `FEE-001` | 17 Fee Management | Configurable, Effective-Dated, Versioned Fee Structure | Critical |
| `FEE-002` | 17 Fee Management | Fee Heads & Schedules | High |
| `FEE-003` | 17 Fee Management | Issued Invoices Are Immutable | Critical |
| `FEE-004` | 17 Fee Management | Duplicate-Invoice Prevention | High |
| `FEE-005` | 17 Fee Management | Pro-Rata for Mid-Period Join/Leave | Medium |
| `FEE-006` | 17 Fee Management | Category-Based Fee Differentiation | Medium |
| `FEE-007` | 17 Fee Management | Dues, Due Dates & Late Fines | High |
| `FEE-008` | 17 Fee Management | Arrears Carry-Forward & Refunds Are Governed | High |
| `FILE-001` | 30 File Management | Private Storage, No Public Access | Critical |
| `FILE-002` | 30 File Management | Upload Validation (Type, Size) | High |
| `FILE-003` | 30 File Management | Malware Scan Before Availability | Critical |
| `FILE-004` | 30 File Management | Subject-Scoped, Custody-Aware Access | Critical |
| `FILE-005` | 30 File Management | Short-Lived, Scoped Signed URLs | High |
| `FILE-006` | 30 File Management | Document Versioning & Metadata Hygiene | Medium |
| `FILE-007` | 30 File Management | Generated Official Documents Are Immutable & Reproducible | High |
| `FILE-008` | 30 File Management | Retention, Secure Deletion & Quota | Medium |
| `GRD-001` | 16 Grading | Configurable, Versioned Grade Scale | Critical |
| `GRD-002` | 16 Grading | Complete, Non-Overlapping Band Coverage | Critical |
| `GRD-003` | 16 Grading | Explicit, Deterministic Rounding & Boundary Rules | Critical |
| `GRD-004` | 16 Grading | GPA / CGPA Computation Rule | High |
| `GRD-005` | 16 Grading | Scope-Specific Schemes (Absolute vs Relative) | Medium |
| `GRD-006` | 16 Grading | Special Grades Are Defined and Excluded Correctly | High |
| `GRD-007` | 16 Grading | Effective-Dated Resolution & Result Stamping | Critical |
| `GRD-008` | 16 Grading | Governed, Transparent Grade Overrides | High |
| `GRD-N-001` | 09 Guardian | Guardian-of-Record Designation | High |
| `GRD-N-002` | 09 Guardian | A Minor Must Always Have ≥1 Active Guardian | Critical |
| `GRD-N-003` | 09 Guardian | One Guardian, Many Students (Sibling Reuse) | Medium |
| `GRD-N-004` | 09 Guardian | Strict Children-Only Ownership Scope | Critical |
| `GRD-N-005` | 09 Guardian | Role-Scoped Guardian Capabilities | Medium |
| `GRD-N-006` | 09 Guardian | Custody / Contact Restrictions Are Enforced | Critical |
| `GRD-N-007` | 09 Guardian | Guardianship Changes Are Reasoned, Dated, and Audited | High |
| `GRD-N-008` | 09 Guardian | Parental Consent for Minors | Critical |
| `HR-001` | 23 HR | Configurable Department/Designation Structure | Medium |
| `HR-002` | 23 HR | Probation-to-Confirmation Lifecycle | High |
| `HR-003` | 23 HR | Versioned Salary Structure | High |
| `HR-004` | 23 HR | Staff Attendance Is Distinct and Payroll-Relevant | High |
| `HR-005` | 23 HR | Payroll Computation Integrity | High |
| `HR-006` | 23 HR | Payroll Separation of Duties | Critical |
| `HR-007` | 23 HR | Governed Separation, Clearance & Final Settlement | High |
| `HR-008` | 23 HR | Rehire Is New Employment, History-Linked | Medium |
| `INST-001` | 03 Institute | Institute Code Uniqueness | High |
| `INST-002` | 03 Institute | Institution Type Drives Templates, Not Code | Critical |
| `INST-003` | 03 Institute | Mandatory Setup Before Activation | High |
| `INST-004` | 03 Institute | Institute Type Change Is Restricted | High |
| `INST-005` | 03 Institute | Configuration Inheritance & Override | Medium |
| `INST-006` | 03 Institute | Institute Is a Mandatory Scope | Critical |
| `INST-007` | 03 Institute | Suspension Cascade | High |
| `INST-008` | 03 Institute | Archive, Never Hard-Delete | Critical |
| `LEV-001` | 24 Leave Management | Configurable Leave Types & Policies | High |
| `LEV-002` | 24 Leave Management | Balance Accrual, Carry-Forward & Encashment | High |
| `LEV-003` | 24 Leave Management | Holiday/Weekend & Sandwich-Leave Computation | High |
| `LEV-004` | 24 Leave Management | Manager-Routed Approval | High |
| `LEV-005` | 24 Leave Management | Escalation, Delegation & Timeout | High |
| `LEV-006` | 24 Leave Management | Attendance & Payroll Integration | High |
| `LEV-007` | 24 Leave Management | Backdated & Exceptional Leave Are Governed | Medium |
| `LEV-008` | 24 Leave Management | Coverage / Substitution Coordination | Medium |
| `LEV-009` | 24 Leave Management | Student Leave Is a Distinct, Balance-Free Sub-Domain | High |
| `NOT-001` | 25 Notification | Event-Driven, Configurable, Templated Notifications | High |
| `NOT-002` | 25 Notification | Custody-Aware, Privacy-Preserving Recipient Resolution | Critical |
| `NOT-003` | 25 Notification | Preferences with Mandatory-Category Override | High |
| `NOT-004` | 25 Notification | Data Minimization in Message Content | High |
| `NOT-005` | 25 Notification | Reliable Delivery via Outbox (Non-Blocking) | Critical |
| `NOT-006` | 25 Notification | Bulk Batching & Deduplication for Peak Events | High |
| `NOT-007` | 25 Notification | Quiet Hours & Timing Respect | Medium |
| `NOT-008` | 25 Notification | Delivery Status Tracking & Failure Visibility | Medium |
| `PAY-001` | 18 Payment | Gapless, Immutable Receipts | Critical |
| `PAY-002` | 18 Payment | Deterministic Payment Allocation | Critical |
| `PAY-003` | 18 Payment | Supported Methods & Reference Capture | Medium |
| `PAY-004` | 18 Payment | Clearance Lifecycle for Non-Immediate Methods | High |
| `PAY-005` | 18 Payment | Reconciliation Against Settlements | High |
| `PAY-006` | 18 Payment | Overpayment Becomes Credit Balance | Medium |
| `PAY-007` | 18 Payment | Payments Are Reversed, Never Deleted | Critical |
| `PAY-008` | 18 Payment | Idempotent Payment Processing (No Double-Credit) | Critical |
| `PAY-009` | 18 Payment | Mis-Posted Payment Correction Is Governed | High |
| `REP-001` | 26 Reporting | Configurable, Parameterized Report Definitions | High |
| `REP-002` | 26 Reporting | Reports Never Bypass Authorization (Scope + Ownership) | Critical |
| `REP-003` | 26 Reporting | Sensitive-Data Masking in Reports | High |
| `REP-004` | 26 Reporting | Projection Freshness & As-Of Transparency | Medium |
| `REP-005` | 26 Reporting | Export Governance & Bulk-PII Controls | Critical |
| `REP-006` | 26 Reporting | Cross-Institute Reporting Is Org-Only and Deployment-Bound | High |
| `REP-007` | 26 Reporting | Asynchronous, Performance-Safe Large Reports | Medium |
| `REP-008` | 26 Reporting | Report Access & Export Auditing | High |
| `RES-001` | 15 Result Processing | Deterministic, Provenance-Stamped Computation | Critical |
| `RES-002` | 15 Result Processing | Layered Pass/Fail (Component → Subject → Aggregate) | Critical |
| `RES-003` | 15 Result Processing | Single-Point, Defined Rounding | Critical |
| `RES-004` | 15 Result Processing | Division / Classification & GPA Derivation | High |
| `RES-005` | 15 Result Processing | Completeness Gate (No Silent Zeros) | Critical |
| `RES-006` | 15 Result Processing | Governed Post-Publish Revision | Critical |
| `RES-007` | 15 Result Processing | Result Withholding Is Defined, Fair & Time-Bound | High |
| `RES-008` | 15 Result Processing | Deterministic Rank/Position with Defined Tie-Breaking | Medium |
| `RES-009` | 15 Result Processing | Asynchronous, Performance-Safe Publishing | High |
| `SCH-001` | 20 Scholarship | Scholarship Is a Structured, Funded Award (Distinct from Discount/Waiver) | Critical |
| `SCH-002` | 20 Scholarship | Defined Eligibility & Fair, Slot-Bounded Selection | High |
| `SCH-003` | 20 Scholarship | Governed Selection & Conflict-of-Interest Controls | High |
| `SCH-004` | 20 Scholarship | Coverage, Stipend & No-Negative-Fee Rule | High |
| `SCH-005` | 20 Scholarship | Fund Tracking & Sponsor Accountability | High |
| `SCH-006` | 20 Scholarship | Scholarship + Discount/Waiver Interaction | Medium |
| `SCH-007` | 20 Scholarship | Renewal & Revocation Are Condition-Based and Governed | High |
| `SCH-008` | 20 Scholarship | Award Changes Never Rewrite History | Medium |
| `SEC-001` | 12 Section | Section Is the Enrollment Leaf | Critical |
| `SEC-002` | 12 Section | Section Name Uniqueness Within Class+Campus+Shift | Medium |
| `SEC-003` | 12 Section | Section Capacity Is the Operative Enrollment Limit | High |
| `SEC-004` | 12 Section | Class-Teacher Assignment Confers Ownership | High |
| `SEC-005` | 12 Section | Shift Is a First-Class Section Attribute | High |
| `SEC-006` | 12 Section | Merge/Split/Close Preserves History via Transfers | High |
| `SESS-001` | 05 Academic Session | Session Date Validity | High |
| `SESS-002` | 05 Academic Session | Controlled Session Overlap (Current + Admission-Open) | Critical |
| `SESS-003` | 05 Academic Session | Single Current Session per Institute | Critical |
| `SESS-004` | 05 Academic Session | Structure Instances Are Per-Session, From the Definition | Critical |
| `SESS-005` | 05 Academic Session | Post-Close Immutability | Critical |
| `SESS-006` | 05 Academic Session | Promotion Honors Outcomes (Advance / Retain / Graduate / Exit) | High |
| `SESS-007` | 05 Academic Session | Close-Readiness Checks | High |
| `SESS-008` | 05 Academic Session | Reopen Is Exceptional and Audited | High |
| `STF-001` | 22 Staff | Unique Employee Identifier | Critical |
| `STF-002` | 22 Staff | Staff–User Account Linkage | High |
| `STF-003` | 22 Staff | Duplicate-Staff Detection | High |
| `STF-004` | 22 Staff | Employment-Status Lifecycle Drives Access | High |
| `STF-005` | 22 Staff | Institute/Campus Scoping & Multi-Assignment | High |
| `STF-006` | 22 Staff | Reporting Hierarchy | High |
| `STF-007` | 22 Staff | Sensitive Staff Data Protection | Critical |
| `STF-008` | 22 Staff | Archive/Anonymize, Never Hard-Delete; Handover on Separation | Critical |
| `STU-001` | 06 Student | Unique Student Identifier | Critical |
| `STU-002` | 06 Student | Core vs Dynamic Profile Fields | High |
| `STU-003` | 06 Student | Duplicate-Student Detection | High |
| `STU-004` | 06 Student | Official-Field Edit Control | High |
| `STU-005` | 06 Student | Sensitive-Data Minimization & Masking | Critical |
| `STU-006` | 06 Student | Mandatory Guardian Linkage for Minors | Critical |
| `STU-007` | 06 Student | Archive/Anonymize, Never Hard-Delete | Critical |
| `STU-008` | 06 Student | Status Reflects Lifecycle, Driven by Enrollment & Outcomes | High |
| `SUB-001` | 13 Subject | Subject Identity & Code | High |
| `SUB-002` | 13 Subject | Curriculum Mapping (Subject ↔ Class/Stream) | Critical |
| `SUB-003` | 13 Subject | Subject Components & Aggregation | High |
| `SUB-004` | 13 Subject | Subject-Teacher Assignment Confers Marks Ownership | High |
| `SUB-005` | 13 Subject | Subject Type, Credit & Weightage | High |
| `SUB-006` | 13 Subject | Elective Selection Rules & Prerequisites | Medium |
| `SUB-007` | 13 Subject | Subject Retirement Preserves History | Medium |
| `TCH-001` | 21 Teacher | Teacher Specializes Staff | High |
| `TCH-002` | 21 Teacher | Qualification-Aware Subject Assignment | High |
| `TCH-003` | 21 Teacher | Workload Limits | Medium |
| `TCH-004` | 21 Teacher | Assignment Confers Ownership (Re-Resolves on Change) | Critical |
| `TCH-005` | 21 Teacher | Substitute = Temporary, Attributed Ownership Transfer | High |
| `TCH-006` | 21 Teacher | Mandatory Handover on Departure/Reassignment | High |
| `TCH-007` | 21 Teacher | Class-Teacher (Homeroom) Singularity | Medium |
| `WFL-001` | 28 Workflow Engine | Declarative Workflow Definitions | Critical |
| `WFL-002` | 28 Workflow Engine | Version-Pinning at Initiation (Fairness) | Critical |
| `WFL-003` | 28 Workflow Engine | Deterministic FSM Execution | Critical |
| `WFL-004` | 28 Workflow Engine | Dynamic Approver Resolution with Separation of Duties | Critical |
| `WFL-005` | 28 Workflow Engine | Escalation on Timeout | High |
| `WFL-006` | 28 Workflow Engine | Bounded, Audited Delegation | High |
| `WFL-007` | 28 Workflow Engine | Safe Timeout Policies — No Silent Auto-Approval of Sensitive Actions | Critical |
| `WFL-008` | 28 Workflow Engine | Sequential, Parallel & Conditional Routing | High |
| `WFL-009` | 28 Workflow Engine | Withdrawal, Cancellation & Reassignment | Medium |
| `WFL-010` | 28 Workflow Engine | Reliability, Idempotency & No Orphaned Instances | Critical |
| `WFL-011` | 28 Workflow Engine | Bounded Return-for-Correction Loops | Medium |

## B. Index by Priority

Critical rules are the load-bearing integrity/safety rules; they must be covered by tests first.

### Critical (87)

`ADM-003`, `ADM-007`, `AUD-001`, `AUD-002`, `AUD-004`, `AUD-005`, `AUTH-001`, `AUTH-002`, `AUTH-004`, `AUTH-005`, `AUTH-009`, `AUTHZ-001`, `AUTHZ-002`, `AUTHZ-003`, `AUTHZ-004`, `AUTHZ-005`, `CAMP-001`, `CAMP-004`, `CFG-001`, `CFG-002`, `CFG-003`, `CFG-004`, `CFG-007`, `CFG-009`, `CFG-011`, `CLS-001`, `CLS-005`, `DSC-001`, `DSC-003`, `DSC-004`, `ENR-001`, `ENR-002`, `EXM-002`, `EXM-004`, `EXM-005`, `EXM-007`, `FEE-001`, `FEE-003`, `FILE-001`, `FILE-003`, `FILE-004`, `GRD-001`, `GRD-002`, `GRD-003`, `GRD-007`, `GRD-N-002`, `GRD-N-004`, `GRD-N-006`, `GRD-N-008`, `HR-006`, `INST-002`, `INST-006`, `INST-008`, `NOT-002`, `NOT-005`, `PAY-001`, `PAY-002`, `PAY-007`, `PAY-008`, `REP-002`, `REP-005`, `RES-001`, `RES-002`, `RES-003`, `RES-005`, `RES-006`, `SCH-001`, `SEC-001`, `SESS-002`, `SESS-003`, `SESS-004`, `SESS-005`, `STF-001`, `STF-007`, `STF-008`, `STU-001`, `STU-005`, `STU-006`, `STU-007`, `SUB-002`, `TCH-004`, `WFL-001`, `WFL-002`, `WFL-003`, `WFL-004`, `WFL-007`, `WFL-010`

### High (121)

`ADM-001`, `ADM-002`, `ADM-004`, `ADM-006`, `ATT-001`, `ATT-002`, `ATT-003`, `ATT-004`, `ATT-005`, `ATT-007`, `ATT-008`, `AUD-003`, `AUD-006`, `AUD-007`, `AUD-008`, `AUTH-003`, `AUTH-006`, `AUTH-007`, `AUTH-008`, `AUTH-010`, `AUTH-011`, `AUTHZ-006`, `AUTHZ-007`, `AUTHZ-008`, `CAMP-002`, `CAMP-006`, `CAMP-007`, `CFG-005`, `CFG-006`, `CFG-008`, `CFG-010`, `CLS-002`, `CLS-003`, `DSC-002`, `DSC-005`, `DSC-006`, `ENR-003`, `ENR-005`, `ENR-006`, `ENR-007`, `ENR-008`, `EXM-001`, `EXM-003`, `EXM-006`, `EXM-008`, `EXM-009`, `FEE-002`, `FEE-004`, `FEE-007`, `FEE-008`, `FILE-002`, `FILE-005`, `FILE-007`, `GRD-004`, `GRD-006`, `GRD-008`, `GRD-N-001`, `GRD-N-007`, `HR-002`, `HR-003`, `HR-004`, `HR-005`, `HR-007`, `INST-001`, `INST-003`, `INST-004`, `INST-007`, `LEV-001`, `LEV-002`, `LEV-003`, `LEV-004`, `LEV-005`, `LEV-006`, `LEV-009`, `NOT-001`, `NOT-003`, `NOT-004`, `NOT-006`, `PAY-004`, `PAY-005`, `PAY-009`, `REP-001`, `REP-003`, `REP-006`, `REP-008`, `RES-004`, `RES-007`, `RES-009`, `SCH-002`, `SCH-003`, `SCH-004`, `SCH-005`, `SCH-007`, `SEC-003`, `SEC-004`, `SEC-005`, `SEC-006`, `SESS-001`, `SESS-006`, `SESS-007`, `SESS-008`, `STF-002`, `STF-003`, `STF-004`, `STF-005`, `STF-006`, `STU-002`, `STU-003`, `STU-004`, `STU-008`, `SUB-001`, `SUB-003`, `SUB-004`, `SUB-005`, `TCH-001`, `TCH-002`, `TCH-005`, `TCH-006`, `WFL-005`, `WFL-006`, `WFL-008`

### Medium (39)

`ADM-005`, `ADM-008`, `ATT-006`, `ATT-009`, `AUTHZ-009`, `CAMP-003`, `CAMP-005`, `CLS-004`, `CLS-006`, `DSC-007`, `ENR-004`, `FEE-005`, `FEE-006`, `FILE-006`, `FILE-008`, `GRD-005`, `GRD-N-003`, `GRD-N-005`, `HR-001`, `HR-008`, `INST-005`, `LEV-007`, `LEV-008`, `NOT-007`, `NOT-008`, `PAY-003`, `PAY-006`, `REP-004`, `REP-007`, `RES-008`, `SCH-006`, `SCH-008`, `SEC-002`, `SUB-006`, `SUB-007`, `TCH-003`, `TCH-007`, `WFL-009`, `WFL-011`

### Low (1)

`CLS-007`

## C. Index by Category

- **Access control** — `AUTHZ-001`
- **Access control (minors)** — `GRD-N-004`
- **Access integration** — `STF-002`
- **Account privacy** — `AUTH-009`
- **Accountability** — `AUD-006`, `REP-008`
- **Assessment integration** — `ATT-009`
- **Assessment integrity** — `EXM-002`, `EXM-006`, `RES-002`, `SUB-003`, `SUB-005`
- **Assignment integrity** — `TCH-002`
- **Authorization** — `GRD-N-005`
- **Authorization (minors)** — `FILE-004`
- **Authorization (ownership)** — `ATT-002`, `EXM-005`, `SEC-004`, `SUB-004`, `TCH-004`
- **Authorization / control** — `WFL-004`
- **Brute-force defense** — `AUTH-004`
- **Calendar integrity** — `ATT-006`
- **Capacity** — `ADM-004`, `CLS-007`, `ENR-003`, `SEC-003`
- **Child safety** — `GRD-N-002`
- **Child safety / communication** — `ATT-008`
- **Child safety / integrity** — `STU-006`
- **Child safety / legal** — `GRD-N-006`
- **Child safety / privacy** — `NOT-002`
- **Clarity / control** — `DSC-006`
- **Compliance** — `AUD-008`
- **Compliance (minors / GDPR)** — `GRD-N-008`
- **Compliance / safety** — `NOT-003`
- **Computation (commonly disputed)** — `LEV-003`
- **Computation integrity** — `FEE-007`, `GRD-004`, `RES-004`
- **Configuration** — `ADM-001`, `ATT-001`, `CLS-002`, `EXM-001`, `FEE-006`, `HR-001`, `INST-002`, `LEV-001`
- **Configuration / correctness** — `REP-001`
- **Configuration / data quality** — `STU-002`
- **Configuration / decoupling** — `NOT-001`
- **Configuration / fairness** — `DSC-002`
- **Configuration / integrity** — `FEE-001`, `GRD-001`
- **Configuration resolution** — `CAMP-005`, `INST-005`
- **Consistency** — `AUTHZ-006`
- **Consistency / performance** — `CFG-008`
- **Continuity** — `LEV-008`, `TCH-006`
- **Continuity / integrity** — `TCH-005`
- **Control** — `PAY-005`
- **Control / SoD** — `AUD-005`
- **Controls / fraud prevention** — `AUTHZ-009`
- **Correctness / transparency** — `REP-004`
- **Credential issuance** — `AUTH-001`
- **Credential strength** — `AUTH-008`
- **Curriculum flexibility** — `SUB-006`
- **Data integrity** — `CAMP-006`, `EXM-004`, `STF-003`, `STU-003`
- **Data integrity / compliance** — `INST-008`, `STU-007`
- **Data model integrity** — `TCH-001`
- **Data protection** — `REP-005`
- **Data quality** — `CFG-002`, `GRD-N-003`, `PAY-003`
- **Data quality / flexibility** — `CFG-010`
- **Definition/instance integrity** — `CLS-001`
- **Definition/instance separation (D38)** — `SESS-004`
- **Domain clarity / integrity** — `SCH-001`
- **Domain separation** — `LEV-009`
- **Eligibility / fairness** — `EXM-003`
- **Entitlement integrity** — `LEV-002`
- **Escalation prevention** — `AUTHZ-005`
- **Evaluation policy** — `AUTHZ-004`
- **Expressiveness / integrity** — `WFL-008`
- **Fairness** — `ADM-005`, `FEE-005`, `RES-008`, `SCH-002`
- **Fairness (dispute prevention)** — `RES-003`
- **Fairness (top dispute source)** — `GRD-003`
- **Fairness / control** — `DSC-005`, `SCH-006`
- **Fairness / feasibility** — `TCH-003`
- **Fairness / finance integration** — `RES-007`
- **Fairness / integrity** — `EXM-008`, `WFL-002`
- **Finance integration** — `ADM-006`
- **Financial control** — `DSC-003`
- **Financial control / fraud prevention** — `HR-006`
- **Financial integrity** — `FEE-003`, `PAY-001`, `PAY-007`
- **Flexibility** — `GRD-005`
- **Fund integrity** — `SCH-005`
- **Governance** — `AUTHZ-008`, `GRD-N-007`, `LEV-007`
- **Governance / flexibility** — `WFL-001`
- **Governance / integrity** — `CFG-001`, `CFG-007`, `SCH-003`
- **Historical fidelity** — `GRD-007`
- **Identity** — `CAMP-003`, `ENR-004`, `INST-001`, `SEC-002`, `STF-001`, `STU-001`, `SUB-001`
- **Identity / integrity** — `ADM-002`
- **Identity / responsibility** — `GRD-N-001`
- **Integration** — `LEV-006`
- **Integration / fairness** — `ATT-005`
- **Integrity** — `ADM-007`, `ADM-008`, `ATT-003`, `ATT-007`, `AUD-007`, `DSC-004`, `DSC-007`, `ENR-002`, `ENR-005`, `FEE-004`, `GRD-002`, `GRD-006`, `HR-008`, `INST-004`, `PAY-002`, `PAY-004`, `PAY-006`, `PAY-009`, `RES-005`, `SCH-004`, `SCH-008`, `SEC-006`, `SUB-007`, `WFL-003`, `WFL-011`
- **Integrity (definition/instance)** — `ENR-001`
- **Integrity (non-repudiation)** — `AUD-001`
- **Integrity (tamper-evidence)** — `EXM-007`
- **Integrity / continuity** — `STF-008`
- **Integrity / governance** — `ATT-004`, `GRD-008`, `RES-006`, `SESS-008`, `STU-004`
- **Integrity / integration** — `HR-004`
- **Integrity / privacy** — `FILE-006`
- **Integrity / reproducibility** — `FILE-007`, `RES-001`
- **Integrity / safety** — `FILE-002`
- **Integrity / transparency** — `DSC-001`
- **Isolation** — `CAMP-004`, `INST-006`, `REP-006`
- **Isolation / onboarding** — `CFG-011`
- **Lifecycle** — `ENR-006`, `HR-002`, `WFL-009`
- **Lifecycle (commonly mis-modeled)** — `SESS-002`
- **Lifecycle / access** — `INST-007`
- **Lifecycle / cost** — `FILE-008`
- **Lifecycle / fairness** — `SCH-007`
- **Lifecycle / integrity** — `FEE-008`, `HR-007`
- **Lifecycle gating** — `INST-003`, `SESS-007`
- **Lifecycle integrity** — `CAMP-007`, `ENR-008`, `STF-004`, `STU-008`
- **Lifecycle invariant** — `CAMP-002`, `SESS-003`
- **Membership management** — `AUTHZ-007`
- **Money correctness** — `PAY-008`
- **Multi-institute isolation** — `AUTHZ-002`
- **Observability** — `NOT-008`
- **Onboarding** — `AUTH-011`
- **Organization** — `STF-006`
- **Ownership clarity** — `TCH-007`
- **Payroll integrity** — `HR-003`, `HR-005`
- **Performance** — `NOT-006`, `REP-007`
- **Performance / reliability** — `RES-009`
- **Privacy** — `STF-007`
- **Privacy (minors)** — `NOT-004`, `STU-005`
- **Privacy (minors/financial/staff)** — `REP-003`
- **Process integrity** — `EXM-009`
- **Progression integrity** — `CLS-005`
- **Promotion** — `ENR-007`
- **Promotion logic** — `SESS-006`
- **Record integrity** — `SESS-005`
- **Reliability** — `AUD-002`, `NOT-005`, `WFL-005`, `WFL-010`
- **Reliability / accountability** — `WFL-006`
- **Reproducibility** — `CFG-004`
- **Resolution integrity** — `CFG-003`
- **Row-level access** — `AUTHZ-003`
- **Safety** — `WFL-007`
- **Safety / governance** — `CFG-006`
- **Scope** — `CLS-004`, `STF-005`
- **Scope integrity** — `CAMP-001`
- **Security** — `CFG-009`, `FILE-003`, `FILE-005`
- **Security (minors/privacy)** — `FILE-001`
- **Security (primary exposure vector)** — `REP-002`
- **Security / privacy** — `AUD-004`
- **Session control** — `AUTH-005`
- **Session management** — `AUTH-006`
- **Session security** — `AUTH-002`, `AUTH-010`
- **Session security / reliability** — `AUTH-003`
- **Strong authentication** — `AUTH-007`
- **Structure** — `FEE-002`
- **Structure (market-specific)** — `SEC-005`
- **Structure flexibility** — `CLS-006`
- **Structure integrity** — `SEC-001`, `SUB-002`
- **Structure invariant** — `CLS-003`
- **Temporal integrity** — `CFG-005`, `SESS-001`
- **Usefulness** — `AUD-003`
- **User respect** — `NOT-007`
- **Workflow** — `LEV-004`
- **Workflow / fairness** — `ADM-003`
- **Workflow reliability** — `LEV-005`