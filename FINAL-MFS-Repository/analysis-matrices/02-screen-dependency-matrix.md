# Screen Dependency Matrix

> Screen inventory per module (§5) and the **shared cross-cutting screen patterns** that recur across modules and resolve to the four platform/cross-cutting services. These patterns should be built **once as shared components** and reused — not re-implemented per module.

## Screen count per module

| Mod | Module | # Screens |
|---|---|---|
| 01 | Authentication Module | 16 |
| 02 | Authorization Module | 14 |
| 03 | Institute Module | 14 |
| 04 | Campus Module | 13 |
| 05 | Academic Session Module | 14 |
| 06 | Admission Module | 15 |
| 07 | Student Module | 15 |
| 08 | Enrollment Module | 15 |
| 09 | Guardian Module | 14 |
| 10 | Attendance Module | 12 |
| 11 | Class Module | 14 |
| 12 | Section Module | 12 |
| 13 | Subject Module | 15 |
| 14 | Examination Module | 15 |
| 15 | Result Processing Module | 13 |
| 16 | Grading Module | 12 |
| 17 | Fee Management Module | 17 |
| 18 | Payment Module | 17 |
| 19 | Discount Module | 12 |
| 20 | Scholarship Module | 12 |
| 21 | Teacher Module | 14 |
| 22 | Staff Module | 16 |
| 23 | HR Module | 14 |
| 24 | Leave Management Module | 14 |
| 25 | Notification Module | 10 |
| 26 | Reporting Module | 14 |
| 27 | Configuration Engine Module | 16 |
| 28 | Workflow Engine Module | 13 |
| 29 | Audit Log Module | 11 |
| 30 | File Management Module | 13 |
| — | **TOTAL** | **416** |

## Shared cross-cutting screen patterns (build once, reuse everywhere)

| Pattern | Modules exhibiting it | Count | Resolves to |
|---|---|---|---|
| Approvals / task inbox (→ Workflow Engine 28) | 02, 03, 04, 05, 06, 07, 08, 09, 10, 11, 12, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28 | 26 | Workflow Engine (28) task inbox |
| Signed-URL download / document view (→ File Mgmt 30) | 07, 14, 15, 18, 22, 23, 26, 30 | 8 | File Management (30) |
| Export (→ Reporting 26 / governed) | 01, 02, 03, 04, 05, 06, 07, 08, 09, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 29 | 28 | Reporting (26) export governance |
| Report / dashboard (→ Reporting 26) | 01, 02, 04, 05, 06, 07, 08, 09, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30 | 29 | Reporting (26) |
| Search / list | 01, 02, 03, 04, 05, 06, 07, 08, 09, 10, 11, 12, 13, 14, 16, 17, 18, 19, 20, 21, 22, 25, 26, 28, 29, 30 | 26 | Shared search/table component |
| Configuration (→ Configuration Engine 27) | 01, 02, 03, 04, 06, 10, 11, 15, 18, 19, 24, 25, 27, 30 | 14 | Configuration Engine (27) |
| Import / bulk | 01, 03, 04, 05, 06, 07, 08, 09, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 26, 27, 28, 30 | 27 | Shared import/bulk component |

## Implication

Seven screen archetypes account for the bulk of UI surface. Building them as **shared, parameterized components** (approval inbox, signed-URL viewer, export dialog, report/dashboard shell, search+table, config editor, import wizard) lets the 30 modules contribute only their domain-specific screens (forms, detail views, domain actions). This is the single largest front-end efficiency lever in the build and keeps cross-cutting guarantees (custody-aware access, scope/ownership injection, export governance) enforced in one place rather than re-derived per module.