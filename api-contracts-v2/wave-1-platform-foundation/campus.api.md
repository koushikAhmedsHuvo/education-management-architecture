# Campus API

## 1. API Overview

**Purpose.** Govern the campus — the secondary, optional scope boundary nested under institute, for multi-branch institutes.

**Module Context.** Implements Business Rules Catalog Doc 04 (Campus) and UI Screen Spec `05-campus-management-screens.md` (SCR-CAMP-01…09/12).

---

## 2. Endpoints

### 1. List Campuses
- **Method:** `GET`
- **URL:** `/api/v1/institutes/{instituteId}/campuses`
- **Description:** Browse an institute's campuses (SCR-CAMP-01).
- **Authentication (Role-Based):** Yes — `campus.view`, institute-scoped (CAMP-004: campus access is bounded by, never exceeds, institute access).
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
    "statusACTIVEINACTIVE": {
      "type": "string",
      "description": "status? (ACTIVE|INACTIVE|ARCHIVED)"
    },
    "sortBynamestudentCou": {
      "type": "string",
      "description": "sortBy? (name|studentCount|status, default name)"
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
      "isDefault": {
        "type": "boolean"
      },
      "studentCount": {
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
      "isDefault",
      "studentCount",
      "status",
      "updatedAt",
      "version"
    ]
  }
}
```
- **Validation Rules:** A campus admin sees only their own campus; an institute admin sees all campuses of their institute (CAMP-004).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CAMP-004 (campus scope filtering, nested within institute).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Campus Detail
- **Method:** `GET`
- **URL:** `/api/v1/campuses/{id}`
- **Description:** Full campus profile (SCR-CAMP-02).
- **Authentication (Role-Based):** Yes — `campus.view`.
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
    "instituteId": {
      "type": "string"
    },
    "name": {
      "type": "string"
    },
    "code": {
      "type": "string"
    },
    "isDefault": {
      "type": "boolean"
    },
    "address": {
      "type": "string"
    },
    "contact": {
      "type": "string"
    },
    "status": {
      "type": "string"
    },
    "studentCount": {
      "type": "integer"
    },
    "sectionCount": {
      "type": "integer"
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
    "instituteId",
    "name",
    "code",
    "isDefault",
    "status",
    "studentCount",
    "sectionCount",
    "createdAt",
    "updatedAt",
    "createdBy",
    "updatedBy",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND` (out-of-institute-scope).
- **Business Rules Applied:** CAMP-001 (campus belongs to exactly one institute), CAMP-004.
- **Pagination:** N/A.

### 3. Create Campus
- **Method:** `POST`
- **URL:** `/api/v1/institutes/{instituteId}/campuses`
- **Description:** Add a branch to an institute (UC-CAMP-01).
- **Authentication (Role-Based):** Yes — `campus.create`, institute-scoped.
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
    "address": {
      "type": "string"
    },
    "contact": {
      "type": "string"
    }
  },
  "required": [
    "name",
    "code"
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
    "instituteId": {
      "type": "string"
    },
    "name": {
      "type": "string"
    },
    "code": {
      "type": "string"
    },
    "isDefault": {
      "type": "boolean",
      "const": false
    },
    "status": {
      "type": "string",
      "const": "ACTIVE"
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
    "instituteId",
    "name",
    "code",
    "isDefault",
    "status",
    "createdAt",
    "version"
  ]
}
```
- **Validation Rules:** `code` unique **within the institute** (not globally — two institutes may each have "MAIN", CAMP-003), immutable after creation; `instituteId` must be one the caller is bound to (no cross-institute creation, CAMP-001).
- **Error Codes:** `409 CAMPUS_CODE_EXISTS_IN_INSTITUTE`.
- **Business Rules Applied:** CAMP-001 (single-institute ownership, no reparenting), CAMP-003 (per-institute code uniqueness).
- **Pagination:** N/A.

### 4. Update Campus
- **Method:** `PUT`
- **URL:** `/api/v1/campuses/{id}`
- **Description:** Edit name, address, contact (SCR-CAMP-02 Edit action).
- **Authentication (Role-Based):** Yes — `campus.update`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "address": {
      "type": "string"
    },
    "contact": {
      "type": "string"
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
- **Validation Rules:** `code` and `instituteId` are never editable via this endpoint (CAMP-001 no reparenting; CAMP-003 immutable code); `version` optimistic lock.
- **Error Codes:** `409 VERSION_CONFLICT`; `422 IMMUTABLE_FIELD` if `code` or `instituteId` is supplied.
- **Business Rules Applied:** CAMP-001, CAMP-003.
- **Pagination:** N/A.

### 5. Get Campus Configuration Overrides
- **Method:** `GET`
- **URL:** `/api/v1/campuses/{id}/config-overrides`
- **Description:** View campus-level overrides for genuinely branch-specific settings (SCR-CAMP-05).
- **Authentication (Role-Based):** Yes — `campus.config.manage`.
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
      "definitionKey": {
        "type": "string"
      },
      "value": {
        "type": "string"
      },
      "inheritedFrom": {
        "type": "string",
        "enum": [
          "organization",
          "institute",
          "campus"
        ]
      }
    },
    "required": [
      "definitionKey",
      "value",
      "inheritedFrom"
    ]
  }
}
```
- **Validation Rules:** Read-only; only `campus-overridable`-flagged definitions appear.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CAMP-005 (campus-level override, most-specific-wins beats institute beats org).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 6. Set Campus Configuration Override
- **Method:** `PUT`
- **URL:** `/api/v1/campuses/{id}/config-overrides/{definitionKey}`
- **Description:** Set or reset a campus-level override.
- **Authentication (Role-Based):** Yes — `campus.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "value": {
          "type": "object"
        }
      },
      "required": [
        "value"
      ]
    },
    {
      "type": "object",
      "properties": {
        "reset": {
          "type": "boolean",
          "const": true
        }
      },
      "required": [
        "reset"
      ]
    }
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "definitionKey": {
      "type": "string"
    },
    "value": {
      "type": "string"
    },
    "inheritedFrom": {
      "type": "string"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "definitionKey",
    "value",
    "inheritedFrom",
    "version"
  ]
}
```
- **Validation Rules:** The setting must be flagged `campus-overridable`; `value` validated against its definition; rejected/disallowed overrides fall back to the inherited (institute) value.
- **Error Codes:** `422 NOT_CAMPUS_OVERRIDABLE`; `422 INVALID_OVERRIDE_VALUE`.
- **Business Rules Applied:** CAMP-005.
- **Pagination:** N/A.

### 7. Transfer a Student Between Campuses
- **Method:** `POST`
- **URL:** `/api/v1/campuses/transfers`
- **Description:** Move a student to a different campus within the **same** institute as a controlled, audited, effective-dated transfer — never a direct edit (SCR-CAMP-06).
- **Authentication (Role-Based):** Yes — `campus.transfer.execute`-class authority (Registrar); routes to approval separately (#10).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string"
    },
    "targetCampusId": {
      "type": "string"
    },
    "effectiveDate": {
      "type": "string"
    },
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "studentId",
    "targetCampusId",
    "effectiveDate",
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
    "studentId": {
      "type": "string"
    },
    "sourceCampusId": {
      "type": "string"
    },
    "targetCampusId": {
      "type": "string"
    },
    "effectiveDate": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "PENDING_APPROVAL"
    }
  },
  "required": [
    "id",
    "studentId",
    "sourceCampusId",
    "targetCampusId",
    "effectiveDate",
    "status"
  ]
}
```
- **Validation Rules:** `targetCampusId ≠` the student's current campus ("Target must differ from current campus"); `targetCampusId` must belong to the **same institute** as the source ("Cross-institute transfer not allowed" — that is a different process entirely, not exposed by this endpoint); `targetCampusId` must be `ACTIVE`.
- **Error Codes:** `422 SAME_CAMPUS`; `409 CROSS_INSTITUTE_TRANSFER_NOT_ALLOWED`; `409 TARGET_CAMPUS_NOT_ACTIVE`.
- **Business Rules Applied:** CAMP-006 (inter-campus movement is a controlled, audited transfer with full history preservation).
- **Pagination:** N/A.

### 8. Bulk Transfer Students
- **Method:** `POST`
- **URL:** `/api/v1/campuses/transfers/bulk`
- **Description:** Transfer a cohort (selection or CSV) to a target campus (SCR-CAMP-07).
- **Authentication (Role-Based):** Yes — same transfer authority as #7.
- **Request DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "studentIds": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "targetCampusId": {
          "type": "string"
        },
        "effectiveDate": {
          "type": "string"
        },
        "reason": {
          "type": "string"
        }
      },
      "required": [
        "studentIds",
        "targetCampusId",
        "effectiveDate",
        "reason"
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
              "studentId": {
                "type": "string"
              },
              "transferId": {
                "type": "string"
              }
            },
            "required": [
              "studentId",
              "transferId"
            ]
          }
        },
        "rejected": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "studentId": {
                "type": "string"
              },
              "reason": {
                "type": "string"
              }
            },
            "required": [
              "studentId",
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
- **Validation Rules:** Each student validated independently per #7's rules (same institute, not source campus, eligible); invalid rows excluded and reported — no silent drops.
- **Error Codes:** `400 MALFORMED_BATCH`; per-row failures reported in `rejected[]`.
- **Business Rules Applied:** CAMP-006; conventions §9 (async/idempotent bulk pattern).
- **Pagination:** N/A.

### 9. Deactivate / Archive Campus
- **Method:** `POST`
- **URL:** `/api/v1/campuses/{id}/deactivate`
- **Description:** Take a campus out of operation, blocked while active dependents exist (SCR-CAMP-08, UC-CAMP-02).
- **Authentication (Role-Based):** Yes — `campus.deactivate` / `campus.archive` respectively; routes to approval separately given student impact (#10).
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
      "enum": [
        "INACTIVE",
        "ARCHIVED",
        "PENDING_APPROVAL"
      ]
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "status",
    "version"
  ]
}
```
- **Validation Rules:** Blocked if active sections/students/staff are still assigned to the campus (CAMP-007 — "Campus has active students/classes"); blocked if this is the institute's **last active campus** while the institute is `ACTIVE` (CAMP-002 — "Cannot deactivate the last active campus"); the default campus cannot be deactivated while it is also the last active one.
- **Error Codes:** `409 ACTIVE_DEPENDENTS_PRESENT`; `409 LAST_ACTIVE_CAMPUS`.
- **Business Rules Applied:** CAMP-002 (default-campus guarantee), CAMP-007 (no active dependents).
- **Pagination:** N/A.

### 10. Get Transfer / Closure Approvals
- **Method:** `GET`
- **URL:** `/api/v1/campuses/approvals`
- **Description:** Inbox for pending campus transfers and closures (SCR-CAMP-12).
- **Authentication (Role-Based):** Yes — campus transfer/closure approval authority, scoped.
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
    "typeTRANSFERBULK_TRA": {
      "type": "string",
      "description": "type? (TRANSFER|BULK_TRANSFER|DEACTIVATE|ARCHIVE)"
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
      "type": {
        "type": "string"
      },
      "summary": {
        "type": "string"
      },
      "requestedBy": {
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
      "type",
      "summary",
      "requestedBy",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Requester ≠ approver (AUTHZ-009).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** Doc 04 §8 (closure/transfer may require approval given student impact), AUTHZ-009.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 11. Decide Campus Request
- **Method:** `POST`
- **URL:** `/api/v1/campuses/approvals/{id}/decide`
- **Description:** Approve/reject a pending transfer or closure.
- **Authentication (Role-Based):** Yes — same authority as #10; blocked if caller raised the request.
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
- **Validation Rules:** Caller ≠ requester.
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED`; `422 REASON_REQUIRED`.
- **Business Rules Applied:** AUTHZ-009, CAMP-006, CAMP-007.
- **Pagination:** N/A.

### 12. Campus Roster & Branch Dashboard
- **Method:** `GET`
- **URL:** `/api/v1/campuses/{id}/dashboard`
- **Description:** Branch-level KPI summary for a campus admin (SCR-CAMP-09).
- **Authentication (Role-Based):** Yes — campus roster/report view authority, scope-bound to the campus.
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
    "studentCount": {
      "type": "integer"
    },
    "staffCount": {
      "type": "integer"
    },
    "sectionCount": {
      "type": "integer"
    },
    "recentTransfersIn": {
      "type": "string"
    },
    "recentTransfersOut": {
      "type": "string"
    }
  },
  "required": [
    "studentCount",
    "staffCount",
    "sectionCount",
    "recentTransfersIn",
    "recentTransfersOut"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CAMP-004 (scope-bound to the campus).
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
