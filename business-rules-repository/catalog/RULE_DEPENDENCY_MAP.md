# Rule Dependency Map

Derived from each rule's **Related Rules** field. Two views: **downstream** (what a rule depends on / feeds) and **upstream** (what depends on a rule). Rules with many upstream dependents are high-blast-radius — change them with extra care.

## A. Highest-impact rules (most depended-upon)

These rules are referenced by the most other rules; a change here ripples widely. Prioritise their stability and test coverage.

| Rule | Module | Dependents | Depended-on-by |
|---|---|---|---|
| `AUTHZ-003` Ownership / Relationship Predicates | 02 | 12 | `ATT-002`, `AUTHZ-002`, `AUTHZ-004`, `CAMP-004`, `EXM-005`, `FILE-004`, `GRD-N-004`, `REP-002`, `SEC-004`, `STU-004`, `SUB-004`, `TCH-004` |
| `SESS-005` Post-Close Immutability | 05 | 12 | `ATT-007`, `EXM-007`, `FEE-001`, `GRD-001`, `GRD-007`, `RES-006`, `SESS-003`, `SESS-006`, `SESS-007`, `SESS-008`, `STU-007`, `SUB-007` |
| `STU-005` Sensitive-Data Minimization & Masking | 06 | 11 | `AUD-004`, `CFG-009`, `FILE-001`, `FILE-004`, `FILE-006`, `GRD-N-008`, `NOT-004`, `REP-003`, `STF-007`, `STU-004`, `STU-006` |
| `AUTHZ-009` Separation of Duties (SoD) Hooks | 02 | 10 | `AUD-005`, `CFG-007`, `DSC-003`, `EXM-008`, `GRD-008`, `HR-006`, `SCH-003`, `SESS-008`, `STF-006`, `WFL-004` |
| `AUTH-005` Immediate Revocation on State Change | 01 | 8 | `AUD-007`, `AUTH-006`, `AUTH-010`, `HR-007`, `INST-007`, `STF-002`, `STF-004`, `STF-008` |
| `AUTHZ-002` Mandatory Scope Filter | 02 | 8 | `AUTHZ-003`, `AUTHZ-004`, `CAMP-004`, `CFG-011`, `INST-006`, `INST-007`, `REP-002`, `REP-006` |
| `FEE-003` Issued Invoices Are Immutable | 17 | 8 | `DSC-001`, `FEE-001`, `FEE-007`, `FILE-007`, `HR-006`, `PAY-001`, `PAY-007`, `SCH-008` |
| `INST-005` Configuration Inheritance & Override | 03 | 6 | `CAMP-005`, `CFG-003`, `CFG-011`, `FEE-001`, `GRD-005`, `INST-003` |
| `STU-002` Core vs Dynamic Profile Fields | 06 | 6 | `ADM-001`, `CFG-002`, `CFG-010`, `STU-001`, `STU-004`, `STU-005` |
| `RES-002` Layered Pass/Fail (Component → Subject → Aggregate) | 15 | 6 | `EXM-002`, `EXM-006`, `EXM-008`, `GRD-006`, `RES-004`, `RES-008` |
| `RES-006` Governed Post-Publish Revision | 15 | 6 | `CFG-006`, `EXM-007`, `FILE-007`, `GRD-008`, `RES-009`, `WFL-007` |
| `PAY-007` Payments Are Reversed, Never Deleted | 18 | 6 | `DSC-007`, `FEE-003`, `FEE-008`, `PAY-001`, `PAY-004`, `PAY-009` |
| `STF-008` Archive/Anonymize, Never Hard-Delete; Handover on Separation | 22 | 6 | `FILE-008`, `HR-007`, `HR-008`, `STF-001`, `STF-004`, `TCH-006` |
| `HR-005` Payroll Computation Integrity | 23 | 6 | `HR-001`, `HR-003`, `HR-004`, `HR-006`, `LEV-006`, `STF-007` |
| `CFG-004` Versioning & Immutable Published Versions | 27 | 6 | `CFG-003`, `CFG-006`, `CFG-008`, `CFG-010`, `REP-001`, `WFL-002` |
| `INST-002` Institution Type Drives Templates, Not Code | 03 | 5 | `CFG-011`, `CLS-002`, `INST-001`, `INST-004`, `SESS-004` |
| `CAMP-001` Campus Belongs to Exactly One Institute | 04 | 5 | `CAMP-003`, `CAMP-004`, `CAMP-006`, `CLS-004`, `INST-003` |
| `ENR-001` Enrollment Binds Student to Session/Level/Section Instance | 08 | 5 | `CLS-003`, `ENR-002`, `ENR-004`, `ENR-007`, `SEC-001` |
| `ATT-009` Attendance Summaries & Exam-Eligibility Threshold | 10 | 5 | `ATT-005`, `ATT-006`, `EXM-003`, `REP-001`, `REP-004` |
| `EXM-007` Submit-and-Lock Immutability | 14 | 5 | `AUD-001`, `AUD-007`, `EXM-005`, `RES-001`, `RES-006` |

## B. Full dependency listing (per rule)

Format: `RULE` → depends on [downstream] · depended-on-by [upstream]

- `ADM-001` → `ADM-002`, `STU-002`  ·  ⬅ `ADM-002`, `CFG-010`
- `ADM-002` → `ADM-001`, `STU-003`  ·  ⬅ `ADM-001`, `ADM-008`
- `ADM-003` → `ADM-005`  ·  ⬅ `ADM-005`, `WFL-002`
- `ADM-004` → `ADM-005`  ·  ⬅ `ADM-005`, `CLS-007`, `ENR-003`, `SEC-003`
- `ADM-005` → `ADM-004`, `ADM-003`  ·  ⬅ `ADM-003`, `ADM-004`, `SCH-002`
- `ADM-006` → `ADM-007`  ·  ⬅ `ADM-007`
- `ADM-007` → `STU-001`, `STU-006`, `ADM-006`  ·  ⬅ `ADM-006`
- `ADM-008` → `ADM-002`, `STU-003`  ·  ⬅ `WFL-009`
- `ATT-001` → `ATT-002`, `ATT-003`  ·  ⬅ `SEC-005`
- `ATT-002` → `SEC-004`, `SUB-004`, `AUTHZ-003`  ·  ⬅ `ATT-001`, `TCH-004`, `TCH-005`, `TCH-007`
- `ATT-003` → `ENR-005`, `ENR-008`  ·  ⬅ `ATT-001`, `SEC-004`, `SUB-004`
- `ATT-004` → `ATT-007`  ·  ⬅ `ATT-007`, `LEV-007`
- `ATT-005` → `ATT-009`  ·  ⬅ `ATT-008`, `ATT-009`, `LEV-009`
- `ATT-006` → `ATT-009`  ·  ⬅ `ATT-009`
- `ATT-007` → `ATT-004`, `SESS-005`  ·  ⬅ `ATT-004`
- `ATT-008` → `GRD-N-006`, `ATT-005`  ·  ⬅ `LEV-009`, `NOT-002`, `NOT-003`
- `ATT-009` → `ATT-006`, `ATT-005`  ·  ⬅ `ATT-005`, `ATT-006`, `EXM-003`, `REP-001`, `REP-004`
- `AUD-001` → `AUD-002`, `AUD-005`, `EXM-007`  ·  ⬅ `AUD-003`, `AUD-005`, `AUD-008`
- `AUD-002` → `AUD-007`, `NOT-005`  ·  ⬅ `AUD-001`, `AUD-007`
- `AUD-003` → `AUD-001`
- `AUD-004` → `AUTH-008`, `CFG-009`, `STU-005`, `STF-007`
- `AUD-005` → `AUTHZ-009`, `AUD-001`, `AUD-006`  ·  ⬅ `AUD-001`, `AUD-006`
- `AUD-006` → `AUD-005`, `REP-008`  ·  ⬅ `AUD-005`
- `AUD-007` → `AUD-002`, `EXM-007`, `AUTH-005`  ·  ⬅ `AUD-002`
- `AUD-008` → `STU-007`, `AUD-001`  ·  ⬅ `FILE-008`
- `AUTH-001` → `AUTH-002`, `AUTH-006`, `AUTHZ-006`  ·  ⬅ `AUTH-004`, `AUTH-007`, `AUTHZ-006`
- `AUTH-002` → `AUTH-003`, `AUTH-006`  ·  ⬅ `AUTH-001`, `AUTH-003`, `AUTH-005`, `AUTH-010`
- `AUTH-003` → `AUTH-002`  ·  ⬅ `AUTH-002`
- `AUTH-004` → `AUTH-001`, `AUTH-008`  ·  ⬅ `AUTH-007`
- `AUTH-005` → `AUTH-002`, `AUTH-006`, `INST-007`  ·  ⬅ `AUD-007`, `AUTH-006`, `AUTH-010`, `HR-007`, `INST-007`, `STF-002`, `STF-004`, `STF-008`
- `AUTH-006` → `AUTH-005`, `AUTH-007`  ·  ⬅ `AUTH-001`, `AUTH-002`, `AUTH-005`
- `AUTH-007` → `AUTH-001`, `AUTH-004`  ·  ⬅ `AUTH-006`
- `AUTH-008` → `AUTH-009`, `AUTH-010`  ·  ⬅ `AUD-004`, `AUTH-004`, `AUTH-009`, `AUTH-011`
- `AUTH-009` → `AUTH-008`, `AUTH-010`  ·  ⬅ `AUTH-008`
- `AUTH-010` → `AUTH-002`, `AUTH-005`  ·  ⬅ `AUTH-008`, `AUTH-009`
- `AUTH-011` → `AUTH-008`, `AUTHZ-007`  ·  ⬅ `STF-002`
- `AUTHZ-001` → `AUTHZ-006`, `AUTHZ-008`  ·  ⬅ `AUTHZ-004`, `AUTHZ-006`, `AUTHZ-008`
- `AUTHZ-002` → `AUTHZ-003`, `INST-006`, `CAMP-004`  ·  ⬅ `AUTHZ-003`, `AUTHZ-004`, `CAMP-004`, `CFG-011`, `INST-006`, `INST-007`, `REP-002`, `REP-006`
- `AUTHZ-003` → `AUTHZ-002`  ·  ⬅ `ATT-002`, `AUTHZ-002`, `AUTHZ-004`, `CAMP-004`, `EXM-005`, `FILE-004`, `GRD-N-004`, `REP-002`, `SEC-004`, `STU-004`, `SUB-004`, `TCH-004`
- `AUTHZ-004` → `AUTHZ-001`, `AUTHZ-002`, `AUTHZ-003`
- `AUTHZ-005` → `AUTHZ-007`, `AUTHZ-008`  ·  ⬅ `AUTHZ-007`
- `AUTHZ-006` → `AUTHZ-001`, `AUTH-001`  ·  ⬅ `AUTH-001`, `AUTHZ-001`, `CFG-008`
- `AUTHZ-007` → `AUTHZ-005`, `CAMP-003`  ·  ⬅ `AUTH-011`, `AUTHZ-005`, `STF-002`
- `AUTHZ-008` → `AUTHZ-001`  ·  ⬅ `AUTHZ-001`, `AUTHZ-005`, `CFG-001`
- `AUTHZ-009`  ·  ⬅ `AUD-005`, `CFG-007`, `DSC-003`, `EXM-008`, `GRD-008`, `HR-006`, `SCH-003`, `SESS-008`, `STF-006`, `WFL-004`
- `CAMP-001` → `INST-006`, `CAMP-004`  ·  ⬅ `CAMP-003`, `CAMP-004`, `CAMP-006`, `CLS-004`, `INST-003`
- `CAMP-002` → `INST-003`, `CAMP-007`  ·  ⬅ `CAMP-007`
- `CAMP-003` → `INST-001`, `CAMP-001`  ·  ⬅ `AUTHZ-007`
- `CAMP-004` → `AUTHZ-002`, `AUTHZ-003`, `CAMP-001`  ·  ⬅ `AUTHZ-002`, `CAMP-001`, `INST-006`, `STF-005`
- `CAMP-005` → `INST-005`  ·  ⬅ `CFG-003`, `INST-005`
- `CAMP-006` → `CAMP-001`  ·  ⬅ `CAMP-007`, `ENR-005`, `SEC-006`
- `CAMP-007` → `CAMP-002`, `CAMP-006`, `SESS-007`  ·  ⬅ `CAMP-002`
- `CFG-001` → `CFG-002`, `AUTHZ-008`  ·  ⬅ `CFG-002`, `WFL-001`
- `CFG-002` → `CFG-001`, `GRD-002`, `STU-002`  ·  ⬅ `CFG-001`, `CFG-010`
- `CFG-003` → `INST-005`, `CAMP-005`, `CFG-004`
- `CFG-004` → `GRD-007`, `FEE-001`, `HR-003`, `RES-001`  ·  ⬅ `CFG-003`, `CFG-006`, `CFG-008`, `CFG-010`, `REP-001`, `WFL-002`
- `CFG-005` → `FEE-001`, `GRD-001`, `SESS-002`
- `CFG-006` → `CFG-004`, `RES-006`, `GRD-008`
- `CFG-007` → `AUTHZ-009`, `INST-004`, `GRD-001`, `STU-001`
- `CFG-008` → `AUTHZ-006`, `CFG-004`
- `CFG-009` → `STF-007`, `STU-005`  ·  ⬅ `AUD-004`, `NOT-004`
- `CFG-010` → `STU-002`, `ADM-001`, `CFG-002`, `CFG-004`  ·  ⬅ `NOT-001`
- `CFG-011` → `INST-002`, `INST-005`, `AUTHZ-002`  ·  ⬅ `REP-006`
- `CLS-001` → `SESS-004`, `CLS-003`
- `CLS-002` → `CLS-005`, `INST-002`
- `CLS-003` → `SEC-001`, `ENR-001`  ·  ⬅ `CLS-001`, `CLS-004`, `SEC-001`
- `CLS-004` → `CAMP-001`, `CLS-003`
- `CLS-005` → `SESS-006`, `ENR-007`  ·  ⬅ `CLS-002`
- `CLS-006` → `SEC-001`, `SUB-005`  ·  ⬅ `SUB-002`
- `CLS-007` → `SEC-003`, `ADM-004`, `ENR-003`  ·  ⬅ `SEC-003`
- `DSC-001` → `FEE-003`, `DSC-004`  ·  ⬅ `DSC-004`, `SCH-001`
- `DSC-002` → `GRD-N-003`, `DSC-007`
- `DSC-003` → `AUTHZ-009`, `DSC-006`  ·  ⬅ `DSC-006`, `WFL-007`, `WFL-008`
- `DSC-004` → `DSC-001`  ·  ⬅ `DSC-001`, `DSC-005`, `SCH-004`
- `DSC-005` → `DSC-004`, `SCH-006`  ·  ⬅ `SCH-006`
- `DSC-006` → `DSC-003`, `FEE-002`  ·  ⬅ `DSC-003`
- `DSC-007` → `FEE-008`, `PAY-007`  ·  ⬅ `DSC-002`
- `ENR-001` → `SESS-004`, `ENR-002`  ·  ⬅ `CLS-003`, `ENR-002`, `ENR-004`, `ENR-007`, `SEC-001`
- `ENR-002` → `ENR-001`, `ENR-005`  ·  ⬅ `ENR-001`, `ENR-005`, `ENR-008`
- `ENR-003` → `ADM-004`  ·  ⬅ `CLS-007`, `SEC-003`
- `ENR-004` → `ENR-001`
- `ENR-005` → `CAMP-006`, `ENR-002`  ·  ⬅ `ATT-003`, `ENR-002`, `FEE-005`, `SEC-006`
- `ENR-006` → `STU-007`  ·  ⬅ `FEE-008`
- `ENR-007` → `SESS-006`, `ENR-001`  ·  ⬅ `CLS-005`
- `ENR-008` → `ENR-002`  ·  ⬅ `ATT-003`, `LEV-009`
- `EXM-001` → `EXM-008`, `RES-001`, `GRD-007`
- `EXM-002` → `SUB-003`, `EXM-004`, `RES-002`  ·  ⬅ `EXM-004`, `RES-002`
- `EXM-003` → `ATT-009`
- `EXM-004` → `EXM-002`, `EXM-005`  ·  ⬅ `EXM-002`
- `EXM-005` → `SUB-004`, `AUTHZ-003`, `EXM-007`  ·  ⬅ `EXM-004`, `EXM-007`, `TCH-004`
- `EXM-006` → `RES-002`, `RES-007`  ·  ⬅ `EXM-009`, `GRD-006`, `RES-007`
- `EXM-007` → `EXM-005`, `RES-006`, `SESS-005`  ·  ⬅ `AUD-001`, `AUD-007`, `EXM-005`, `RES-001`, `RES-006`
- `EXM-008` → `AUTHZ-009`, `RES-002`  ·  ⬅ `EXM-001`, `GRD-008`
- `EXM-009` → `EXM-006`, `RES-001`, `RES-005`  ·  ⬅ `RES-005`, `TCH-006`
- `FEE-001` → `FEE-003`, `INST-005`, `SESS-005`  ·  ⬅ `CFG-004`, `CFG-005`, `FEE-006`, `HR-003`
- `FEE-002` → `FEE-005`, `FEE-007`  ·  ⬅ `DSC-006`, `FEE-005`
- `FEE-003` → `FEE-008`, `PAY-007`  ·  ⬅ `DSC-001`, `FEE-001`, `FEE-007`, `FILE-007`, `HR-006`, `PAY-001`, `PAY-007`, `SCH-008`
- `FEE-004` → `FEE-007`, `PAY-008`  ·  ⬅ `PAY-008`
- `FEE-005` → `ENR-005`, `FEE-002`  ·  ⬅ `FEE-002`
- `FEE-006` → `FEE-001`  ·  ⬅ `SCH-006`
- `FEE-007` → `FEE-003`, `PAY-002`  ·  ⬅ `FEE-002`, `FEE-004`, `PAY-002`, `REP-001`
- `FEE-008` → `SESS-007`, `PAY-007`, `ENR-006`  ·  ⬅ `DSC-007`, `FEE-003`, `PAY-005`, `PAY-006`, `PAY-007`
- `FILE-001` → `FILE-004`, `STU-005`  ·  ⬅ `FILE-003`, `FILE-005`
- `FILE-002` → `FILE-003`  ·  ⬅ `FILE-003`
- `FILE-003` → `FILE-001`, `FILE-002`  ·  ⬅ `FILE-002`
- `FILE-004` → `AUTHZ-003`, `GRD-N-006`, `STU-005`  ·  ⬅ `FILE-001`, `FILE-005`, `FILE-006`
- `FILE-005` → `FILE-001`, `FILE-004`
- `FILE-006` → `STU-005`, `FILE-004`
- `FILE-007` → `RES-006`, `GRD-007`, `FEE-003`, `HR-003`
- `FILE-008` → `STU-007`, `STF-008`, `AUD-008`
- `GRD-001` → `GRD-007`, `SESS-005`  ·  ⬅ `CFG-005`, `CFG-007`, `GRD-002`, `GRD-005`, `GRD-007`
- `GRD-002` → `GRD-003`, `GRD-001`  ·  ⬅ `CFG-002`, `GRD-003`
- `GRD-003` → `GRD-002`, `RES-003`  ·  ⬅ `GRD-002`, `RES-003`
- `GRD-004` → `SUB-005`, `GRD-006`, `RES-004`  ·  ⬅ `GRD-006`, `RES-004`
- `GRD-005` → `GRD-001`, `INST-005`
- `GRD-006` → `EXM-006`, `RES-002`, `GRD-004`  ·  ⬅ `GRD-004`
- `GRD-007` → `GRD-001`, `RES-008`, `SESS-005`  ·  ⬅ `CFG-004`, `EXM-001`, `FILE-007`, `GRD-001`, `RES-001`
- `GRD-008` → `AUTHZ-009`, `EXM-008`, `RES-006`  ·  ⬅ `CFG-006`, `RES-006`
- `GRD-N-001` → `GRD-N-002`, `GRD-N-003`  ·  ⬅ `GRD-N-003`, `GRD-N-005`
- `GRD-N-002` → `STU-006`, `GRD-N-007`  ·  ⬅ `GRD-N-001`, `GRD-N-004`, `GRD-N-007`
- `GRD-N-003` → `GRD-N-001`, `STU-003`  ·  ⬅ `DSC-002`, `GRD-N-001`, `PAY-009`
- `GRD-N-004` → `AUTHZ-003`, `GRD-N-002`  ·  ⬅ `GRD-N-006`
- `GRD-N-005` → `GRD-N-001`  ·  ⬅ `GRD-N-006`, `NOT-002`
- `GRD-N-006` → `GRD-N-004`, `GRD-N-005`  ·  ⬅ `ATT-008`, `FILE-004`, `GRD-N-007`, `NOT-002`
- `GRD-N-007` → `GRD-N-002`, `GRD-N-006`  ·  ⬅ `GRD-N-002`
- `GRD-N-008` → `STU-005`
- `HR-001` → `STF-005`, `STF-006`, `HR-005`
- `HR-002` → `STF-004`, `HR-007`  ·  ⬅ `HR-008`, `LEV-001`
- `HR-003` → `HR-005`, `FEE-001`, `STF-007`  ·  ⬅ `CFG-004`, `FILE-007`, `HR-005`
- `HR-004` → `HR-005`  ·  ⬅ `HR-005`, `LEV-006`
- `HR-005` → `HR-003`, `HR-004`, `HR-006`  ·  ⬅ `HR-001`, `HR-003`, `HR-004`, `HR-006`, `LEV-006`, `STF-007`
- `HR-006` → `AUTHZ-009`, `HR-005`, `FEE-003`  ·  ⬅ `HR-005`, `LEV-006`, `LEV-007`
- `HR-007` → `STF-008`, `TCH-006`, `AUTH-005`  ·  ⬅ `HR-002`, `LEV-002`, `STF-008`
- `HR-008` → `STF-003`, `STF-008`, `HR-002`  ·  ⬅ `STF-003`
- `INST-001` → `INST-002`  ·  ⬅ `CAMP-003`
- `INST-002` → `INST-004`  ·  ⬅ `CFG-011`, `CLS-002`, `INST-001`, `INST-004`, `SESS-004`
- `INST-003` → `INST-005`, `CAMP-001`, `SESS-001`  ·  ⬅ `CAMP-002`
- `INST-004` → `INST-002`, `INST-008`  ·  ⬅ `CFG-007`, `INST-002`, `INST-008`
- `INST-005` → `CAMP-005`  ·  ⬅ `CAMP-005`, `CFG-003`, `CFG-011`, `FEE-001`, `GRD-005`, `INST-003`
- `INST-006` → `AUTHZ-002`, `CAMP-004`  ·  ⬅ `AUTHZ-002`, `CAMP-001`, `REP-006`, `STF-005`
- `INST-007` → `AUTH-005`, `AUTHZ-002`  ·  ⬅ `AUTH-005`
- `INST-008` → `INST-004`, `SESS-007`  ·  ⬅ `INST-004`
- `LEV-001` → `LEV-002`, `HR-002`  ·  ⬅ `LEV-002`
- `LEV-002` → `LEV-001`, `HR-007`  ·  ⬅ `LEV-001`, `LEV-003`
- `LEV-003` → `LEV-002`
- `LEV-004` → `STF-006`, `LEV-005`  ·  ⬅ `LEV-005`, `LEV-008`
- `LEV-005` → `STF-006`, `LEV-004`  ·  ⬅ `LEV-004`, `WFL-005`, `WFL-006`
- `LEV-006` → `HR-004`, `HR-005`, `HR-006`
- `LEV-007` → `ATT-004`, `HR-006`
- `LEV-008` → `TCH-005`, `LEV-004`
- `LEV-009` → `ATT-005`, `ATT-008`, `ENR-008`
- `NOT-001` → `CFG-010`, `NOT-006`
- `NOT-002` → `GRD-N-005`, `GRD-N-006`, `ATT-008`  ·  ⬅ `NOT-004`
- `NOT-003` → `ATT-008`  ·  ⬅ `NOT-007`
- `NOT-004` → `STU-005`, `CFG-009`, `NOT-002`  ·  ⬅ `NOT-007`
- `NOT-005` → `RES-009`, `NOT-008`  ·  ⬅ `AUD-002`, `NOT-006`, `NOT-008`, `REP-007`
- `NOT-006` → `RES-009`, `NOT-005`  ·  ⬅ `NOT-001`
- `NOT-007` → `NOT-003`, `NOT-004`
- `NOT-008` → `NOT-005`  ·  ⬅ `NOT-005`
- `PAY-001` → `PAY-007`, `FEE-003`  ·  ⬅ `PAY-007`
- `PAY-002` → `FEE-007`, `PAY-006`  ·  ⬅ `FEE-007`, `PAY-006`
- `PAY-003` → `PAY-004`, `PAY-005`  ·  ⬅ `PAY-004`
- `PAY-004` → `PAY-003`, `PAY-007`  ·  ⬅ `PAY-003`
- `PAY-005` → `PAY-008`, `FEE-008`  ·  ⬅ `PAY-003`, `PAY-008`
- `PAY-006` → `FEE-008`, `PAY-002`  ·  ⬅ `PAY-002`
- `PAY-007` → `PAY-001`, `FEE-008`, `FEE-003`  ·  ⬅ `DSC-007`, `FEE-003`, `FEE-008`, `PAY-001`, `PAY-004`, `PAY-009`
- `PAY-008` → `FEE-004`, `PAY-005`  ·  ⬅ `FEE-004`, `PAY-005`, `WFL-010`
- `PAY-009` → `PAY-007`, `GRD-N-003`
- `REP-001` → `ATT-009`, `FEE-007`, `CFG-004`
- `REP-002` → `AUTHZ-002`, `AUTHZ-003`, `REP-005`  ·  ⬅ `REP-005`
- `REP-003` → `STU-005`, `STF-007`, `REP-005`  ·  ⬅ `REP-005`
- `REP-004` → `RES-009`, `ATT-009`
- `REP-005` → `REP-002`, `REP-003`  ·  ⬅ `REP-002`, `REP-003`, `REP-008`
- `REP-006` → `AUTHZ-002`, `INST-006`, `CFG-011`
- `REP-007` → `RES-009`, `NOT-005`
- `REP-008` → `REP-005`  ·  ⬅ `AUD-006`
- `RES-001` → `EXM-007`, `GRD-007`, `RES-003`  ·  ⬅ `CFG-004`, `EXM-001`, `EXM-009`, `RES-003`
- `RES-002` → `SUB-003`, `EXM-002`, `RES-004`  ·  ⬅ `EXM-002`, `EXM-006`, `EXM-008`, `GRD-006`, `RES-004`, `RES-008`
- `RES-003` → `GRD-003`, `RES-001`  ·  ⬅ `GRD-003`, `RES-001`
- `RES-004` → `GRD-004`, `RES-002`  ·  ⬅ `GRD-004`, `RES-002`, `RES-008`
- `RES-005` → `EXM-009`, `RES-007`  ·  ⬅ `EXM-009`, `RES-007`
- `RES-006` → `EXM-007`, `GRD-008`, `RES-008`, `SESS-005`  ·  ⬅ `CFG-006`, `EXM-007`, `FILE-007`, `GRD-008`, `RES-009`, `WFL-007`
- `RES-007` → `EXM-006`, `RES-005`  ·  ⬅ `EXM-006`, `RES-005`
- `RES-008` → `RES-004`, `RES-002`  ·  ⬅ `GRD-007`, `RES-006`
- `RES-009` → `RES-006`  ·  ⬅ `NOT-005`, `NOT-006`, `REP-004`, `REP-007`
- `SCH-001` → `DSC-001`, `SCH-004`
- `SCH-002` → `SCH-003`, `ADM-005`  ·  ⬅ `SCH-003`
- `SCH-003` → `AUTHZ-009`, `SCH-002`  ·  ⬅ `SCH-002`, `WFL-008`
- `SCH-004` → `DSC-004`, `SCH-005`  ·  ⬅ `SCH-001`, `SCH-005`
- `SCH-005` → `SCH-004`  ·  ⬅ `SCH-004`
- `SCH-006` → `DSC-005`, `FEE-006`  ·  ⬅ `DSC-005`
- `SCH-007` → `SCH-008`  ·  ⬅ `SCH-008`
- `SCH-008` → `FEE-003`, `SCH-007`  ·  ⬅ `SCH-007`
- `SEC-001` → `CLS-003`, `ENR-001`, `SEC-005`  ·  ⬅ `CLS-003`, `CLS-006`, `SEC-002`, `SEC-005`
- `SEC-002` → `SEC-001`, `SEC-005`
- `SEC-003` → `ADM-004`, `ENR-003`, `CLS-007`  ·  ⬅ `CLS-007`, `SEC-006`
- `SEC-004` → `AUTHZ-003`, `ATT-003`  ·  ⬅ `ATT-002`, `TCH-004`, `TCH-007`
- `SEC-005` → `SEC-001`, `ATT-001`  ·  ⬅ `SEC-001`, `SEC-002`
- `SEC-006` → `ENR-005`, `CAMP-006`, `SEC-003`
- `SESS-001` → `SESS-002`, `SESS-003`  ·  ⬅ `INST-003`
- `SESS-002` → `SESS-003`  ·  ⬅ `CFG-005`, `SESS-001`, `SESS-003`
- `SESS-003` → `SESS-002`, `SESS-005`  ·  ⬅ `SESS-001`, `SESS-002`
- `SESS-004` → `INST-002`  ·  ⬅ `CLS-001`, `ENR-001`
- `SESS-005` → `SESS-007`  ·  ⬅ `ATT-007`, `EXM-007`, `FEE-001`, `GRD-001`, `GRD-007`, `RES-006`, `SESS-003`, `SESS-006`, `SESS-007`, `SESS-008`, `STU-007`, `SUB-007`
- `SESS-006` → `SESS-005`  ·  ⬅ `CLS-005`, `ENR-007`, `STU-008`
- `SESS-007` → `SESS-005`, `SESS-008`  ·  ⬅ `CAMP-007`, `FEE-008`, `INST-008`, `SESS-005`
- `SESS-008` → `SESS-005`, `AUTHZ-009`  ·  ⬅ `SESS-007`
- `STF-001` → `STF-003`, `STF-008`  ·  ⬅ `STF-003`, `TCH-001`
- `STF-002` → `AUTH-011`, `AUTH-005`, `AUTHZ-007`
- `STF-003` → `STF-001`, `HR-008`  ·  ⬅ `HR-008`, `STF-001`
- `STF-004` → `AUTH-005`, `STF-008`  ·  ⬅ `HR-002`, `TCH-001`
- `STF-005` → `CAMP-004`, `INST-006`, `TCH-002`  ·  ⬅ `HR-001`, `TCH-002`
- `STF-006` → `AUTHZ-009`  ·  ⬅ `HR-001`, `LEV-004`, `LEV-005`, `WFL-004`, `WFL-006`
- `STF-007` → `HR-005`, `STU-005`  ·  ⬅ `AUD-004`, `CFG-009`, `HR-003`, `REP-003`
- `STF-008` → `AUTH-005`, `TCH-005`, `HR-007`  ·  ⬅ `FILE-008`, `HR-007`, `HR-008`, `STF-001`, `STF-004`, `TCH-006`
- `STU-001` → `STU-002`, `STU-003`  ·  ⬅ `ADM-007`, `CFG-007`, `STU-003`
- `STU-002` → `STU-004`  ·  ⬅ `ADM-001`, `CFG-002`, `CFG-010`, `STU-001`, `STU-004`, `STU-005`
- `STU-003` → `STU-001`  ·  ⬅ `ADM-002`, `ADM-008`, `GRD-N-003`, `STU-001`
- `STU-004` → `STU-002`, `STU-005`, `AUTHZ-003`  ·  ⬅ `STU-002`
- `STU-005` → `STU-002`  ·  ⬅ `AUD-004`, `CFG-009`, `FILE-001`, `FILE-004`, `FILE-006`, `GRD-N-008`, `NOT-004`, `REP-003`, `STF-007`, `STU-004`, `STU-006`
- `STU-006` → `STU-005`  ·  ⬅ `ADM-007`, `GRD-N-002`
- `STU-007` → `STU-008`, `SESS-005`  ·  ⬅ `AUD-008`, `ENR-006`, `FILE-008`, `STU-008`
- `STU-008` → `STU-007`, `SESS-006`  ·  ⬅ `STU-007`
- `SUB-001` → `SUB-002`  ·  ⬅ `SUB-002`
- `SUB-002` → `SUB-001`, `CLS-006`  ·  ⬅ `SUB-001`, `SUB-006`, `SUB-007`, `TCH-002`
- `SUB-003`  ·  ⬅ `EXM-002`, `RES-002`
- `SUB-004` → `AUTHZ-003`, `ATT-003`  ·  ⬅ `ATT-002`, `EXM-005`, `TCH-002`, `TCH-004`
- `SUB-005` → `SUB-006`  ·  ⬅ `CLS-006`, `GRD-004`, `SUB-006`
- `SUB-006` → `SUB-002`, `SUB-005`  ·  ⬅ `SUB-005`
- `SUB-007` → `SUB-002`, `SESS-005`
- `TCH-001` → `STF-001`, `STF-004`
- `TCH-002` → `SUB-002`, `SUB-004`, `STF-005`  ·  ⬅ `STF-005`, `TCH-003`
- `TCH-003` → `TCH-002`
- `TCH-004` → `AUTHZ-003`, `SEC-004`, `SUB-004`, `EXM-005`, `ATT-002`  ·  ⬅ `TCH-005`, `TCH-006`
- `TCH-005` → `TCH-004`, `ATT-002`  ·  ⬅ `LEV-008`, `STF-008`
- `TCH-006` → `STF-008`, `EXM-009`, `TCH-004`  ·  ⬅ `HR-007`
- `TCH-007` → `SEC-004`, `ATT-002`
- `WFL-001` → `CFG-001`, `WFL-003`  ·  ⬅ `WFL-003`
- `WFL-002` → `ADM-003`, `CFG-004`
- `WFL-003` → `WFL-001`, `WFL-010`, `WFL-011`  ·  ⬅ `WFL-001`, `WFL-008`, `WFL-010`, `WFL-011`
- `WFL-004` → `AUTHZ-009`, `STF-006`, `WFL-005`  ·  ⬅ `WFL-006`
- `WFL-005` → `LEV-005`, `WFL-006`, `WFL-007`  ·  ⬅ `WFL-004`, `WFL-007`, `WFL-010`
- `WFL-006` → `WFL-004`, `LEV-005`, `STF-006`  ·  ⬅ `WFL-005`
- `WFL-007` → `WFL-005`, `RES-006`, `DSC-003`  ·  ⬅ `WFL-005`
- `WFL-008` → `SCH-003`, `DSC-003`, `WFL-003`
- `WFL-009` → `ADM-008`, `WFL-010`
- `WFL-010` → `WFL-003`, `WFL-005`, `PAY-008`  ·  ⬅ `WFL-003`, `WFL-009`
- `WFL-011` → `WFL-003`  ·  ⬅ `WFL-003`