# Student API

## 1. API Overview

**Purpose.** Govern the Student entity — the institution's most sensitive data subject. Masked-by-default sensitive fields with audited unmasking, dedup-checked creation, governed official-field edits, dynamic fields, lifecycle status, guardian linking, document management, and retention-aware archive/anonymization.

**Module Context.** Implements Business Rules Catalog Doc 06 (Student) and UI Screen Spec `08-student-management-screens.md` (SCR-STU-01…14).

---

## 2. Endpoints

### 1. List Students
- **Method:** `GET`
- **URL:** `/api/v1/students`
- **Description:** Browse student records within scope, with sensitive fields masked by default (SCR-STU-01).
- **Authentication (Role-Based):** Yes — `student.view`, scoped + ownership (teacher → own students, AUTHZ-003).
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
    "statuscommaseparated": {
      "type": "string",
      "description": "status? (comma-separated)"
    },
    "classId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "campusId": {
      "type": "string"
    },
    "hasGuardianboolean": {
      "type": "string",
      "description": "hasGuardian? (boolean)"
    },
    "sortBynamestudentId": {
      "type": "string",
      "description": "sortBy? (name|studentId|updatedAt, default name)"
    }
  }
}
```
- **Response DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": {
            "type": "string"
          },
          "studentId": {
            "type": "string"
          },
          "name": {
            "type": "string"
          },
          "classId": {
            "type": "string"
          },
          "sectionId": {
            "type": "string"
          },
          "campusId": {
            "type": "string"
          },
          "status": {
            "type": "string"
          },
          "guardianCount": {
            "type": "integer"
          },
          "isMinor": {
            "type": "boolean"
          },
          "hasActiveGuardian": {
            "type": "boolean"
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
          "studentId",
          "name",
          "classId",
          "sectionId",
          "campusId",
          "status",
          "guardianCount",
          "isMinor",
          "hasActiveGuardian",
          "updatedAt",
          "version"
        ]
      }
    },
    {
      "type": "object",
      "properties": {
        "XXXXX1234": {
          "type": "string",
          "description": "\"XXX-XX-1234\""
        }
      }
    },
    {
      "type": "object",
      "properties": {
        "studentsensitiveview": {
          "type": "string",
          "description": "student.sensitive.view"
        }
      }
    }
  ]
}
```
- **Validation Rules:** No free-text search is permitted against masked sensitive fields without `student.sensitive.view`.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** STU-005 (sensitive-data minimization & masking), AUTHZ-002/003.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Student Detail
- **Method:** `GET`
- **URL:** `/api/v1/students/{id}`
- **Description:** Full profile (masked), guardians, documents, and lifecycle (SCR-STU-02).
- **Authentication (Role-Based):** Yes — `student.view` (masked); self (own profile); guardian (linked child only, GRD-N-004); teacher (own students only).
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
    "studentId": {
      "type": "string"
    },
    "firstName": {
      "type": "string"
    },
    "lastName": {
      "type": "string"
    },
    "dob": {
      "type": "string"
    },
    "gender": {
      "type": "string"
    },
    "nationalId": {
      "type": "string",
      "description": "\"MASKED\"|value"
    },
    "contact": {
      "type": "string",
      "description": "\"MASKED\"|value"
    },
    "dynamicFields": {
      "type": "object"
    },
    "isMinor": {
      "type": "boolean"
    },
    "status": {
      "type": "string"
    },
    "guardianSummary": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
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
    "studentId",
    "firstName",
    "lastName",
    "dob",
    "gender",
    "nationalId",
    "contact",
    "dynamicFields",
    "isMinor",
    "status",
    "guardianSummary",
    "createdAt",
    "updatedAt",
    "createdBy",
    "updatedBy",
    "version"
  ]
}
```
- **Validation Rules:** Sensitive fields are masked by default for any viewer lacking `student.sensitive.view`; the student's own self-view and a linked guardian's view of their own child are **not** automatically unmasked — masking applies uniformly and unmasking always goes through #3.
- **Error Codes:** `404 NOT_FOUND` (out of scope/ownership).
- **Business Rules Applied:** STU-002 (core + dynamic fields), STU-005 (masking).
- **Pagination:** N/A.

### 3. Unmask a Sensitive Field
- **Method:** `POST`
- **URL:** `/api/v1/students/{id}/unmask`
- **Description:** Reveal a specific sensitive field's true value for a need-to-know viewer — every unmask is individually audited (SCR-STU-02).
- **Authentication (Role-Based):** Yes — `student.sensitive.view` (need-to-know, narrowly granted).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "field": {
      "type": "string",
      "description": "\"nationalId\"|\"contact\"|..."
    }
  },
  "required": [
    "field"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "field": {
      "type": "string"
    },
    "value": {
      "type": "string"
    }
  },
  "required": [
    "field",
    "value"
  ]
}
```
- **Validation Rules:** Caller must hold `student.sensitive.view` **and** pass the same scope/ownership checks as #2; the request itself is logged regardless of outcome.
- **Error Codes:** `403 FORBIDDEN` (the field simply stays masked client-side — no error detail leaked).
- **Business Rules Applied:** STU-005 (need-to-know unmasking), Doc 06 §11 (`SENSITIVE_FIELD_ACCESSED` audited).
- **Pagination:** N/A.

## Create

### 4. Check for Duplicate Candidates
- **Method:** `GET`
- **URL:** `/api/v1/students/dedup-check`
- **Description:** Surface likely-duplicate students before creating a new record (SCR-STU-03).
- **Authentication (Role-Based):** Yes — `student.create`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "firstName": {
      "type": "string"
    },
    "lastName": {
      "type": "string"
    },
    "dob": {
      "type": "string"
    },
    "guardianContact": {
      "type": "string"
    }
  },
  "required": [
    "firstName",
    "lastName",
    "dob"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "candidates": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": {
            "type": "string"
          },
          "studentId": {
            "type": "string"
          },
          "name": {
            "type": "string"
          },
          "dob": {
            "type": "string"
          },
          "matchedOn": {
            "type": "array",
            "items": {
              "type": "string"
            }
          }
        },
        "required": [
          "id",
          "studentId",
          "name",
          "dob",
          "matchedOn"
        ]
      }
    }
  },
  "required": [
    "candidates"
  ]
}
```
- **Validation Rules:** Matches on name + DOB + guardian, or government ID where captured; presented for human review — never auto-merged and never auto-blocked.
- **Error Codes:** None (empty `candidates[]` is a normal "no match" result).
- **Business Rules Applied:** STU-003 (duplicate-student detection).
- **Pagination:** N/A.

### 5. Create Student
- **Method:** `POST`
- **URL:** `/api/v1/students`
- **Description:** Materialize a student record outside the admission-conversion path (migration/manual case), with a unique, immutable ID (SCR-STU-03, UC-STU-01).
- **Authentication (Role-Based):** Yes — `student.create` (elevated for this direct-creation path; conversion's own creation runs through `12-admission-api.md` #16 instead).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "firstName": {
      "type": "string"
    },
    "lastName": {
      "type": "string"
    },
    "dob": {
      "type": "string"
    },
    "gender": {
      "type": "string"
    },
    "dynamicFields": {
      "type": "object"
    },
    "dedupConfirmation": {
      "type": "string",
      "description": "\"CONFIRMED_NEW\"|{ mergeIntoId: string }"
    }
  },
  "required": [
    "firstName",
    "lastName",
    "dob",
    "gender",
    "dedupConfirmation"
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
    "studentId": {
      "type": "string",
      "description": "auto, immutable"
    },
    "status": {
      "type": "string",
      "const": "PRE_ENROLLED"
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
    "studentId",
    "status",
    "createdAt",
    "version"
  ]
}
```
- **Validation Rules:** `studentId` is generated atomically per the configured scheme and is **never editable thereafter** (STU-001); `dynamicFields` validated against their published Configuration Engine definitions (CFG-010, STU-002); the dedup check (#4) result must be explicitly confirmed (`CONFIRMED_NEW`) before creation proceeds — the system never silently creates or silently blocks on a duplicate match (STU-003); a minor (computed from `dob` against the configured age of majority) is created in `PRE_ENROLLED` but **cannot transition to `ACTIVE` until ≥1 guardian is linked** ("A minor must have at least one guardian," STU-006) — that gate is enforced at enrollment finalization (`15-enrollment-api.md`), not blocking creation itself.
- **Error Codes:** `422 DUPLICATE_NOT_CONFIRMED`; `422 DYNAMIC_FIELD_VALIDATION_FAILED`.
- **Business Rules Applied:** STU-001 (unique immutable ID), STU-002 (core + dynamic fields), STU-003 (duplicate detection), STU-006 (minor guardian gate, enforced downstream).
- **Pagination:** N/A.

## Official-Field Edits (Governed)

### 6. Update Official Fields
- **Method:** `PUT`
- **URL:** `/api/v1/students/{id}/official-fields`
- **Description:** Edit legally/operationally significant fields (legal name, DOB, government ID) under governance (SCR-STU-04).
- **Authentication (Role-Based):** Yes — `student.official.update` (raises the request).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "firstName": {
      "type": "string"
    },
    "lastName": {
      "type": "string"
    },
    "dob": {
      "type": "string"
    },
    "nationalId": {
      "type": "string"
    },
    "justification": {
      "type": "string"
    },
    "version": {
      "type": "number"
    }
  },
  "required": [
    "justification",
    "version"
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
        "id": {
          "type": "string"
        },
        "status": {
          "type": "string",
          "const": "PENDING_APPROVAL"
        },
        "requestId": {
          "type": "string"
        }
      },
      "required": [
        "id",
        "status",
        "requestId"
      ]
    },
    {
      "type": "object",
      "properties": {
        "studentId": {
          "type": "string"
        }
      },
      "required": [
        "studentId"
      ]
    }
  ]
}
```
- **Validation Rules:** `justification` required ("This change needs approval"); the immutable `studentId` is rejected if supplied ("Student ID can't be changed," STU-001); operations on a `GRADUATED`/`ARCHIVED` student are blocked (STU-007); `version` optimistic lock.
- **Error Codes:** `422 IMMUTABLE_FIELD` (if `studentId` supplied); `409 STUDENT_NOT_EDITABLE` (graduated/archived); `409 VERSION_CONFLICT`.
- **Business Rules Applied:** STU-001 (immutable ID), STU-004 (official-field edit control), STU-007.
- **Pagination:** N/A.

### 7. List Sensitive-Change Approvals
- **Method:** `GET`
- **URL:** `/api/v1/students/sensitive-change-approvals`
- **Description:** Inbox for pending official-field changes (SCR-STU-14).
- **Authentication (Role-Based):** Yes — `student.change.approve` *(named per the established view/manage/approve pairing convention from `00-api-conventions.md` §3; the approved screen names the combination "`student.change.approve`-class" without committing to one literal string)*, scoped.
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
    "status": {
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
      "id": {
        "type": "string"
      },
      "studentId": {
        "type": "string"
      },
      "change": {
        "type": "object",
        "properties": {
          "field": {
            "type": "string"
          },
          "before": {
            "type": "string"
          },
          "after": {
            "type": "string"
          }
        },
        "required": [
          "field",
          "before",
          "after"
        ]
      },
      "requestedBy": {
        "type": "string"
      },
      "justification": {
        "type": "string"
      },
      "slaDueAt": {
        "type": "string"
      },
      "status": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "studentId",
      "change",
      "requestedBy",
      "justification",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Requester ≠ approver (AUTHZ-009).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** STU-004, AUTHZ-009.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 8. Decide Sensitive-Change Request
- **Method:** `POST`
- **URL:** `/api/v1/students/sensitive-change-approvals/{id}/decide`
- **Description:** Approve or reject a pending official-field change; on approval, the change in #6 is committed with full before/after audit.
- **Authentication (Role-Based):** Yes — `student.change.approve`; **blocked if the caller raised the request**.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "decision": {
      "type": "string",
      "enum": [
        "APPROVE",
        "REJECT"
      ]
    },
    "reason": {
      "type": "string",
      "description": "required on REJECT"
    }
  },
  "required": [
    "decision"
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
      "enum": [
        "APPROVED",
        "REJECTED"
      ]
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** Caller ≠ requester; `reason` required on `REJECT`.
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED`; `422 REASON_REQUIRED`.
- **Business Rules Applied:** AUTHZ-009, STU-004.
- **Pagination:** N/A.

## Dynamic Fields & Lifecycle Status

### 9. Update Dynamic Fields
- **Method:** `PUT`
- **URL:** `/api/v1/students/{id}/dynamic-fields`
- **Description:** Edit institute-defined custom fields, engine-validated (SCR-STU-05).
- **Authentication (Role-Based):** Yes — `student.dynamic.update`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "dynamicFields": {
      "type": "object"
    },
    "version": {
      "type": "number"
    }
  },
  "required": [
    "dynamicFields",
    "version"
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
    "dynamicFields": {
      "type": "string"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "dynamicFields",
    "version"
  ]
}
```
- **Validation Rules:** Each field validated against its published Configuration Engine schema definition (CFG-010); `version` optimistic lock.
- **Error Codes:** `422 DYNAMIC_FIELD_VALIDATION_FAILED`; `409 VERSION_CONFLICT`.
- **Business Rules Applied:** STU-002, STU-004 (non-sensitive edits apply immediately), CFG-010.
- **Pagination:** N/A.

### 10. Change Lifecycle Status
- **Method:** `POST`
- **URL:** `/api/v1/students/{id}/status`
- **Description:** Manage derived/managed status — most transitions are driven by Enrollment/Result events, but explicit manual transitions with reason are also supported (SCR-STU-06).
- **Authentication (Role-Based):** Yes — `student.status.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "newStatus": {
      "type": "string"
    },
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "newStatus",
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
      "type": "string"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** Only transitions allowed by the state machine and consistent with the student's actual enrollment/results are permitted; an admin cannot jump a student to a status that contradicts reality (e.g., `GRADUATED` without reaching the final level) except via an explicit, audited, approved override (STU-008).
- **Error Codes:** `409 INVALID_STATE_TRANSITION`; `409 INCONSISTENT_WITH_ENROLLMENT` (requires override approval).
- **Business Rules Applied:** STU-008 (status reflects lifecycle, event-driven), Doc 06 §6 (state machine).
- **Pagination:** N/A.

## Guardians

### 11. List Student's Guardians
- **Method:** `GET`
- **URL:** `/api/v1/students/{id}/guardians`
- **Description:** View a student's linked guardians with roles and restrictions (SCR-STU-07).
- **Authentication (Role-Based):** Yes — `student.view`.
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
      "guardianId": {
        "type": "string"
      },
      "name": {
        "type": "string"
      },
      "relationship": {
        "type": "string"
      },
      "isFinancialResponsible": {
        "type": "boolean"
      },
      "isPrimary": {
        "type": "boolean"
      },
      "restrictions": {
        "type": "array",
        "items": {
          "type": "string"
        }
      }
    },
    "required": [
      "guardianId",
      "name",
      "relationship",
      "isFinancialResponsible",
      "isPrimary",
      "restrictions"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** STU-006, GRD-N-001.
- **Pagination:** N/A.

### 12. Link Guardian
- **Method:** `POST`
- **URL:** `/api/v1/students/{id}/guardians`
- **Description:** Link a (new or existing) guardian to the student (SCR-STU-07).
- **Authentication (Role-Based):** Yes — `student.guardian.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "guardianIdstringex": {
          "type": "string",
          "description": "{ guardianId: string (existing, or use"
        }
      }
    },
    {
      "type": "object",
      "properties": {
        "4tocreatefirstrel": {
          "type": "string",
          "description": "#4 to create first), relationship: string, isFinancialResponsible?: boolean }"
        }
      }
    }
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
    "guardianId": {
      "type": "string"
    },
    "relationship": {
      "type": "string"
    },
    "linkedAt": {
      "type": "string"
    }
  },
  "required": [
    "studentId",
    "guardianId",
    "relationship",
    "linkedAt"
  ]
}
```
- **Validation Rules:** None beyond access (this is an additive link; see #14 for the protected removal path).
- **Error Codes:** `404 GUARDIAN_NOT_FOUND`.
- **Business Rules Applied:** STU-006 (guardian linkage), GRD-N-003 (sibling reuse — link existing rather than duplicate).
- **Pagination:** N/A.

### 13. Set Guardian Restrictions on a Link
- **Method:** `PUT`
- **URL:** `/api/v1/students/{id}/guardians/{guardianId}/restrictions`
- **Description:** Apply custody/contact restrictions for this specific student-guardian relationship, enforced across data, notifications, and files (SCR-STU-07).
- **Authentication (Role-Based):** Yes — `guardian.restriction.manage` (elevated, distinct from general guardian-link management).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "restrictions": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "(\"NO_ACCESS\"|\"NO_CONTACT\"|\"SUPERVISED\")"
      }
    },
    "reason": {
      "type": "string"
    },
    "documentRef": {
      "type": "string"
    }
  },
  "required": [
    "restrictions",
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
    "guardianId": {
      "type": "string"
    },
    "restrictions": {
      "type": "string"
    },
    "appliedAt": {
      "type": "string"
    }
  },
  "required": [
    "studentId",
    "guardianId",
    "restrictions",
    "appliedAt"
  ]
}
```
- **Validation Rules:** `reason` required; the restriction takes effect **immediately** across portal access, notification routing, and file access — never delayed.
- **Error Codes:** `422 REASON_REQUIRED`.
- **Business Rules Applied:** GRD-N-006 (custody/contact restrictions enforced system-wide).
- **Pagination:** N/A.

### 14. Unlink Guardian
- **Method:** `DELETE`
- **URL:** `/api/v1/students/{id}/guardians/{guardianId}`
- **Description:** Remove a guardian link — **hard-blocked** if it would leave a minor with zero active guardians (SCR-STU-07).
- **Authentication (Role-Based):** Yes — `student.guardian.manage`.
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
  "type": "null",
  "description": "204 No Content."
}
```
- **Validation Rules:** If the student is a minor and this is their only active guardian link, the removal is rejected outright ("Cannot unlink the last guardian of a minor," STU-006/GRD-N-002) — a replacement must be linked first.
- **Error Codes:** `409 LAST_GUARDIAN_OF_MINOR`.
- **Business Rules Applied:** STU-006, GRD-N-002 (a minor must always have ≥1 active guardian).
- **Pagination:** N/A.

## Photo & Documents

### 15. List Student Documents
- **Method:** `GET`
- **URL:** `/api/v1/students/{id}/documents`
- **Description:** List documents and photo with version history (SCR-STU-08).
- **Authentication (Role-Based):** Yes — `student.view`.
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
      "id": {
        "type": "string"
      },
      "type": {
        "type": "string"
      },
      "version": {
        "type": "integer"
      },
      "uploadedBy": {
        "type": "string"
      },
      "uploadedAt": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "type",
      "version",
      "uploadedBy",
      "uploadedAt"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** FILE-006 (versioning).
- **Pagination:** N/A.

### 16. Upload Document
- **Method:** `POST`
- **URL:** `/api/v1/students/{id}/documents`
- **Description:** Upload a document or photo — scanned for malware, photo metadata stripped (SCR-STU-08).
- **Authentication (Role-Based):** Yes — `student.document.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "multipartformdatafi": {
      "type": "string",
      "description": "multipart/form-data { file, type: \"PHOTO\"|\"BIRTH_CERTIFICATE\"|... }"
    }
  }
}
```
- **Response DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "status": {
          "type": "string",
          "const": "SCAN_PENDING"
        }
      },
      "required": [
        "id",
        "status"
      ]
    },
    {
      "type": "object",
      "properties": {
        "READY": {
          "type": "string"
        }
      },
      "required": [
        "READY"
      ]
    }
  ]
}
```
- **Validation Rules:** File type/size validated; uploads are malware-scanned before becoming available — a failed scan quarantines the file and it is never served (FILE-003); photo uploads have EXIF/GPS location metadata stripped on ingest (FILE-006).
- **Error Codes:** `422 INVALID_FILE_TYPE`; the scan-failed state is surfaced via the document's status, not a synchronous error.
- **Business Rules Applied:** FILE-003 (malware scan), FILE-006 (versioning & metadata hygiene).
- **Pagination:** N/A.

### 17. Replace Document (New Version)
- **Method:** `POST`
- **URL:** `/api/v1/students/{id}/documents/{docId}/replace`
- **Description:** Upload a new version of an existing document, preserving history (SCR-STU-08).
- **Authentication (Role-Based):** Yes — `student.document.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "multipartformdatafi": {
      "type": "string",
      "description": "multipart/form-data { file }"
    }
  }
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
    "version": {
      "type": "number"
    },
    "status": {
      "type": "string",
      "const": "SCAN_PENDING"
    }
  },
  "required": [
    "id",
    "version",
    "status"
  ]
}
```
- **Validation Rules:** Same scan/metadata rules as #16; the prior version is retained, not overwritten (FILE-006).
- **Error Codes:** `422 INVALID_FILE_TYPE`.
- **Business Rules Applied:** FILE-006.
- **Pagination:** N/A.

### 18. Get Document Download URL
- **Method:** `GET`
- **URL:** `/api/v1/students/{id}/documents/{docId}/download-url`
- **Description:** Obtain a short-lived signed URL to view/download a document — never a raw public URL (SCR-STU-08).
- **Authentication (Role-Based):** Yes — `student.document.manage`, or the viewing party's own subject-scoped access (self, linked non-restricted guardian).
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
- **Validation Rules:** Access is subject-scoped and custody-aware — a restricted guardian (per `13. Set Guardian Restrictions`) is denied even if otherwise linked (GRD-N-006).
- **Error Codes:** `403 FORBIDDEN` (restricted or out of scope).
- **Business Rules Applied:** FILE-004 (subject-scoped, custody-aware access), FILE-005 (short-lived signed URLs).
- **Pagination:** N/A.

### 19. Delete Document (Governed)
- **Method:** `DELETE`
- **URL:** `/api/v1/students/{id}/documents/{docId}`
- **Description:** Remove a document — a governed action, not a casual delete (SCR-STU-08).
- **Authentication (Role-Based):** Yes — `student.document.manage`.
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
  "type": "null",
  "description": "204 No Content."
}
```
- **Validation Rules:** `reason` required; deletion is logged and, where the document is referenced by other records (e.g., an admission application), handled per the owning record's own integrity rules.
- **Error Codes:** `422 REASON_REQUIRED`.
- **Business Rules Applied:** FILE-004 (governed access/deletion).
- **Pagination:** N/A.

## Archive, Anonymize & Subject Access

### 20. Archive Student
- **Method:** `POST`
- **URL:** `/api/v1/students/{id}/archive`
- **Description:** Move a student to `WITHDRAWN`/`GRADUATED`/`ARCHIVED`, preserving all academic/financial history immutably — **never** a hard delete (SCR-STU-09).
- **Authentication (Role-Based):** Yes — `student.archive`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "targetStatus": {
      "type": "string",
      "enum": [
        "WITHDRAWN",
        "GRADUATED",
        "ARCHIVED"
      ]
    },
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "targetStatus",
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
      "type": "string"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** `reason` required; transition must be consistent with the student's actual enrollment/results history (STU-008); a hard-delete attempt on a data-bearing student is rejected outright at the data layer regardless of caller intent.
- **Error Codes:** `422 REASON_REQUIRED`; `409 INCONSISTENT_WITH_HISTORY`.
- **Business Rules Applied:** STU-007 (archive, never hard-delete), STU-008.
- **Pagination:** N/A.

### 21. Anonymize Student (Erasure)
- **Method:** `POST`
- **URL:** `/api/v1/students/{id}/anonymize`
- **Description:** Lawful, retention-driven erasure of personal data — export-before-purge, preserving non-identifying integrity references. Reached either directly (SCR-STU-09) or via a formal Data-Subject erasure request (SCR-STU-10), both of which converge on this same governed operation.
- **Authentication (Role-Based):** Yes — `student.anonymize` (direct) or `student.subject_access` (when reached via a formal DSR — same underlying action, different entry point/verification).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "reason": {
      "type": "string"
    },
    "requestType": {
      "type": "string",
      "enum": [
        "DIRECT",
        "DATA_SUBJECT_REQUEST"
      ]
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
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "status": {
          "type": "string",
          "const": "ANONYMIZED"
        }
      },
      "required": [
        "id",
        "status"
      ]
    },
    {
      "type": "object",
      "properties": {
        "409": {
          "type": "string",
          "description": "409"
        }
      }
    }
  ]
}
```
- **Validation Rules:** Erasure is **blocked outright while a legal hold is active** on the student's record, with the block explained, not silently ignored ("erasure under legal hold blocked with explanation," C-08); statutory data (financial/academic records required by law) is retained even after PII is stripped; non-identifying audit/integrity references persist permanently.
- **Error Codes:** `409 LEGAL_HOLD_ACTIVE`.
- **Business Rules Applied:** STU-007 (archive/anonymize, never hard-delete), C-08 (legal-hold > statutory > erasure precedence).
- **Pagination:** N/A.

### 22. Generate Subject-Access Report
- **Method:** `POST`
- **URL:** `/api/v1/students/{id}/subject-access-report`
- **Description:** Generate a governed export of all data held about the student, for a formal subject-access request (SCR-STU-10).
- **Authentication (Role-Based):** Yes — `student.subject_access`.
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
- **Validation Rules:** Scope-limited; generation is audited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** STU-007, REP-002, conventions §10.
- **Pagination:** N/A.

## Reporting & Data Transfer

### 23. Roster & Demographics Report
- **Method:** `GET`
- **URL:** `/api/v1/students/report/roster-demographics`
- **Description:** Roster and demographic breakdowns within scope, with sensitive fields masked in the output (SCR-STU-11).
- **Authentication (Role-Based):** Yes — `student.view` (the approved screen names a generic "report view"; reused here per the established convention).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "classId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "campusId": {
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
      "classId": {
        "type": "string"
      },
      "sectionId": {
        "type": "string"
      },
      "totalStudents": {
        "type": "string"
      },
      "byGender": {
        "type": "object",
        "properties": {
          "note": {
            "type": "string",
            "description": "..."
          }
        }
      },
      "byStatus": {
        "type": "object",
        "properties": {
          "note": {
            "type": "string",
            "description": "..."
          }
        }
      }
    },
    "required": [
      "classId",
      "sectionId",
      "totalStudents",
      "byGender",
      "byStatus"
    ]
  }
}
```
- **Validation Rules:** Sensitive fields masked in all report output, with no separate unmask path within the report itself (REP-003).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** STU-005, REP-002, REP-003 (masking in reports).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 24. Import Students
- **Method:** `POST`
- **URL:** `/api/v1/students/import`
- **Description:** Bulk-import students with dedup and minor-guardian-gate validation (SCR-STU-12).
- **Authentication (Role-Based):** Yes — `student.import`.
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
              "firstName": {
                "type": "string"
              },
              "lastName": {
                "type": "string"
              },
              "dob": {
                "type": "string"
              },
              "gender": {
                "type": "string"
              },
              "dynamicFields": {
                "type": "string"
              }
            },
            "required": [
              "firstName",
              "lastName",
              "dob",
              "gender"
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
              "studentId": {
                "type": "string"
              }
            },
            "required": [
              "row",
              "studentId"
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
- **Validation Rules:** Each row run through the same dedup heuristic as #4 (flagged, not silently merged or blocked, STU-003); minors imported without an accompanying guardian-linkage path are flagged rather than silently activated (STU-006); invalid/flagged rows excluded and always reported.
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** STU-001, STU-003, STU-006, CFG-010; conventions §9.
- **Pagination:** N/A.

### 25. Export Student Data
- **Method:** `GET`
- **URL:** `/api/v1/students/export`
- **Description:** Governed, masked export of student data (SCR-STU-13).
- **Authentication (Role-Based):** Yes — `student.export`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "classId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "status": {
      "type": "string"
    }
  }
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
- **Validation Rules:** Scope-limited; export is audited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** STU-005, REP-002, REP-008 (audited export), conventions §10.
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
