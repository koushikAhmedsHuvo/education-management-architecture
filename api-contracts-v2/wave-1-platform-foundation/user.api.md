# User API

## 1. API Overview

**Purpose.** Manage the User account lifecycle — invitation, activation, deactivation, suspension, unlock, MFA reset, and session control — distinct from role/permission assignment.

**Module Context.** Implements Business Rules Catalog Doc 01 (Authentication) §6 state machine and §9 permission rules, and UI Screen Spec `02-user-management-screens.md` (SCR-USER-01…04).

---

## 2. Endpoints

### 1. List Users
- **Method:** `GET`
- **URL:** `/api/v1/users`
- **Description:** Scope-filtered user directory (SCR-USER-01).
- **Authentication (Role-Based):** Yes — `auth.user.view`, scoped to the caller's active institute/campus (AUTHZ-002).
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
    "qsearchnameemail": {
      "type": "string",
      "description": "q? (search name/email)"
    },
    "statuscommaseparated": {
      "type": "string",
      "description": "status? (comma-separated: INVITED|ACTIVE|LOCKED|SUSPENDED|DEACTIVATED)"
    },
    "roleId": {
      "type": "string"
    },
    "scopeId": {
      "type": "string"
    },
    "mfaEnabledboolean": {
      "type": "string",
      "description": "mfaEnabled? (boolean)"
    },
    "lastLoginFrom": {
      "type": "string"
    },
    "lastLoginTo": {
      "type": "string"
    },
    "sortBynamelastLoginA": {
      "type": "string",
      "description": "sortBy? (name|lastLoginAt|status, default name)"
    },
    "sortOrder": {
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
      "displayName": {
        "type": "string"
      },
      "email": {
        "type": "string"
      },
      "scope": {
        "type": "object",
        "properties": {
          "instituteId": {
            "type": "string"
          },
          "campusId": {
            "type": "string"
          }
        },
        "required": [
          "instituteId"
        ]
      },
      "roles": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "id": {
              "type": "string"
            },
            "name": {
              "type": "string"
            }
          },
          "required": [
            "id",
            "name"
          ]
        }
      },
      "status": {
        "type": "string"
      },
      "lastLoginAt": {
        "type": "string"
      },
      "mfaEnabled": {
        "type": "string"
      },
      "createdAt": {
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
      "displayName",
      "email",
      "scope",
      "roles",
      "status",
      "mfaEnabled",
      "createdAt",
      "updatedAt",
      "version"
    ]
  }
}
```
- **Validation Rules:** `status` values must be valid User states; rows outside the caller's scope are never returned (not filtered client-side).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-002 (mandatory scope filter), AUTHZ-003 (ownership/relationship predicates), Doc 01 §6 (status values).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get User Detail
- **Method:** `GET`
- **URL:** `/api/v1/users/{id}`
- **Description:** Full profile, security state, and session summary for one user (SCR-USER-02).
- **Authentication (Role-Based):** Yes — `auth.user.view` in the target's scope, **or** self.
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
    "displayName": {
      "type": "string"
    },
    "email": {
      "type": "string"
    },
    "scope": {
      "type": "string"
    },
    "roles": {
      "type": "string",
      "description": "roles[]"
    },
    "status": {
      "type": "string"
    },
    "mfaEnabled": {
      "type": "string"
    },
    "activeSessionCount": {
      "type": "integer"
    },
    "lastLoginAt": {
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
    "displayName",
    "email",
    "scope",
    "status",
    "mfaEnabled",
    "activeSessionCount",
    "createdAt",
    "updatedAt",
    "createdBy",
    "updatedBy",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access checks.
- **Error Codes:** `404 NOT_FOUND` (out-of-scope target — never reveals existence, AUTHZ-002).
- **Business Rules Applied:** AUTHZ-002, AUTHZ-003.
- **Pagination:** N/A.

### 3. Invite User
- **Method:** `POST`
- **URL:** `/api/v1/users/invite`
- **Description:** Create a user in `INVITED` state and dispatch an activation token (SCR-USER-03, AUTH-011).
- **Authentication (Role-Based):** Yes — `auth.user.invite`, scoped; the scope and role pickers are pre-constrained server-side to scopes the inviter governs and roles within the inviter's own grant (AUTHZ-005, AUTHZ-007).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "email": {
      "type": "string"
    },
    "displayName": {
      "type": "string"
    },
    "initialScope": {
      "type": "object",
      "properties": {
        "instituteId": {
          "type": "string"
        },
        "campusId": {
          "type": "string"
        }
      },
      "required": [
        "instituteId"
      ]
    },
    "initialRoleId": {
      "type": "string"
    }
  },
  "required": [
    "email",
    "displayName",
    "initialScope",
    "initialRoleId"
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
    "displayName": {
      "type": "string"
    },
    "email": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "INVITED"
    },
    "scope": {
      "type": "string"
    },
    "roles": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": {
            "type": "string"
          },
          "name": {
            "type": "string"
          }
        },
        "required": [
          "id",
          "name"
        ]
      }
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
    "displayName",
    "email",
    "status",
    "scope",
    "roles",
    "createdAt",
    "version"
  ]
}
```
- **Validation Rules:** `email` unique among non-deactivated users; `initialScope` ⊆ inviter's own scope authority; `initialRoleId` ⊆ permissions the inviter themselves holds in that scope (no escalation).
- **Error Codes:** `409 EMAIL_ALREADY_EXISTS`; `403 OUT_OF_SCOPE` ("You can't invite to this scope"); `403 PRIVILEGE_ESCALATION_BLOCKED` ("Role exceeds your grant authority").
- **Business Rules Applied:** AUTH-011 (invitation-based activation), AUTHZ-005 (no privilege escalation), AUTHZ-007 (scoped role assignment authority).
- **Pagination:** N/A.

### 4. Bulk Invite Users (CSV)
- **Method:** `POST`
- **URL:** `/api/v1/users/bulk-invite`
- **Description:** Invite many users from a validated CSV-derived batch (SCR-USER-04).
- **Authentication (Role-Based):** Yes — `auth.user.invite`, scoped.
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
              "email": {
                "type": "string"
              },
              "displayName": {
                "type": "string"
              },
              "scope": {
                "type": "object",
                "properties": {
                  "instituteId": {
                    "type": "string"
                  },
                  "campusId": {
                    "type": "string"
                  }
                },
                "required": [
                  "instituteId"
                ]
              },
              "roleId": {
                "type": "string"
              }
            },
            "required": [
              "email",
              "displayName",
              "scope",
              "roleId"
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
              "userId": {
                "type": "string"
              }
            },
            "required": [
              "row",
              "userId"
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
    },
    {
      "type": "object",
      "properties": {
        "GETapiv1jobsjobId": {
          "type": "string",
          "description": "GET /api/v1/jobs/{jobId}"
        }
      }
    }
  ]
}
```
- **Validation Rules:** Each row validated independently against the same rules as #3 (unique email, scope/role authority); out-of-scope or over-authority rows are flagged invalid and **never dispatched**, but always appear in the result — no silent drops.
- **Error Codes:** `400 MALFORMED_BATCH` (schema-level failure on the whole payload); individual row failures are reported in `rejected[]`, not as a top-level error.
- **Business Rules Applied:** AUTH-011, AUTHZ-005, AUTHZ-007; conventions §9 (async/idempotent bulk pattern).
- **Pagination:** N/A (job-status polling has no pagination; `rejected[]` is bounded by batch size).

### 5. Update User Profile
- **Method:** `PATCH`
- **URL:** `/api/v1/users/{id}`
- **Description:** Update the user's display name.
- **Authentication (Role-Based):** Yes — self, or an `auth.user.invite`-holder in scope (display-name correction is treated as a management edit; gated by `auth.user.invite` since no separate "edit" permission is named in the approved rules).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "displayName": {
      "type": "string"
    },
    "version": {
      "type": "number"
    }
  },
  "required": [
    "displayName",
    "version"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "description": "Updated user summary (as #2) .",
  "$ref": "#endpoint-2"
}
```
- **Validation Rules:** `displayName` non-empty; `version` must match current.
- **Error Codes:** `409 VERSION_CONFLICT`; `403 FORBIDDEN`.
- **Business Rules Applied:** Conventions §8 (optimistic concurrency). *(Email change is not specified by the approved rules and is intentionally out of scope for this contract.)*
- **Pagination:** N/A.

### 6. Deactivate User
- **Method:** `POST`
- **URL:** `/api/v1/users/{id}/deactivate`
- **Description:** Offboard a user (state → `DEACTIVATED`); blocks authentication immediately.
- **Authentication (Role-Based):** Yes — `auth.user.deactivate`, scoped to the target user's scope (AUTH §9).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "reason": {
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
    "id": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "DEACTIVATED"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "status",
    "version"
  ]
}
```
- **Validation Rules:** Allowed only from `ACTIVE`, `LOCKED`, or `SUSPENDED` (Doc 01 §6 state machine).
- **Error Codes:** `409 INVALID_STATE_TRANSITION`; `403 FORBIDDEN` (out of scope).
- **Business Rules Applied:** Doc 01 §6 (state machine), AUTH-005 (immediate session revocation cascades on deactivation).
- **Pagination:** N/A.

### 7. Reactivate User
- **Method:** `POST`
- **URL:** `/api/v1/users/{id}/reactivate`
- **Description:** Restore access for a `DEACTIVATED` user via a fresh, explicit, audited activation — **not** a silent state flip.
- **Authentication (Role-Based):** Yes — `auth.user.reactivate`, scoped.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "justification": {
      "type": "string"
    }
  },
  "required": [
    "justification"
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
      "const": "INVITED"
    },
    "invitationReissued": {
      "type": "boolean",
      "const": true
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "status",
    "invitationReissued",
    "version"
  ]
}
```
- **Validation Rules:** Only allowed from `DEACTIVATED`; `justification` required.
- **Error Codes:** `409 INVALID_STATE_TRANSITION` ("Direct reactivation is forbidden; a fresh activation is issued instead").
- **Business Rules Applied:** Doc 01 §6 — *"Forbidden Transitions: → ACTIVE (reactivation requires a new explicit admin action creating a fresh activation, audited) — direct silent reactivation forbidden."* This endpoint implements that literally: it reissues an AUTH-011 invitation rather than setting `ACTIVE` directly.
- **Pagination:** N/A.

### 8. Suspend User
- **Method:** `POST`
- **URL:** `/api/v1/users/{id}/suspend`
- **Description:** Administratively disable a user (investigation, leave) while preserving the account for later reinstatement.
- **Authentication (Role-Based):** Yes — `auth.user.deactivate` (the approved rules do not name a distinct suspend-permission; suspend and deactivate share the same management capability per Doc 01 §9).
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
      "const": "SUSPENDED"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "status",
    "version"
  ]
}
```
- **Validation Rules:** Allowed only from `ACTIVE` or `LOCKED`.
- **Error Codes:** `409 INVALID_STATE_TRANSITION`.
- **Business Rules Applied:** Doc 01 §6 state machine, AUTH-005.
- **Pagination:** N/A.

### 9. Reinstate Suspended User
- **Method:** `POST`
- **URL:** `/api/v1/users/{id}/reinstate`
- **Description:** Return a `SUSPENDED` user to `ACTIVE`.
- **Authentication (Role-Based):** Yes — `auth.user.deactivate`-equivalent management permission, scoped.
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
    "id": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "ACTIVE"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "status",
    "version"
  ]
}
```
- **Validation Rules:** Allowed only from `SUSPENDED` (Doc 01 §6: `SUSPENDED → ACTIVE` is allowed, unlike `DEACTIVATED → ACTIVE`).
- **Error Codes:** `409 INVALID_STATE_TRANSITION`.
- **Business Rules Applied:** Doc 01 §6.
- **Pagination:** N/A.

### 10. Unlock User
- **Method:** `POST`
- **URL:** `/api/v1/users/{id}/unlock`
- **Description:** Admin-unlock a `LOCKED` account before its cooldown expires — a single privileged, audited action with no multi-step approval (Doc 01 §8).
- **Authentication (Role-Based):** Yes — `auth.user.unlock`.
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
    "id": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "ACTIVE"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "status",
    "version"
  ]
}
```
- **Validation Rules:** Allowed only from `LOCKED`.
- **Error Codes:** `409 INVALID_STATE_TRANSITION` (not currently locked).
- **Business Rules Applied:** AUTH-004 (progressive lockout), Doc 01 §8 (admin unlock is a single privileged action).
- **Pagination:** N/A.

### 11. Reset User's MFA
- **Method:** `POST`
- **URL:** `/api/v1/users/{id}/mfa/reset`
- **Description:** Admin-assisted MFA re-enrollment after a lost device and exhausted recovery codes — clears the user's enrollment so they re-enroll their own new factor; **never sets a TOTP secret on the admin's behalf** (Doc 01 §13 edge case).
- **Authentication (Role-Based):** Yes — `auth.user.mfa.reset`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "justification": {
      "type": "string"
    }
  },
  "required": [
    "justification"
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
    "mfaEnabled": {
      "type": "boolean",
      "const": false
    }
  },
  "required": [
    "id",
    "mfaEnabled"
  ]
}
```
- **Validation Rules:** None beyond permission/scope.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTH-007 (edge case: lost MFA device → admin-assisted re-enrollment, never admin-sets-TOTP).
- **Pagination:** N/A.

### 12. Force Logout
- **Method:** `POST`
- **URL:** `/api/v1/users/{id}/force-logout`
- **Description:** Admin-forced logout — revokes all of the target user's sessions immediately (Doc 01 AUTH-005 example scenario).
- **Authentication (Role-Based):** Yes — `auth.session.revoke`, scoped.
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
    "revokedCount": {
      "type": "number"
    }
  },
  "required": [
    "revokedCount"
  ]
}
```
- **Validation Rules:** None beyond permission/scope.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTH-005 (immediate revocation on admin-forced logout), AUTH-006.
- **Pagination:** N/A.

### 13. List a User's Sessions (Admin)
- **Method:** `GET`
- **URL:** `/api/v1/users/{id}/sessions`
- **Description:** View another user's active sessions (admin equivalent of the self-service screen, SCR-AUTH-08 note).
- **Authentication (Role-Based):** Yes — `auth.session.revoke`-class permission (Doc 01 AUTH-006: "Viewing another user's sessions/devices requires `auth.session.manage` in scope"; this contract exposes the read under the same governing capability as #12's write, since the approved screens name a single combined permission for the Security tab's session actions).
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
  "type": "object",
  "properties": {
    "01authenticationapimd": {
      "type": "string",
      "description": "01-authentication-api.md"
    }
  }
}
```
- **Validation Rules:** Target must be in caller's scope.
- **Error Codes:** `403 FORBIDDEN`; `404` out-of-scope.
- **Business Rules Applied:** AUTH-006.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 14. Revoke a User's Session (Admin)
- **Method:** `DELETE`
- **URL:** `/api/v1/users/{id}/sessions/{sessionId}`
- **Description:** Revoke one specific session of a target user.
- **Authentication (Role-Based):** Yes — `auth.session.revoke`, scoped.
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
- **Validation Rules:** `sessionId` must belong to `{id}`, and `{id}` must be in scope.
- **Error Codes:** `403 FORBIDDEN`; `404` not found / out of scope.
- **Business Rules Applied:** AUTH-005, AUTH-006.
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
