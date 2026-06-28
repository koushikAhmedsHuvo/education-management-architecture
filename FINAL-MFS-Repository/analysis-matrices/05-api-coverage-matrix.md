# API Coverage Matrix

> Endpoint **purpose** coverage per module from §14 (method + actor only; request/response contracts are intentionally out of scope at the functional-spec layer — see Readiness Review §Missing Requirements).

| Mod | Module | Endpoints | GET | POST | PUT | DELETE |
|---|---|---|---|---|---|---|
| 01 | Authentication Module | 21 | 4 | 16 | 1 | 1 |
| 02 | Authorization Module | 10 | 7 | 6 | 2 | 3 |
| 03 | Institute Module | 10 | 4 | 6 | 2 | 0 |
| 04 | Campus Module | 10 | 4 | 6 | 2 | 0 |
| 05 | Academic Session Module | 11 | 4 | 7 | 1 | 0 |
| 06 | Admission Module | 13 | 6 | 7 | 2 | 0 |
| 07 | Student Module | 12 | 4 | 7 | 2 | 1 |
| 08 | Enrollment Module | 9 | 3 | 6 | 1 | 0 |
| 09 | Guardian Module | 10 | 4 | 6 | 2 | 1 |
| 10 | Attendance Module | 11 | 6 | 6 | 2 | 1 |
| 11 | Class Module | 9 | 3 | 5 | 3 | 1 |
| 12 | Section Module | 8 | 3 | 4 | 2 | 0 |
| 13 | Subject Module | 11 | 3 | 8 | 3 | 1 |
| 14 | Examination Module | 10 | 3 | 8 | 1 | 0 |
| 15 | Result Processing Module | 8 | 4 | 5 | 1 | 0 |
| 16 | Grading Module | 8 | 4 | 4 | 1 | 0 |
| 17 | Fee Management Module | 10 | 3 | 8 | 2 | 0 |
| 18 | Payment Module | 10 | 4 | 7 | 1 | 0 |
| 19 | Discount Module | 9 | 4 | 6 | 1 | 0 |
| 20 | Scholarship Module | 9 | 3 | 7 | 1 | 0 |
| 21 | Teacher Module | 10 | 2 | 9 | 1 | 0 |
| 22 | Staff Module | 11 | 3 | 6 | 3 | 0 |
| 23 | HR Module | 13 | 3 | 11 | 2 | 0 |
| 24 | Leave Management Module | 11 | 4 | 8 | 1 | 0 |
| 25 | Notification Module | 9 | 7 | 3 | 3 | 0 |
| 26 | Reporting Module | 10 | 4 | 6 | 1 | 0 |
| 27 | Configuration Engine Module | 13 | 4 | 10 | 1 | 1 |
| 28 | Workflow Engine Module | 12 | 4 | 8 | 1 | 0 |
| 29 | Audit Log Module | 9 | 3 | 5 | 1 | 0 |
| 30 | File Management Module | 10 | 4 | 6 | 1 | 0 |
| — | **TOTAL** | **317** | 118 | 207 | 48 | 10 |

**Total endpoint purposes specified: 317** across 30 modules (GET 118 · POST 207 · PUT 48 · DELETE 10).

Write-heavy skew (POST-dominant) reflects the governed, event-driven nature of the platform: most state changes flow through approval workflows, idempotent commands, and immutable-record creation rather than in-place edits.