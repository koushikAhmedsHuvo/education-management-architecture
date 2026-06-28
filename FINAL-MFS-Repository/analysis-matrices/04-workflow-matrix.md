# Workflow Matrix

> Modules that invoke the **Workflow Engine (28)** for governed approvals, and the workflow rules (`WFL-xxx`) each cites. The Workflow Engine is the single substrate for every approval/review/escalation across the platform (version-pinned, SoD-enforcing, no silent auto-approval).

| Mod | Module | Cites WFL rules | Governed processes (from spec §10) |
|---|---|---|---|
| 01 | Authentication Module | — | — |
| 02 | Authorization Module | WFL-004 | High-impact role/permission change approval |
| 03 | Institute Module | — | Institute config / type-lock changes |
| 04 | Campus Module | WFL-004 | Campus structural changes |
| 05 | Academic Session Module | WFL-002 | Session rollover / close & reopen |
| 06 | Admission Module | WFL-002, WFL-011 | Admission decision (version-pinned), return-for-correction, bulk decision |
| 07 | Student Module | — | Sensitive student-field & status change |
| 08 | Enrollment Module | WFL-002 | Inter-section transfer, withdrawal, cross-institution exit |
| 09 | Guardian Module | — | Guardianship change, custody-restriction removal |
| 10 | Attendance Module | WFL-002 | Attendance correction, backdated/window-override |
| 11 | Class Module | WFL-004 | Class structure change (promotion path, retirement) |
| 12 | Section Module | WFL-004 | Section closure / merge / split / restructure |
| 13 | Subject Module | WFL-004 | Elective selection; curriculum structural change |
| 14 | Examination Module | WFL-002 | Post-lock mark correction, moderation/grace |
| 15 | Result Processing Module | WFL-002 | Governed post-publish result revision (version-pinned, SoD) |
| 16 | Grading Module | WFL-002 | Grade-scale change, governed grade override |
| 17 | Fee Management Module | WFL-002 | Refund, fine waiver, credit note |
| 18 | Payment Module | WFL-002 | Payment reversal, mis-post correction |
| 19 | Discount Module | WFL-002 | Discretionary discount / waiver approval |
| 20 | Scholarship Module | WFL-002 | Scholarship selection (slots, COI) & disbursement |
| 21 | Teacher Module | WFL-004 | Over-workload exception, mandatory handover |
| 22 | Staff Module | WFL-004 | Sensitive staff change, separation |
| 23 | HR Module | WFL-002 | Payroll (compute→approve→disburse), salary change, separation/settlement |
| 24 | Leave Management Module | WFL-004, WFL-005, WFL-007 | Leave approval, escalation/delegation, backdated leave |
| 25 | Notification Module | — | Dead-letter discard (governed) |
| 26 | Reporting Module | WFL-002 | Bulk-PII export approval |
| 27 | Configuration Engine Module | WFL-002 | High-impact config change (immutable-after-use) |
| 28 | Workflow Engine Module | WFL-001, WFL-002, WFL-003, WFL-004, WFL-005, WFL-006, WFL-007, WFL-008, WFL-009, WFL-010, WFL-011, WFL-012, WFL-013, WFL-014, WFL-015, WFL-016, WFL-017, WFL-018, WFL-019 | (engine itself) definition-change approval |
| 29 | Audit Log Module | — | Regulator export, legal-hold removal |
| 30 | File Management Module | — | Secure deletion, quarantine release |

**23 of 30 modules explicitly cite WFL rules**; nearly every module initiates governed processes through the engine. The four most-cited workflow rules are WFL-002 (version-pinning), WFL-004 (approver resolution + SoD), WFL-005/006/007 (escalation/delegation/safe-timeout) — the fairness, control, and stall-proofing guarantees every approval inherits.