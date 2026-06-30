# Class API

## 1. API Overview

**Purpose.** Govern the class (grade/standard/level) — a level in the timeless structure definition, instantiated per session, with ordering, campus applicability, streams, advisory aggregate capacity, and an acyclic promotion path.

**Module Context.** Implements Business Rules Catalog Doc 11 (Class) and UI Screen Spec `11-class-management-screens.md` (SCR-CLS-01…13).

---

## 2. Endpoints

### 1. List Classes
- **Method:** `GET`
- **URL:** `/api/v1/classes`
- **Description:** Browse the institute's class structure definition (SCR-CLS-01).
- **Authentication (Role-Based):** Yes — `class.view`, institute-scoped.
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
    "campusId": {
      "type": "string"
    },
    "stream": {
      "type": "string"
    },
    "statusDRAFTACTIVE_DE": {
      "type": "string",
      "description": "status? (DRAFT|ACTIVE_DEFINITION|DEPRECATED_DEFINITION)"
    },
    "sortByorderinglabel": {
      "type": "string",
      "description": "sortBy? (ordering|label, default ordering)"
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
      "label": {
        "type": "string"
      },
      "code": {
        "type": "string"
      },
      "ordering": {
        "type": "number"
      },
      "campusApplicability": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "streams": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "aggregateCapacity": {
        "type": "number"
      },
      "sectionCount": {
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
      "label",
      "ordering",
      "campusApplicability",
      "streams",
      "sectionCount",
      "status",
      "updatedAt",
      "version"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CLS-001 (definition vs instance), CLS-002 (configurable label & ordering).
- **Pagination:** Yes — default `page=1, pageSize=25`, sorted by `ordering asc`.

### 2. Get Class Detail
- **Method:** `GET`
- **URL:** `/api/v1/classes/{id}`
- **Description:** Full class definition — ordering, campuses, streams, capacity, promotion target (SCR-CLS-02).
- **Authentication (Role-Based):** Yes — `class.view`.
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
    "label": {
      "type": "string"
    },
    "code": {
      "type": "string"
    },
    "ordering": {
      "type": "string"
    },
    "campusApplicability": {
      "type": "string",
      "description": "campusApplicability[]"
    },
    "streams": {
      "type": "string",
      "description": "streams[]"
    },
    "aggregateCapacity": {
      "type": "number"
    },
    "promotionPath": {
      "type": "object",
      "properties": {
        "nextClassId": {
          "type": "string"
        },
        "isTerminal": {
          "type": "boolean"
        }
      },
      "required": [
        "isTerminal"
      ]
    },
    "sectionCount": {
      "type": "integer"
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
    "label",
    "ordering",
    "promotionPath",
    "sectionCount",
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
- **Business Rules Applied:** CLS-001, CLS-005 (promotion target shown), CLS-007 (capacity shown).
- **Pagination:** N/A.

### 3. Define Class
- **Method:** `POST`
- **URL:** `/api/v1/classes`
- **Description:** Add a class (level) to the institute's structure definition; defining a class auto-provisions a default section (UC-CLS-01, SCR-CLS-03).
- **Authentication (Role-Based):** Yes — `class.create`, institute-scoped.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "label": {
      "type": "string"
    },
    "code": {
      "type": "string"
    },
    "ordering": {
      "type": "number"
    },
    "campusApplicability": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "\u22651"
    },
    "streams": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "aggregateCapacity": {
      "type": "number"
    }
  },
  "required": [
    "label",
    "ordering",
    "campusApplicability"
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
    "label": {
      "type": "string"
    },
    "ordering": {
      "type": "string"
    },
    "campusApplicability": {
      "type": "string",
      "description": "campusApplicability[]"
    },
    "status": {
      "type": "string",
      "const": "DRAFT"
    },
    "defaultSectionProvisioned": {
      "type": "boolean",
      "const": true
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
    "label",
    "ordering",
    "status",
    "defaultSectionProvisioned",
    "createdAt",
    "version"
  ]
}
```
- **Validation Rules:** `ordering` unique within the institute ("Ordering already used," CLS-002); `campusApplicability` requires at least one active campus ("Select at least one campus," CLS-004); on creation, a default section is auto-provisioned so the class always has ≥1 section (CLS-003).
- **Error Codes:** `409 ORDERING_ALREADY_USED`; `422 NO_CAMPUS_SELECTED`.
- **Business Rules Applied:** CLS-002 (unique ordering), CLS-003 (default-section guarantee), CLS-004 (campus applicability).
- **Pagination:** N/A.

### 4. Edit Class
- **Method:** `PUT`
- **URL:** `/api/v1/classes/{id}`
- **Description:** Edit label, code, ordering, and campus applicability (SCR-CLS-04). Edits apply to **future sessions only** — already-instantiated session records are never rewritten.
- **Authentication (Role-Based):** Yes — `class.update`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "label": {
      "type": "string"
    },
    "code": {
      "type": "string"
    },
    "ordering": {
      "type": "string"
    },
    "campusApplicability": {
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
- **Validation Rules:** Same ordering-uniqueness and campus-applicability rules as #3, re-validated on edit; `version` optimistic lock; this endpoint never touches the promotion path (#9) or streams (#5) — those have their own governance.
- **Error Codes:** `409 ORDERING_ALREADY_USED`; `422 NO_CAMPUS_SELECTED`; `409 VERSION_CONFLICT`.
- **Business Rules Applied:** CLS-001 (future-only effect), CLS-002, CLS-004.
- **Pagination:** N/A.

### 5. Bulk Reorder Classes
- **Method:** `POST`
- **URL:** `/api/v1/classes/reorder`
- **Description:** Commit a drag-reordered sequence across many classes at once (SCR-CLS-01 inline reorder, SCR-CLS-08 Label & Ordering).
- **Authentication (Role-Based):** Yes — `class.update`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "ordering": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "classId": {
            "type": "string"
          },
          "ordering": {
            "type": "number"
          }
        },
        "required": [
          "classId",
          "ordering"
        ]
      }
    }
  },
  "required": [
    "ordering"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "updated": {
      "type": "number"
    }
  },
  "required": [
    "updated"
  ]
}
```
- **Validation Rules:** The submitted `ordering` values must be unique as a set (CLS-002) — validated across the whole batch, not just pairwise against existing rows.
- **Error Codes:** `422 DUPLICATE_ORDERING_IN_BATCH`.
- **Business Rules Applied:** CLS-002.
- **Pagination:** N/A.

### 6. Manage Streams
- **Method:** `PUT`
- **URL:** `/api/v1/classes/{id}/streams`
- **Description:** Configure a class's stream sub-groupings — e.g., Science/Commerce/Arts (SCR-CLS-06).
- **Authentication (Role-Based):** Yes — `class.stream.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "streams": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": {
            "type": "string"
          },
          "version": {
            "type": "number"
          }
        },
        "required": [
          "name",
          "version"
        ]
      }
    }
  },
  "required": [
    "streams"
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
    "streams": {
      "type": "array",
      "items": {
        "type": "string"
      }
    }
  },
  "required": [
    "id",
    "streams"
  ]
}
```
- **Validation Rules:** Stream sub-grouping must stay consistent with subject curriculum mapping (CLS-006); removing a stream still referenced by an active curriculum mapping (Doc 13, SUB-002) or an active section is blocked.
- **Error Codes:** `409 STREAM_IN_USE`.
- **Business Rules Applied:** CLS-006 (streams as configurable sub-grouping), SUB-002 (curriculum-mapping consistency).
- **Pagination:** N/A.

### 7. Set Aggregate Capacity
- **Method:** `PUT`
- **URL:** `/api/v1/classes/{id}/capacity`
- **Description:** Set or clear the optional, **advisory** aggregate class capacity (SCR-CLS-07).
- **Authentication (Role-Based):** Yes — `class.capacity.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "aggregateCapacity": {
      "type": "string",
      "description": "number | null"
    }
  },
  "required": [
    "aggregateCapacity"
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
    "aggregateCapacity": {
      "type": "number"
    }
  },
  "required": [
    "id",
    "aggregateCapacity"
  ]
}
```
- **Validation Rules:** Cannot be set below the class's current total active enrollment; the UI and this endpoint both make clear that **section capacity is the authoritative, hard enrollment cap** (`05-campus-api.md`/Section module) — this aggregate figure is advisory only and never overrides it (CLS-007, C-01).
- **Error Codes:** `409 BELOW_CURRENT_ENROLLMENT`.
- **Business Rules Applied:** CLS-007 (aggregate class capacity, optional/advisory).
- **Pagination:** N/A.

### 8. Get Promotion Path
- **Method:** `GET`
- **URL:** `/api/v1/classes/{id}/promotion-path`
- **Description:** View the class's current promotion target or terminal (graduating) flag.
- **Authentication (Role-Based):** Yes — `class.view`.
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
    "classId": {
      "type": "string"
    },
    "nextClassId": {
      "type": "string"
    },
    "isTerminal": {
      "type": "boolean"
    }
  },
  "required": [
    "classId",
    "isTerminal"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CLS-005.
- **Pagination:** N/A.

### 9. Set Promotion Path
- **Method:** `PUT`
- **URL:** `/api/v1/classes/{id}/promotion-path`
- **Description:** Set or change the class's promotion target — a structural change, distinct from a basic edit (#4), governed separately (SCR-CLS-05).
- **Authentication (Role-Based):** Yes — `class.promotion.manage` to request; routes to approval (#14) rather than applying immediately, given its breadth across the institute's progression model.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "nextClassId": {
      "type": "string",
      "description": "omit/null if terminal"
    },
    "isTerminal": {
      "type": "boolean"
    }
  },
  "required": [
    "isTerminal"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "classId": {
      "type": "string"
    },
    "nextClassId": {
      "type": "string"
    },
    "isTerminal": {
      "type": "boolean"
    },
    "status": {
      "type": "string",
      "const": "PENDING_APPROVAL"
    }
  },
  "required": [
    "classId",
    "isTerminal",
    "status"
  ]
}
```
- **Validation Rules:** The resulting promotion graph across the institute's classes must remain **acyclic** — every non-terminal class must have a valid next class, and no chain of promotions may loop back on itself ("This would create a promotion cycle," CLS-005).
- **Error Codes:** `422 PROMOTION_CYCLE_DETECTED`.
- **Business Rules Applied:** CLS-005 (acyclic promotion path), Doc 11 §8 (structural changes routed to approval).
- **Pagination:** N/A.

### 10. Retire / Archive Class
- **Method:** `POST`
- **URL:** `/api/v1/classes/{id}/retire`
- **Description:** Retire a class from future sessions while preserving all historical instances and records intact (SCR-CLS-09).
- **Authentication (Role-Based):** Yes — `class.retire`.
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
      "const": "DEPRECATED_DEFINITION"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** `reason` required; this is **never** a hard delete — historical session instances and their records remain fully intact and queryable (CLS-001); a class that other classes promote to cannot be retired until their promotion targets are repointed (dependent-class check, mirroring CLS-005's acyclic-graph integrity).
- **Error Codes:** `422 REASON_REQUIRED`; `409 PROMOTION_TARGET_IN_USE`.
- **Business Rules Applied:** CLS-001 (no hard delete, history preserved).
- **Pagination:** N/A.

### 11. Class Structure & Capacity Report
- **Method:** `GET`
- **URL:** `/api/v1/classes/report/structure-capacity`
- **Description:** Institute-wide structure and capacity utilization across classes (SCR-CLS-10).
- **Authentication (Role-Based):** Yes — `class.view` (the approved screen names a generic "report view"; this contract reuses the resource's own view permission, consistent with how report screens are gated elsewhere in the approved specs).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "campusId": {
      "type": "string"
    },
    "stream": {
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
      "label": {
        "type": "string"
      },
      "sectionCount": {
        "type": "integer"
      },
      "aggregateCapacity": {
        "type": "number"
      },
      "totalEnrolled": {
        "type": "string"
      },
      "utilizationPercent": {
        "type": "number"
      }
    },
    "required": [
      "classId",
      "label",
      "sectionCount",
      "totalEnrolled",
      "utilizationPercent"
    ]
  }
}
```
- **Validation Rules:** Scope-limited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CLS-007.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 12. Bulk-Define Class Structure
- **Method:** `POST`
- **URL:** `/api/v1/classes/bulk-define`
- **Description:** Define many classes at once from an import batch (SCR-CLS-11).
- **Authentication (Role-Based):** Yes — `class.import`.
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
              "label": {
                "type": "string"
              },
              "code": {
                "type": "string"
              },
              "ordering": {
                "type": "string"
              },
              "campusApplicability": {
                "type": "string"
              },
              "streams": {
                "type": "string"
              },
              "aggregateCapacity": {
                "type": "number"
              }
            },
            "required": [
              "label",
              "ordering",
              "campusApplicability"
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
              "classId": {
                "type": "string"
              }
            },
            "required": [
              "row",
              "classId"
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
- **Validation Rules:** Each row validated against #3's rules (ordering uniqueness across the whole batch and against existing classes, ≥1 campus, default-section provisioning); invalid rows excluded and always reported — never silently imported.
- **Error Codes:** `400 MALFORMED_BATCH`; per-row failures in `rejected[]`.
- **Business Rules Applied:** CLS-002, CLS-003, CLS-004; conventions §9.
- **Pagination:** N/A.

### 13. Export Class Structure
- **Method:** `GET`
- **URL:** `/api/v1/classes/export`
- **Description:** Governed export of the class structure (SCR-CLS-12).
- **Authentication (Role-Based):** Yes — `class.export`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "campusId": {
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
- **Business Rules Applied:** conventions §10.
- **Pagination:** N/A.

### 14. List Structure-Change Approvals
- **Method:** `GET`
- **URL:** `/api/v1/classes/approvals`
- **Description:** Inbox for pending promotion-path (and other structural) changes (SCR-CLS-13).
- **Authentication (Role-Based):** Yes — `class.change.approve`, scoped.
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
      "classId": {
        "type": "string"
      },
      "className": {
        "type": "string"
      },
      "change": {
        "type": "object",
        "properties": {
          "description": {
            "type": "string"
          }
        },
        "required": [
          "description"
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
      "classId",
      "className",
      "change",
      "requestedBy",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Requester ≠ approver (AUTHZ-009).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CLS-005, AUTHZ-009.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 15. Decide Structure-Change Request
- **Method:** `POST`
- **URL:** `/api/v1/classes/approvals/{id}/decide`
- **Description:** Approve or reject a pending structural change (e.g., a promotion-path edit from #9).
- **Authentication (Role-Based):** Yes — `class.change.approve`; **blocked if the caller raised the request** — self-approval is disabled.
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
- **Business Rules Applied:** AUTHZ-009, CLS-005.
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
