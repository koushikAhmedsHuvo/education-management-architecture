# ID Generation API

## 1. API Overview

**Purpose.** Configure and execute the format that auto-generates unique, immutable institutional identifiers — the Student ID (`STU-2026-MN-0001`-style) and the Employee ID — with a token-based pattern (prefix, year/session segment, campus code, zero-padded sequence), a uniqueness scope, and a reset policy.

**Module Context.** Implements Business Rules Catalog Doc 06 (Student) Rule STU-001 ("a unique student ID — format configurable per client/institute — prefix, sequence, year segment — assigned at creation, unique within the deployment, and never reused or re-edited") together with Doc 27 (Configuration Engine) Rule CFG-007 (definitions flagged **immutable-after-use** cannot be changed once dependent data exists — only set during setup). The equivalent Employee ID rule is the staff-identity counterpart of STU-001, scoped under the same Configuration Engine pattern. **Scope note:** the approved Configuration Engine UI Screen Spec does not list a standalone "ID Generation" screen — `STU-001` specifies only that an ID is "assigned at creation" by a configured scheme. This file realizes that scheme through the Configuration Engine's typed-definition + immutable-after-use mechanism, consistent with how this project's design-system work (Student ID Rules / Employee ID Rules) already modeled the same capability. No new business logic is introduced: this is the configuration and atomic-generation surface behind the `studentId`/`employeeId` fields already specified as auto-generated and immutable in `13-student-api.md` and the (forthcoming) Staff API.

---

## 2. Endpoints

### 1. Get ID Format Configuration
- **Method:** `GET`
- **URL:** `/api/v1/config/id-formats/{target}`
- **Description:** View the current ID-generation pattern for `student` or `employee` identifiers.
- **Authentication (Role-Based):** Yes — `config.view`, scoped.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "target": { "type": "string", "enum": ["student", "employee"] }
  },
  "required": ["target"]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "target": { "type": "string", "enum": ["student", "employee"] },
    "tokens": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "type": { "type": "string", "enum": ["STATIC_PREFIX", "SESSION_OR_YEAR", "DEPARTMENT_OR_JOIN_YEAR", "CAMPUS_CODE", "SEQUENCE"] },
          "value": { "type": "string", "description": "literal text for STATIC_PREFIX; zero-pad width for SEQUENCE" },
          "separator": { "type": "string" },
          "order": { "type": "integer" }
        },
        "required": ["type", "order"]
      }
    },
    "uniquenessScope": { "type": "string", "enum": ["DEPLOYMENT", "INSTITUTE", "CAMPUS"] },
    "resetPolicy": { "type": "string", "enum": ["PER_SESSION", "PER_CAMPUS", "NEVER"] },
    "samplePreview": { "type": "string", "description": "e.g. STU-2026-MN-0001" },
    "locked": { "type": "boolean", "description": "true once any ID has been issued under this format" },
    "version": { "type": "integer" }
  },
  "required": ["target", "tokens", "uniquenessScope", "resetPolicy", "locked", "version"]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** STU-001 (configurable ID scheme), CFG-007 (immutable-after-use flag surfaced as `locked`).
- **Pagination:** N/A.

### 2. Set ID Format Configuration
- **Method:** `PUT`
- **URL:** `/api/v1/config/id-formats/{target}`
- **Description:** Configure the token pattern, uniqueness scope, and reset policy for Student or Employee ID generation. **Locked once any ID has been issued under it** — settable only during setup.
- **Authentication (Role-Based):** Yes — `config.definition.manage` (platform/institute setup authority, consistent with other immutable-after-use definitions).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "tokens": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "type": { "type": "string", "enum": ["STATIC_PREFIX", "SESSION_OR_YEAR", "DEPARTMENT_OR_JOIN_YEAR", "CAMPUS_CODE", "SEQUENCE"] },
          "value": { "type": "string" },
          "separator": { "type": "string" },
          "order": { "type": "integer" }
        },
        "required": ["type", "order"]
      }
    },
    "uniquenessScope": { "type": "string", "enum": ["DEPLOYMENT", "INSTITUTE", "CAMPUS"] },
    "resetPolicy": { "type": "string", "enum": ["PER_SESSION", "PER_CAMPUS", "NEVER"] },
    "version": { "type": "integer" }
  },
  "required": ["tokens", "uniquenessScope", "resetPolicy", "version"]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "target": { "type": "string" },
    "tokens": { "type": "array", "items": { "type": "object" } },
    "samplePreview": { "type": "string" },
    "version": { "type": "integer" }
  },
  "required": ["target", "tokens", "version"]
}
```
- **Validation Rules:** The configuration is **rejected outright once any identifier has been issued under it** — an ID format is settable only during institute/deployment setup, before dependent data (students or employees) exists ("a used setting can only be superseded forward, not mutated," CFG-007; "changing an institute's type after enrollments exist is blocked" is the exact same `immutable-after-use` family of rule applied here to ID schemes); exactly one `SEQUENCE` token is required, with an explicit zero-pad width; `version` optimistic lock.
- **Error Codes:** `409 FORMAT_LOCKED_IDS_ISSUED` ("Locked after first use — supersede forward only"); `422 MISSING_SEQUENCE_TOKEN`; `409 VERSION_CONFLICT`.
- **Business Rules Applied:** CFG-007 (immutable-after-use settings are settable only during setup), STU-001.
- **Pagination:** N/A.

### 3. Preview Sample ID
- **Method:** `POST`
- **URL:** `/api/v1/config/id-formats/{target}/preview`
- **Description:** Render a sample identifier from a candidate (possibly unsaved) token configuration, for live preview during setup.
- **Authentication (Role-Based):** Yes — `config.definition.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "tokens": { "type": "array", "items": { "type": "object" } }
  },
  "required": ["tokens"]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "samplePreview": { "type": "string" }
  },
  "required": ["samplePreview"]
}
```
- **Validation Rules:** Purely computational — produces no side effects and never checks the `locked` state, since it is a what-if preview only.
- **Error Codes:** `422 INVALID_TOKEN_CONFIGURATION`.
- **Business Rules Applied:** STU-001.
- **Pagination:** N/A.

### 4. Generate Next Identifier
- **Method:** `POST`
- **URL:** `/api/v1/config/id-formats/{target}/generate`
- **Description:** Atomically generate and reserve the next identifier in sequence per the published format — the operation `13-student-api.md` #5 (`Create Student`) and the Staff API's equivalent creation endpoint call internally to assign `studentId`/`employeeId`.
- **Authentication (Role-Based):** Yes — internal/system-scoped (called by `student.create`-authorized and the equivalent `staff.create`-authorized flows; not intended for direct end-user invocation).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "scope": { "type": "object", "properties": { "instituteId": { "type": "string" }, "campusId": { "type": "string" }, "sessionId": { "type": "string" } }, "required": ["instituteId"] },
    "idempotencyKey": { "type": "string" }
  },
  "required": ["scope", "idempotencyKey"]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "identifier": { "type": "string" },
    "sequenceNumber": { "type": "integer" }
  },
  "required": ["identifier", "sequenceNumber"]
}
```
- **Validation Rules:** Generation is **atomic** — concurrent requests can never receive the same sequence number, even under high concurrent load (the same integrity guarantee STU-001 names: "never reused or re-edited"); generation also **locks the format configuration going forward**, transitioning `locked: false → true` on the very first successful call (#2's `409 FORMAT_LOCKED_IDS_ISSUED` thereafter applies); the uniqueness scope (deployment/institute/campus) and reset policy (per-session/per-campus/never) determine how the sequence counter is partitioned and when it resets.
- **Error Codes:** `422 FORMAT_NOT_CONFIGURED` (no ID format has been set for this target/scope yet).
- **Business Rules Applied:** STU-001 (unique, atomically assigned, immutable identifier), CFG-007 (first generation locks the format).
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — not applicable to this file: ID-format configuration is immutable-after-use rather than deletable, and generated identifiers are permanent by design (STU-001).
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
