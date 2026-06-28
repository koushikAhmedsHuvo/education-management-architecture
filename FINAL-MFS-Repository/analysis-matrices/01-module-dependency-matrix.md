# Module Dependency Matrix

> Derived from §12 cross-module rule citations across all 30 module specifications. A row module **depends on** a column module when its functional spec cites a business rule owned by that column module. `●` = direct dependency.

**Reading:** row → depends on → column.

| Mod | 01 | 02 | 03 | 04 | 05 | 06 | 07 | 08 | 09 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 | 21 | 22 | 23 | 24 | 25 | 26 | 27 | 28 | 29 | 30 | **#Deps** |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **01** |  | ● |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ● |  | ● |  | 3 |
| **02** |  |  | ● | ● |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ● | ● | ● |  | 5 |
| **03** |  | ● |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ● | ● |  | ● |  | 4 |
| **04** |  | ● | ● |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ● |  | ● | ● | ● |  | 6 |
| **05** |  |  | ● |  |  |  |  |  |  |  |  |  |  |  | ● |  |  |  |  |  |  |  |  |  | ● |  |  | ● | ● |  | 5 |
| **06** |  | ● |  |  |  |  | ● | ● |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ● |  | ● | ● | ● |  | 7 |
| **07** |  | ● |  |  |  | ● |  |  | ● |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ● | ● |  | ● | ● | 7 |
| **08** |  |  |  |  | ● |  |  |  |  |  |  | ● |  |  |  |  |  |  |  |  |  |  |  |  | ● |  |  | ● | ● | ● | 6 |
| **09** |  | ● |  |  |  | ● |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ● |  |  |  | ● | ● | 5 |
| **10** |  |  |  |  |  |  |  | ● |  |  |  | ● | ● | ● |  |  |  |  |  |  |  |  |  | ● | ● |  |  | ● | ● |  | 8 |
| **11** |  |  |  |  | ● |  |  |  |  |  |  | ● | ● |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ● | ● |  | 5 |
| **12** |  |  |  |  |  |  |  | ● |  |  | ● |  |  |  |  |  |  |  |  |  | ● |  |  |  |  |  |  | ● | ● |  | 5 |
| **13** |  |  |  |  |  |  |  |  |  |  | ● |  |  | ● | ● | ● |  |  |  |  | ● |  |  |  |  |  |  | ● | ● |  | 7 |
| **14** |  | ● |  |  |  |  |  |  |  | ● |  |  | ● |  | ● |  |  |  |  |  |  |  |  |  |  |  |  | ● | ● | ● | 7 |
| **15** |  | ● |  |  |  |  |  |  |  |  |  |  |  | ● |  | ● |  |  |  |  |  |  |  |  | ● |  |  | ● | ● | ● | 7 |
| **16** |  | ● |  |  |  |  |  |  |  |  |  |  | ● |  | ● |  |  |  |  |  |  |  |  |  |  |  | ● | ● | ● |  | 6 |
| **17** |  | ● |  |  | ● |  |  |  | ● |  |  |  |  |  |  |  |  | ● | ● |  |  |  |  |  |  |  | ● | ● | ● | ● | 9 |
| **18** |  | ● |  |  |  |  |  |  | ● |  |  |  |  |  |  |  | ● |  |  |  |  |  |  |  |  |  | ● | ● | ● | ● | 7 |
| **19** |  | ● |  |  |  |  |  |  | ● |  |  |  |  |  |  |  | ● |  |  | ● |  |  |  |  |  |  | ● | ● | ● |  | 7 |
| **20** |  | ● |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ● |  | ● |  |  |  |  |  |  |  | ● | ● | ● |  | 6 |
| **21** |  |  |  |  |  |  |  |  |  |  |  | ● | ● |  |  |  |  |  |  |  |  | ● |  | ● |  |  |  | ● | ● |  | 6 |
| **22** | ● | ● |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ● |  |  |  |  |  | ● | ● | ● |  | 6 |
| **23** |  | ● |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ● |  |  | ● | ● |  | ● |  |  | ● | ● | ● | ● | 9 |
| **24** |  | ● |  |  |  |  |  |  | ● | ● |  |  |  |  |  |  |  |  |  |  | ● | ● | ● |  |  |  |  | ● | ● |  | 8 |
| **25** |  |  |  |  |  |  |  |  | ● |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ● |  | ● |  | 3 |
| **26** |  | ● |  |  |  | ● |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ● | ● | ● | ● | 6 |
| **27** |  | ● | ● |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ● | ● |  | 4 |
| **28** |  | ● |  |  |  |  | ● |  |  |  |  |  |  |  | ● |  |  |  | ● | ● |  | ● | ● | ● |  |  | ● |  | ● |  | 10 |
| **29** |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ● |  |  |  | 1 |
| **30** |  |  |  |  |  |  |  |  | ● |  |  |  |  |  | ● |  | ● |  |  |  |  |  | ● |  |  |  | ● |  | ● |  | 6 |

## Dependency detail (who each module depends on)

- **01 Authentication Module** → 02 Authorization Module, 27 Configuration Engine Module, 29 Audit Log Module
- **02 Authorization Module** → 03 Institute Module, 04 Campus Module, 27 Configuration Engine Module, 28 Workflow Engine Module, 29 Audit Log Module
- **03 Institute Module** → 02 Authorization Module, 26 Reporting Module, 27 Configuration Engine Module, 29 Audit Log Module
- **04 Campus Module** → 02 Authorization Module, 03 Institute Module, 25 Notification Module, 27 Configuration Engine Module, 28 Workflow Engine Module, 29 Audit Log Module
- **05 Academic Session Module** → 03 Institute Module, 15 Result Processing Module, 25 Notification Module, 28 Workflow Engine Module, 29 Audit Log Module
- **06 Admission Module** → 02 Authorization Module, 07 Student Module, 08 Enrollment Module, 25 Notification Module, 27 Configuration Engine Module, 28 Workflow Engine Module, 29 Audit Log Module
- **07 Student Module** → 02 Authorization Module, 06 Admission Module, 09 Guardian Module, 26 Reporting Module, 27 Configuration Engine Module, 29 Audit Log Module, 30 File Management Module
- **08 Enrollment Module** → 05 Academic Session Module, 12 Section Module, 25 Notification Module, 28 Workflow Engine Module, 29 Audit Log Module, 30 File Management Module
- **09 Guardian Module** → 02 Authorization Module, 06 Admission Module, 25 Notification Module, 29 Audit Log Module, 30 File Management Module
- **10 Attendance Module** → 08 Enrollment Module, 12 Section Module, 13 Subject Module, 14 Examination Module, 24 Leave Management Module, 25 Notification Module, 28 Workflow Engine Module, 29 Audit Log Module
- **11 Class Module** → 05 Academic Session Module, 12 Section Module, 13 Subject Module, 28 Workflow Engine Module, 29 Audit Log Module
- **12 Section Module** → 08 Enrollment Module, 11 Class Module, 21 Teacher Module, 28 Workflow Engine Module, 29 Audit Log Module
- **13 Subject Module** → 11 Class Module, 14 Examination Module, 15 Result Processing Module, 16 Grading Module, 21 Teacher Module, 28 Workflow Engine Module, 29 Audit Log Module
- **14 Examination Module** → 02 Authorization Module, 10 Attendance Module, 13 Subject Module, 15 Result Processing Module, 28 Workflow Engine Module, 29 Audit Log Module, 30 File Management Module
- **15 Result Processing Module** → 02 Authorization Module, 14 Examination Module, 16 Grading Module, 25 Notification Module, 28 Workflow Engine Module, 29 Audit Log Module, 30 File Management Module
- **16 Grading Module** → 02 Authorization Module, 13 Subject Module, 15 Result Processing Module, 27 Configuration Engine Module, 28 Workflow Engine Module, 29 Audit Log Module
- **17 Fee Management Module** → 02 Authorization Module, 05 Academic Session Module, 09 Guardian Module, 18 Payment Module, 19 Discount Module, 27 Configuration Engine Module, 28 Workflow Engine Module, 29 Audit Log Module, 30 File Management Module
- **18 Payment Module** → 02 Authorization Module, 09 Guardian Module, 17 Fee Management Module, 27 Configuration Engine Module, 28 Workflow Engine Module, 29 Audit Log Module, 30 File Management Module
- **19 Discount Module** → 02 Authorization Module, 09 Guardian Module, 17 Fee Management Module, 20 Scholarship Module, 27 Configuration Engine Module, 28 Workflow Engine Module, 29 Audit Log Module
- **20 Scholarship Module** → 02 Authorization Module, 17 Fee Management Module, 19 Discount Module, 27 Configuration Engine Module, 28 Workflow Engine Module, 29 Audit Log Module
- **21 Teacher Module** → 12 Section Module, 13 Subject Module, 22 Staff Module, 24 Leave Management Module, 28 Workflow Engine Module, 29 Audit Log Module
- **22 Staff Module** → 01 Authentication Module, 02 Authorization Module, 21 Teacher Module, 27 Configuration Engine Module, 28 Workflow Engine Module, 29 Audit Log Module
- **23 HR Module** → 02 Authorization Module, 18 Payment Module, 21 Teacher Module, 22 Staff Module, 24 Leave Management Module, 27 Configuration Engine Module, 28 Workflow Engine Module, 29 Audit Log Module, 30 File Management Module
- **24 Leave Management Module** → 02 Authorization Module, 09 Guardian Module, 10 Attendance Module, 21 Teacher Module, 22 Staff Module, 23 HR Module, 28 Workflow Engine Module, 29 Audit Log Module
- **25 Notification Module** → 09 Guardian Module, 27 Configuration Engine Module, 29 Audit Log Module
- **26 Reporting Module** → 02 Authorization Module, 06 Admission Module, 27 Configuration Engine Module, 28 Workflow Engine Module, 29 Audit Log Module, 30 File Management Module
- **27 Configuration Engine Module** → 02 Authorization Module, 03 Institute Module, 28 Workflow Engine Module, 29 Audit Log Module
- **28 Workflow Engine Module** → 02 Authorization Module, 07 Student Module, 15 Result Processing Module, 19 Discount Module, 20 Scholarship Module, 22 Staff Module, 23 HR Module, 24 Leave Management Module, 27 Configuration Engine Module, 29 Audit Log Module
- **29 Audit Log Module** → 27 Configuration Engine Module
- **30 File Management Module** → 09 Guardian Module, 15 Result Processing Module, 17 Fee Management Module, 23 HR Module, 27 Configuration Engine Module, 29 Audit Log Module

## Most-depended-upon modules (fan-in)

| Module | Depended on by | Count |
|---|---|---|
| 29 Audit Log Module | 01, 02, 03, 04, 05, 06, 07, 08, 09, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 30 | 29 |
| 28 Workflow Engine Module | 02, 04, 05, 06, 08, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 26, 27 | 22 |
| 02 Authorization Module | 01, 03, 04, 06, 07, 09, 14, 15, 16, 17, 18, 19, 20, 22, 23, 24, 26, 27, 28 | 19 |
| 27 Configuration Engine Module | 01, 02, 03, 04, 06, 07, 16, 17, 18, 19, 20, 22, 23, 25, 26, 28, 29, 30 | 18 |
| 30 File Management Module | 07, 08, 09, 14, 15, 17, 18, 23, 26 | 9 |
| 25 Notification Module | 04, 05, 06, 08, 09, 10, 15 | 7 |
| 09 Guardian Module | 07, 17, 18, 19, 24, 25, 30 | 7 |
| 15 Result Processing Module | 05, 13, 14, 16, 28, 30 | 6 |
| 13 Subject Module | 10, 11, 14, 16, 21 | 5 |
| 21 Teacher Module | 12, 13, 22, 23, 24 | 5 |
| 03 Institute Module | 02, 04, 05, 27 | 4 |
| 12 Section Module | 08, 10, 11, 21 | 4 |
| 24 Leave Management Module | 10, 21, 23, 28 | 4 |
| 17 Fee Management Module | 18, 19, 20, 30 | 4 |
| 22 Staff Module | 21, 23, 24, 28 | 4 |
| 08 Enrollment Module | 06, 10, 12 | 3 |
| 06 Admission Module | 07, 09, 26 | 3 |
| 05 Academic Session Module | 08, 11, 17 | 3 |
| 14 Examination Module | 10, 13, 15 | 3 |
| 19 Discount Module | 17, 20, 28 | 3 |
| 23 HR Module | 24, 28, 30 | 3 |
| 26 Reporting Module | 03, 07 | 2 |
| 07 Student Module | 06, 28 | 2 |
| 11 Class Module | 12, 13 | 2 |
| 16 Grading Module | 13, 15 | 2 |
| 10 Attendance Module | 14, 24 | 2 |
| 18 Payment Module | 17, 23 | 2 |
| 20 Scholarship Module | 19, 28 | 2 |
| 04 Campus Module | 02 | 1 |
| 01 Authentication Module | 22 | 1 |