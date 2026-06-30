# Exam API

## 1. API Overview

**Purpose.** Define exams (per-subject maxima/components/schedule, consistent with curriculum), determine eligibility and issue admit cards, and track mark-entry progress and reporting. Mark capture itself is split out to `marks.api.md` in this same wave.

**Module Context.** Implements Business Rules Catalog Doc 14 (Examination) Rules EXM-001/002/003/009 and UI Screen Spec `15-examination-screens.md` SCR-EXM-01…03/09…11.

---

## 2. Endpoints

### 1. List Exams
- **Method:** `GET`
- **URL:** `/api/v1/exams`
- **Description:** Browse exams within scope and their lifecycle status (SCR-EXM-01).
- **Authentication (Role-Based):** Yes — `exam.view`, scoped.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "page": {
      "type": "integer"
    },
    "pageSize": {
      "type": "integer"
    },
    "q": {
      "type": "string"
    },
    "sessionId": {
      "type": "string"
    },
    "classId": {
      "type": "string"
    },
    "type": {
      "type": "string"
    },
    "statusDEFINEDSCHEDUL": {
      "type": "string",
      "description": "status? (DEFINED|SCHEDULED|RESCHEDULED|IN_PROGRESS|MARK_ENTRY|UNDER_VERIFICATION|LOCKED|CANCELLED|ARCHIVED)"
    },
    "sortByschedulestatus": {
      "type": "string",
      "description": "sortBy? (schedule|status, default schedule)"
    }
  }
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "id": {
        "type": "string"
      },
      "name": {
        "type": "string"
      },
      "type": {
        "type": "string"
      },
      "sessionId": {
        "type": "string"
      },
      "classIds": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "schedule": {
        "type": "object",
        "properties": {
          "start": {
            "type": "string"
          },
          "end": {
            "type": "string"
          }
        },
        "required": [
          "start",
          "end"
        ]
      },
      "markEntryWindow": {
        "type": "object",
        "properties": {
          "from": {
            "type": "string"
          },
          "to": {
            "type": "string"
          }
        },
        "required": [
          "from",
          "to"
        ]
      },
      "status": {
        "type": "string"
      },
      "updatedAt": {
        "type": "string"
      },
      "version": {
        "type": "integer"
      }
    },
    "required": [
      "id",
      "name",
      "type",
      "sessionId",
      "classIds",
      "schedule",
      "markEntryWindow",
      "status",
      "updatedAt",
      "version"
    ]
  }
}
```
- **Validation Rules:** Scope-limited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** EXM-001, AUTHZ-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Exam Detail
- **Method:** `GET`
- **URL:** `/api/v1/exams/{id}`
- **Description:** Full exam definition — subjects, maxima, components, schedule (SCR-EXM-02).
- **Authentication (Role-Based):** Yes — `exam.view`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {},
  "description": "No request body."
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "id": {
      "type": "string"
    },
    "name": {
      "type": "string"
    },
    "type": {
      "type": "string"
    },
    "sessionId": {
      "type": "string"
    },
    "classIds": {
      "type": "string",
      "description": "classIds[]"
    },
    "subjects": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "subjectId": {
            "type": "string"
          },
          "maxMarks": {
            "type": "number"
          },
          "passMark": {
            "type": "number"
          },
          "components": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "name": {
                  "type": "string"
                },
                "maxMarks": {
                  "type": "number"
                },
                "passMark": {
                  "type": "number"
                },
                "weightage": {
                  "type": "number"
                }
              },
              "required": [
                "name",
                "maxMarks",
                "passMark",
                "weightage"
              ]
            }
          },
          "weightage": {
            "type": "number"
          }
        },
        "required": [
          "subjectId",
          "maxMarks",
          "passMark",
          "components",
          "weightage"
        ]
      }
    },
    "schedule": {
      "type": "string"
    },
    "markEntryWindow": {
      "type": "number"
    },
    "status": {
      "type": "string"
    },
    "createdAt": {
      "type": "string"
    },
    "updatedAt": {
      "type": "string"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "name",
    "type",
    "sessionId",
    "subjects",
    "schedule",
    "markEntryWindow",
    "status",
    "createdAt",
    "updatedAt",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** EXM-001, EXM-002.
- **Pagination:** N/A.

### 3. Define Exam
- **Method:** `POST`
- **URL:** `/api/v1/exams`
- **Description:** Define an exam's per-subject maxima, components, weightage, and schedule, versioned (UC-EXM-01, SCR-EXM-02).
- **Authentication (Role-Based):** Yes — `exam.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "type": {
      "type": "string"
    },
    "sessionId": {
      "type": "string"
    },
    "classIds": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "subjects": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "subjectId": {
            "type": "string"
          },
          "maxMarks": {
            "type": "number"
          },
          "passMark": {
            "type": "number"
          },
          "components": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "name": {
                  "type": "string"
                },
                "maxMarks": {
                  "type": "number"
                },
                "passMark": {
                  "type": "number"
                },
                "weightage": {
                  "type": "number"
                }
              },
              "required": [
                "name",
                "maxMarks",
                "passMark",
                "weightage"
              ]
            }
          },
          "weightage": {
            "type": "number"
          }
        },
        "required": [
          "subjectId",
          "maxMarks",
          "passMark",
          "weightage"
        ]
      }
    },
    "schedule": {
      "type": "object",
      "properties": {
        "start": {
          "type": "string"
        },
        "end": {
          "type": "string"
        }
      },
      "required": [
        "start",
        "end"
      ]
    },
    "markEntryWindow": {
      "type": "object",
      "properties": {
        "from": {
          "type": "string"
        },
        "to": {
          "type": "string"
        }
      },
      "required": [
        "from",
        "to"
      ]
    }
  },
  "required": [
    "name",
    "type",
    "sessionId",
    "classIds",
    "subjects",
    "schedule",
    "markEntryWindow"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "id": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "DEFINED"
    },
    "version": {
      "type": "integer"
    },
    "createdAt": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "status",
    "version",
    "createdAt"
  ]
}
```
- **Validation Rules:** Each subject must already be curriculum-mapped to the listed classes (`11-subject-api.md` #6); each subject's maxima/component structure must be **consistent with the subject's published components** ("Component structure must match the subject's published components," EXM-002, SUB-003) — component maxima must sum to the subject max; cross-subject weightages must be internally consistent.
- **Error Codes:** `409 SUBJECT_NOT_MAPPED_TO_CLASS`; `422 COMPONENT_STRUCTURE_MISMATCH`.
- **Business Rules Applied:** EXM-001 (configurable exam definition), EXM-002 (per-subject maxima/pass/components), SUB-002/SUB-003 (curriculum and component consistency), CFG-004 (versioned).
- **Pagination:** N/A.

### 4. Edit Exam Definition (Pre-Marks Only)
- **Method:** `PUT`
- **URL:** `/api/v1/exams/{id}`
- **Description:** Edit the exam definition — **permitted only before any mark entry has begun** for it (SCR-EXM-02).
- **Authentication (Role-Based):** Yes — `exam.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "subjects": {
      "type": "string"
    },
    "schedule": {
      "type": "string"
    },
    "markEntryWindow": {
      "type": "number"
    },
    "version": {
      "type": "number"
    }
  },
  "required": [
    "version"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "description": "Updated detail (as #2).",
  "$ref": "#endpoint-2"
}
```
- **Validation Rules:** The endpoint rejects any edit once a single mark has been entered against this exam — the definition is locked the moment mark entry begins ("editing is disabled once marks exist"); component-consistency re-validated as in #3; `version` optimistic lock.
- **Error Codes:** `409 DEFINITION_LOCKED_MARKS_EXIST`; `422 COMPONENT_STRUCTURE_MISMATCH`; `409 VERSION_CONFLICT`.
- **Business Rules Applied:** EXM-002, EXM-008 (definition locks once entry begins, mirroring the moderation bounding pattern).
- **Pagination:** N/A.

## Eligibility & Admit Cards

### 5. List Eligibility
- **Method:** `GET`
- **URL:** `/api/v1/exams/{id}/eligibility`
- **Description:** View each student's eligibility status and reason (SCR-EXM-03).
- **Authentication (Role-Based):** Yes — `exam.eligibility.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "eligibleboolean": {
      "type": "string",
      "description": "eligible? (boolean)"
    },
    "classId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    }
  }
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "studentId": {
        "type": "string"
      },
      "studentName": {
        "type": "string"
      },
      "attendancePercent": {
        "type": "number"
      },
      "duesStatus": {
        "type": "string"
      },
      "eligible": {
        "type": "boolean"
      },
      "reason": {
        "type": "string"
      },
      "admitCardIssued": {
        "type": "boolean"
      }
    },
    "required": [
      "studentId",
      "studentName",
      "attendancePercent",
      "duesStatus",
      "eligible",
      "admitCardIssued"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** EXM-003.
- **Pagination:** N/A.

### 6. Determine Eligibility
- **Method:** `POST`
- **URL:** `/api/v1/exams/{id}/eligibility/determine`
- **Description:** Evaluate eligibility against configured criteria — attendance threshold and fee clearance (SCR-EXM-03).
- **Authentication (Role-Based):** Yes — `exam.eligibility.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {}
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "evaluated": {
      "type": "number"
    },
    "eligible": {
      "type": "number"
    },
    "ineligible": {
      "type": "number"
    }
  },
  "required": [
    "evaluated",
    "eligible",
    "ineligible"
  ]
}
```
- **Validation Rules:** Eligibility is computed from the attendance threshold (the Attendance module's own ATT-009 rule, a later wave) and dues status; the evaluation is re-runnable and always reflects current data.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** EXM-003 (eligibility & admit-card gating, consuming ATT-009).
- **Pagination:** N/A.

### 7. Issue Admit Cards
- **Method:** `POST`
- **URL:** `/api/v1/exams/{id}/admit-cards/issue`
- **Description:** Issue admit cards to eligible students as immutable, reproducible documents (SCR-EXM-03).
- **Authentication (Role-Based):** Yes — `exam.eligibility.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentIds": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "omit for all currently eligible"
    }
  }
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "issued": {
      "type": "number"
    }
  },
  "required": [
    "issued"
  ]
}
```
- **Validation Rules:** Admit cards issue **only** to students currently flagged eligible (or eligible via an override, #8); the generated document is immutable and reproducible — reissuing produces the identical artifact (FILE-007).
- **Error Codes:** `409 STUDENT_NOT_ELIGIBLE`.
- **Business Rules Applied:** EXM-003, FILE-007 (immutable, reproducible documents).
- **Pagination:** N/A.

### 8. Override Eligibility (Governed)
- **Method:** `POST`
- **URL:** `/api/v1/exams/{id}/eligibility/{studentId}/override`
- **Description:** Authorize an exceptional eligibility override with reason (SCR-EXM-03).
- **Authentication (Role-Based):** Yes — `exam.eligibility.override` (elevated, per Doc 14 §9).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "reason"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string"
    },
    "eligible": {
      "type": "boolean",
      "const": true
    },
    "overrideReason": {
      "type": "string",
      "description": "reason"
    }
  },
  "required": [
    "studentId",
    "eligible",
    "overrideReason"
  ]
}
```
- **Validation Rules:** `reason` required; the override is recorded distinctly from the computed eligibility, never overwriting the original determination.
- **Error Codes:** `422 REASON_REQUIRED`.
- **Business Rules Applied:** EXM-003 (overrides require authorization + reason and are audited).
- **Pagination:** N/A.

### 9. Get Admit Card Download URL
- **Method:** `GET`
- **URL:** `/api/v1/exams/{id}/admit-cards/{studentId}/download-url`
- **Description:** Short-lived signed URL to download an admit card (SCR-EXM-03).
- **Authentication (Role-Based):** Yes — `exam.eligibility.manage`, or the student/guardian's own subject-scoped access.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {},
  "description": "No request body."
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "downloadUrl": {
      "type": "string"
    },
    "expiresAt": {
      "type": "string"
    }
  },
  "required": [
    "downloadUrl",
    "expiresAt"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 ADMIT_CARD_NOT_ISSUED`.
- **Business Rules Applied:** FILE-005 (short-lived signed URLs).
- **Pagination:** N/A.

## Mark Entry

### 10. Get Mark-Entry Progress
- **Method:** `GET`
- **URL:** `/api/v1/exams/{id}/mark-entry/progress`
- **Description:** Track which sheets are pending entry/lock, by subject/section/teacher (SCR-EXM-09).
- **Authentication (Role-Based):** Yes — `exam.view`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {},
  "description": "No request body."
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "subjectId": {
        "type": "string"
      },
      "sectionId": {
        "type": "string"
      },
      "teacherId": {
        "type": "string"
      },
      "teacherName": {
        "type": "string"
      },
      "eligibleCount": {
        "type": "integer"
      },
      "enteredCount": {
        "type": "integer"
      },
      "status": {
        "type": "string",
        "enum": [
          "NOT_STARTED",
          "DRAFT",
          "LOCKED"
        ]
      }
    },
    "required": [
      "subjectId",
      "sectionId",
      "teacherId",
      "teacherName",
      "eligibleCount",
      "enteredCount",
      "status"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** EXM-009 (completeness tracking).
- **Pagination:** N/A.

### 11. Remind Pending Teachers
- **Method:** `POST`
- **URL:** `/api/v1/exams/{id}/mark-entry/remind`
- **Description:** Dispatch a reminder notification to teachers with incomplete sheets (SCR-EXM-09).
- **Authentication (Role-Based):** Yes — `exam.view` (the approved screen states this permission for the whole progress screen, reminder included).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "subjectSectionPairs": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "subjectId": {
            "type": "string"
          },
          "sectionId": {
            "type": "string"
          }
        },
        "required": [
          "subjectId",
          "sectionId"
        ]
      },
      "description": "omit for all pending"
    }
  }
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "remindersSent": {
      "type": "number"
    }
  },
  "required": [
    "remindersSent"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** EXM-009.
- **Pagination:** N/A.

### 12. Mark Report
- **Method:** `GET`
- **URL:** `/api/v1/exams/report/marks`
- **Description:** Marks/analysis report within scope (SCR-EXM-11).
- **Authentication (Role-Based):** Yes — `exam.report.view`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "examId": {
      "type": "string"
    },
    "subjectId": {
      "type": "string"
    },
    "classId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    }
  },
  "required": [
    "examId"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "subjectId": {
        "type": "string"
      },
      "sectionId": {
        "type": "string"
      },
      "avgMark": {
        "type": "number"
      },
      "highestMark": {
        "type": "number"
      },
      "lowestMark": {
        "type": "number"
      },
      "passRate": {
        "type": "string"
      }
    },
    "required": [
      "subjectId",
      "sectionId",
      "avgMark",
      "highestMark",
      "lowestMark",
      "passRate"
    ]
  }
}
```
- **Validation Rules:** Scope-limited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** REP-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
