# Custom Fields API

## 1. API Overview

**Purpose.** Define and version dynamic, institute-specific custom-field schemas (e.g., a 'blood group' field on Student) — engine-validated like any other configuration, with historical values resolving against the schema version active when they were captured.

**Module Context.** Implements Business Rules Catalog Doc 27 (Configuration Engine) Rule CFG-010 and UI Screen Spec `28-configuration-engine-screens.md` SCR-CFG-07 (Dynamic Custom-Field Schema).

---

## 2. Endpoints

### 1. List Custom-Field Schemas
- **Method:** `GET`
- **URL:** `/api/v1/config/custom-fields`
- **Description:** Browse dynamic custom-field schemas by entity (e.g., Student, Staff) (SCR-CFG-07).
- **Authentication (Role-Based):** Yes — `config.view`, scoped.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "entityegstudent": {
      "type": "string",
      "description": "entity? (e.g. \"student\"|\"staff\")"
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
      "entity": {
        "type": "string"
      },
      "fieldCount": {
        "type": "integer"
      },
      "currentVersion": {
        "type": "string"
      },
      "updatedAt": {
        "type": "string"
      }
    },
    "required": [
      "entity",
      "fieldCount",
      "currentVersion",
      "updatedAt"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CFG-010 (dynamic custom-field schemas are engine-validated).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Custom-Field Schema Detail
- **Method:** `GET`
- **URL:** `/api/v1/config/custom-fields/{entity}`
- **Description:** Current field list for an entity's dynamic schema.
- **Authentication (Role-Based):** Yes — `config.view`.
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
    "entity": {
      "type": "string"
    },
    "fields": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "key": {
            "type": "string"
          },
          "label": {
            "type": "string"
          },
          "dataType": {
            "type": "string"
          },
          "options": {
            "type": "string"
          },
          "required": {
            "type": "string"
          },
          "validation": {
            "type": "string"
          }
        },
        "required": [
          "key",
          "label",
          "dataType",
          "required"
        ]
      }
    },
    "version": {
      "type": "integer"
    },
    "updatedAt": {
      "type": "string"
    }
  },
  "required": [
    "entity",
    "fields",
    "version",
    "updatedAt"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** CFG-010.
- **Pagination:** N/A.

### 3. Create / Version a Custom-Field Schema
- **Method:** `POST`
- **URL:** `/api/v1/config/custom-fields/{entity}`
- **Description:** Define or amend an entity's dynamic schema; saving creates a new version so historical values continue to resolve against the version they were captured under (SCR-CFG-07).
- **Authentication (Role-Based):** Yes — `config.definition.manage`, **institute-scoped for custom-field schemas specifically** *(note: Doc 27 frames `config.definition.manage` as platform-tier for core system-setting definitions, but CFG-010's own worked example — "an institute adds a validated 'blood group' student field" — shows institute admins defining custom fields directly. This contract follows that example: the same permission string is grantable at institute scope for custom-field-schema actions, while remaining platform-only for the system definitions in §"Definition Registry" above.)*
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "fields": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "key": {
            "type": "string"
          },
          "label": {
            "type": "string"
          },
          "dataType": {
            "type": "string",
            "enum": [
              "string",
              "number",
              "boolean",
              "enum",
              "date"
            ]
          },
          "options": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "required": {
            "type": "boolean"
          },
          "validation": {
            "type": "string"
          }
        },
        "required": [
          "key",
          "label",
          "dataType",
          "required"
        ]
      }
    }
  },
  "required": [
    "fields"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "entity": {
      "type": "string"
    },
    "fields": {
      "type": "string",
      "description": "fields[]"
    },
    "version": {
      "type": "integer"
    },
    "status": {
      "type": "string",
      "const": "PUBLISHED"
    }
  },
  "required": [
    "entity",
    "version",
    "status"
  ]
}
```
- **Validation Rules:** Each field individually type/option/required-valid (CFG-002 applied to schema fields themselves); values entered against prior schema versions are unaffected — they continue to resolve against the version active when they were created.
- **Error Codes:** `422 INVALID_FIELD_DEFINITION`.
- **Business Rules Applied:** CFG-010, CFG-004 (versioned, immutable schema history).
- **Pagination:** N/A.

## Sensitive / Secret Configuration

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
