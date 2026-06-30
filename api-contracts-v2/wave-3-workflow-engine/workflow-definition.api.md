# Workflow Definition API

## 1. API Overview

**Purpose.** Declare, draft, validate, and publish workflow definitions as version-pinned finite state machines — states, transitions, approver rules, SoD constraints, routing, and timeout policy.

**Module Context.** Implements Business Rules Catalog Doc 28 (Workflow Engine) Rules WFL-001/003/007/008/011 and UI Screen Spec `29-workflow-engine-screens.md` SCR-WFL-01/02.

---

## 2. Endpoints

### 1. List Workflow Definitions
- **Method:** `GET`
- **URL:** `/api/v1/workflows`
- **Description:** Browse declared workflows and their versions across all consuming modules (SCR-WFL-01).
- **Authentication (Role-Based):** Yes — `workflow.view`.
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
    "consumingModule": {
      "type": "string"
    },
    "statusDRAFTPUBLISHED": {
      "type": "string",
      "description": "status? (DRAFT|PUBLISHED|SUPERSEDED|ARCHIVED)"
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
      "key": {
        "type": "string"
      },
      "name": {
        "type": "string"
      },
      "consumingModule": {
        "type": "string"
      },
      "currentVersion": {
        "type": "number"
      },
      "status": {
        "type": "string"
      },
      "updatedAt": {
        "type": "string"
      }
    },
    "required": [
      "key",
      "name",
      "consumingModule",
      "currentVersion",
      "status",
      "updatedAt"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** WFL-001 (declarative definitions), CFG-004 (versioning pattern shared with Configuration Engine).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Workflow Definition Detail
- **Method:** `GET`
- **URL:** `/api/v1/workflows/{key}`
- **Description:** Full FSM — states, transitions, approver rules, SoD constraints, routing, timeout policy, return bound (SCR-WFL-02).
- **Authentication (Role-Based):** Yes — `workflow.view`; full edit access requires `workflow.definition.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "version": {
      "type": "integer"
    }
  }
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "key": {
      "type": "string"
    },
    "name": {
      "type": "string"
    },
    "version": {
      "type": "integer"
    },
    "status": {
      "type": "string"
    },
    "states": {
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
          "isTerminal": {
            "type": "boolean"
          }
        },
        "required": [
          "key",
          "label",
          "isTerminal"
        ]
      }
    },
    "transitions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "from": {
            "type": "string"
          },
          "to": {
            "type": "string"
          },
          "action": {
            "type": "string"
          }
        },
        "required": [
          "from",
          "to",
          "action"
        ]
      }
    },
    "approverRules": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "stepKey": {
            "type": "string"
          },
          "resolution": {
            "type": "string",
            "enum": [
              "role",
              "reportingLine",
              "scope",
              "amountTier"
            ]
          },
          "config": {
            "type": "string"
          }
        },
        "required": [
          "stepKey",
          "resolution",
          "config"
        ]
      }
    },
    "sodConstraints": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "type": {
            "type": "string",
            "const": "REQUESTER_NOT_APPROVER"
          },
          "config": {
            "type": "string"
          }
        },
        "required": [
          "type",
          "config"
        ]
      }
    },
    "routing": {
      "type": "object",
      "properties": {
        "type": {
          "type": "string",
          "enum": [
            "sequential",
            "parallel",
            "conditional"
          ]
        },
        "config": {
          "type": "string"
        }
      },
      "required": [
        "type",
        "config"
      ]
    },
    "timeoutPolicy": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "stepKey": {
            "type": "string"
          },
          "ttlHours": {
            "type": "string"
          },
          "onTimeout": {
            "type": "string",
            "enum": [
              "escalate",
              "hold",
              "autoReject",
              "autoAdvance"
            ]
          }
        },
        "required": [
          "stepKey",
          "ttlHours",
          "onTimeout"
        ]
      }
    },
    "returnBound": {
      "type": "number"
    },
    "publishedAt": {
      "type": "string"
    }
  },
  "required": [
    "key",
    "name",
    "version",
    "status",
    "states",
    "transitions",
    "approverRules",
    "sodConstraints",
    "routing",
    "timeoutPolicy",
    "returnBound",
    "publishedAt",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** WFL-001, WFL-002 (version exposed for transparency).
- **Pagination:** N/A.

### 3. Define / Draft a Workflow
- **Method:** `POST`
- **URL:** `/api/v1/workflows`
- **Description:** Declare a new workflow (or a new draft version of an existing key) as states, transitions, approver rules, and policies — never hard-coded process logic (UC-WFL-01, SCR-WFL-02).
- **Authentication (Role-Based):** Yes — `workflow.definition.manage` (governance role, tightly held).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "key": {
      "type": "string"
    },
    "name": {
      "type": "string"
    },
    "consumingModule": {
      "type": "string"
    },
    "states": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    },
    "transitions": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    },
    "approverRules": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    },
    "sodConstraints": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    },
    "routing": {
      "type": "object",
      "properties": {
        "note": {
          "type": "string",
          "description": "..."
        }
      }
    },
    "timeoutPolicy": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    },
    "returnBound": {
      "type": "number"
    }
  },
  "required": [
    "key",
    "name",
    "consumingModule",
    "states",
    "transitions",
    "approverRules",
    "sodConstraints",
    "routing",
    "timeoutPolicy",
    "returnBound"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "key": {
      "type": "string"
    },
    "version": {
      "type": "integer"
    },
    "status": {
      "type": "string",
      "const": "DRAFT"
    }
  },
  "required": [
    "key",
    "version",
    "status"
  ]
}
```
- **Validation Rules:** None blocking at draft save beyond basic shape completeness (full FSM validation runs at publish, #4) — drafts may be incomplete while being authored.
- **Error Codes:** `422 INCOMPLETE_DEFINITION`.
- **Business Rules Applied:** WFL-001 (declarative definitions).
- **Pagination:** N/A.

### 4. Validate & Publish Workflow Version
- **Method:** `POST`
- **URL:** `/api/v1/workflows/{key}/versions/{version}/publish`
- **Description:** Validate the FSM is reachable and terminating, that no sensitive step auto-approves, and publish an immutable version with an effective date (UC-WFL-01).
- **Authentication (Role-Based):** Yes — `workflow.definition.manage`; high-impact definition changes route to approval (#23) instead of publishing immediately.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "effectiveDate": {
      "type": "string",
      "description": "default now"
    }
  }
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "key": {
      "type": "string"
    },
    "version": {
      "type": "integer"
    },
    "status": {
      "type": "string",
      "enum": [
        "PUBLISHED",
        "PENDING_APPROVAL"
      ]
    },
    "publishedAt": {
      "type": "string"
    }
  },
  "required": [
    "key",
    "version",
    "status",
    "publishedAt"
  ]
}
```
- **Validation Rules:** FSM must be reachable from its initial state and every path must reach a terminal state ("Unreachable state \<X\>", "Workflow never terminates" — WFL-001/WFL-003); a timeout policy of `autoAdvance` on any step flagged sensitive is rejected outright ("Sensitive step cannot auto-approve on timeout" — WFL-007); routing (parallel quorum / conditional branches) must be well-formed and deterministic (WFL-008); `returnBound` must be a defined, finite positive integer (WFL-011); once published, the version is immutable (CFG-004 pattern) — further edits create a new draft version, never an in-place mutation.
- **Error Codes:** `422 UNREACHABLE_STATE`; `422 NON_TERMINATING_FSM`; `422 UNSAFE_AUTO_APPROVE`; `422 AMBIGUOUS_ROUTING`; `422 UNBOUNDED_RETURN_LOOP`.
- **Business Rules Applied:** WFL-001, WFL-003 (deterministic, reachable, terminating FSM), WFL-007 (no silent auto-approval of sensitive steps), WFL-008 (sequential/parallel/conditional routing), WFL-011 (bounded return loops).
- **Pagination:** N/A.

### 5. List Workflow Versions
- **Method:** `GET`
- **URL:** `/api/v1/workflows/{key}/versions`
- **Description:** Full version history of a workflow definition — what any in-flight or historical instance was pinned to.
- **Authentication (Role-Based):** Yes — `workflow.view`.
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
      "version": {
        "type": "integer"
      },
      "status": {
        "type": "string",
        "enum": [
          "DRAFT",
          "PUBLISHED",
          "SUPERSEDED",
          "ARCHIVED"
        ]
      },
      "publishedAt": {
        "type": "string"
      },
      "publishedBy": {
        "type": "string"
      }
    },
    "required": [
      "version",
      "status"
    ]
  }
}
```
- **Validation Rules:** None beyond access; published versions are never edited in place or hard-deleted (consumers may depend on them for fairness reconstruction).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** WFL-002 (version-pinning relies on full history being retained).
- **Pagination:** Yes — default `page=1, pageSize=25`, sorted `version desc`.

## Instances

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
