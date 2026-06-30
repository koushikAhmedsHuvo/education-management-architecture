# Settings API

## 1. API Overview

**Purpose.** The generic Configuration Engine — a definition registry, typed validation, scoped most-specific-wins resolution, immutable versioning, effective-dating, governed rollback, cache propagation, secret protection, and configuration audit/drift reporting.

**Module Context.** Implements Business Rules Catalog Doc 27 (Configuration Engine) and UI Screen Spec `28-configuration-engine-screens.md` (SCR-CFG-01…06, 08…15). Custom-field schema management is split out to `custom-fields.api.md` in this same wave.

---

## 2. Endpoints

### 1. List Definitions
- **Method:** `GET`
- **URL:** `/api/v1/config/definitions`
- **Description:** Browse the catalog of registered, configurable setting definitions (SCR-CFG-01).
- **Authentication (Role-Based):** Yes — `config.view`, scoped (non-sensitive definitions only — sensitive-flagged definitions' metadata is visible but their values are never included here, CFG-009).
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
    "statusREGISTEREDDEPR": {
      "type": "string",
      "description": "status? (REGISTERED|DEPRECATED)"
    },
    "sensitiveboolean": {
      "type": "string",
      "description": "sensitive? (boolean)"
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
      "dataType": {
        "type": "string"
      },
      "allowedScopeLevels": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "overridable": {
        "type": "boolean"
      },
      "sensitive": {
        "type": "boolean"
      },
      "immutableAfterUse": {
        "type": "boolean"
      },
      "highImpact": {
        "type": "boolean"
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
      "dataType",
      "allowedScopeLevels",
      "overridable",
      "sensitive",
      "immutableAfterUse",
      "highImpact",
      "status",
      "updatedAt"
    ]
  }
}
```
- **Validation Rules:** None beyond access; sensitive definitions show metadata flags only, never any stored value.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CFG-001 (definition registry — no undeclared configuration), CFG-009 (sensitive metadata, not value).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Definition Detail
- **Method:** `GET`
- **URL:** `/api/v1/config/definitions/{key}`
- **Description:** Full definition — type, range/enum, default, allowed scopes, flags.
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
    "key": {
      "type": "string"
    },
    "dataType": {
      "type": "string"
    },
    "allowedValues": {
      "type": "string"
    },
    "range": {
      "type": "string"
    },
    "default": {
      "type": "string"
    },
    "allowedScopeLevels": {
      "type": "string",
      "description": "allowedScopeLevels[]"
    },
    "overridable": {
      "type": "string"
    },
    "sensitive": {
      "type": "string"
    },
    "immutableAfterUse": {
      "type": "string"
    },
    "highImpact": {
      "type": "string"
    },
    "status": {
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
    "key",
    "dataType",
    "default",
    "overridable",
    "sensitive",
    "immutableAfterUse",
    "highImpact",
    "status",
    "createdAt",
    "updatedAt",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** CFG-001, CFG-002 (type/range/enum shape exposed for client-side validation hints).
- **Pagination:** N/A.

### 3. Register Definition
- **Method:** `POST`
- **URL:** `/api/v1/config/definitions`
- **Description:** Declare a new configurable setting before any value can be set (UC-CFG-01).
- **Authentication (Role-Based):** Yes — `config.definition.manage` (platform; tightly held, per Doc 27 §9 — registers system settings, distinct from institute-level custom-field schemas, see §"Dynamic Custom-Field Schema" below).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "key": {
      "type": "string",
      "description": "unique, namespaced"
    },
    "dataType": {
      "type": "string",
      "enum": [
        "string",
        "number",
        "boolean",
        "enum",
        "date",
        "json"
      ]
    },
    "allowedValues": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "range": {
      "type": "object",
      "properties": {
        "min": {
          "type": "string"
        },
        "max": {
          "type": "string"
        }
      }
    },
    "default": {
      "type": "object"
    },
    "allowedScopeLevels": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "overridable": {
      "type": "boolean"
    },
    "sensitive": {
      "type": "boolean"
    },
    "immutableAfterUse": {
      "type": "boolean"
    },
    "highImpact": {
      "type": "boolean"
    }
  },
  "required": [
    "key",
    "dataType",
    "default",
    "allowedScopeLevels",
    "overridable"
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
    "status": {
      "type": "string",
      "const": "REGISTERED"
    },
    "createdAt": {
      "type": "string"
    }
  },
  "required": [
    "key",
    "status",
    "createdAt"
  ]
}
```
- **Validation Rules:** `key` unique and complete (every required field present — CFG-001 "definitions complete before use"); `default` must itself conform to `dataType`/`allowedValues`/`range`.
- **Error Codes:** `409 DEFINITION_KEY_EXISTS`; `422 INCOMPLETE_DEFINITION`.
- **Business Rules Applied:** CFG-001 (no undeclared configuration).
- **Pagination:** N/A.

### 4. Deprecate Definition
- **Method:** `POST`
- **URL:** `/api/v1/config/definitions/{key}/deprecate`
- **Description:** Retire a definition — stops new use, retains it for historical resolution (SCR-CFG-10).
- **Authentication (Role-Based):** Yes — `config.definition.manage`.
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
- **Validation Rules:** A deprecated definition is never hard-deleted (Doc 27 §6: "Forbidden Transitions: edits/hard-delete — consumers may depend on history"); existing values remain resolvable for historical/effective-dated queries.
- **Error Codes:** `409 INVALID_STATE_TRANSITION` (already deprecated).
- **Business Rules Applied:** CFG-004 (history preserved), Doc 27 §6 state machine.
- **Pagination:** N/A.

## Resolution & Browser

### 5. Resolve a Value
- **Method:** `GET`
- **URL:** `/api/v1/config/resolve`
- **Description:** Deterministically resolve the effective value of a key for a scope (and optional effective date), per the most-specific-wins chain (UC-CFG-03).
- **Authentication (Role-Based):** Yes — `config.view`, scoped (returns a masked indicator, never the raw value, if the definition is `sensitive` and the caller lacks `config.sensitive.manage`).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "key": {
      "type": "string",
      "description": "required"
    },
    "instituteId": {
      "type": "string"
    },
    "campusId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "effectiveDatedefault": {
      "type": "string",
      "description": "effectiveDate? (default now)"
    }
  },
  "required": [
    "key"
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
    "value": {
      "type": "object"
    },
    "resolvedAtScope": {
      "type": "string"
    },
    "version": {
      "type": "number"
    },
    "effectiveFrom": {
      "type": "string"
    }
  },
  "required": [
    "key",
    "value",
    "resolvedAtScope",
    "version",
    "effectiveFrom"
  ]
}
```
- **Validation Rules:** `key` must be registered (CFG-001); if no value is set at any applicable scope, the definition's `default` is returned — resolution is never null/ambiguous (CFG-003).
- **Error Codes:** `422 UNREGISTERED_KEY`.
- **Business Rules Applied:** CFG-003 (scoped values & most-specific-wins resolution), CFG-005 (effective-dating).
- **Pagination:** N/A.

### 6. Configuration Browser (List Resolved Values)
- **Method:** `GET`
- **URL:** `/api/v1/config`
- **Description:** Browse resolved configuration across keys for a scope, grouped by category — the backing list for the Configuration Browser screen (SCR-CFG-02).
- **Authentication (Role-Based):** Yes — `config.view`, scoped.
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
    "instituteId": {
      "type": "string"
    },
    "campusId": {
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
      "key": {
        "type": "string"
      },
      "value": {
        "type": "object",
        "description": "masked if sensitive and unauthorized"
      },
      "resolvedAtScope": {
        "type": "string"
      },
      "version": {
        "type": "integer"
      },
      "status": {
        "type": "string"
      }
    },
    "required": [
      "key",
      "value",
      "resolvedAtScope",
      "version",
      "status"
    ]
  }
}
```
- **Validation Rules:** Same masking rule as #5.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CFG-003, CFG-009.
- **Pagination:** Yes — default `page=1, pageSize=25`.

## Set Value, Versioning & Rollback

### 7. Set Value (Versioned Save)
- **Method:** `POST`
- **URL:** `/api/v1/config/values`
- **Description:** Set or change a configuration value at a scope, publishing a new immutable version — the engine's central write operation (SCR-CFG-03, UC-CFG-02). An `effectiveDate` in the future implements the "Effective-Dated Change" flow (SCR-CFG-04) as the same operation, not a separate endpoint.
- **Authentication (Role-Based):** Yes — `config.value.manage`, scoped to the target's allowed scope levels.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "definitionKey": {
      "type": "string"
    },
    "value": {
      "type": "object"
    },
    "scope": {
      "type": "object",
      "properties": {
        "instituteId": {
          "type": "string"
        },
        "campusId": {
          "type": "string"
        },
        "sectionId": {
          "type": "string"
        }
      }
    },
    "effectiveDate": {
      "type": "string",
      "description": "default now"
    }
  },
  "required": [
    "definitionKey",
    "value",
    "scope"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "definitionKey": {
      "type": "string"
    },
    "value": {
      "type": "string"
    },
    "scope": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "enum": [
        "PUBLISHED",
        "PENDING_APPROVAL"
      ]
    },
    "version": {
      "type": "integer"
    },
    "effectiveFrom": {
      "type": "string"
    }
  },
  "required": [
    "definitionKey",
    "value",
    "scope",
    "status",
    "version",
    "effectiveFrom"
  ]
}
```
- **Validation Rules:** `definitionKey` must be registered ("Unregistered key", CFG-001); `value` typed/range/enum/required-valid against the definition, including any registered cross-field consistency hook ("Value fails the definition's type/range", CFG-002); `scope` must be among the definition's `allowedScopeLevels` (CFG-003); if `immutableAfterUse = true` and dependent data already exists, the write is blocked outright ("This setting is locked after first use", CFG-007); `effectiveDate` periods must not overlap an existing scheduled version for the same key/scope (CFG-005); if the definition is `highImpact`, the write does not publish immediately — it returns `PENDING_APPROVAL` and routes to `23. List Change Approvals` below.
- **Error Codes:** `422 UNREGISTERED_KEY`; `422 VALUE_VALIDATION_FAILED`; `409 SCOPE_NOT_ALLOWED`; `409 IMMUTABLE_AFTER_USE_LOCKED`; `409 EFFECTIVE_PERIOD_OVERLAP`.
- **Business Rules Applied:** CFG-001, CFG-002, CFG-003, CFG-004 (immutable version on publish), CFG-005, CFG-007 (high-impact governance, immutable-after-use), CFG-008 (cache invalidation/version bump on publish).
- **Pagination:** N/A.

### 8. Get Scheduled (Future-Effective) Versions
- **Method:** `GET`
- **URL:** `/api/v1/config/values/{definitionKey}/scheduled`
- **Description:** List not-yet-active versions queued by their effective date (SCR-CFG-04).
- **Authentication (Role-Based):** Yes — `config.view`, scoped.
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
      "value": {
        "type": "string"
      },
      "effectiveFrom": {
        "type": "string"
      },
      "scheduledBy": {
        "type": "string"
      },
      "scheduledAt": {
        "type": "string"
      }
    },
    "required": [
      "version",
      "value",
      "effectiveFrom",
      "scheduledBy",
      "scheduledAt"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CFG-005 (future-dated changes activate on schedule without manual intervention).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 9. Get Value Version History
- **Method:** `GET`
- **URL:** `/api/v1/config/values/{definitionKey}/history`
- **Description:** Full immutable version history for a key/scope — the basis for reproducibility and for choosing a rollback target.
- **Authentication (Role-Based):** Yes — `config.view`, scoped.
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
      "version": {
        "type": "integer"
      },
      "value": {
        "type": "string"
      },
      "effectiveFrom": {
        "type": "string"
      },
      "effectiveUntil": {
        "type": "string"
      },
      "status": {
        "type": "string",
        "enum": [
          "PUBLISHED",
          "SUPERSEDED",
          "ARCHIVED"
        ]
      },
      "publishedBy": {
        "type": "string"
      },
      "publishedAt": {
        "type": "string"
      }
    },
    "required": [
      "version",
      "value",
      "effectiveFrom",
      "status",
      "publishedBy",
      "publishedAt"
    ]
  }
}
```
- **Validation Rules:** None beyond access; published versions are never edited in place (CFG-004).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CFG-004 (immutable versions retained for reproducibility), CFG-006.
- **Pagination:** Yes — default `page=1, pageSize=25`, sorted `version desc`.

### 10. Governed Rollback
- **Method:** `POST`
- **URL:** `/api/v1/config/values/{definitionKey}/rollback`
- **Description:** Re-publish a prior version as the new current version, with a recorded reason — history is never lost, and rollback is **forward-only** (SCR-CFG-05, UC-CFG-03).
- **Authentication (Role-Based):** Yes — `config.rollback` (elevated); high-impact rollbacks route to approval the same way as #7.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "scope": {
      "type": "object",
      "properties": {
        "instituteId": {
          "type": "string"
        },
        "campusId": {
          "type": "string"
        }
      }
    },
    "targetVersion": {
      "type": "number"
    },
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "scope",
    "targetVersion",
    "reason"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "definitionKey": {
      "type": "string"
    },
    "newVersion": {
      "type": "number"
    },
    "rolledBackFrom": {
      "type": "number"
    },
    "rolledBackTo": {
      "type": "number"
    },
    "status": {
      "type": "string",
      "enum": [
        "PUBLISHED",
        "PENDING_APPROVAL"
      ]
    }
  },
  "required": [
    "definitionKey",
    "newVersion",
    "rolledBackFrom",
    "rolledBackTo",
    "status"
  ]
}
```
- **Validation Rules:** `targetVersion` must be a prior `PUBLISHED`/`SUPERSEDED` version of this key/scope; `reason` required; rollback **never** retroactively alters already-produced dependent records (e.g., invoices issued under the superseded version) — it only changes forward resolution; those records are corrected, if ever needed, through their own governed revision process in their owning module.
- **Error Codes:** `404 VERSION_NOT_FOUND`; `422 REASON_REQUIRED`.
- **Business Rules Applied:** CFG-006 (governed rollback, forward-only effect, history preserved), CFG-007 (approval for high-impact rollback).
- **Pagination:** N/A.

## Cache / Config-Version Management

### 11. Get Cache / Config-Version Status
- **Method:** `GET`
- **URL:** `/api/v1/config/cache-status`
- **Description:** Inspect current config-version stamps per scope, for diagnosing propagation (SCR-CFG-11).
- **Authentication (Role-Based):** Yes — `config.view`.
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
      "scope": {
        "type": "string"
      },
      "currentConfigVersion": {
        "type": "number"
      },
      "lastBumpedAt": {
        "type": "string"
      }
    },
    "required": [
      "scope",
      "currentConfigVersion",
      "lastBumpedAt"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CFG-008 (config-version cache invalidation & fast propagation).
- **Pagination:** N/A.

### 12. Force Cache Invalidation
- **Method:** `POST`
- **URL:** `/api/v1/config/cache/invalidate`
- **Description:** Manually force a cache-invalidation sweep — a rare, platform-level recovery action for suspected propagation inconsistency.
- **Authentication (Role-Based):** Yes — `config.definition.manage` (no distinct permission is named for this operational action in Doc 27 §9; as the rule's only platform-tier permission, and consistent with its framing of platform actions as "tightly held," this contract gates the rare manual sweep behind it rather than the scoped `config.value.manage`).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "scope": {
      "type": "object",
      "properties": {
        "instituteId": {
          "type": "string"
        },
        "campusId": {
          "type": "string"
        }
      }
    }
  }
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "invalidated": {
      "type": "boolean",
      "const": true
    },
    "scope": {
      "type": "string"
    }
  },
  "required": [
    "invalidated",
    "scope"
  ]
}
```
- **Validation Rules:** None beyond access; if the cache layer is unreachable, the system already resolves from source on the next request rather than serving stale data (CFG-008 failure behavior) — this endpoint is a convenience, not a correctness requirement.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CFG-008.
- **Pagination:** N/A.

## Dynamic Custom-Field Schema

### 13. Get Sensitive Configuration (Masked)
- **Method:** `GET`
- **URL:** `/api/v1/config/sensitive/{key}`
- **Description:** Confirm a sensitive value is set without ever exposing it (SCR-CFG-08).
- **Authentication (Role-Based):** Yes — `config.sensitive.manage` (narrowly granted).
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
    "key": {
      "type": "string"
    },
    "isSet": {
      "type": "boolean"
    },
    "lastUpdatedAt": {
      "type": "string"
    },
    "lastUpdatedBy": {
      "type": "string"
    }
  },
  "required": [
    "key",
    "isSet"
  ]
}
```
- **Validation Rules:** The definition must be flagged `sensitive`.
- **Error Codes:** `403 FORBIDDEN`; `422 NOT_A_SENSITIVE_DEFINITION`.
- **Business Rules Applied:** CFG-009 (sensitive configuration protection — encrypted at rest, masked in all reads).
- **Pagination:** N/A.

### 14. Set Sensitive Configuration
- **Method:** `PUT`
- **URL:** `/api/v1/config/sensitive/{key}`
- **Description:** Set or rotate a sensitive value (integration secrets, key references).
- **Authentication (Role-Based):** Yes — `config.sensitive.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "value": {
      "type": "string"
    },
    "scope": {
      "type": "object",
      "properties": {
        "instituteId": {
          "type": "string"
        }
      }
    }
  },
  "required": [
    "value",
    "scope"
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
    "isSet": {
      "type": "boolean",
      "const": true
    },
    "updatedAt": {
      "type": "string"
    }
  },
  "required": [
    "key",
    "isSet",
    "updatedAt"
  ]
}
```
- **Validation Rules:** Same type/scope validation as #7; the value is encrypted at rest and excluded from general config exports (#22) and from audit value-capture — only the fact of the change is audited, never the value.
- **Error Codes:** `422 VALUE_VALIDATION_FAILED`.
- **Business Rules Applied:** CFG-009.
- **Pagination:** N/A.

## Audit & Drift Reporting

### 15. List Configuration Audit Events
- **Method:** `GET`
- **URL:** `/api/v1/config/audit`
- **Description:** Query the immutable configuration change trail (SCR-CFG-12).
- **Authentication (Role-Based):** Yes — `config.view` (named per the established view/manage pairing convention from `00-api-conventions.md` §3, scoped to non-sensitive events; access to sensitive-config audit events specifically requires `config.sensitive.manage`).
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
    "dateFrom": {
      "type": "string"
    },
    "dateTo": {
      "type": "string"
    },
    "eventTypecommasepara": {
      "type": "string",
      "description": "eventType? (comma-separated)"
    },
    "key": {
      "type": "string"
    }
  }
}
```
- **Response DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": {
            "type": "string"
          },
          "eventType": {
            "type": "string"
          },
          "definitionKey": {
            "type": "string"
          },
          "scope": {
            "type": "string"
          },
          "actor": {
            "type": "string"
          },
          "before": {
            "type": "string"
          },
          "after": {
            "type": "string"
          },
          "version": {
            "type": "integer"
          },
          "timestamp": {
            "type": "string"
          }
        },
        "required": [
          "id",
          "eventType",
          "scope",
          "actor",
          "timestamp"
        ]
      }
    },
    {
      "type": "object",
      "properties": {
        "before": {
          "type": "string"
        }
      },
      "required": [
        "before"
      ]
    },
    {
      "type": "object",
      "properties": {
        "after": {
          "type": "string"
        }
      },
      "required": [
        "after"
      ]
    }
  ]
}
```
- **Validation Rules:** `eventType` restricted to the mandated set (`CONFIG_DEFINITION_REGISTERED`, `CONFIG_VALUE_PUBLISHED`, `CONFIG_VERSION_BUMPED`, `CONFIG_ROLLED_BACK`, `SENSITIVE_CONFIG_ACCESSED`, etc., per Doc 27 §11).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** Doc 27 §11 (audit requirements), CFG-009 (sensitive values never in audit).
- **Pagination:** Yes — default `page=1, pageSize=25`, sorted `timestamp desc`.

### 16. Configuration Drift Report
- **Method:** `GET`
- **URL:** `/api/v1/config/drift-report`
- **Description:** Compare an institute's/campus's current configuration against its seeded type template or a chosen baseline, surfacing where it has diverged (SCR-CFG-12).
- **Authentication (Role-Based):** Yes — `config.view`, scoped.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "instituteId": {
      "type": "string",
      "description": "required"
    },
    "campusId": {
      "type": "string"
    },
    "baseline": {
      "type": "string",
      "enum": [
        "type-template",
        "organization"
      ],
      "description": "default type-template"
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
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "key": {
        "type": "string"
      },
      "baselineValue": {
        "type": "string"
      },
      "currentValue": {
        "type": "string"
      },
      "diverged": {
        "type": "boolean"
      }
    },
    "required": [
      "key",
      "baselineValue",
      "currentValue",
      "diverged"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CFG-011 (institution-type templates seed values, which are then editable — drift is expected and visible, not an error state).
- **Pagination:** Yes — default `page=1, pageSize=25` (drifted keys only, by default).

## Bulk Operations & Export

### 17. Bulk-Set Values (Import)
- **Method:** `POST`
- **URL:** `/api/v1/config/values/bulk`
- **Description:** Set many values at once from an import batch (SCR-CFG-13).
- **Authentication (Role-Based):** Yes — `config.value.manage`, scoped per row.
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
              "definitionKey": {
                "type": "string"
              },
              "value": {
                "type": "string"
              },
              "scope": {
                "type": "string"
              },
              "effectiveDate": {
                "type": "string"
              }
            },
            "required": [
              "definitionKey",
              "value",
              "scope"
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
              "definitionKey": {
                "type": "string"
              },
              "version": {
                "type": "integer"
              }
            },
            "required": [
              "row",
              "definitionKey",
              "version"
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
    }
  ]
}
```
- **Validation Rules:** Each row validated independently against the same rules as #7 (registered key, type/range, allowed scope, immutable-after-use, effective-period non-overlap); invalid rows are excluded and always reported — never silently dropped; high-impact rows within the batch are individually routed to approval rather than blocking the whole batch.
- **Error Codes:** `400 MALFORMED_BATCH`; per-row failures in `rejected[]`.
- **Business Rules Applied:** CFG-001, CFG-002, CFG-003, CFG-007; conventions §9.
- **Pagination:** N/A.

### 18. Export Configuration
- **Method:** `GET`
- **URL:** `/api/v1/config/export`
- **Description:** Governed export of resolved configuration for a scope, with secrets redacted (SCR-CFG-14).
- **Authentication (Role-Based):** Yes — `config.view`, scoped.
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
    },
    "category": {
      "type": "string"
    }
  }
}
```
- **Response DTO (JSON Schema):**
```json
{
  "oneOf": [
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
    },
    {
      "type": "object",
      "properties": {
        "sensitive": {
          "type": "string"
        }
      },
      "required": [
        "sensitive"
      ]
    }
  ]
}
```
- **Validation Rules:** Sensitive values are structurally excluded, never just masked, from the export payload (CFG-009: "excluded from general config exports").
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CFG-009, conventions §10.
- **Pagination:** N/A.

## Change Approvals

### 19. List Pending Configuration Change Approvals
- **Method:** `GET`
- **URL:** `/api/v1/config/approvals`
- **Description:** Inbox of high-impact value changes and rollbacks awaiting decision (SCR-CFG-06, SCR-CFG-15).
- **Authentication (Role-Based):** Yes — `config.value.approve`, scoped.
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
      "type": {
        "type": "string",
        "enum": [
          "SET_VALUE",
          "ROLLBACK"
        ]
      },
      "definitionKey": {
        "type": "string"
      },
      "scope": {
        "type": "string"
      },
      "requestedValue": {
        "type": "string"
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
      "type",
      "definitionKey",
      "scope",
      "requestedValue",
      "requestedBy",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Requests the caller themselves raised are listed but cannot be decided by them (#23).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** CFG-007 (high-impact change governance), AUTHZ-009 (SoD — requester ≠ approver).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 20. Decide Configuration Change Request
- **Method:** `POST`
- **URL:** `/api/v1/config/approvals/{id}/decide`
- **Description:** Approve or reject a pending high-impact value change or rollback.
- **Authentication (Role-Based):** Yes — `config.value.approve`; **blocked if the caller is the request's own requester** — self-approval is structurally impossible (Doc 27 §7).
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
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED` ("A different approver is required"); `422 REASON_REQUIRED`; `409 ALREADY_DECIDED`.
- **Business Rules Applied:** CFG-007, AUTHZ-009.
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
