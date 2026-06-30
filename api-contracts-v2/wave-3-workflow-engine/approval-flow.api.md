# Approval Flow API

## 1. API Overview

**Purpose.** SLA/bottleneck reporting over workflow execution, and governance of workflow-definition changes themselves — a separately-approved, SoD-enforced change-control layer so no one can both author and approve their own process change.

**Module Context.** Implements Business Rules Catalog Doc 28 (Workflow Engine) §11 and CFG-007-pattern governance, and UI Screen Spec `29-workflow-engine-screens.md` SCR-WFL-09/12.

---

## 2. Endpoints

### 1. SLA / Bottleneck Report
- **Method:** `GET`
- **URL:** `/api/v1/workflows/report/sla`
- **Description:** Average time-to-decision, breach rate, and bottlenecks by step/approver, scope-limited (SCR-WFL-09).
- **Authentication (Role-Based):** Yes — `workflow.report.view`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "workflowKey": {
      "type": "string"
    },
    "dateFrom": {
      "type": "string"
    },
    "dateTo": {
      "type": "string"
    },
    "groupBystepapprov": {
      "type": "string",
      "description": "groupBy? (\"step\"|\"approver\", default step)"
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
      "groupKey": {
        "type": "string"
      },
      "avgTimeToDecisionHours": {
        "type": "number"
      },
      "breachRate": {
        "type": "number"
      },
      "instanceCount": {
        "type": "number"
      }
    },
    "required": [
      "groupKey",
      "avgTimeToDecisionHours",
      "breachRate",
      "instanceCount"
    ]
  }
}
```
- **Validation Rules:** Results scope-limited to the caller's authority; large date ranges run async per the approved screen's own deviation note (returns `202` + `jobId` above a configured threshold, conventions §9).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** WFL-005 (SLA tracking that underlies this report), Doc 28 §9 (`workflow.report.view`, scoped).
- **Pagination:** Yes — default `page=1, pageSize=25` (when not async).

### 2. Export SLA / Audit Report
- **Method:** `GET`
- **URL:** `/api/v1/workflows/report/sla/export`
- **Description:** Governed export of the SLA/bottleneck/decision-audit report (signed URL).
- **Authentication (Role-Based):** Yes — `workflow.report.view` (no distinct export permission is named in Doc 28 §9; this contract reuses the report's own view permission for export, consistent with how this screen's "ExportDialog (governed)" is described as part of the same report surface).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "description": "Same filters as #21.",
  "$ref": "#endpoint-21"
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
- **Validation Rules:** Same scope limits as #21; export is itself audited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** Doc 28 §11 (audit requirements), conventions §10.
- **Pagination:** N/A.

## Change Approvals (Definition Governance)

### 3. List Pending Workflow-Definition Change Approvals
- **Method:** `GET`
- **URL:** `/api/v1/workflows/approvals`
- **Description:** Inbox for high-impact workflow-definition changes awaiting a separate approver's decision (SCR-WFL-12).
- **Authentication (Role-Based):** Yes — `workflow.definition.approve` *(named per the established view/manage/approve pairing convention from `00-api-conventions.md` §3; the approved screen names the combination "`workflow.definition.manage` + approval authority" without a single literal permission string for the decide-side, so this contract introduces the explicit, separately-grantable `workflow.definition.approve` so the requester and approver can never be the same authority by construction)*.
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
    "statusPENDINGAPPROVE": {
      "type": "string",
      "description": "status? (PENDING|APPROVED|REJECTED)"
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
      "workflowKey": {
        "type": "string"
      },
      "proposedVersion": {
        "type": "number"
      },
      "diff": {
        "type": "object",
        "properties": {
          "added": {
            "type": "array",
            "items": {
              "type": "string",
              "description": "..."
            }
          },
          "removed": {
            "type": "array",
            "items": {
              "type": "string",
              "description": "..."
            }
          },
          "changed": {
            "type": "array",
            "items": {
              "type": "string",
              "description": "..."
            }
          }
        },
        "required": [
          "added",
          "removed",
          "changed"
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
      "workflowKey",
      "proposedVersion",
      "diff",
      "requestedBy",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Requests the caller themselves raised are listed but cannot be decided by them (#24).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CFG-007 (governed high-impact change pattern shared with Configuration Engine), AUTHZ-009 / WFL-004 (requester ≠ approver).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 4. Decide Workflow-Definition Change Request
- **Method:** `POST`
- **URL:** `/api/v1/workflows/approvals/{id}/decide`
- **Description:** Approve or reject a pending workflow-definition change.
- **Authentication (Role-Based):** Yes — `workflow.definition.approve`; **blocked if the caller proposed the change** — self-raised definition changes render this action disabled regardless of any other role held (SCR-WFL-12 deviation note).
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
- **Validation Rules:** `reason` required on `REJECT`; caller ≠ requester.
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED`; `422 REASON_REQUIRED`; `409 ALREADY_DECIDED`.
- **Business Rules Applied:** WFL-004, AUTHZ-009, CFG-007.
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
