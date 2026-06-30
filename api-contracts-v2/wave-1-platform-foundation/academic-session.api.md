# Academic Session API

## 1. API Overview

**Purpose.** Govern the academic session — the time dimension everything academic and financial attaches to, including controlled overlap, structure instancing, rollover/promotion, and close/reopen.

**Module Context.** Implements Business Rules Catalog Doc 05 (Academic Session) and UI Screen Spec `06-academic-session-screens.md` (SCR-SESS-01…09).

---

## 2. Endpoints

### 1. List Sessions
- **Method:** `GET`
- **URL:** `/api/v1/institutes/{instituteId}/sessions`
- **Description:** Browse an institute's sessions (SCR-SESS-01).
- **Authentication (Role-Based):** Yes — `session.view`, institute-scoped.
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
    "statusDRAFTACTIVECL": {
      "type": "string",
      "description": "status? (DRAFT|ACTIVE|CLOSED|ARCHIVED)"
    },
    "sortBydefaultstartDa": {
      "type": "string",
      "description": "sortBy? (default startDate desc)"
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
      "startDate": {
        "type": "string"
      },
      "endDate": {
        "type": "string"
      },
      "admissionOpen": {
        "type": "boolean"
      },
      "isCurrent": {
        "type": "boolean"
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
      "startDate",
      "endDate",
      "admissionOpen",
      "isCurrent",
      "status",
      "updatedAt",
      "version"
    ]
  }
}
```
- **Validation Rules:** Institute-scoped only.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** SESS-002 (more than one non-archived session may exist), SESS-003 (exactly one `current`).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Session Detail
- **Method:** `GET`
- **URL:** `/api/v1/sessions/{id}`
- **Description:** Status, dates, structure-instancing state, and close-readiness summary (SCR-SESS-02).
- **Authentication (Role-Based):** Yes — `session.view`.
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
    "startDate": {
      "type": "string"
    },
    "endDate": {
      "type": "string"
    },
    "admissionOpen": {
      "type": "string"
    },
    "isCurrent": {
      "type": "boolean"
    },
    "status": {
      "type": "string"
    },
    "structureInstantiated": {
      "type": "boolean"
    },
    "closeReadiness": {
      "type": "object",
      "properties": {
        "unpublishedResults": {
          "type": "string"
        },
        "outstandingDues": {
          "type": "string"
        },
        "openWorkflows": {
          "type": "string"
        }
      },
      "required": [
        "unpublishedResults",
        "outstandingDues",
        "openWorkflows"
      ]
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
    "startDate",
    "endDate",
    "admissionOpen",
    "isCurrent",
    "status",
    "structureInstantiated",
    "createdAt",
    "updatedAt",
    "createdBy",
    "updatedBy",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND` (out of scope).
- **Business Rules Applied:** SESS-004 (structure instances are per-session), SESS-007 (close-readiness fields).
- **Pagination:** N/A.

### 3. Create / Define Session
- **Method:** `POST`
- **URL:** `/api/v1/institutes/{instituteId}/sessions`
- **Description:** Define a new session — year, term, or semester (SCR-SESS-03, UC-SESS-01).
- **Authentication (Role-Based):** Yes — `session.create`, institute-scoped.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "startDate": {
      "type": "string"
    },
    "endDate": {
      "type": "string"
    },
    "admissionOpen": {
      "type": "boolean",
      "description": "default false"
    },
    "openEnded": {
      "type": "boolean",
      "description": "only for institution types that permit it"
    }
  },
  "required": [
    "name",
    "startDate",
    "endDate"
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
    "startDate": {
      "type": "string"
    },
    "endDate": {
      "type": "string"
    },
    "admissionOpen": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "DRAFT"
    },
    "isCurrent": {
      "type": "boolean",
      "const": false
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
    "startDate",
    "endDate",
    "admissionOpen",
    "status",
    "isCurrent",
    "createdAt",
    "version"
  ]
}
```
- **Validation Rules:** `startDate < endDate` unless `openEnded` is explicitly permitted for the institute's type ("End date must be after start date"); overlap with an existing non-archived session is rejected **unless** the new session is the explicitly-supported current-plus-admission-open-next pattern ("Overlapping session not allowed except admission-open next").
- **Error Codes:** `422 INVALID_DATE_RANGE`; `409 OVERLAP_NOT_PERMITTED`.
- **Business Rules Applied:** SESS-001 (date validity), SESS-002 (controlled overlap).
- **Pagination:** N/A.

### 4. Update Session
- **Method:** `PUT`
- **URL:** `/api/v1/sessions/{id}`
- **Description:** Edit a session's name/dates/admission-open flag before it is closed (SCR-SESS-10).
- **Authentication (Role-Based):** Yes — `session.update`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "startDate": {
      "type": "string"
    },
    "endDate": {
      "type": "string"
    },
    "admissionOpen": {
      "type": "boolean"
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
- **Validation Rules:** Same date/overlap rules as #3; **blocked entirely once `status = CLOSED`** — edits to a closed session's records are rejected outright, with the only path being the governed reopen (#9); `version` optimistic lock.
- **Error Codes:** `409 SESSION_CLOSED_IMMUTABLE`; `422 INVALID_DATE_RANGE`; `409 VERSION_CONFLICT`.
- **Business Rules Applied:** SESS-001, SESS-005 (post-close immutability).
- **Pagination:** N/A.

### 5. Instantiate Session Structure
- **Method:** `POST`
- **URL:** `/api/v1/sessions/{id}/structure/instantiate`
- **Description:** Generate this session's classes/sections/subject-offering instances from the institute's timeless structure **definition** — never duplicating the definition itself (SCR-SESS-04, D38).
- **Authentication (Role-Based):** Yes — `session.structure.instance`.
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
    "sessionId": {
      "type": "string"
    },
    "instancesGenerated": {
      "type": "number"
    },
    "structureInstantiated": {
      "type": "boolean",
      "const": true
    }
  },
  "required": [
    "sessionId",
    "instancesGenerated",
    "structureInstantiated"
  ]
}
```
- **Validation Rules:** The institute's structure definition must exist; instances reference the definition by identifier and are pinned to this session — re-running this endpoint is **idempotent** (does not duplicate existing instances).
- **Error Codes:** `422 STRUCTURE_DEFINITION_MISSING` (blocks session activation until structure setup is complete).
- **Business Rules Applied:** SESS-004 (structure instances are per-session, from the definition, D38). *Editing the definition later never retroactively alters this session's instances.*
- **Pagination:** N/A.

### 6. Activate / Set Current Session
- **Method:** `POST`
- **URL:** `/api/v1/sessions/{id}/activate`
- **Description:** Make a `DRAFT` session operational and, optionally, the institute's single `current` session (SCR-SESS-05).
- **Authentication (Role-Based):** Yes — `session.activate`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "setAsCurrent": {
      "type": "boolean"
    }
  },
  "required": [
    "setAsCurrent"
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
      "const": "ACTIVE"
    },
    "isCurrent": {
      "type": "boolean"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "status",
    "isCurrent",
    "version"
  ]
}
```
- **Validation Rules:** If `setAsCurrent = true`, this institute must have **no other** session simultaneously flagged `current` (SESS-003) — if one exists, the caller must explicitly go through rollover/close first; the demotion of the previous current session (if any) and promotion of this one happen atomically.
- **Error Codes:** `409 ANOTHER_CURRENT_SESSION_EXISTS` ("Close or roll over the current session first").
- **Business Rules Applied:** SESS-003 (single current session per institute, enforced atomically).
- **Pagination:** N/A.

### 7. Run Rollover & Promotion
- **Method:** `POST`
- **URL:** `/api/v1/sessions/{id}/rollover`
- **Description:** Move from the current session to the next, promoting students per their outcome — advance, retain, graduate, or exit (SCR-SESS-06, UC-SESS-02).
- **Authentication (Role-Based):** Yes — `session.rollover`; high-impact, routes to approval (#11) given its breadth.
- **Request DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "targetSessionId": {
          "type": "string"
        },
        "outcomeMap": {
          "type": "string",
          "description": "required for ADVANCE/RETAIN) }"
        },
        "carryForwardDues": {
          "type": "boolean"
        }
      },
      "required": [
        "targetSessionId",
        "outcomeMap",
        "carryForwardDues"
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
  "type": "null",
  "description": "202 Accepted { jobId, status: \"PENDING_APPROVAL\" | \"QUEUED\" } (always async per conventions \u00a79 given typical cohort size; poll via GET /api/v1/jobs/{jobId}). On completion, the job result is { promoted: number, retained: number, graduated: number, exited: number, blocked: [{ studentId, reason }] }."
}
```
- **Validation Rules:** Every `ADVANCE`/`RETAIN` row must resolve to an existing next-session class instance (#5 must have run for `targetSessionId`) — if the target level's instance doesn't exist, that student is blocked, never silently dropped; students with unresolved/unpublished results are blocked unless an explicit, audited override is supplied per row; the whole operation is atomic/recoverable — never leaves a half-promoted cohort, and is reversible within a configured rollback window.
- **Error Codes:** `422 PROMOTION_TARGET_MISSING` (per-row, surfaced in `blocked[]`, not a hard failure of the whole batch); `409 UNRESOLVED_RESULTS_BLOCK_PROMOTION` (per-row, same).
- **Business Rules Applied:** SESS-006 (promotion honors outcomes — advance/retain/graduate/exit), SESS-004 (target instances from the definition), Doc 05 §8 (rollover may require approval, reversible within a window).
- **Pagination:** N/A (job-result lists are bounded by cohort size, not paginated).

### 8. Bulk Promotion (Cohort Shortcut)
- **Method:** `POST`
- **URL:** `/api/v1/sessions/{id}/rollover/bulk-advance`
- **Description:** Apply a single outcome (typically `ADVANCE`) to an entire eligible cohort in one call, as a convenience over #7's per-student `outcomeMap` (SCR-SESS-07).
- **Authentication (Role-Based):** Yes — `session.rollover`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "targetSessionId": {
      "type": "string"
    },
    "sourceClassId": {
      "type": "string"
    },
    "targetClassId": {
      "type": "string"
    },
    "exceptions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "studentId": {
            "type": "string"
          },
          "outcome": {
            "type": "string",
            "enum": [
              "RETAIN",
              "GRADUATE",
              "EXIT"
            ]
          }
        },
        "required": [
          "studentId",
          "outcome"
        ]
      }
    }
  },
  "required": [
    "targetSessionId",
    "sourceClassId",
    "targetClassId"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "null",
  "description": "202 Accepted { jobId } (same job-result shape as #7)."
}
```
- **Validation Rules:** Same as #7, applied per student; `exceptions` override the default outcome for named students.
- **Error Codes:** Same as #7.
- **Business Rules Applied:** SESS-006, SESS-004.
- **Pagination:** N/A.

### 9. Close Session
- **Method:** `POST`
- **URL:** `/api/v1/sessions/{id}/close`
- **Description:** End a session, making its academic and financial records immutable, after close-readiness checks (SCR-SESS-08).
- **Authentication (Role-Based):** Yes — `session.close`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "overrideWarnings": {
      "type": "boolean"
    },
    "carryForwardDecision": {
      "type": "string",
      "enum": [
        "carry",
        "writeOff"
      ]
    },
    "justification": {
      "type": "string",
      "description": "required if overrideWarnings = true"
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
    "status": {
      "type": "string",
      "const": "CLOSED"
    },
    "closeReadinessCheck": {
      "type": "object",
      "properties": {
        "unpublishedResults": {
          "type": "string"
        },
        "outstandingDues": {
          "type": "string"
        },
        "openWorkflows": {
          "type": "string"
        }
      },
      "required": [
        "unpublishedResults",
        "outstandingDues",
        "openWorkflows"
      ]
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "status",
    "closeReadinessCheck",
    "version"
  ]
}
```
- **Validation Rules:** Hard-stop items (per configured policy) block close outright; soft items only warn and require `overrideWarnings: true` with a `justification` to proceed; once closed, marks/results/attendance/invoices for the session become read-only through normal operations.
- **Error Codes:** `409 CLOSE_BLOCKED` (hard-stop items present, listed); `422 JUSTIFICATION_REQUIRED` (overriding a soft warning without justification).
- **Business Rules Applied:** SESS-007 (close-readiness checks), SESS-005 (post-close immutability begins here).
- **Pagination:** N/A.

### 10. Reopen Session (Governed)
- **Method:** `POST`
- **URL:** `/api/v1/sessions/{id}/reopen`
- **Description:** Exceptionally lift immutability on a closed session for a specific, audited correction (SCR-SESS-09).
- **Authentication (Role-Based):** Yes — `session.reopen` (elevated, separately held from `session.close`); routes to approval (#11) with SoD on the approver where configured.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "justification": {
      "type": "string"
    },
    "scopeOfChange": {
      "type": "string"
    }
  },
  "required": [
    "justification",
    "scopeOfChange"
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
        "ACTIVE",
        "PENDING_APPROVAL"
      ]
    },
    "reopenedAt": {
      "type": "string"
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
- **Validation Rules:** Only from `CLOSED`; `justification` and `scopeOfChange` required; the session must be **re-closed** after the correction (tracked via `closeDeadlineAfterReopen`, surfaced as a flagged item if left open too long).
- **Error Codes:** `409 INVALID_STATE_TRANSITION` (not currently closed); `422 JUSTIFICATION_REQUIRED`.
- **Business Rules Applied:** SESS-008 (reopen is exceptional and audited; backdating constrained and logged), AUTHZ-009 (SoD on the approver).
- **Pagination:** N/A.

### 11. List Rollover / Reopen Approvals
- **Method:** `GET`
- **URL:** `/api/v1/sessions/approvals`
- **Description:** Inbox for pending rollover and reopen requests (SCR-SESS-13).
- **Authentication (Role-Based):** Yes — session change-approval authority, scoped.
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
    "typeROLLOVERREOPEN": {
      "type": "string",
      "description": "type? (ROLLOVER|REOPEN)"
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
      "sessionId": {
        "type": "string"
      },
      "sessionName": {
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
      "sessionId",
      "sessionName",
      "requestedBy",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Requester ≠ approver (AUTHZ-009).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** SESS-008, Doc 05 §8, AUTHZ-009.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 12. Decide Session Change Request
- **Method:** `POST`
- **URL:** `/api/v1/sessions/approvals/{id}/decide`
- **Description:** Approve/reject a pending rollover or reopen.
- **Authentication (Role-Based):** Yes — same authority as #11; blocked if caller raised the request.
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
- **Business Rules Applied:** AUTHZ-009, SESS-008.
- **Pagination:** N/A.

### 13. Promotion Summary Report
- **Method:** `GET`
- **URL:** `/api/v1/sessions/{id}/promotion-summary`
- **Description:** Post-rollover report of who advanced, retained, graduated, or exited (SCR-SESS-11).
- **Authentication (Role-Based):** Yes — `session.view` (read-only report over the session's own data).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "outcomeADVANCERETAIN": {
      "type": "string",
      "description": "outcome? (ADVANCE|RETAIN|GRADUATE|EXIT)"
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
      "fromClass": {
        "type": "string"
      },
      "toClass": {
        "type": "string"
      },
      "outcome": {
        "type": "string"
      },
      "basis": {
        "type": "string"
      },
      "decidedAt": {
        "type": "string"
      }
    },
    "required": [
      "studentId",
      "studentName",
      "fromClass",
      "outcome",
      "basis",
      "decidedAt"
    ]
  }
}
```
- **Validation Rules:** Only available once a rollover (#7/#8) has completed for this session.
- **Error Codes:** `404 NO_ROLLOVER_RECORDED`.
- **Business Rules Applied:** SESS-006 (per-student promotion audit trail — `STUDENT_PROMOTED/RETAINED/GRADUATED`).
- **Pagination:** Yes — default `page=1, pageSize=25`.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
