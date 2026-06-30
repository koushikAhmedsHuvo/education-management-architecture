# Role & Permission API

## 1. API Overview

**Purpose.** Govern Roles, the Permission Registry, Memberships, Separation-of-Duties constraints, scope-switching, and access governance/reporting.

**Module Context.** Implements Business Rules Catalog Doc 02 (Authorization) and UI Screen Spec `03-role-permission-screens.md` (SCR-ROLE-01…13).

---

## 2. Endpoints

### 1. List Roles
- **Method:** `GET`
- **URL:** `/api/v1/roles`
- **Description:** Browse roles with their permission/member footprint (SCR-ROLE-01).
- **Authentication (Role-Based):** Yes — `authz.role.view`, scoped.
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
    "typeSYSTEMCUSTOM": {
      "type": "string",
      "description": "type? (SYSTEM|CUSTOM)"
    },
    "scopeLevel": {
      "type": "string"
    },
    "statusDRAFTACTIVEDE": {
      "type": "string",
      "description": "status? (DRAFT|ACTIVE|DEPRECATED|ARCHIVED)"
    },
    "sortBynamemembersCou": {
      "type": "string",
      "description": "sortBy? (name|membersCount|updatedAt, default name)"
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
      "type": {
        "type": "string"
      },
      "permissionsCount": {
        "type": "integer"
      },
      "scopeLevels": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "membersCount": {
        "type": "integer"
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
      "type",
      "permissionsCount",
      "scopeLevels",
      "membersCount",
      "status",
      "updatedAt",
      "version"
    ]
  }
}
```
- **Validation Rules:** Results scoped to roles the caller's institute(s) can see (org-wide roles visible to all in-scope viewers).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-001 (deny-by-default), AUTHZ-002 (scope filter), Doc 02 §6 (status values).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Role Detail
- **Method:** `GET`
- **URL:** `/api/v1/roles/{id}`
- **Description:** Permissions (grouped), scope levels, and member list for one role (SCR-ROLE-02).
- **Authentication (Role-Based):** Yes — `authz.role.view`.
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
    "description": {
      "type": "string"
    },
    "type": {
      "type": "string"
    },
    "scopeLevels": {
      "type": "string",
      "description": "scopeLevels[]"
    },
    "permissions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "key": {
            "type": "string"
          },
          "category": {
            "type": "string"
          }
        },
        "required": [
          "key",
          "category"
        ]
      }
    },
    "status": {
      "type": "string"
    },
    "membersCount": {
      "type": "integer"
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
    "name",
    "description",
    "type",
    "permissions",
    "status",
    "membersCount",
    "createdAt",
    "updatedAt",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** AUTHZ-001, AUTHZ-008 (permissions shown reference the registry).
- **Pagination:** N/A.

### 3. Create Role (Draft)
- **Method:** `POST`
- **URL:** `/api/v1/roles`
- **Description:** Compose a new custom role from registered permissions, optionally cloned from an existing role (SCR-ROLE-03, UC-AUTHZ-02).
- **Authentication (Role-Based):** Yes — `authz.role.manage`, scoped.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "description": {
      "type": "string"
    },
    "basis": {
      "type": "string",
      "description": "clone-from roleId"
    },
    "permissions": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "\u22651"
    },
    "scopeLevels": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "\u22651"
    }
  },
  "required": [
    "name",
    "permissions",
    "scopeLevels"
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
    "status": {
      "type": "string",
      "const": "ROLE_DRAFT"
    },
    "permissions": {
      "type": "string",
      "description": "permissions[]"
    },
    "scopeLevels": {
      "type": "string",
      "description": "scopeLevels[]"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "name",
    "status",
    "version"
  ]
}
```
- **Validation Rules:** `name` unique within scope-level; every entry in `permissions` must exist in the Permission Registry (AUTHZ-008) **and** be a permission the creator themselves currently holds in the target scope (AUTHZ-005 — no escalation); `permissions` and `scopeLevels` each require ≥1 entry.
- **Error Codes:** `409 ROLE_NAME_EXISTS`; `422 UNREGISTERED_PERMISSION`; `403 PRIVILEGE_ESCALATION_BLOCKED` ("You can't grant permissions you don't hold").
- **Business Rules Applied:** AUTHZ-005 (no privilege escalation), AUTHZ-007, AUTHZ-008 (registry integrity), Doc 02 §6 (created in `ROLE_DRAFT`).
- **Pagination:** N/A.

### 4. Publish Role
- **Method:** `POST`
- **URL:** `/api/v1/roles/{id}/publish`
- **Description:** Move a role from `ROLE_DRAFT` to `ROLE_ACTIVE`, making it assignable (Doc 02 §6).
- **Authentication (Role-Based):** Yes — `authz.role.manage`.
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
      "const": "ROLE_ACTIVE"
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
- **Validation Rules:** Re-validates permissions ⊆ publisher's grant and ∈ registry (Doc 02 §6 system action) — a role's permission set can drift between draft and publish if the registry or the publisher's own grant changed.
- **Error Codes:** `409 INVALID_STATE_TRANSITION` (not in `ROLE_DRAFT`); `403 PRIVILEGE_ESCALATION_BLOCKED`.
- **Business Rules Applied:** Doc 02 §6 state machine, AUTHZ-005.
- **Pagination:** N/A.

### 5. Edit Role (New Version)
- **Method:** `PUT`
- **URL:** `/api/v1/roles/{id}`
- **Description:** Edit an active custom role; saving creates a new version, not an in-place mutation (SCR-ROLE-04).
- **Authentication (Role-Based):** Yes — `authz.role.manage`; **blocked for `type: SYSTEM` roles** (system roles are read-only/clone-only per the approved UI spec).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "description": {
      "type": "string"
    },
    "scopeLevels": {
      "type": "string"
    },
    "permissions": {
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
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "RoleDetail": {
          "type": "string"
        }
      },
      "required": [
        "RoleDetail"
      ]
    },
    {
      "type": "object",
      "properties": {
        "version": {
          "type": "integer"
        }
      },
      "required": [
        "version"
      ]
    },
    {
      "type": "object",
      "properties": {
        "impactedMemberCount": {
          "type": "integer"
        }
      },
      "required": [
        "impactedMemberCount"
      ]
    }
  ]
}
```
- **Validation Rules:** Same anti-escalation and registry checks as #3; `version` must match current (optimistic lock); permission removal previews member impact before commit is expected to have been confirmed client-side (the API itself proceeds and reports the count).
- **Error Codes:** `403 SYSTEM_ROLE_IMMUTABLE`; `409 VERSION_CONFLICT`; `422 UNREGISTERED_PERMISSION`; `403 PRIVILEGE_ESCALATION_BLOCKED`.
- **Business Rules Applied:** AUTHZ-005, AUTHZ-006 (edit bumps affected users' permissions-version), AUTHZ-008.
- **Pagination:** N/A.

### 6. Deprecate Role
- **Method:** `POST`
- **URL:** `/api/v1/roles/{id}/deprecate`
- **Description:** Retire a role (Doc 02 §6: `ROLE_ACTIVE → ROLE_DEPRECATED`).
- **Authentication (Role-Based):** Yes — `authz.role.manage`; blocked for `SYSTEM` roles.
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
      "const": "ROLE_DEPRECATED"
    },
    "impactedMemberCount": {
      "type": "integer"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "status",
    "impactedMemberCount",
    "version"
  ]
}
```
- **Validation Rules:** Only from `ROLE_ACTIVE`; existing memberships are **not** auto-revoked (Doc 02 §6: "existing assignments handled per policy") — the response surfaces the count so the caller can decide.
- **Error Codes:** `409 INVALID_STATE_TRANSITION`; `403 SYSTEM_ROLE_IMMUTABLE`.
- **Business Rules Applied:** Doc 02 §6.
- **Pagination:** N/A.

## Memberships

### 7. List Memberships
- **Method:** `GET`
- **URL:** `/api/v1/memberships`
- **Description:** Browse (user, scope, role) assignments (SCR-ROLE-05).
- **Authentication (Role-Based):** Yes — `authz.membership.view`, scoped.
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
    "userId": {
      "type": "string"
    },
    "scopeId": {
      "type": "string"
    },
    "roleId": {
      "type": "string"
    },
    "statusACTIVESUSPENDE": {
      "type": "string",
      "description": "status? (ACTIVE|SUSPENDED|REVOKED)"
    },
    "sortBygrantedAtuser": {
      "type": "string",
      "description": "sortBy? (grantedAt|user, default grantedAt)"
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
      "user": {
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
      "role": {
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
      },
      "grantedBy": {
        "type": "string"
      },
      "grantedAt": {
        "type": "string"
      },
      "status": {
        "type": "string"
      },
      "version": {
        "type": "integer"
      }
    },
    "required": [
      "id",
      "user",
      "scope",
      "role",
      "grantedBy",
      "grantedAt",
      "status",
      "version"
    ]
  }
}
```
- **Validation Rules:** Scope-bound results only.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-002, AUTHZ-007 (scoped role assignment authority).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 8. Grant Membership
- **Method:** `POST`
- **URL:** `/api/v1/memberships`
- **Description:** Assign a user a role in a scope, with authority and SoD enforced server-side so a violating grant cannot be constructed (SCR-ROLE-06, UC-AUTHZ-02-adjacent).
- **Authentication (Role-Based):** Yes — `authz.membership.grant`, scoped; the scope/role pickers are pre-filtered server-side to the granter's own authority.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "userId": {
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
    "userId",
    "scope",
    "roleId"
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
        "ACTIVE",
        "PENDING_APPROVAL"
      ]
    },
    "user": {
      "type": "string"
    },
    "scope": {
      "type": "string"
    },
    "role": {
      "type": "string"
    },
    "grantedBy": {
      "type": "string"
    },
    "grantedAt": {
      "type": "string"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "status",
    "user",
    "scope",
    "role",
    "grantedBy",
    "grantedAt",
    "version"
  ]
}
```
- **Validation Rules:** Target `scope` ⊆ granter's own scope authority; `roleId` ⊆ permissions the granter holds in that scope (AUTHZ-005); if an SoD constraint (see #11) would be violated by this grant, the request is rejected outright — it cannot be constructed even partially.
- **Error Codes:** `403 OUT_OF_SCOPE_AUTHORITY`; `403 PRIVILEGE_ESCALATION_BLOCKED`; `403 SOD_VIOLATION_BLOCKED`. High-impact grants (per configured policy) return `202 Accepted` with `status: "PENDING_APPROVAL"` instead of immediate `201`, routed through `12. List Access Change Approvals` below.
- **Business Rules Applied:** AUTHZ-005, AUTHZ-007 (scoped role assignment authority), AUTHZ-009 (SoD hooks), Doc 02 §8 (high-impact grants may require approval).
- **Pagination:** N/A.

### 9. Revoke Membership
- **Method:** `DELETE`
- **URL:** `/api/v1/memberships/{id}`
- **Description:** Immediately end a (user, scope, role) assignment.
- **Authentication (Role-Based):** Yes — `authz.membership.revoke`, scoped.
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
- **Validation Rules:** Allowed from `ACTIVE` or `SUSPENDED`; terminal — Doc 02 §6 forbids `REVOKED → ACTIVE` (a re-grant creates a new membership via #8).
- **Error Codes:** `409 INVALID_STATE_TRANSITION`; `403 FORBIDDEN`.
- **Business Rules Applied:** Doc 02 §6 (membership state machine), AUTHZ-006 (revocation bumps the user's permissions-version immediately).
- **Pagination:** N/A.

### 10. Get My Access (Scope Switcher)
- **Method:** `GET`
- **URL:** `/api/v1/me/access`
- **Description:** List the caller's own memberships and effective permissions, grouped by module, for the scope switcher (SCR-ROLE-07, UC-AUTHZ-03).
- **Authentication (Role-Based):** Yes — self (no permission string; everyone may see their own access).
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
    "scopes": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "instituteId": {
            "type": "string"
          },
          "campusId": {
            "type": "string"
          },
          "roleNames": {
            "type": "string",
            "description": "roleNames[]"
          }
        },
        "required": [
          "instituteId"
        ]
      }
    },
    "activeScope": {
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
    "effectivePermissions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "category": {
            "type": "string"
          },
          "permissions": {
            "type": "array",
            "items": {
              "type": "string"
            }
          }
        },
        "required": [
          "category",
          "permissions"
        ]
      }
    }
  },
  "required": [
    "scopes",
    "activeScope",
    "effectivePermissions"
  ]
}
```
- **Validation Rules:** Returns only scopes backed by a current `ACTIVE` membership.
- **Error Codes:** `401` invalid session.
- **Business Rules Applied:** AUTHZ-001 (effective permissions = union of role permissions in the active scope), Doc 02 §6.
- **Pagination:** N/A.

### 11. Switch Active Scope
- **Method:** `POST`
- **URL:** `/api/v1/me/active-scope`
- **Description:** Change the session's active institute/campus (UC-AUTHZ-03).
- **Authentication (Role-Based):** Yes — self.
- **Request DTO (JSON Schema):**
```json
{
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
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "activeScope": {
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
    "permissionsVersion": {
      "type": "number"
    }
  },
  "required": [
    "activeScope",
    "permissionsVersion"
  ]
}
```
- **Validation Rules:** The requested scope must be backed by a current membership for the caller.
- **Error Codes:** `403 NO_MEMBERSHIP_FOR_SCOPE`.
- **Business Rules Applied:** AUTHZ-002, AUTHZ-006 (permissions re-resolved for the new scope on switch).
- **Pagination:** N/A.

## Permission Registry

### 12. List Permission Registry
- **Method:** `GET`
- **URL:** `/api/v1/permissions`
- **Description:** Browse the catalog of namespaced, code-enforced permission keys (SCR-ROLE-08).
- **Authentication (Role-Based):** Yes — `authz.registry.manage` for the management view; read access is part of the same screen per the approved spec ("Others read-only/hidden" implies registry visibility itself is gated to `authz.registry.manage`, with no separate broader view permission named).
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
    "category": {
      "type": "string"
    },
    "statusREGISTEREDDEPR": {
      "type": "string",
      "description": "status? (REGISTERED|DEPRECATED)"
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
      "category": {
        "type": "string"
      },
      "description": {
        "type": "string"
      },
      "rolesUsingCount": {
        "type": "integer"
      },
      "status": {
        "type": "string"
      }
    },
    "required": [
      "key",
      "category",
      "description",
      "rolesUsingCount",
      "status"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-008 (registry integrity).
- **Pagination:** Yes — default `page=1, pageSize=25`, grouped/sorted by `category`.

### 13. Register Permission
- **Method:** `POST`
- **URL:** `/api/v1/permissions`
- **Description:** Register a new code-enforced permission before any role can reference it.
- **Authentication (Role-Based):** Yes — `authz.registry.manage` (platform-level).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "key": {
      "type": "string",
      "description": "namespaced \"entity.action\", unique"
    },
    "description": {
      "type": "string"
    },
    "category": {
      "type": "string"
    }
  },
  "required": [
    "key",
    "description",
    "category"
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
    "description": {
      "type": "string"
    },
    "category": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "REGISTERED"
    }
  },
  "required": [
    "key",
    "description",
    "category",
    "status"
  ]
}
```
- **Validation Rules:** `key` unique and matches the `entity.action` namespace format.
- **Error Codes:** `409 KEY_ALREADY_EXISTS`.
- **Business Rules Applied:** AUTHZ-008.
- **Pagination:** N/A.

### 14. Deprecate Permission
- **Method:** `POST`
- **URL:** `/api/v1/permissions/{key}/deprecate`
- **Description:** Retire a permission key — **deprecate-not-delete** for any key still referenced by a role.
- **Authentication (Role-Based):** Yes — `authz.registry.manage`.
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
    "key": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "DEPRECATED"
    }
  },
  "required": [
    "key",
    "status"
  ]
}
```
- **Validation Rules:** A referenced key cannot be hard-deleted (no delete endpoint is exposed for permissions at all — deprecation is the only removal path).
- **Error Codes:** `409 CANNOT_DELETE_REFERENCED_PERMISSION` is not applicable here since this is deprecate-only; returned only if a hard-delete were attempted via an unsupported path.
- **Business Rules Applied:** AUTHZ-008, UI spec: *"Cannot delete a referenced permission — deprecate instead."*
- **Pagination:** N/A.

## Separation of Duties (SoD)

### 15. List SoD Constraints
- **Method:** `GET`
- **URL:** `/api/v1/sod-constraints`
- **Description:** Browse configured SoD action-pair constraints (SCR-ROLE-09).
- **Authentication (Role-Based):** Yes — `authz.sod.manage`.
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
    "typeMUTUALLY_EXCLUSIV": {
      "type": "string",
      "description": "type? (MUTUALLY_EXCLUSIVE|REQUESTER_NOT_APPROVER)"
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
      "dutyA": {
        "type": "string"
      },
      "dutyB": {
        "type": "string"
      },
      "type": {
        "type": "string"
      },
      "scope": {
        "type": "string"
      },
      "status": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "dutyA",
      "dutyB",
      "type",
      "status"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-009.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 16. Create SoD Constraint
- **Method:** `POST`
- **URL:** `/api/v1/sod-constraints`
- **Description:** Define a new SoD constraint between two duties (permissions or roles).
- **Authentication (Role-Based):** Yes — `authz.sod.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "dutyA": {
      "type": "string"
    },
    "dutyB": {
      "type": "string"
    },
    "type": {
      "type": "string",
      "enum": [
        "MUTUALLY_EXCLUSIVE",
        "REQUESTER_NOT_APPROVER"
      ]
    },
    "scope": {
      "type": "string"
    }
  },
  "required": [
    "dutyA",
    "dutyB",
    "type"
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
    "dutyA": {
      "type": "string"
    },
    "dutyB": {
      "type": "string"
    },
    "type": {
      "type": "string"
    },
    "scope": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "ACTIVE"
    },
    "impactedMembershipCount": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "dutyA",
    "dutyB",
    "type",
    "scope",
    "status",
    "impactedMembershipCount"
  ]
}
```
- **Validation Rules:** `dutyA ≠ dutyB`; must not contradict an existing constraint; if applying it would invalidate existing memberships, the response surfaces the count (the constraint is still created — invalidation is handled per configured policy, not silently).
- **Error Codes:** `422 SELF_CONFLICTING_DUTY`; `409 CONTRADICTS_EXISTING_CONSTRAINT`.
- **Business Rules Applied:** AUTHZ-009 (SoD hooks).
- **Pagination:** N/A.

### 17. Update SoD Constraint
- **Method:** `PUT`
- **URL:** `/api/v1/sod-constraints/{id}`
- **Description:** Edit an existing constraint.
- **Authentication (Role-Based):** Yes — `authz.sod.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "dutyA": {
      "type": "string"
    },
    "dutyB": {
      "type": "string"
    },
    "type": {
      "type": "string"
    },
    "scope": {
      "type": "string"
    },
    "status": {
      "type": "string"
    },
    "version": {
      "type": "integer"
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
  "description": "Updated constraint (as #15).",
  "$ref": "#endpoint-15"
}
```
- **Validation Rules:** Same as #15; `version` optimistic lock.
- **Error Codes:** `409 VERSION_CONFLICT`; `422 SELF_CONFLICTING_DUTY`.
- **Business Rules Applied:** AUTHZ-009.
- **Pagination:** N/A.

## Access Reporting

### 18. Who Has Permission X
- **Method:** `GET`
- **URL:** `/api/v1/access/who-has`
- **Description:** Find every holder (user, via which role, in which scope) of a given permission (SCR-ROLE-10).
- **Authentication (Role-Based):** Yes — `authz.role.view` (matrix-view class, per the approved spec).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "permissionKey": {
      "type": "string",
      "description": "required"
    },
    "scope": {
      "type": "string"
    },
    "holderTypeUSERROLE": {
      "type": "string",
      "description": "holderType? (USER|ROLE)"
    },
    "page": {
      "type": "integer"
    },
    "pageSize": {
      "type": "integer"
    }
  },
  "required": [
    "permissionKey"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "holder": {
        "type": "object",
        "properties": {
          "type": {
            "type": "string"
          },
          "id": {
            "type": "string"
          },
          "name": {
            "type": "string"
          }
        },
        "required": [
          "type",
          "id",
          "name"
        ]
      },
      "viaRole": {
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
      },
      "scope": {
        "type": "string"
      },
      "status": {
        "type": "string"
      }
    },
    "required": [
      "holder",
      "viaRole",
      "scope",
      "status"
    ]
  }
}
```
- **Validation Rules:** `permissionKey` must exist in the registry; results limited to the caller's own authority.
- **Error Codes:** `422 UNKNOWN_PERMISSION_KEY`.
- **Business Rules Applied:** AUTHZ-001, AUTHZ-002 (results scope-limited to viewer authority).
- **Pagination:** Yes — default `page=1, pageSize=25`; large queries run async per conventions §9 (`202` + `jobId`) above a configured row threshold.

### 19. Access Matrix Report
- **Method:** `GET`
- **URL:** `/api/v1/access/matrix`
- **Description:** Subject × Permission × Scope matrix with SoD-conflict highlighting (SCR-ROLE-11).
- **Authentication (Role-Based):** Yes — matrix-view access for read; see #19 for export.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "scope": {
      "type": "string"
    },
    "role": {
      "type": "string"
    },
    "subject": {
      "type": "string"
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
  "type": "object",
  "properties": {
    "subjects": {
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
          "type": {
            "type": "string"
          }
        },
        "required": [
          "id",
          "name",
          "type"
        ]
      }
    },
    "permissions": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "grants": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "subjectId": {
            "type": "string"
          },
          "permissionKey": {
            "type": "string"
          },
          "granted": {
            "type": "boolean"
          },
          "sodConflict": {
            "type": "boolean"
          }
        },
        "required": [
          "subjectId",
          "permissionKey",
          "granted",
          "sodConflict"
        ]
      }
    }
  },
  "required": [
    "subjects",
    "permissions",
    "grants"
  ]
}
```
- **Validation Rules:** Scope-limited to caller's authority.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-001, AUTHZ-002, AUTHZ-009 (SoD-conflict cells flagged).
- **Pagination:** Yes — subjects paginated, default `page=1, pageSize=25`; permissions/grants returned for the current page of subjects.

### 20. Export Access Matrix
- **Method:** `GET`
- **URL:** `/api/v1/access/matrix/export`
- **Description:** Governed, audited export of the access matrix (signed URL).
- **Authentication (Role-Based):** Yes — `authz.matrix.export`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "description": "Same filters as #18.",
  "$ref": "#endpoint-18"
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
- **Validation Rules:** Same as #18; export itself is audited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-002, conventions §10.
- **Pagination:** N/A.

### 21. Export Access Audit
- **Method:** `GET`
- **URL:** `/api/v1/access/audit/export`
- **Description:** Governed export of the access-change audit trail (SCR-ROLE-13).
- **Authentication (Role-Based):** Yes — `authz.matrix.export` / audit-export authority, scope-limited (per the approved spec).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "dateFrom": {
      "type": "string"
    },
    "dateTo": {
      "type": "string"
    },
    "scope": {
      "type": "string"
    },
    "subject": {
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
- **Validation Rules:** Scope-limited results only.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** Doc 02 §11 (audit requirements — `MEMBERSHIP_GRANTED/REVOKED`, `PRIVILEGE_ESCALATION_BLOCKED`, `SOD_VIOLATION_BLOCKED`, etc.), conventions §10.
- **Pagination:** N/A.

## Access Change Approvals

### 22. List Pending Access Change Approvals
- **Method:** `GET`
- **URL:** `/api/v1/access/approvals`
- **Description:** Inbox of high-impact membership/role changes awaiting decision (SCR-ROLE-12).
- **Authentication (Role-Based):** Yes — `authz.change.approve`, scoped.
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
      "description": "status? (PENDING|APPROVED|REJECTED|RETURNED)"
    },
    "scope": {
      "type": "string"
    },
    "sortByslarequestedAt": {
      "type": "string",
      "description": "sortBy? (sla|requestedAt, default sla)"
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
      "requestType": {
        "type": "string"
      },
      "subjectUser": {
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
      "change": {
        "type": "object",
        "properties": {
          "description": {
            "type": "string"
          }
        },
        "required": [
          "description"
        ]
      },
      "requestedBy": {
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
      "id",
      "requestType",
      "subjectUser",
      "change",
      "requestedBy",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Scope-limited; requests the caller themselves raised are still listed (visible) but cannot be decided by them (see #22).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-009 (SoD: requester ≠ approver), Doc 02 §8.
- **Pagination:** Yes — default `page=1, pageSize=25`, sorted by SLA due date ascending.

### 23. Decide Access Change Request
- **Method:** `POST`
- **URL:** `/api/v1/access/approvals/{id}/decide`
- **Description:** Approve, reject, or return a pending access-change request.
- **Authentication (Role-Based):** Yes — `authz.change.approve`, scoped; **blocked if the caller is the request's own requester** (AUTHZ-009, enforced server-side regardless of what the UI hides).
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
  "oneOf": [
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
            "REJECTED",
            "RETURNED"
          ]
        }
      },
      "required": [
        "id",
        "status"
      ]
    },
    {
      "type": "object",
      "properties": {
        "APPROVE": {
          "type": "string"
        }
      },
      "required": [
        "APPROVE"
      ]
    }
  ]
}
```
- **Validation Rules:** `reason` required for `REJECT`/`RETURN`; caller ≠ requester (SoD hook).
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED` ("You raised this request — a different approver is required"); `422 REASON_REQUIRED`; `409 ALREADY_DECIDED`.
- **Business Rules Applied:** AUTHZ-009 (Separation of Duties), Doc 02 §10 (notification on decision).
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
