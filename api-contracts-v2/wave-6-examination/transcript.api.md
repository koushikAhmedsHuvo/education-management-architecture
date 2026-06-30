# Transcript API

## 1. API Overview

**Purpose.** View and download a single exam's published result/marksheet, and generate a **cumulative, multi-session academic transcript** — an immutable, reproducible document aggregating a student's results across multiple exams/sessions, suitable for alumni, transfer, and verification purposes.

**Module Context.** The single-exam marksheet endpoints (1–3) implement Business Rules Catalog Doc 15 (Result Processing) Rule RES-007 (withholding hides a result from the student until released) and UI Screen Spec `16-result-processing-screens.md` SCR-RES-04, moved here from `result.api.md` since marksheet/transcript *viewing* is a distinct concern from result *computation and publishing*. The cumulative transcript endpoints (4–7) are new in this consolidation: the approved repository does not name a standalone "Transcript" business-rules module, but the **concept itself is explicitly named** across the catalog — Doc 06 (Student) Rule STU-007 ("an alumnus's transcript remains retrievable years later"), Doc 05 (Academic Session) compliance notes ("academic record permanence... transcripts retained"), and this project's own design-system work for Wave 6 (Examination & Results), which specified a distinct "Transcript Generation" screen alongside "Report Card View." This file implements that already-named capability as a read-only aggregation **over already-published, already-immutable Result records** (`result.api.md`) — it computes nothing new and stamps no new provenance of its own; it reproduces, verbatim, the per-exam results that Result Processing already finalized, combined with each result's own `{ruleVer, configVer, gradeVer}` provenance carried through unchanged (RES-001, FILE-007).

---

## 2. Endpoints

### 1. Get Result / Marksheet
- **Method:** `GET`
- **URL:** `/api/v1/results/{id}/marksheet`
- **Description:** View a published marksheet with its full provenance footer (SCR-RES-04).
- **Authentication (Role-Based):** Yes — `result.view`, scoped/ownership (staff scoped; student/guardian own/linked-child only, custody-aware per GRD-N-006).
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
    "examName": {
      "type": "string"
    },
    "subjects": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "subjectId": {
            "type": "string"
          },
          "marks": {
            "type": "number"
          },
          "grade": {
            "type": "string"
          },
          "gradePoint": {
            "type": "string"
          }
        },
        "required": [
          "subjectId",
          "marks",
          "grade",
          "gradePoint"
        ]
      }
    },
    "aggregate": {
      "type": "string"
    },
    "division": {
      "type": "string"
    },
    "gpa": {
      "type": "string"
    },
    "rank": {
      "type": "integer"
    },
    "provenance": {
      "type": "object",
      "properties": {
        "ruleVer": {
          "type": "string"
        },
        "configVer": {
          "type": "string"
        },
        "gradeVer": {
          "type": "string"
        }
      },
      "required": [
        "ruleVer",
        "configVer",
        "gradeVer"
      ]
    },
    "status": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "studentId",
    "examName",
    "subjects",
    "aggregate",
    "division",
    "gpa",
    "provenance",
    "status"
  ]
}
```
- **Validation Rules:** A `WITHHELD` result returns a neutral "not yet available" response to a student/guardian viewer — never the underlying computed marks ("withheld results not shown to student until released," RES-007); a custody-restricted guardian is denied per the same rule as `13-student-api.md`/`14-guardian-api.md` (GRD-N-006).
- **Error Codes:** `404 NOT_FOUND` (not published, withheld-from-this-viewer, or out of scope — indistinguishable to the caller).
- **Business Rules Applied:** RES-007 (withholding hides the result from the student until release), FILE-007 (immutable, reproducible), REP-002, GRD-N-006.
- **Pagination:** N/A.

### 2. Get Marksheet Download URL
- **Method:** `GET`
- **URL:** `/api/v1/results/{id}/marksheet/download-url`
- **Description:** Short-lived signed URL to download the marksheet document.
- **Authentication (Role-Based):** Yes — `result.view`, same scope/ownership rules as #9.
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
- **Validation Rules:** Same withholding/custody gate as #9.
- **Error Codes:** `404 NOT_AVAILABLE`.
- **Business Rules Applied:** FILE-005, FILE-007.
- **Pagination:** N/A.

### 3. Reprint Marksheet
- **Method:** `POST`
- **URL:** `/api/v1/results/{id}/marksheet/reprint`
- **Description:** Regenerate the marksheet document — always produces the **identical** artifact, since it is computed from the immutable, stamped provenance (SCR-RES-04).
- **Authentication (Role-Based):** Yes — `result.view`.
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
- **Validation Rules:** A reprint of a result that has since been revised (#12) returns the **current** version's marksheet, not a blend — the prior version remains separately retrievable via its own version history.
- **Error Codes:** `404 NOT_AVAILABLE`.
- **Business Rules Applied:** FILE-007 (immutable, reproducible).
- **Pagination:** N/A.

## Governed Revision

### 4. List Transcript-Eligible Sessions
- **Method:** `GET`
- **URL:** `/api/v1/students/{id}/transcript/eligible-sessions`
- **Description:** List the sessions/exams for a student that have a finalized (published, non-withheld) result and are therefore eligible for inclusion in a cumulative transcript.
- **Authentication (Role-Based):** Yes — `result.view`, scoped/ownership (staff scoped; student/guardian own/linked-child only).
- **Request DTO (JSON Schema):**
```json
{ "type": "object", "properties": {}, "description": "No request body." }
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "sessionId": { "type": "string" },
      "examId": { "type": "string" },
      "examName": { "type": "string" },
      "resultId": { "type": "string" },
      "status": { "type": "string", "enum": ["PUBLISHED", "REVISED"] },
      "gpa": { "type": "number" }
    },
    "required": ["sessionId", "examId", "resultId", "status"]
  }
}
```
- **Validation Rules:** A session/exam appears only if its result is `PUBLISHED` or `REVISED` (the current, approved state) — `WITHHELD`, `INCOMPLETE`, or unpublished results are never included or even revealed to exist, consistent with the same withholding rule applied to single marksheets (`1. Get Result / Marksheet` above, RES-007).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** RES-007 (withheld/unpublished results never surfaced to the student), STU-007 (transcript permanence).
- **Pagination:** N/A.

### 5. Generate Cumulative Transcript
- **Method:** `POST`
- **URL:** `/api/v1/students/{id}/transcript/generate`
- **Description:** Generate the immutable, reproducible cumulative transcript document, aggregating the selected (or all eligible) sessions' already-published results.
- **Authentication (Role-Based):** Yes — `result.view` for self/own-child generation; staff-initiated generation (e.g., for a transfer certificate) requires `fee.invoice.issue`-tier registrar authority or the equivalent `enrollment.withdraw`-adjacent clearance authority, scoped — generation never bypasses the per-result withholding/ownership checks already enforced on each constituent result.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "sessionIds": { "type": "array", "items": { "type": "string" }, "description": "Omit to include all currently eligible sessions" },
    "idempotencyKey": { "type": "string" }
  },
  "required": ["idempotencyKey"]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "transcriptId": { "type": "string" },
    "studentId": { "type": "string" },
    "includedResults": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "resultId": { "type": "string" },
          "sessionId": { "type": "string" },
          "provenance": { "type": "object", "properties": { "ruleVer": { "type": "integer" }, "configVer": { "type": "integer" }, "gradeVer": { "type": "integer" } } }
        }
      }
    },
    "cumulativeGpa": { "type": "number" },
    "generatedAt": { "type": "string" },
    "status": { "type": "string", "const": "GENERATED" }
  },
  "required": ["transcriptId", "studentId", "includedResults", "generatedAt", "status"]
}
```
- **Validation Rules:** Only sessions returned by `4. List Transcript-Eligible Sessions` may be included — a withheld or unpublished result can never enter a generated transcript, even by explicit request ("withheld results not shown to student until released," RES-007, applied transitively); each included result's own provenance stamp is carried through **unchanged** — the transcript never recomputes a result, it reproduces what Result Processing already finalized (RES-001); the resulting document is **immutable and reproducible**: regenerating with the same  and the same set of constituent results' current versions always yields the byte-identical document (FILE-007, mirroring `3. Reprint Marksheet` above); generation is idempotent on `idempotencyKey`.
- **Error Codes:** `422 NO_ELIGIBLE_SESSIONS`; `409 SESSION_RESULT_WITHHELD` (an explicitly requested  is not eligible).
- **Business Rules Applied:** RES-001 (provenance carried through unchanged, never recomputed), RES-007 (withheld results excluded), FILE-007 (immutable, reproducible document), STU-007 (transcript permanence — "an alumnus's transcript remains retrievable years later").
- **Pagination:** N/A.

### 6. Get Transcript Download URL
- **Method:** `GET`
- **URL:** `/api/v1/students/{id}/transcript/{transcriptId}/download-url`
- **Description:** Short-lived signed URL to download the generated cumulative transcript document.
- **Authentication (Role-Based):** Yes — `result.view`, same scope/ownership rules as endpoint 1.
- **Request DTO (JSON Schema):**
```json
{ "type": "object", "properties": {}, "description": "No request body." }
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "downloadUrl": { "type": "string" },
    "expiresAt": { "type": "string" }
  },
  "required": ["downloadUrl", "expiresAt"]
}
```
- **Validation Rules:** None beyond access and the same withholding/custody gate as endpoint 1.
- **Error Codes:** `404 NOT_AVAILABLE`.
- **Business Rules Applied:** FILE-005 (short-lived signed URLs), FILE-007.
- **Pagination:** N/A.

### 7. Reprint Cumulative Transcript
- **Method:** `POST`
- **URL:** `/api/v1/students/{id}/transcript/{transcriptId}/reprint`
- **Description:** Regenerate the cumulative transcript document — always the **identical** artifact, since every constituent result's provenance is immutable and the aggregation logic is deterministic.
- **Authentication (Role-Based):** Yes — `result.view`.
- **Request DTO (JSON Schema):**
```json
{ "type": "object", "properties": {}, "description": "No request body." }
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "downloadUrl": { "type": "string" },
    "expiresAt": { "type": "string" }
  },
  "required": ["downloadUrl", "expiresAt"]
}
```
- **Validation Rules:** If any constituent result has since been **revised** (`result.api.md` #14, governed revision), the reprinted transcript reflects the **current** version of that result, with the revision itself visible in that result's own version history — a reprint never silently masks a disclosed revision.
- **Error Codes:** `404 NOT_AVAILABLE`.
- **Business Rules Applied:** FILE-007 (immutable, reproducible), RES-006 (revisions are disclosed, never silent).
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — not applicable: transcripts are immutable, generated documents; they are never edited or deleted, only regenerated to reflect a disclosed underlying revision.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
