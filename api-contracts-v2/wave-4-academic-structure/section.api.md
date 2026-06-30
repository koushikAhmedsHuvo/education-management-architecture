# Section API

## 1. API Overview

**Purpose.** Govern the section — the enrollment leaf and the operative unit for capacity, roll numbers, class-teacher ownership, attendance, and timetabling.

**Module Context.** Implements Business Rules Catalog Doc 12 (Section) and UI Screen Spec `12-section-management-screens.md` (SCR-SEC-01…11).

---

## 2. Endpoints

### 1. List Sections
- **Method:** `GET`
- **URL:** `/api/v1/sections`
- **Description:** Browse sections within scope with live fill and class-teacher ownership (SCR-SEC-01).
- **Authentication (Role-Based):** Yes — `section.view`, institute/campus-scoped.
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
    "classId": {
      "type": "string"
    },
    "shift": {
      "type": "string"
    },
    "campusId": {
      "type": "string"
    },
    "statusACTIVEINACTIVE": {
      "type": "string",
      "description": "status? (ACTIVE|INACTIVE|MERGED|CLOSED)"
    },
    "hasClassTeacherboolea": {
      "type": "string",
      "description": "hasClassTeacher? (boolean)"
    },
    "sortByclassfillPerce": {
      "type": "string",
      "description": "sortBy? (class|fillPercent|name, default class)"
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
      "classId": {
        "type": "string"
      },
      "className": {
        "type": "string"
      },
      "shift": {
        "type": "string"
      },
      "campusId": {
        "type": "string"
      },
      "capacity": {
        "type": "number"
      },
      "enrolled": {
        "type": "string"
      },
      "fillPercent": {
        "type": "number"
      },
      "classTeacher": {
        "type": "object",
        "properties": {
          "id": {
            "type": "string"
          },
          "displayName": {
            "type": "string"
          }
        },
        "required": [
          "id",
          "displayName"
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
      "classId",
      "className",
      "shift",
      "campusId",
      "capacity",
      "enrolled",
      "fillPercent",
      "status",
      "updatedAt",
      "version"
    ]
  }
}
```
- **Validation Rules:** Scope-limited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** SEC-001 (enrollment leaf), SEC-003 (capacity is the operative limit), SEC-004 (class-teacher ownership shown), AUTHZ-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Section Detail
- **Method:** `GET`
- **URL:** `/api/v1/sections/{id}`
- **Description:** Roster summary, capacity, class-teacher, and history (SCR-SEC-02).
- **Authentication (Role-Based):** Yes — `section.view`.
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
    "classId": {
      "type": "string"
    },
    "campusId": {
      "type": "string"
    },
    "shift": {
      "type": "string"
    },
    "stream": {
      "type": "string"
    },
    "capacity": {
      "type": "number"
    },
    "enrolled": {
      "type": "string"
    },
    "fillPercent": {
      "type": "number"
    },
    "classTeacher": {
      "type": "object",
      "properties": {
        "id": {
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
        "id",
        "displayName",
        "assignedAt"
      ]
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
    "classId",
    "campusId",
    "shift",
    "capacity",
    "enrolled",
    "fillPercent",
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
- **Business Rules Applied:** SEC-001, SEC-003, SEC-004.
- **Pagination:** N/A.

### 3. Create Section
- **Method:** `POST`
- **URL:** `/api/v1/sections`
- **Description:** Add a section under a class instance at a campus (UC-SEC-01, SCR-SEC-03).
- **Authentication (Role-Based):** Yes — `section.create`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "classId": {
      "type": "string"
    },
    "name": {
      "type": "string"
    },
    "capacity": {
      "type": "number",
      "description": "\u22651"
    },
    "shift": {
      "type": "string"
    },
    "campusId": {
      "type": "string"
    }
  },
  "required": [
    "classId",
    "name",
    "capacity",
    "shift",
    "campusId"
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
    "classId": {
      "type": "string"
    },
    "campusId": {
      "type": "string"
    },
    "shift": {
      "type": "string"
    },
    "capacity": {
      "type": "number"
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
    "name",
    "classId",
    "campusId",
    "shift",
    "capacity",
    "status",
    "createdAt",
    "version"
  ]
}
```
- **Validation Rules:** `name` unique within the **(class, campus, shift)** composite — shift participates in uniqueness, a frequent miss ("A section with this name already exists for this class/campus/shift," SEC-002, SEC-005); `capacity` must be ≥1 ("Capacity must be at least 1," SEC-003); the section becomes the immediate enrollment hard cap.
- **Error Codes:** `409 DUPLICATE_SECTION_NAME`; `422 CAPACITY_BELOW_MINIMUM`.
- **Business Rules Applied:** SEC-001 (enrollment leaf), SEC-002 (uniqueness within class+campus+shift), SEC-003 (operative capacity), SEC-005 (shift is a first-class attribute).
- **Pagination:** N/A.

### 4. Edit Section (Capacity / Details)
- **Method:** `PUT`
- **URL:** `/api/v1/sections/{id}`
- **Description:** Edit capacity and basic details (SCR-SEC-04).
- **Authentication (Role-Based):** Yes — `section.update`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "capacity": {
      "type": "number"
    },
    "name": {
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
- **Validation Rules:** `capacity` can never be set below the section's **current active enrollment count** — hard-blocked, never auto-drops students ("not below current enrolled count," SEC-003); `name` uniqueness re-validated within (class, campus, shift) on change (SEC-002); `version` optimistic lock.
- **Error Codes:** `409 CAPACITY_BELOW_ENROLLED`; `409 DUPLICATE_SECTION_NAME`; `409 VERSION_CONFLICT`.
- **Business Rules Applied:** SEC-002, SEC-003.
- **Pagination:** N/A.

### 5. Set Shift
- **Method:** `PUT`
- **URL:** `/api/v1/sections/{id}/shift`
- **Description:** Manage the section's shift attribute — a first-class structural attribute, not a label (SCR-SEC-07).
- **Authentication (Role-Based):** Yes — `section.update`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "shift": {
      "type": "string"
    }
  },
  "required": [
    "shift"
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
    "shift": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "shift"
  ]
}
```
- **Validation Rules:** The shift must be one defined for the institute; changing it re-validates the (class, campus, shift) name-uniqueness composite, since shift participates in that uniqueness (SEC-005).
- **Error Codes:** `422 UNDEFINED_SHIFT`; `409 DUPLICATE_SECTION_NAME` (the new shift collides with an existing section).
- **Business Rules Applied:** SEC-005 (shift as a first-class section attribute).
- **Pagination:** N/A.

### 6. Assign / Change Class-Teacher
- **Method:** `POST`
- **URL:** `/api/v1/sections/{id}/class-teacher`
- **Description:** Assign exactly one class-teacher per section, conferring ownership over its daily attendance and roster (SCR-SEC-05).
- **Authentication (Role-Based):** Yes — `section.classteacher.assign`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "teacherId": {
      "type": "string"
    }
  },
  "required": [
    "teacherId"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "sectionId": {
      "type": "string"
    },
    "classTeacher": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "displayName": {
          "type": "string"
        }
      },
      "required": [
        "id",
        "displayName"
      ]
    },
    "assignedAt": {
      "type": "string"
    },
    "priorTeacher": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "displayName": {
          "type": "string"
        },
        "ownershipEndedAt": {
          "type": "string"
        }
      },
      "required": [
        "id",
        "displayName",
        "ownershipEndedAt"
      ]
    }
  },
  "required": [
    "sectionId",
    "classTeacher",
    "assignedAt"
  ]
}
```
- **Validation Rules:** `teacherId` must be active staff, qualified, and within scope; exactly one class-teacher per section is enforced — assigning a new one **immediately ends the prior teacher's live ownership** (attendance/roster access), with the change fully attributed in history (SEC-004; the prior teacher's *past* actions remain audited and intact).
- **Error Codes:** `422 TEACHER_NOT_ELIGIBLE`; `422 ASSIGNMENT_LIMIT_EXCEEDED` (configurable per-teacher cap).
- **Business Rules Applied:** SEC-004 (class-teacher assignment confers ownership; reassignment ends prior ownership immediately with history attribution).
- **Pagination:** N/A.

### 7. Submit Section Restructure (Merge / Split / Close)
- **Method:** `POST`
- **URL:** `/api/v1/sections/restructure`
- **Description:** Merge, split, or close sections with a per-student transfer map, preserving period-accurate history; routes to approval given student impact (SCR-SEC-06).
- **Authentication (Role-Based):** Yes — `section.restructure` (request).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "operation": {
      "type": "string",
      "enum": [
        "MERGE",
        "SPLIT",
        "CLOSE"
      ]
    },
    "sourceSectionIds": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "targetSectionIds": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "transferMap": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "studentId": {
            "type": "string"
          },
          "targetSectionId": {
            "type": "string"
          }
        },
        "required": [
          "studentId",
          "targetSectionId"
        ]
      }
    },
    "effectiveDate": {
      "type": "string"
    }
  },
  "required": [
    "operation",
    "sourceSectionIds",
    "targetSectionIds",
    "transferMap",
    "effectiveDate"
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
    "operation": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "PENDING_APPROVAL"
    },
    "transferMapValidation": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "studentId": {
            "type": "string"
          },
          "targetSectionId": {
            "type": "string"
          },
          "validation": {
            "type": "string",
            "enum": [
              "OK",
              "TARGET_FULL",
              "UNMAPPED"
            ]
          }
        },
        "required": [
          "studentId",
          "targetSectionId",
          "validation"
        ]
      }
    }
  },
  "required": [
    "id",
    "operation",
    "status",
    "transferMapValidation"
  ]
}
```
- **Validation Rules:** Every affected student must appear in `transferMap` with a valid target — unmapped students block submission ("Unmapped students remain"); each target section must have available capacity for its incoming students ("Target section is full," SEC-003); closing a section with active students is blocked unless every one of them is included in the transfer map (SEC-006).
- **Error Codes:** `422 TARGET_SECTION_FULL`; `422 UNMAPPED_STUDENTS`.
- **Business Rules Applied:** SEC-006 (merge/split/close preserves history via transfers), SEC-003 (target capacity respected), ENR-005 (period-accurate transfer history).
- **Pagination:** N/A.

### 8. List Restructure Approvals
- **Method:** `GET`
- **URL:** `/api/v1/sections/restructure-approvals`
- **Description:** Inbox for pending section restructures (SCR-SEC-11).
- **Authentication (Role-Based):** Yes — `section.restructure.approve`, scoped.
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
      "operation": {
        "type": "string"
      },
      "sectionsInvolved": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "studentCount": {
        "type": "integer"
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
      "operation",
      "sectionsInvolved",
      "studentCount",
      "requestedBy",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Requester ≠ approver (AUTHZ-009).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** SEC-006, AUTHZ-009, WFL-004 (requester ≠ approver).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 9. Decide Restructure Request
- **Method:** `POST`
- **URL:** `/api/v1/sections/restructure-approvals/{id}/decide`
- **Description:** Approve or reject a pending restructure; on approval, the transfers in #7 are applied with preserved history.
- **Authentication (Role-Based):** Yes — `section.restructure.approve`; **blocked if the caller raised the request**.
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
- **Validation Rules:** Caller ≠ requester; `reason` required on `REJECT`; target capacity is **re-checked** at approval time (it may have changed since submission) before committing the transfers.
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED`; `422 REASON_REQUIRED`; `409 TARGET_CAPACITY_CHANGED` (re-validation failed at approval time).
- **Business Rules Applied:** AUTHZ-009, SEC-006, SEC-003.
- **Pagination:** N/A.

### 10. Section Fill & Capacity Report
- **Method:** `GET`
- **URL:** `/api/v1/sections/report/fill-capacity`
- **Description:** Fill rate and capacity utilization across sections (SCR-SEC-08).
- **Authentication (Role-Based):** Yes — `section.view` (the approved screen names a generic "report view"; this contract reuses the resource's own view permission, consistent with the convention applied across these report screens).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "classId": {
      "type": "string"
    },
    "campusId": {
      "type": "string"
    },
    "shift": {
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
      "sectionId": {
        "type": "string"
      },
      "name": {
        "type": "string"
      },
      "className": {
        "type": "string"
      },
      "capacity": {
        "type": "number"
      },
      "enrolled": {
        "type": "string"
      },
      "fillPercent": {
        "type": "number"
      },
      "hasClassTeacher": {
        "type": "boolean"
      }
    },
    "required": [
      "sectionId",
      "name",
      "className",
      "capacity",
      "enrolled",
      "fillPercent",
      "hasClassTeacher"
    ]
  }
}
```
- **Validation Rules:** Scope-limited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** SEC-003.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 11. Bulk-Create / Import Sections
- **Method:** `POST`
- **URL:** `/api/v1/sections/bulk-create`
- **Description:** Bulk-create sections from a file (SCR-SEC-09).
- **Authentication (Role-Based):** Yes — `section.import`.
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
              "classId": {
                "type": "string"
              },
              "name": {
                "type": "string"
              },
              "capacity": {
                "type": "number"
              },
              "shift": {
                "type": "string"
              },
              "campusId": {
                "type": "string"
              }
            },
            "required": [
              "classId",
              "name",
              "capacity",
              "shift",
              "campusId"
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
              "sectionId": {
                "type": "string"
              }
            },
            "required": [
              "row",
              "sectionId"
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
- **Validation Rules:** Each row validated against #3's rules (uniqueness within class+campus+shift, capacity ≥1); duplicate/invalid rows are excluded and always reported — never silently imported.
- **Error Codes:** `400 MALFORMED_BATCH`; per-row failures in `rejected[]`.
- **Business Rules Applied:** SEC-002, SEC-003; conventions §9.
- **Pagination:** N/A.

### 12. Export Section List
- **Method:** `GET`
- **URL:** `/api/v1/sections/export`
- **Description:** Governed export of sections (SCR-SEC-10).
- **Authentication (Role-Based):** Yes — `section.export`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "classId": {
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

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
