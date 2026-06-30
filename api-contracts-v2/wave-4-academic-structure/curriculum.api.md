# Curriculum API

## 1. API Overview

**Purpose.** Map subjects to classes/streams as core or elective offerings, and report on curriculum coverage — the contract that keeps exams and results consistent with what is actually taught.

**Module Context.** Implements Business Rules Catalog Doc 13 (Subject) Rule SUB-002 and UI Screen Spec `13-subject-management-screens.md` SCR-SUB-05/11/13.

---

## 2. Endpoints

### 1. Get Curriculum Mapping
- **Method:** `GET`
- **URL:** `/api/v1/subjects/{id}/curriculum-mapping`
- **Description:** View which classes/streams a subject is mapped to, and whether mandatory or optional.
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
    "mappings": {
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
    }
  },
  "required": [
    "subjectId",
    "mappings"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** SUB-002.
- **Pagination:** N/A.

### 2. Set Curriculum Mapping
- **Method:** `PUT`
- **URL:** `/api/v1/subjects/{id}/curriculum-mapping`
- **Description:** Map a subject to classes/streams as core or elective — required before the subject can be assessed or assigned for those classes (UC-SUB-01, SCR-SUB-05).
- **Authentication (Role-Based):** Yes — `subject.curriculum.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "mappings": {
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
    }
  },
  "required": [
    "mappings"
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
    "mappings": {
      "type": "string",
      "description": "mappings[]"
    }
  },
  "required": [
    "subjectId"
  ]
}
```
- **Validation Rules:** Each referenced `classId`/`stream` must exist and be consistent with that class's defined streams (CLS-006); a subject may be `mandatory: true` (core) in one class and `false` (elective) in another, and stream-specific where streams apply; unmapped subject-class pairs **block** assessment and teacher assignment for that pairing downstream (enforced at those write points, not here).
- **Error Codes:** `422 INVALID_CLASS_OR_STREAM`.
- **Business Rules Applied:** SUB-002 (curriculum mapping is required before assessment/assignment), CLS-006 (stream consistency).
- **Pagination:** N/A.

### 3. Curriculum Map & Subject Report
- **Method:** `GET`
- **URL:** `/api/v1/subjects/report/curriculum-map`
- **Description:** Institute-wide curriculum mapping and subject coverage report (SCR-SUB-11).
- **Authentication (Role-Based):** Yes — `subject.view` (the approved screen names a generic "report view"; this contract reuses the resource's own view permission, consistent with the convention applied across these report screens).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "classId": {
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
      "className": {
        "type": "string"
      },
      "mappedSubjects": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "subjectId": {
              "type": "string"
            },
            "name": {
              "type": "string"
            },
            "mandatory": {
              "type": "boolean"
            }
          },
          "required": [
            "subjectId",
            "name",
            "mandatory"
          ]
        }
      }
    },
    "required": [
      "classId",
      "className",
      "mappedSubjects"
    ]
  }
}
```
- **Validation Rules:** Scope-limited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** SUB-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 4. Export Curriculum
- **Method:** `GET`
- **URL:** `/api/v1/subjects/export`
- **Description:** Governed export of the subject catalog and curriculum mapping (SCR-SUB-13).
- **Authentication (Role-Based):** Yes — `subject.export`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "classId": {
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
