# Institute API

## 1. API Overview

**Purpose.** Govern the institute — the primary multi-tenant scope boundary that nearly every business object in the system belongs to.

**Module Context.** Implements Business Rules Catalog Doc 03 (Institute) and UI Screen Spec `04-institute-management-screens.md` (SCR-INST-01…13).

---

## 2. Endpoints

### 1. List Institutes
- **Method:** `GET`
- **URL:** `/api/v1/institutes`
- **Description:** Browse institutes; org admins see all, institute admins see only their own (SCR-INST-01).
- **Authentication (Role-Based):** Yes — `institute.view` (org-wide for org admins; ownership-filtered otherwise, AUTHZ-003).
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
    "typecommaseparated": {
      "type": "string",
      "description": "type? (comma-separated)"
    },
    "statusDRAFTACTIVESU": {
      "type": "string",
      "description": "status? (DRAFT|ACTIVE|SUSPENDED|ARCHIVED)"
    },
    "sortBynamestatusupd": {
      "type": "string",
      "description": "sortBy? (name|status|updatedAt, default name)"
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
      "campusCount": {
        "type": "integer"
      },
      "status": {
        "type": "string"
      },
      "setupCompletenessPercent": {
        "type": "string",
        "description": "setupCompletenessPercent? (DRAFT only)"
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
      "campusCount",
      "status",
      "updatedAt",
      "version"
    ]
  }
}
```
- **Validation Rules:** Non-org admins receive only their own institute(s) regardless of filters supplied.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-003 (ownership predicate for non-org admins), INST-006 (institute is a mandatory scope).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Institute Detail
- **Method:** `GET`
- **URL:** `/api/v1/institutes/{id}`
- **Description:** Overview, status, and setup checklist (SCR-INST-02).
- **Authentication (Role-Based):** Yes — `institute.view`.
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
    "address": {
      "type": "string"
    },
    "contact": {
      "type": "string"
    },
    "status": {
      "type": "string"
    },
    "setupChecklist": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "item": {
            "type": "string"
          },
          "complete": {
            "type": "string"
          }
        },
        "required": [
          "item",
          "complete"
        ]
      }
    },
    "campusCount": {
      "type": "integer"
    },
    "sessionCount": {
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
    "name",
    "code",
    "type",
    "status",
    "setupChecklist",
    "campusCount",
    "sessionCount",
    "createdAt",
    "updatedAt",
    "createdBy",
    "updatedBy",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND` (out-of-scope, AUTHZ-002).
- **Business Rules Applied:** INST-003 (setup checklist), AUTHZ-002.
- **Pagination:** N/A.

### 3. Create Institute (Draft)
- **Method:** `POST`
- **URL:** `/api/v1/institutes`
- **Description:** Stand up a new institute, seeding type-based templates (SCR-INST-03, UC-INST-01).
- **Authentication (Role-Based):** Yes — `institute.create` (org).
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
    "institutionType": {
      "type": "string",
      "enum": [
        "school",
        "college",
        "university",
        "madrasa",
        "coaching",
        "training"
      ]
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
    "code",
    "institutionType"
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
    "institutionType": {
      "type": "string"
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
    "institutionType",
    "status",
    "createdAt",
    "version"
  ]
}
```
- **Validation Rules:** `code` unique across the deployment, case-insensitive (INST-001), and **immutable after creation**; `institutionType` ∈ supported catalog with an existing seed template (INST-002).
- **Error Codes:** `409 INSTITUTE_CODE_EXISTS`; `422 UNSUPPORTED_INSTITUTION_TYPE`.
- **Business Rules Applied:** INST-001 (code uniqueness), INST-002 (type seeds templates, never hard-coded behavior).
- **Pagination:** N/A.

### 4. Get Setup Checklist / Progress
- **Method:** `GET`
- **URL:** `/api/v1/institutes/{id}/setup`
- **Description:** Drive the Setup Wizard's stepper (SCR-INST-04).
- **Authentication (Role-Based):** Yes — `institute.update` + the relevant child permissions for items the caller can complete (`campus.create`, `session.create`, `membership.grant`).
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
    "items": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "key": {
            "type": "string",
            "enum": [
              "campus",
              "session",
              "locale",
              "adminAssigned",
              "coreConfig"
            ]
          },
          "complete": {
            "type": "boolean"
          }
        },
        "required": [
          "key",
          "complete"
        ]
      }
    },
    "readyToActivate": {
      "type": "boolean"
    }
  },
  "required": [
    "items",
    "readyToActivate"
  ]
}
```
- **Validation Rules:** None (read).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** INST-003 (mandatory setup before activation).
- **Pagination:** N/A.

### 5. Update Institute
- **Method:** `PUT`
- **URL:** `/api/v1/institutes/{id}`
- **Description:** Edit name, address, contact (SCR-INST-05).
- **Authentication (Role-Based):** Yes — `institute.update`.
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
- **Validation Rules:** `code` is never editable via this endpoint (INST-001: immutable post-creation — no field for it is accepted); `institutionType` is never editable here either (see #10 for the sole, draft-only, no-dependent-data exception); `version` optimistic lock.
- **Error Codes:** `409 VERSION_CONFLICT`; `422 IMMUTABLE_FIELD` if `code` or `institutionType` is supplied.
- **Business Rules Applied:** INST-001, INST-004 (type change is restricted — not via this endpoint).
- **Pagination:** N/A.

### 6. Get Institute Configuration Overrides
- **Method:** `GET`
- **URL:** `/api/v1/institutes/{id}/config-overrides`
- **Description:** View institute-level configuration overrides (SCR-INST-06).
- **Authentication (Role-Based):** Yes — `institute.config.manage`.
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
          "institute"
        ]
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
}
```
- **Validation Rules:** None (read).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** INST-005 (configuration inheritance & override, most-specific-wins).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 7. Set Institute Configuration Override
- **Method:** `PUT`
- **URL:** `/api/v1/institutes/{id}/config-overrides/{definitionKey}`
- **Description:** Set or reset an institute-level override.
- **Authentication (Role-Based):** Yes — `institute.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "value": {
          "type": "object",
          "description": "typed per the definition"
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
- **Validation Rules:** `value` validated against the configuration definition's type/range; rejected overrides fall back to the inherited value, never to an undefined state.
- **Error Codes:** `422 INVALID_OVERRIDE_VALUE`.
- **Business Rules Applied:** INST-005.
- **Pagination:** N/A.

### 8. Activate Institute
- **Method:** `POST`
- **URL:** `/api/v1/institutes/{id}/activate`
- **Description:** Transition `DRAFT → ACTIVE`, unlocking operational modules.
- **Authentication (Role-Based):** Yes — `institute.activate`.
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
    "id": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "ACTIVE"
    },
    "checklistSnapshot": {
      "type": "object"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "status",
    "checklistSnapshot",
    "version"
  ]
}
```
- **Validation Rules:** Blocked unless every required setup-checklist item is complete (INST-003); only from `DRAFT`.
- **Error Codes:** `422 SETUP_INCOMPLETE` (with the precise list of missing items); `409 INVALID_STATE_TRANSITION`.
- **Business Rules Applied:** INST-003 (mandatory setup before activation).
- **Pagination:** N/A.

### 9. Suspend Institute
- **Method:** `POST`
- **URL:** `/api/v1/institutes/{id}/suspend`
- **Description:** Temporarily block operational access for institute-scoped users while preserving data; org admins retain management access (SCR-INST-07).
- **Authentication (Role-Based):** Yes — `institute.suspend`; high-impact, may route to approval via `institute.change.approve`-class authority (Doc 03 §8).
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
        "SUSPENDED",
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
- **Validation Rules:** Only from `ACTIVE`; `DRAFT` institutes cannot be suspended (Doc 03 §6: "cannot suspend a never-active institute").
- **Error Codes:** `409 INVALID_STATE_TRANSITION`.
- **Business Rules Applied:** INST-007 (suspension cascade — no data altered/deleted), Doc 03 §8 (approval for high-impact lifecycle changes).
- **Pagination:** N/A.

### 10. Reinstate Institute
- **Method:** `POST`
- **URL:** `/api/v1/institutes/{id}/reinstate`
- **Description:** Lift a suspension, restoring operational access.
- **Authentication (Role-Based):** Yes — `institute.suspend`-equivalent (the inverse of #9).
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
    "id": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "ACTIVE"
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
- **Validation Rules:** Only from `SUSPENDED`.
- **Error Codes:** `409 INVALID_STATE_TRANSITION`.
- **Business Rules Applied:** INST-007.
- **Pagination:** N/A.

### 11. Archive Institute
- **Method:** `POST`
- **URL:** `/api/v1/institutes/{id}/archive`
- **Description:** Permanently close an institute, preserving all data immutably (SCR-INST-08).
- **Authentication (Role-Based):** Yes — `institute.archive`; high-impact, may route to approval.
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
- **Validation Rules:** Blocked if any active session/enrollment dependency is unresolved (INST-008); resolve or explicitly handle first (see `06-academic-session-api.md` close-readiness).
- **Error Codes:** `409 BLOCKING_DEPENDENCIES` (with the list of blockers).
- **Business Rules Applied:** INST-008 (archive, never hard-delete, once any data exists).
- **Pagination:** N/A.

### 12. Reactivate Archived Institute
- **Method:** `POST`
- **URL:** `/api/v1/institutes/{id}/reactivate`
- **Description:** Exceptional, policy-gated reactivation of an `ARCHIVED` institute (SCR-INST-09).
- **Authentication (Role-Based):** Yes — `institute.reactivate` (explicitly elevated, distinct from `institute.suspend`'s reinstate path).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "justification": {
      "type": "string"
    }
  },
  "required": [
    "justification"
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
- **Validation Rules:** Only from `ARCHIVED`; `justification` required.
- **Error Codes:** `409 INVALID_STATE_TRANSITION`.
- **Business Rules Applied:** INST-008 (Doc 03 §6: "→ ACTIVE only via explicit, audited reactivation policy (rare)").
- **Pagination:** N/A.

### 13. Hard-Delete Empty Draft Institute
- **Method:** `DELETE`
- **URL:** `/api/v1/institutes/{id}`
- **Description:** The sole hard-delete path — an empty `DRAFT` institute with no dependent data (Doc 03 §8 edge case).
- **Authentication (Role-Based):** Yes — `institute.create` (the same org-admin authority that creates institutes governs removing an unused draft).
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
  "type": "null",
  "description": "204 No Content."
}
```
- **Validation Rules:** Only permitted when `status = DRAFT` **and** zero dependent records exist (no campuses beyond the unsaved default, no sessions, no memberships beyond the creator).
- **Error Codes:** `409 CANNOT_HARD_DELETE` ("This institute has data and must be archived, not deleted") if any dependency is found, or if `status ≠ DRAFT`.
- **Business Rules Applied:** INST-008 ("Hard-delete is reserved only for an empty `DRAFT` institute with no data").
- **Pagination:** N/A.

### 14. Apply / Re-seed Type Template
- **Method:** `POST`
- **URL:** `/api/v1/institutes/{id}/apply-type-template`
- **Description:** Re-apply the institution-type's seed template — draft-only (SCR-INST-10).
- **Authentication (Role-Based):** Yes — `institute.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "institutionType": {
      "type": "string",
      "description": "to also change type, draft-only"
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
    "institutionType": {
      "type": "string"
    },
    "templateApplied": {
      "type": "boolean",
      "const": true
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "institutionType",
    "templateApplied",
    "version"
  ]
}
```
- **Validation Rules:** Permitted only while `status = DRAFT` **and** no dependent data exists; otherwise the type is locked (INST-004).
- **Error Codes:** `409 TYPE_LOCKED` ("A live institute cannot change type mid-year; create a new institute instead").
- **Business Rules Applied:** INST-002 (type seeds templates), INST-004 (type change restricted to draft, no-data state).
- **Pagination:** N/A.

### 15. Cross-Institute Summary
- **Method:** `GET`
- **URL:** `/api/v1/institutes/summary`
- **Description:** Org-level aggregate view across all institutes in the deployment (SCR-INST-11).
- **Authentication (Role-Based):** Yes — `institute.summary.view` (org only).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "status": {
      "type": "string"
    },
    "type": {
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
    "totalInstitutes": {
      "type": "string"
    },
    "byStatus": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "status": {
            "type": "string"
          },
          "count": {
            "type": "integer"
          }
        },
        "required": [
          "status",
          "count"
        ]
      }
    },
    "byType": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "type": {
            "type": "string"
          },
          "count": {
            "type": "integer"
          }
        },
        "required": [
          "type",
          "count"
        ]
      }
    }
  },
  "required": [
    "totalInstitutes",
    "byStatus",
    "byType"
  ]
}
```
- **Validation Rules:** Always scoped to the **current deployment only** — never crosses deployments, per the approved spec's explicit cross-reference (REP-006 in the Reporting module, applied here as a hard boundary).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** INST-006, deployment isolation (per architecture: one database per deployment).
- **Pagination:** N/A (aggregate, not a list).

### 16. Get Institute Change Approvals
- **Method:** `GET`
- **URL:** `/api/v1/institutes/approvals`
- **Description:** Inbox for pending suspend/archive/reactivate requests (SCR-INST-13).
- **Authentication (Role-Based):** Yes — `institute.change.approve`-class authority.
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
      "instituteId": {
        "type": "string"
      },
      "instituteName": {
        "type": "string"
      },
      "requestType": {
        "type": "string",
        "enum": [
          "SUSPEND",
          "ARCHIVE",
          "REACTIVATE"
        ]
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
      "instituteId",
      "instituteName",
      "requestType",
      "requestedBy",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Requester ≠ approver enforced server-side (AUTHZ-009).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** Doc 03 §8 (workflow for high-impact lifecycle changes), AUTHZ-009.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 17. Decide Institute Change Request
- **Method:** `POST`
- **URL:** `/api/v1/institutes/approvals/{id}/decide`
- **Description:** Approve/reject a pending suspend/archive/reactivate request.
- **Authentication (Role-Based):** Yes — `institute.change.approve`-class authority; blocked if caller raised the request (SoD).
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
- **Business Rules Applied:** AUTHZ-009, Doc 03 §8.
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
