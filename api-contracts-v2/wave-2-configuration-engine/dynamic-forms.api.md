# Dynamic Forms API

## 1. API Overview

**Purpose.** Compose registered dynamic custom fields (`custom-fields.api.md`) into ordered, versioned, engine-validated forms — e.g., an admission application form or a student-profile intake form — and render/validate submissions against the published form version.

**Module Context.** Implements Business Rules Catalog Doc 27 (Configuration Engine) Rule CFG-010 (dynamic custom-field schemas are engine-validated) together with the Architecture Blueprint's form-engine decision (Part C: "static forms use typed schemas; dynamic forms render from definitions and validate against them, returning the same error envelope shape as the backend"). **Scope note:** the approved Configuration Engine UI Screen Spec (`28-configuration-engine-screens.md`) names only `SCR-CFG-07` (Dynamic Custom-Field Schema) as a distinct screen — it does not list a separate "Dynamic Forms" screen. This file exposes the form-composition layer the architecture already commits to building on top of that schema, consistent with the design-system work for this project (the Admission and Student modules' dynamic intake forms are exactly this capability in use). No new business logic is introduced: a form is an ordered arrangement of already-registered, already-validated fields; this API does not invent field types, validation rules, or bypass the schema in any way.

---

## 2. Endpoints

### 1. List Dynamic Forms
- **Method:** `GET`
- **URL:** `/api/v1/config/dynamic-forms`
- **Description:** Browse dynamic forms by entity/context (e.g., "admission-application", "student-intake"), with their current published version.
- **Authentication (Role-Based):** Yes — `config.view`, scoped.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "entity": { "type": "string", "description": "e.g. admission, student" },
    "page": { "type": "integer" },
    "pageSize": { "type": "integer" }
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
      "id": { "type": "string" },
      "entity": { "type": "string" },
      "name": { "type": "string" },
      "currentVersion": { "type": "integer" },
      "status": { "type": "string", "enum": ["DRAFT", "PUBLISHED", "SUPERSEDED", "ARCHIVED"] },
      "updatedAt": { "type": "string" }
    },
    "required": ["id", "entity", "name", "currentVersion", "status"]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CFG-010 (dynamic schemas are engine-validated); CFG-004 (versioning pattern shared with the rest of the Configuration Engine).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Dynamic Form Detail
- **Method:** `GET`
- **URL:** `/api/v1/config/dynamic-forms/{id}`
- **Description:** Full form definition — ordered sections and fields, each referencing a registered custom-field schema entry, plus version history.
- **Authentication (Role-Based):** Yes — `config.view`.
- **Request DTO (JSON Schema):**
```json
{ "type": "object", "properties": {}, "description": "No request body." }
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "id": { "type": "string" },
    "entity": { "type": "string" },
    "name": { "type": "string" },
    "version": { "type": "integer" },
    "sections": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "key": { "type": "string" },
          "label": { "type": "string" },
          "fields": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "fieldSchemaId": { "type": "string", "description": "References a registered Custom Field (see custom-fields.api.md)" },
                "label": { "type": "string" },
                "required": { "type": "boolean" },
                "validation": { "type": "string" },
                "helpText": { "type": "string" },
                "order": { "type": "integer" }
              },
              "required": ["fieldSchemaId", "label", "required", "order"]
            }
          }
        },
        "required": ["key", "label", "fields"]
      }
    },
    "status": { "type": "string", "enum": ["DRAFT", "PUBLISHED", "SUPERSEDED", "ARCHIVED"] },
    "versionHistory": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "version": { "type": "integer" },
          "publishedAt": { "type": "string" },
          "status": { "type": "string" }
        }
      }
    }
  },
  "required": ["id", "entity", "name", "version", "sections", "status"]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** CFG-010, CFG-004.
- **Pagination:** N/A.

### 3. Create / Draft Dynamic Form
- **Method:** `POST`
- **URL:** `/api/v1/config/dynamic-forms`
- **Description:** Compose a new dynamic form (or a new draft version of an existing one) from registered custom-field schema entries, in ordered sections.
- **Authentication (Role-Based):** Yes — `config.definition.manage`, scoped per the same institute-level grantability established for custom-field schemas (`custom-fields.api.md` #3 note).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "entity": { "type": "string" },
    "name": { "type": "string" },
    "sections": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "key": { "type": "string" },
          "label": { "type": "string" },
          "fields": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "fieldSchemaId": { "type": "string" },
                "label": { "type": "string" },
                "required": { "type": "boolean" },
                "validation": { "type": "string" },
                "helpText": { "type": "string" },
                "order": { "type": "integer" }
              },
              "required": ["fieldSchemaId", "label", "required", "order"]
            }
          }
        },
        "required": ["key", "label", "fields"]
      }
    }
  },
  "required": ["entity", "name", "sections"]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "id": { "type": "string" },
    "status": { "type": "string", "const": "DRAFT" },
    "version": { "type": "integer" }
  },
  "required": ["id", "status", "version"]
}
```
- **Validation Rules:** Every `fieldSchemaId` referenced must exist in the registered Custom Field catalog (`custom-fields.api.md`) for the same `entity` — a form can be composed only from fields the engine already knows how to validate ("build forms only from registered schema fields; do not invent field types or bypass the schema"); a draft may be saved incomplete while being authored, mirroring the draft/publish pattern used across the Configuration and Workflow Engines.
- **Error Codes:** `422 UNREGISTERED_FIELD_SCHEMA` (a referenced `fieldSchemaId` is not registered for this entity).
- **Business Rules Applied:** CFG-010 (dynamic schemas are engine-validated; values are never stored unvalidated), CFG-001 (no undeclared configuration — a form field must reference a registered definition).
- **Pagination:** N/A.

### 4. Publish Dynamic Form Version
- **Method:** `POST`
- **URL:** `/api/v1/config/dynamic-forms/{id}/versions/{version}/publish`
- **Description:** Publish a draft form as the new immutable, current version. Submissions made against a prior version continue to resolve against the version active when they were submitted.
- **Authentication (Role-Based):** Yes — `config.definition.manage`.
- **Request DTO (JSON Schema):**
```json
{ "type": "object", "properties": {}, "description": "No request body." }
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "id": { "type": "string" },
    "version": { "type": "integer" },
    "status": { "type": "string", "const": "PUBLISHED" }
  },
  "required": ["id", "version", "status"]
}
```
- **Validation Rules:** Every field reference is re-validated against the current Custom Field registry at publish time, since a referenced field's own schema may have changed since the form was drafted; once published, the version is immutable — further edits create a new draft version, never an in-place mutation; historical submissions remain bound to the version they were captured under ("form versions are immutable; submissions resolve against their version").
- **Error Codes:** `422 UNREGISTERED_FIELD_SCHEMA`; `409 STALE_VERSION_CONFLICT`.
- **Business Rules Applied:** CFG-010, CFG-004 (immutable version on publish).
- **Pagination:** N/A.

### 5. Get Published Form Schema (Render)
- **Method:** `GET`
- **URL:** `/api/v1/config/dynamic-forms/render`
- **Description:** Fetch the currently published form definition for a given entity/context, for client-side rendering and validation — the same endpoint shape consumed by `12-admission-api.md`'s `1. Get Application Schema`.
- **Authentication (Role-Based):** No (public where the consuming context is itself public, e.g. an admission application; otherwise scoped to the same authentication as the consuming screen).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "entity": { "type": "string" },
    "context": { "type": "string", "description": "e.g. sessionId, levelClassId — disambiguates which published form applies" }
  },
  "required": ["entity"]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "formVersion": { "type": "integer" },
    "sections": { "type": "array", "items": { "type": "object" } }
  },
  "required": ["formVersion", "sections"]
}
```
- **Validation Rules:** Only the currently `PUBLISHED` version is ever returned; if multiple forms could apply to the same entity, the most specific resolves per the Configuration Engine's standard most-specific-wins precedence (CFG-003).
- **Error Codes:** `404 NO_PUBLISHED_FORM`.
- **Business Rules Applied:** CFG-003 (most-specific-wins resolution), CFG-010.
- **Pagination:** N/A.

### 6. Validate Form Submission
- **Method:** `POST`
- **URL:** `/api/v1/config/dynamic-forms/{id}/validate-submission`
- **Description:** Server-side validation of a candidate submission against the published form's field definitions, without persisting it — used by consuming modules (e.g., Admission) before final submit, returning the same error-envelope shape as static validation.
- **Authentication (Role-Based):** No (mirrors the consuming screen's own authentication; this is a stateless validation pass).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "formVersion": { "type": "integer" },
    "fieldValues": { "type": "object", "description": "Map of fieldSchemaId to submitted value" }
  },
  "required": ["formVersion", "fieldValues"]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "valid": { "type": "boolean" },
    "fieldErrors": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "fieldSchemaId": { "type": "string" },
          "message": { "type": "string" }
        }
      }
    }
  },
  "required": ["valid", "fieldErrors"]
}
```
- **Validation Rules:** Each submitted value is validated against its referenced field's own type/range/options/required rules from the Custom Field registry (CFG-002 applied through CFG-010); this endpoint returns the same error envelope shape used by static request-DTO validation across the rest of the API (per the Architecture Blueprint's two-tier validation decision), so a consuming frontend handles both uniformly.
- **Error Codes:** None beyond the `valid: false` / `fieldErrors[]` result shape — this endpoint never itself errors on an invalid submission, only reports it.
- **Business Rules Applied:** CFG-002 (typed, validated values), CFG-010.
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
