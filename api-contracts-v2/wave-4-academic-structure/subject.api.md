# Subject API

## 1. API Overview

**Purpose.** Govern subject identity, components/aggregation, credit/weightage, teacher assignment, and elective rules — the foundation that keeps assessment and results coherent. Curriculum mapping itself is split out to `curriculum.api.md` in this same wave.

**Module Context.** Implements Business Rules Catalog Doc 13 (Subject) and UI Screen Spec `13-subject-management-screens.md` (SCR-SUB-01…04/06…10/12).

---

## 2. Endpoints

### 1. List Subjects
- **Method:** `GET`
- **URL:** `/api/v1/subjects`
- **Description:** Browse the institute's subject catalog (SCR-SUB-01).
- **Authentication (Role-Based):** Yes — `subject.view`, institute-scoped.
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
    "typecoreelectivepra": {
      "type": "string",
      "description": "type? (core|elective|practical|co-curricular)"
    },
    "mappedClassId": {
      "type": "string"
    },
    "statusDRAFTACTIVEDE": {
      "type": "string",
      "description": "status? (DRAFT|ACTIVE|DEPRECATED)"
    },
    "sortBynamecodetype": {
      "type": "string",
      "description": "sortBy? (name|code|type, default name)"
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
      "code": {
        "type": "string"
      },
      "type": {
        "type": "string"
      },
      "credit": {
        "type": "number"
      },
      "weightage": {
        "type": "number"
      },
      "mappedClassCount": {
        "type": "integer"
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
      "code",
      "type",
      "mappedClassCount",
      "status",
      "updatedAt",
      "version"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** SUB-001 (subject identity), SUB-005 (type/credit/weightage shown).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Subject Detail
- **Method:** `GET`
- **URL:** `/api/v1/subjects/{id}`
- **Description:** Full subject definition — curriculum mapping, components, teachers, electives (SCR-SUB-02).
- **Authentication (Role-Based):** Yes — `subject.view`.
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
    "code": {
      "type": "string"
    },
    "type": {
      "type": "string"
    },
    "credit": {
      "type": "number"
    },
    "weightage": {
      "type": "number"
    },
    "languageGroup": {
      "type": "string"
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
          "weightage": {
            "type": "number"
          },
          "passMark": {
            "type": "number"
          }
        },
        "required": [
          "name",
          "maxMarks",
          "weightage",
          "passMark"
        ]
      }
    },
    "mappedClasses": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "classId": {
            "type": "string"
          },
          "stream": {
            "type": "string"
          },
          "mandatory": {
            "type": "boolean"
          }
        },
        "required": [
          "classId",
          "mandatory"
        ]
      }
    },
    "teacherAssignments": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "sectionId": {
            "type": "string"
          },
          "teacherId": {
            "type": "string"
          },
          "displayName": {
            "type": "string"
          }
        },
        "required": [
          "sectionId",
          "teacherId",
          "displayName"
        ]
      }
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
    "createdBy": {
      "type": "string"
    },
    "updatedBy": {
      "type": "string"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "name",
    "code",
    "type",
    "components",
    "mappedClasses",
    "teacherAssignments",
    "status",
    "createdAt",
    "updatedAt",
    "createdBy",
    "updatedBy",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** SUB-001, SUB-002, SUB-003, SUB-004, SUB-005.
- **Pagination:** N/A.

### 3. Define Subject
- **Method:** `POST`
- **URL:** `/api/v1/subjects`
- **Description:** Create a subject (UC-SUB-01, SCR-SUB-03).
- **Authentication (Role-Based):** Yes — `subject.create`, institute-scoped.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "code": {
      "type": "string"
    },
    "type": {
      "type": "string",
      "enum": [
        "core",
        "elective",
        "practical"
      ]
    },
    "credit": {
      "type": "number"
    },
    "weightage": {
      "type": "number"
    }
  },
  "required": [
    "name",
    "code",
    "type"
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
    "name": {
      "type": "string"
    },
    "code": {
      "type": "string"
    },
    "type": {
      "type": "string"
    },
    "credit": {
      "type": "number"
    },
    "weightage": {
      "type": "number"
    },
    "status": {
      "type": "string",
      "const": "DRAFT"
    },
    "createdAt": {
      "type": "string"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "name",
    "code",
    "type",
    "status",
    "createdAt",
    "version"
  ]
}
```
- **Validation Rules:** `code` unique within the institute and **immutable thereafter** ("Subject code already exists," SUB-001); `type`/`credit` must be valid and consistent with the grading scheme they will later feed (SUB-005).
- **Error Codes:** `409 SUBJECT_CODE_EXISTS`; `422 INVALID_TYPE_OR_CREDIT`.
- **Business Rules Applied:** SUB-001 (subject identity & code), SUB-005 (type, credit & weightage).
- **Pagination:** N/A.

### 4. Edit Subject (Name / Type / Credit / Weightage)
- **Method:** `PUT`
- **URL:** `/api/v1/subjects/{id}`
- **Description:** Edit name, type, credit, and weightage — this single endpoint backs both the general "Edit Subject" screen (SCR-SUB-04) and the focused "Type/Credit/Weightage" screen (SCR-SUB-09), which the approved spec presents as two UI entry points to the same underlying fields.
- **Authentication (Role-Based):** Yes — `subject.update`.
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
    "credit": {
      "type": "number"
    },
    "weightage": {
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
- **Validation Rules:** `code` is never accepted here — it is immutable-after-creation (SUB-001); `type`/`credit` re-validated for consistency with the grading scheme (SUB-005); `version` optimistic lock.
- **Error Codes:** `422 IMMUTABLE_FIELD` (if `code` is supplied); `422 INVALID_TYPE_OR_CREDIT`; `409 VERSION_CONFLICT`.
- **Business Rules Applied:** SUB-001, SUB-005.
- **Pagination:** N/A.

### 5. Set Components & Aggregation
- **Method:** `PUT`
- **URL:** `/api/v1/subjects/{id}/components`
- **Description:** Define a subject's assessment components (e.g., theory + practical) and how they aggregate to the subject result (SCR-SUB-06).
- **Authentication (Role-Based):** Yes — `subject.component.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
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
          "weightage": {
            "type": "number"
          },
          "passMark": {
            "type": "number"
          }
        },
        "required": [
          "name",
          "maxMarks",
          "weightage"
        ]
      }
    }
  },
  "required": [
    "components"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "subjectId": {
      "type": "string"
    },
    "components": {
      "type": "string",
      "description": "components[]"
    }
  },
  "required": [
    "subjectId"
  ]
}
```
- **Validation Rules:** Component `weightage` values must sum to the configured total, typically 100% ("Weightages must sum to 100% (currently \<X\>%)," SUB-003); per-component `passMark` rules, where set, are enforced independently in result computation (a student can fail the subject by failing one component even with a high overall score) — this endpoint only validates the configuration's internal consistency, not live results.
- **Error Codes:** `422 WEIGHTAGES_DO_NOT_SUM` (with the current total in the error detail).
- **Business Rules Applied:** SUB-003 (subject components & aggregation).
- **Pagination:** N/A.

### 6. Assign Subject-Teacher
- **Method:** `POST`
- **URL:** `/api/v1/subjects/{id}/teacher-assignments`
- **Description:** Assign a teacher to teach a subject in a specific section, conferring marks/attendance ownership for that pairing (UC-SUB-02, SCR-SUB-07).
- **Authentication (Role-Based):** Yes — `subject.teacher.assign`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "sectionId": {
      "type": "string"
    },
    "teacherId": {
      "type": "string"
    }
  },
  "required": [
    "sectionId",
    "teacherId"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
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
    "displayName": {
      "type": "string"
    },
    "assignedAt": {
      "type": "string"
    }
  },
  "required": [
    "subjectId",
    "sectionId",
    "teacherId",
    "displayName",
    "assignedAt"
  ]
}
```
- **Validation Rules:** The subject must already be mapped to the section's class via curriculum (#6) — assigning to an unmapped class is rejected ("Teacher not qualified for this subject" is also returned when the teacher's own qualification record doesn't cover the subject, a Teacher-module check); the assignment respects the teacher's configurable workload/assignment limit ("Assignment exceeds workload limit").
- **Error Codes:** `409 SUBJECT_NOT_MAPPED_TO_CLASS`; `422 TEACHER_NOT_QUALIFIED`; `422 WORKLOAD_LIMIT_EXCEEDED`.
- **Business Rules Applied:** SUB-002 (mapping required first), SUB-004 (subject-teacher assignment confers marks ownership), TCH-002/TCH-003 (qualification and workload, Teacher module).
- **Pagination:** N/A.

### 7. Get Elective Configuration
- **Method:** `GET`
- **URL:** `/api/v1/subjects/{id}/electives-config`
- **Description:** View the elective group's selection rules and prerequisites for a subject (SCR-SUB-08).
- **Authentication (Role-Based):** Yes — `subject.view`.
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
    "subjectId": {
      "type": "string"
    },
    "selectionRules": {
      "type": "object",
      "properties": {
        "groupKey": {
          "type": "string"
        },
        "min": {
          "type": "number"
        },
        "max": {
          "type": "number"
        }
      },
      "required": [
        "groupKey",
        "min",
        "max"
      ]
    },
    "prerequisites": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "subject ids"
    }
  },
  "required": [
    "subjectId",
    "selectionRules",
    "prerequisites"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** SUB-006.
- **Pagination:** N/A.

### 8. Set Elective Configuration
- **Method:** `PUT`
- **URL:** `/api/v1/subjects/{id}/electives-config`
- **Description:** Configure an elective group's min/max selection limits and prerequisite subjects (SCR-SUB-08).
- **Authentication (Role-Based):** Yes — `subject.elective.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "selectionRules": {
      "type": "object",
      "properties": {
        "groupKey": {
          "type": "string"
        },
        "min": {
          "type": "number"
        },
        "max": {
          "type": "number"
        }
      },
      "required": [
        "groupKey",
        "min",
        "max"
      ]
    },
    "prerequisites": {
      "type": "array",
      "items": {
        "type": "string"
      }
    }
  },
  "required": [
    "selectionRules",
    "prerequisites"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "description": "Updated config (as #9).",
  "$ref": "#endpoint-9"
}
```
- **Validation Rules:** `min ≤ max`; `prerequisites` must reference existing subjects.
- **Error Codes:** `422 INVALID_SELECTION_RULES`.
- **Business Rules Applied:** SUB-006 (elective selection rules & prerequisites).
- **Pagination:** N/A.

### 9. Select Electives (Student/Guardian)
- **Method:** `POST`
- **URL:** `/api/v1/subjects/elective-selections`
- **Description:** A student (or their guardian on their behalf) selects electives from an offered group within its rules (UC-SUB-03, SCR-SUB-14).
- **Authentication (Role-Based):** Yes — `academic.elective.select`, **self-scoped** (the selecting student's own context only; Doc 13 §9 names this distinct, self-scoped permission separately from the configuration-side `subject.elective.manage`).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string",
      "description": "must be self or own child"
    },
    "sessionId": {
      "type": "string"
    },
    "selectedSubjectIds": {
      "type": "array",
      "items": {
        "type": "string"
      }
    }
  },
  "required": [
    "studentId",
    "sessionId",
    "selectedSubjectIds"
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
    "sessionId": {
      "type": "string"
    },
    "selectedSubjectIds": {
      "type": "string",
      "description": "selectedSubjectIds[]"
    },
    "status": {
      "type": "string",
      "const": "RECORDED"
    }
  },
  "required": [
    "studentId",
    "sessionId",
    "status"
  ]
}
```
- **Validation Rules:** The selection count must fall within the group's configured `min`/`max` ("Select between \<min\> and \<max\>," SUB-006); every prerequisite subject for each selected elective must already be satisfied by the student's record ("Prerequisite \<subject\> not met," SUB-006); a full elective routes to waitlist/alternative rather than a hard rejection, per configured policy; the selection window must be open.
- **Error Codes:** `422 SELECTION_OUT_OF_RANGE`; `422 PREREQUISITE_NOT_MET`; `409 SELECTION_WINDOW_CLOSED`; `409 ELECTIVE_FULL` (returns waitlist status instead where policy allows).
- **Business Rules Applied:** SUB-006 (elective selection rules & prerequisites).
- **Pagination:** N/A.

### 10. List Elective Selections (Admin)
- **Method:** `GET`
- **URL:** `/api/v1/subjects/{id}/elective-selections`
- **Description:** Admin view of who has selected a given elective, for capacity and reporting purposes.
- **Authentication (Role-Based):** Yes — `subject.elective.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "sessionId": {
      "type": "string"
    },
    "page": {
      "type": "integer"
    },
    "pageSize": {
      "type": "integer"
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
      "selectedAt": {
        "type": "string"
      },
      "status": {
        "type": "string"
      }
    },
    "required": [
      "studentId",
      "studentName",
      "selectedAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Scope-limited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** SUB-006.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 11. Retire Subject
- **Method:** `POST`
- **URL:** `/api/v1/subjects/{id}/retire`
- **Description:** Retire a subject from future sessions/curriculum, preserving all historical assessment data intact (SCR-SUB-10).
- **Authentication (Role-Based):** Yes — `subject.retire`.
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
    "id": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "DEPRECATED"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** `reason` required; **never** a hard delete — a subject with any assessment history is deprecated/unmapped forward only, while past results referencing it remain valid and retrievable ("a subject dropped from the 2027 curriculum still appears correctly on 2025 marksheets," SUB-007); a hard-delete attempt on an assessed subject is rejected outright.
- **Error Codes:** `422 REASON_REQUIRED`; `409 CANNOT_HARD_DELETE_ASSESSED_SUBJECT` (returned only if a hard-delete path is attempted via an unsupported route — this endpoint itself only deprecates).
- **Business Rules Applied:** SUB-007 (subject retirement preserves history).
- **Pagination:** N/A.

### 12. Bulk-Define Subjects
- **Method:** `POST`
- **URL:** `/api/v1/subjects/bulk-define`
- **Description:** Define many subjects at once from an import batch (SCR-SUB-12).
- **Authentication (Role-Based):** Yes — `subject.import`.
- **Request DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "rows": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "name": {
                "type": "string"
              },
              "code": {
                "type": "string"
              },
              "type": {
                "type": "string"
              },
              "credit": {
                "type": "number"
              },
              "weightage": {
                "type": "number"
              }
            },
            "required": [
              "name",
              "code",
              "type"
            ]
          }
        }
      },
      "required": [
        "rows"
      ]
    },
    {
      "type": "object",
      "properties": {
        "IdempotencyKey": {
          "type": "string",
          "description": "Idempotency-Key"
        }
      }
    }
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "accepted": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "row": {
                "type": "string"
              },
              "subjectId": {
                "type": "string"
              }
            },
            "required": [
              "row",
              "subjectId"
            ]
          }
        },
        "rejected": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "row": {
                "type": "string"
              },
              "reason": {
                "type": "string"
              }
            },
            "required": [
              "row",
              "reason"
            ]
          }
        }
      },
      "required": [
        "accepted",
        "rejected"
      ]
    },
    {
      "type": "object",
      "properties": {
        "202Accepted": {
          "type": "string",
          "description": "202 Accepted"
        }
      }
    },
    {
      "type": "object",
      "properties": {
        "jobId": {
          "type": "string"
        }
      },
      "required": [
        "jobId"
      ]
    }
  ]
}
```
- **Validation Rules:** Each row validated against #3's rules (unique code, valid type/credit); duplicate/invalid rows excluded and always reported — never silently imported.
- **Error Codes:** `400 MALFORMED_BATCH`; per-row failures in `rejected[]`.
- **Business Rules Applied:** SUB-001, SUB-005; conventions §9.
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
