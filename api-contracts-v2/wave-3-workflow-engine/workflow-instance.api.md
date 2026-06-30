# Workflow Instance API

## 1. API Overview

**Purpose.** Initiate, search, and progress workflow instances — version-pinned at initiation, with dynamic SoD-respecting approver resolution, the approver inbox, decision recording, bulk action on low-risk tasks, delegation, withdrawal/cancellation/reassignment, and idempotent orphan recovery.

**Module Context.** Implements Business Rules Catalog Doc 28 (Workflow Engine) Rules WFL-002/003/004/005/006/009/010/011 and UI Screen Spec `29-workflow-engine-screens.md` SCR-WFL-03…07/10/11.

---

## 2. Endpoints

### 1. Initiate Workflow Instance
- **Method:** `POST`
- **URL:** `/api/v1/workflow-instances`
- **Description:** Start a process instance against a published workflow — pins the current definition version (the fairness guarantee), resolves the first step's approver(s) with SoD, and starts SLA timers (UC-WFL-02). Called by consuming modules (Admission, Leave, Discount, etc.) when their own domain action requires governed approval.
- **Authentication (Role-Based):** Yes — `workflow.instance.initiate` (often granted implicitly alongside the domain action that triggers it, per Doc 28 §9).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "workflowKey": {
      "type": "string"
    },
    "context": {
      "type": "object",
      "description": "domain payload \u2014 e.g., requestType, amount, targetEntityId"
    },
    "initiatedBy": {
      "type": "string",
      "description": "defaults to caller"
    }
  },
  "required": [
    "workflowKey",
    "context",
    "initiatedBy"
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
    "workflowKey": {
      "type": "string"
    },
    "pinnedVersion": {
      "type": "number"
    },
    "status": {
      "type": "string",
      "const": "INITIATED"
    },
    "currentStep": {
      "type": "object",
      "properties": {
        "key": {
          "type": "string"
        },
        "resolvedApprovers": {
          "type": "array",
          "items": {
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
          }
        }
      },
      "required": [
        "key",
        "resolvedApprovers"
      ]
    },
    "slaDueAt": {
      "type": "string"
    },
    "initiatedBy": {
      "type": "string"
    },
    "initiatedAt": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "workflowKey",
    "pinnedVersion",
    "status",
    "currentStep",
    "slaDueAt",
    "initiatedBy",
    "initiatedAt"
  ]
}
```
- **Validation Rules:** `workflowKey` must reference a `PUBLISHED` definition; the **currently published version is snapshotted and pinned** to this instance — a later edit to the definition never alters it (WFL-002); the first step's approver(s) are resolved dynamically (by role, reporting line, scope, or amount tier) with SoD applied so the initiator can never be resolved as their own approver — if the only eligible approver would be the initiator, resolution instead routes to the next eligible authority, or the instance is flagged rather than silently auto-approved (WFL-004, WFL-007).
- **Error Codes:** `404 WORKFLOW_NOT_PUBLISHED`; `422 NO_ELIGIBLE_APPROVER` (flagged, not silently approved).
- **Business Rules Applied:** WFL-002 (version-pinning at initiation), WFL-004 (dynamic approver resolution with SoD), WFL-007 (safety on resolution failure).
- **Pagination:** N/A.

### 2. Search Instances / Tasks
- **Method:** `GET`
- **URL:** `/api/v1/workflow-instances`
- **Description:** Scope-limited search across instances and tasks by module, state, participant, and context (SCR-WFL-08).
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
    "workflowKey": {
      "type": "string"
    },
    "consumingModule": {
      "type": "string"
    },
    "statuscommaseparated": {
      "type": "string",
      "description": "status? (comma-separated)"
    },
    "participantId": {
      "type": "string"
    },
    "contextRef": {
      "type": "string"
    },
    "sortByinitiatedAtsla": {
      "type": "string",
      "description": "sortBy? (initiatedAt|slaDueAt, default initiatedAt desc)"
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
      "workflowName": {
        "type": "string"
      },
      "pinnedVersion": {
        "type": "string"
      },
      "status": {
        "type": "string"
      },
      "currentStep": {
        "type": "string"
      },
      "initiatedBy": {
        "type": "string"
      },
      "initiatedAt": {
        "type": "string"
      },
      "slaDueAt": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "workflowKey",
      "workflowName",
      "pinnedVersion",
      "status",
      "initiatedBy",
      "initiatedAt"
    ]
  }
}
```
- **Validation Rules:** Results scope-limited to the caller's authority (AUTHZ-002); a non-administrator sees only instances they initiated or are/were a participant in.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-002 (mandatory scope filter), Doc 28 §9 (`workflow.view` is scoped; initiators see their own).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 3. Get Instance Detail (State, Lineage, Participants)
- **Method:** `GET`
- **URL:** `/api/v1/workflow-instances/{id}`
- **Description:** Current state, the **complete decision lineage** (every step, actor, decision, timestamp), participants, and the pinned definition version (SCR-WFL-04).
- **Authentication (Role-Based):** Yes — `workflow.view`, scoped (participant or administrator); manage actions on the instance require `workflow.instance.manage`.
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
    "workflowKey": {
      "type": "string"
    },
    "pinnedVersion": {
      "type": "string"
    },
    "status": {
      "type": "string"
    },
    "currentStep": {
      "type": "object",
      "properties": {
        "key": {
          "type": "string"
        },
        "resolvedApprovers": {
          "type": "string"
        },
        "slaDueAt": {
          "type": "string"
        }
      },
      "required": [
        "key",
        "resolvedApprovers",
        "slaDueAt"
      ]
    },
    "participants": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": {
            "type": "string"
          },
          "displayName": {
            "type": "string"
          },
          "role": {
            "type": "string"
          }
        },
        "required": [
          "id",
          "displayName",
          "role"
        ]
      }
    },
    "lineage": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "stepKey": {
            "type": "string"
          },
          "actor": {
            "type": "string"
          },
          "decision": {
            "type": "string",
            "enum": [
              "APPROVE",
              "REJECT",
              "RETURN",
              "DELEGATE",
              "ESCALATE"
            ]
          },
          "reason": {
            "type": "string"
          },
          "actingFor": {
            "type": "string"
          },
          "timestamp": {
            "type": "string"
          }
        },
        "required": [
          "stepKey",
          "actor",
          "decision",
          "timestamp"
        ]
      }
    },
    "context": {
      "type": "string"
    },
    "initiatedBy": {
      "type": "string"
    },
    "initiatedAt": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "workflowKey",
    "pinnedVersion",
    "status",
    "participants",
    "lineage",
    "context",
    "initiatedBy",
    "initiatedAt"
  ]
}
```
- **Validation Rules:** The full lineage is always shown to authorized viewers — never truncated or summarized away (SCR-WFL-04 deviation note); the displayed definition is always the **pinned** version, never the current one if they differ (WFL-002).
- **Error Codes:** `404 NOT_FOUND` (out of scope).
- **Business Rules Applied:** WFL-002 (pinned version always shown), WFL-003 (state), Doc 28 §11 (full attributable lineage).
- **Pagination:** N/A.

## Approving / Acting

### 4. List My Tasks (Approver Inbox)
- **Method:** `GET`
- **URL:** `/api/v1/workflow-tasks/my`
- **Description:** The unified inbox of tasks routed to the caller as a resolved approver — including tasks held via an active delegation (SCR-WFL-03).
- **Authentication (Role-Based):** Yes — `workflow.act` (a user sees only tasks the engine actually resolved to them — never a broader list); delegated tasks are tagged.
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
    "statusPENDINGESCALAT": {
      "type": "string",
      "description": "status? (PENDING|ESCALATED)"
    },
    "sortByslaDueAtdefau": {
      "type": "string",
      "description": "sortBy? (slaDueAt, default asc)"
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
      "taskId": {
        "type": "string"
      },
      "instanceId": {
        "type": "string"
      },
      "workflowKey": {
        "type": "string"
      },
      "workflowName": {
        "type": "string"
      },
      "stepKey": {
        "type": "string"
      },
      "stepLabel": {
        "type": "string"
      },
      "context": {
        "type": "string"
      },
      "delegatedFrom": {
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
      "slaDueAt": {
        "type": "string"
      },
      "status": {
        "type": "string"
      }
    },
    "required": [
      "taskId",
      "instanceId",
      "workflowKey",
      "workflowName",
      "stepKey",
      "stepLabel",
      "context",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Only tasks where the caller is the engine-resolved approver (directly or via an active, unexpired delegation) are returned.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** WFL-004 (dynamic approver resolution), WFL-006 (delegated tasks visible, tagged).
- **Pagination:** Yes — default `page=1, pageSize=25`, sorted by `slaDueAt asc`.

### 5. Decide a Task (Approve / Reject / Return)
- **Method:** `POST`
- **URL:** `/api/v1/workflow-instances/{id}/tasks/{taskId}/decide`
- **Description:** Act on an assigned task, enforcing only-declared-transitions, SoD, and a bounded return counter (SCR-WFL-05).
- **Authentication (Role-Based):** Yes — `workflow.act`, **resolved approver for this specific task only**; self-approval is structurally disabled — if the caller is also this instance's initiator, the action is rejected regardless of any other role they hold.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "decision": {
      "type": "string",
      "enum": [
        "APPROVE",
        "REJECT",
        "RETURN"
      ]
    },
    "reason": {
      "type": "string",
      "description": "required for REJECT/RETURN"
    },
    "idempotencyKey": {
      "type": "string"
    }
  },
  "required": [
    "decision",
    "idempotencyKey"
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
        "taskId": {
          "type": "string"
        },
        "instanceId": {
          "type": "string"
        },
        "decision": {
          "type": "string"
        },
        "newState": {
          "type": "string"
        },
        "returnCount": {
          "type": "number"
        },
        "returnsRemaining": {
          "type": "number"
        }
      },
      "required": [
        "taskId",
        "instanceId",
        "decision",
        "newState"
      ]
    },
    {
      "type": "object",
      "properties": {
        "RETURN": {
          "type": "string"
        }
      },
      "required": [
        "RETURN"
      ]
    },
    {
      "type": "object",
      "properties": {
        "RETURNED": {
          "type": "string"
        }
      },
      "required": [
        "RETURNED"
      ]
    }
  ]
}
```
- **Validation Rules:** Only the **declared** transition from the current state is permitted — out-of-band state jumps are impossible ("only declared transitions," WFL-003); `reason` required on `REJECT`/`RETURN`; caller ≠ this instance's initiator (WFL-004); `RETURN` increments the instance's correction-round counter — once it reaches the definition's `returnBound`, `RETURN` becomes unavailable and the step **escalates** instead (WFL-011); the action is idempotent on `idempotencyKey` — a double-submitted click decides exactly once (WFL-010).
- **Error Codes:** `409 INVALID_TRANSITION`; `403 SOD_VIOLATION_BLOCKED` ("You raised this request — a different approver is required"); `422 REASON_REQUIRED`; `409 RETURN_BOUND_EXCEEDED` (escalates automatically rather than failing silently).
- **Business Rules Applied:** WFL-003 (deterministic FSM execution), WFL-004 (SoD), WFL-011 (bounded return loops), WFL-010 (idempotent actions).
- **Pagination:** N/A.

### 6. Bulk Approve / Act
- **Method:** `POST`
- **URL:** `/api/v1/workflow-tasks/bulk-act`
- **Description:** Act on many **low-risk** tasks at once where the platform permits it; sensitive and SoD-bearing task types are categorically excluded from bulk action (SCR-WFL-10).
- **Authentication (Role-Based):** Yes — `workflow.bulk.act`.
- **Request DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "taskIds": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "decision": {
          "type": "string",
          "enum": [
            "APPROVE",
            "REJECT"
          ]
        },
        "reason": {
          "type": "string"
        }
      },
      "required": [
        "taskIds",
        "decision"
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
  "type": "null",
  "description": "202 Accepted { jobId } (always async per the approved screen's own deviation note; poll via GET /api/v1/jobs/{jobId} \u2014 conventions \u00a79 \u2014 for the per-task result list { accepted: [{ taskId }], rejected: [{ taskId, reason }] })."
}
```
- **Validation Rules:** Only task types flagged **low-risk** in their workflow definition are eligible for bulk action; any sensitive, SoD-bearing, or high-impact task in the submitted set is excluded and reported in `rejected[]`, never silently force-acted ("only low-risk task types eligible; SoD/sensitive excluded," WFL-007); the whole batch is idempotent on the supplied key (WFL-010).
- **Error Codes:** `400 MALFORMED_BATCH`; per-task exclusions reported in the job result, not as a top-level error.
- **Business Rules Applied:** WFL-007 (sensitive steps excluded from any auto/bulk shortcut), WFL-010 (idempotency).
- **Pagination:** N/A.

## Delegation

### 7. List My Delegations
- **Method:** `GET`
- **URL:** `/api/v1/workflow-delegations`
- **Description:** View delegations the caller has granted or received.
- **Authentication (Role-Based):** Yes — self (`workflow.delegate` for ones the caller granted).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "directiongrantedr": {
      "type": "string",
      "description": "direction? (\"granted\"|\"received\")"
    },
    "statusACTIVEEXPIRED": {
      "type": "string",
      "description": "status? (ACTIVE|EXPIRED|REVOKED)"
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
      "delegator": {
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
      "delegate": {
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
      "scope": {
        "type": "string"
      },
      "periodFrom": {
        "type": "string"
      },
      "periodTo": {
        "type": "string"
      },
      "status": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "delegator",
      "delegate",
      "periodFrom",
      "periodTo",
      "status"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** WFL-006.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 8. Set Delegation
- **Method:** `POST`
- **URL:** `/api/v1/workflow-delegations`
- **Description:** Delegate approval authority for a bounded period — e.g., while away (SCR-WFL-06).
- **Authentication (Role-Based):** Yes — `workflow.delegate` (an approver delegating their own authority).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "delegateUserId": {
      "type": "string"
    },
    "periodFrom": {
      "type": "string"
    },
    "periodTo": {
      "type": "string"
    },
    "scope": {
      "type": "object"
    }
  },
  "required": [
    "delegateUserId",
    "periodFrom",
    "periodTo"
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
    "delegator": {
      "type": "string"
    },
    "delegate": {
      "type": "string"
    },
    "periodFrom": {
      "type": "string"
    },
    "periodTo": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "ACTIVE"
    }
  },
  "required": [
    "id",
    "delegator",
    "delegate",
    "periodFrom",
    "periodTo",
    "status"
  ]
}
```
- **Validation Rules:** `delegateUserId` must be SoD-eligible (the delegate is still bound by SoD — they cannot approve a request the delegator themselves raised, WFL-004); delegation chains are bounded — a delegate who has themselves delegated further cannot be re-delegated to beyond the configured chain depth, preventing loops; the delegation auto-expires at `periodTo` without manual action and is revocable before then.
- **Error Codes:** `422 DELEGATE_NOT_ELIGIBLE`; `422 DELEGATION_CHAIN_BOUND_EXCEEDED`.
- **Business Rules Applied:** WFL-006 (bounded, audited delegation), WFL-004 (SoD still applies to the delegate).
- **Pagination:** N/A.

### 9. Revoke Delegation
- **Method:** `DELETE`
- **URL:** `/api/v1/workflow-delegations/{id}`
- **Description:** End a delegation before its scheduled `periodTo`.
- **Authentication (Role-Based):** Yes — `workflow.delegate`, the original delegator only.
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
- **Validation Rules:** Allowed only from `ACTIVE`; tasks already decided by the delegate remain attributed as acting-for the delegator (history is never rewritten).
- **Error Codes:** `409 INVALID_STATE_TRANSITION`.
- **Business Rules Applied:** WFL-006.
- **Pagination:** N/A.

## Withdraw, Cancel, Reassign

### 10. Withdraw Instance
- **Method:** `POST`
- **URL:** `/api/v1/workflow-instances/{id}/withdraw`
- **Description:** The initiator ends their own instance before a terminal decision (SCR-WFL-07).
- **Authentication (Role-Based):** Yes — this instance's initiator only (no broader permission required).
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
      "const": "WITHDRAWN"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** Only permitted while the instance has **not** yet reached a terminal state (`APPROVED`/`REJECTED`/already `WITHDRAWN`/`CANCELLED`).
- **Error Codes:** `409 ALREADY_TERMINAL` ("This instance has already reached a final decision").
- **Business Rules Applied:** WFL-009 (governed withdrawal, pre-terminal only).
- **Pagination:** N/A.

### 11. Cancel Instance (Admin)
- **Method:** `POST`
- **URL:** `/api/v1/workflow-instances/{id}/cancel`
- **Description:** An administrator ends an instance with a recorded reason.
- **Authentication (Role-Based):** Yes — `workflow.instance.manage` (elevated).
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
      "const": "CANCELLED"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** Same pre-terminal constraint as #15; `reason` required.
- **Error Codes:** `409 ALREADY_TERMINAL`; `422 REASON_REQUIRED`.
- **Business Rules Applied:** WFL-009.
- **Pagination:** N/A.

### 12. Reassign a Stuck Task
- **Method:** `POST`
- **URL:** `/api/v1/workflow-instances/{id}/tasks/{taskId}/reassign`
- **Description:** Move a pending step to another eligible approver when the resolved approver is stuck/unavailable.
- **Authentication (Role-Based):** Yes — `workflow.instance.manage` (elevated).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "toUserId": {
      "type": "string"
    },
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "toUserId",
    "reason"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "taskId": {
      "type": "string"
    },
    "reassignedTo": {
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
    }
  },
  "required": [
    "taskId",
    "reassignedTo"
  ]
}
```
- **Validation Rules:** `toUserId` must be SoD-eligible for this task (cannot be the instance's initiator); `reason` required.
- **Error Codes:** `422 TARGET_NOT_ELIGIBLE`; `422 REASON_REQUIRED`.
- **Business Rules Applied:** WFL-009, WFL-004 (SoD still applies to the reassignment target).
- **Pagination:** N/A.

## Orphan Detection & Recovery

### 13. List Orphaned / Stuck Instances
- **Method:** `GET`
- **URL:** `/api/v1/workflow-instances/orphans`
- **Description:** Surface instances stuck beyond configured limits for administrator intervention (SCR-WFL-11).
- **Authentication (Role-Based):** Yes — `workflow.instance.manage` (elevated).
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
    "workflowKey": {
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
      "workflowKey": {
        "type": "string"
      },
      "currentStep": {
        "type": "string"
      },
      "stuckSince": {
        "type": "string"
      },
      "reason": {
        "type": "string",
        "enum": [
          "NO_ESCALATION_TARGET",
          "PROCESS_FAILURE",
          "UNKNOWN"
        ]
      }
    },
    "required": [
      "id",
      "workflowKey",
      "currentStep",
      "stuckSince",
      "reason"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** WFL-010 (reliability — no instance is ever silently lost; orphan detection flags stuck instances).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 14. Recover an Orphaned Instance
- **Method:** `POST`
- **URL:** `/api/v1/workflow-instances/{id}/recover`
- **Description:** Safely resume a stuck instance — **idempotent**, re-running is always safe and never double-decides or duplicates an outcome.
- **Authentication (Role-Based):** Yes — `workflow.instance.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "idempotencyKey": {
      "type": "string"
    }
  },
  "required": [
    "idempotencyKey"
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
      "type": "string"
    },
    "recovered": {
      "type": "boolean",
      "const": true
    }
  },
  "required": [
    "id",
    "status",
    "recovered"
  ]
}
```
- **Validation Rules:** Re-invoking recovery on an instance already recovered (or no longer stuck) is a safe no-op, not an error — idempotent by `idempotencyKey` (WFL-010).
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** WFL-010 (idempotent recovery, no duplicated outcomes).
- **Pagination:** N/A.

### 15. Bulk Recover Orphaned Instances
- **Method:** `POST`
- **URL:** `/api/v1/workflow-instances/orphans/bulk-recover`
- **Description:** Recover many stuck instances at once.
- **Authentication (Role-Based):** Yes — `workflow.instance.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "instanceIds": {
          "type": "array",
          "items": {
            "type": "string"
          }
        }
      },
      "required": [
        "instanceIds"
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
  "type": "null",
  "description": "202 Accepted { jobId } (per-instance results via GET /api/v1/jobs/{jobId})."
}
```
- **Validation Rules:** Same idempotency guarantee as #19, applied per instance.
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** WFL-010.
- **Pagination:** N/A.

## SLA, Bottleneck & Audit Reporting

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
