# Cross-Module Rule References

How the 30 modules reference each other (counted from inline rule-ID citations across all module documents). This reveals the system's coupling structure: which modules are foundational (heavily referenced) and which are integrative (reference many others).

## A. Reference counts — who each module depends on

Read as: *Module (row) cites Module (listed) N times.*

- **01 Authentication** → 02 Authorization (2), 03 Institute (2)
- **02 Authorization** → 03 Institute (2), 04 Campus (2), 01 Authentication (1)
- **03 Institute** → 02 Authorization (4), 04 Campus (3), 05 Academic Session (3), 01 Authentication (1)
- **04 Campus** → 03 Institute (4), 02 Authorization (3), 05 Academic Session (1)
- **05 Academic Session** → 03 Institute (1), 02 Authorization (1)
- **06 Student** → 05 Academic Session (2), 02 Authorization (1)
- **07 Admission** → 06 Student (10), 05 Academic Session (2)
- **08 Enrollment** → 05 Academic Session (5), 04 Campus (3), 07 Admission (2), 06 Student (2)
- **09 Guardian** → 06 Student (3), 02 Authorization (1)
- **10 Attendance** → 05 Academic Session (4), 08 Enrollment (4), 12 Section (3), 09 Guardian (3), 13 Subject (2), 02 Authorization (1)
- **11 Class** → 05 Academic Session (7), 08 Enrollment (5), 12 Section (3), 03 Institute (1), 04 Campus (1), 13 Subject (1), 07 Admission (1)
- **12 Section** → 08 Enrollment (8), 11 Class (4), 02 Authorization (3), 07 Admission (2), 10 Attendance (2), 04 Campus (1)
- **13 Subject** → 02 Authorization (3), 05 Academic Session (2), 11 Class (1), 10 Attendance (1)
- **14 Examination** → 15 Result Processing (10), 13 Subject (7), 10 Attendance (4), 02 Authorization (3), 16 Grading (1), 05 Academic Session (1)
- **15 Result Processing** → 14 Examination (10), 16 Grading (10), 05 Academic Session (3), 13 Subject (2), 09 Guardian (1)
- **16 Grading** → 15 Result Processing (6), 13 Subject (3), 05 Academic Session (2), 14 Examination (2), 03 Institute (1), 02 Authorization (1)
- **17 Fee Management** → 18 Payment (6), 08 Enrollment (5), 05 Academic Session (4), 03 Institute (1), 02 Authorization (1), 09 Guardian (1), 15 Result Processing (1)
- **18 Payment** → 17 Fee Management (8), 09 Guardian (2), 02 Authorization (1)
- **19 Discount** → 17 Fee Management (6), 18 Payment (3), 02 Authorization (2), 20 Scholarship (2), 09 Guardian (1)
- **20 Scholarship** → 19 Discount (3), 17 Fee Management (2), 07 Admission (1), 02 Authorization (1)
- **21 Teacher** → 22 Staff (10), 13 Subject (8), 12 Section (6), 10 Attendance (5), 14 Examination (4), 02 Authorization (2)
- **22 Staff** → 01 Authentication (9), 23 HR (4), 21 Teacher (3), 02 Authorization (2), 04 Campus (1), 03 Institute (1), 06 Student (1)
- **23 HR** → 22 Staff (10), 21 Teacher (4), 01 Authentication (3), 17 Fee Management (3), 02 Authorization (2), 10 Attendance (1)
- **24 Leave Management** → 10 Attendance (11), 23 HR (11), 21 Teacher (4), 22 Staff (4), 08 Enrollment (1)
- **25 Notification** → 10 Attendance (3), 15 Result Processing (3), 09 Guardian (3), 27 Configuration Engine (2), 06 Student (1)
- **26 Reporting** → 02 Authorization (5), 10 Attendance (4), 27 Configuration Engine (3), 06 Student (2), 22 Staff (2), 15 Result Processing (2), 17 Fee Management (1), 03 Institute (1), 25 Notification (1)
- **27 Configuration Engine** → 03 Institute (6), 02 Authorization (5), 16 Grading (5), 06 Student (5), 17 Fee Management (2), 15 Result Processing (2), 07 Admission (2), 04 Campus (1), 23 HR (1), 05 Academic Session (1), 22 Staff (1)
- **28 Workflow Engine** → 27 Configuration Engine (3), 22 Staff (3), 07 Admission (2), 02 Authorization (2), 24 Leave Management (2), 19 Discount (2), 20 Scholarship (2), 15 Result Processing (1), 18 Payment (1)
- **29 Audit Log** → 14 Examination (2), 27 Configuration Engine (2), 01 Authentication (2), 06 Student (2), 25 Notification (1), 22 Staff (1), 02 Authorization (1), 26 Reporting (1)
- **30 File Management** → 06 Student (7), 09 Guardian (4), 22 Staff (3), 02 Authorization (2), 15 Result Processing (2), 16 Grading (1), 17 Fee Management (1), 23 HR (1), 29 Audit Log (1)

## B. Most-referenced modules (foundational layer)

Modules other modules depend on most — the platform's gravitational centre. Changes here have the widest impact.

| Rank | Module | Inbound references |
|---|---|---|
| 1 | 02 Authorization | 49 |
| 2 | 05 Academic Session | 37 |
| 3 | 22 Staff | 34 |
| 4 | 06 Student | 33 |
| 5 | 10 Attendance | 31 |
| 6 | 15 Result Processing | 27 |
| 7 | 13 Subject | 23 |
| 8 | 08 Enrollment | 23 |
| 9 | 17 Fee Management | 23 |
| 10 | 03 Institute | 20 |
| 11 | 14 Examination | 18 |
| 12 | 16 Grading | 17 |

## C. Most integrative modules (reference the most others)

Modules that tie the system together by citing many others — typically lifecycle/orchestration modules.

| Rank | Module | Outbound references |
|---|---|---|
| 1 | 21 Teacher | 35 |
| 2 | 24 Leave Management | 31 |
| 3 | 27 Configuration Engine | 31 |
| 4 | 14 Examination | 26 |
| 5 | 15 Result Processing | 26 |
| 6 | 23 HR | 23 |
| 7 | 30 File Management | 22 |
| 8 | 22 Staff | 21 |
| 9 | 26 Reporting | 21 |
| 10 | 12 Section | 20 |
| 11 | 11 Class | 19 |
| 12 | 17 Fee Management | 19 |

## D. The two engines

The Configuration Engine (27) and Workflow Engine (28) are referenced as *mechanisms* throughout. Their inbound references represent the 'configuration-driven' and 'governed-process' promises made by other modules:

- **27 Configuration Engine** — inbound: 10 (version-stamping, scoped resolution, custom fields)
- **28 Workflow Engine** — inbound: 0 (approval, escalation, SoD, version-pinning)
- **29 Audit Log** — inbound: 1 (every consequential action emits an audit event)
