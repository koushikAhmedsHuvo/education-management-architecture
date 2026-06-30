# Fee Structure API

## 1. API Overview

**Purpose.** Configure fee structures, heads (including the authoritative non-waivable classification), late-fine policy, and structure import — the effective-dated, versioned definition layer that invoice generation resolves against.

**Module Context.** Implements Business Rules Catalog Doc 17 (Fee Management) Rules FEE-001/002/006/007 and UI Screen Spec `18-fee-management-screens.md` SCR-FEE-01…03/11/15.

---

## 2. Endpoints

### 1. List Fee Structures
- **Method:** `GET`
- **URL:** `/api/v1/fee-structures`
- **Description:** Browse fee structures and their versions (SCR-FEE-01).
- **Authentication (Role-Based):** Yes — `fee.view`, scoped.
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
    "statusDRAFTACTIVESU": {
      "type": "string",
      "description": "status? (DRAFT|ACTIVE|SUPERSEDED|ARCHIVED)"
    },
    "sortBynameeffectiveD": {
      "type": "string",
      "description": "sortBy? (name|effectiveDate, default name)"
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
      "category": {
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
          },
          "classId": {
            "type": "string"
          }
        },
        "required": [
          "instituteId"
        ]
      },
      "version": {
        "type": "integer"
      },
      "effectiveDate": {
        "type": "string"
      },
      "status": {
        "type": "string"
      },
      "updatedAt": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "name",
      "category",
      "scope",
      "version",
      "effectiveDate",
      "status",
      "updatedAt"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** FEE-001 (versioned, effective-dated), FEE-006 (category differentiation), AUTHZ-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Fee Structure Detail
- **Method:** `GET`
- **URL:** `/api/v1/fee-structures/{id}`
- **Description:** Full structure — heads, amounts, schedule, version history (SCR-FEE-01).
- **Authentication (Role-Based):** Yes — `fee.view`.
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
    "category": {
      "type": "string"
    },
    "scope": {
      "type": "string"
    },
    "heads": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "feeHeadId": {
            "type": "string"
          },
          "amount": {
            "type": "number"
          },
          "schedule": {
            "type": "string"
          }
        },
        "required": [
          "feeHeadId",
          "amount",
          "schedule"
        ]
      }
    },
    "effectiveDate": {
      "type": "string"
    },
    "version": {
      "type": "integer"
    },
    "versionHistory": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "version": {
            "type": "integer"
          },
          "effectiveDate": {
            "type": "string"
          },
          "status": {
            "type": "string"
          }
        },
        "required": [
          "version",
          "effectiveDate",
          "status"
        ]
      }
    }
  },
  "required": [
    "id",
    "name",
    "category",
    "scope",
    "heads",
    "effectiveDate",
    "version",
    "versionHistory"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** FEE-001.
- **Pagination:** N/A.

### 3. Define / Version Fee Structure
- **Method:** `POST`
- **URL:** `/api/v1/fee-structures`
- **Description:** Define or version a fee structure — heads mapped to amounts for a scope, with a schedule and effective date (UC-FEE-01, SCR-FEE-02).
- **Authentication (Role-Based):** Yes — `fee.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "category": {
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
        },
        "classId": {
          "type": "string"
        }
      },
      "required": [
        "instituteId"
      ]
    },
    "heads": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "feeHeadId": {
            "type": "string"
          },
          "amount": {
            "type": "number"
          },
          "schedule": {
            "type": "string",
            "enum": [
              "ONE_TIME",
              "MONTHLY",
              "TERM",
              "INSTALLMENT"
            ]
          }
        },
        "required": [
          "feeHeadId",
          "amount",
          "schedule"
        ]
      }
    },
    "effectiveDate": {
      "type": "string"
    }
  },
  "required": [
    "name",
    "category",
    "scope",
    "heads",
    "effectiveDate"
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
        "DRAFT",
        "ACTIVE"
      ]
    },
    "version": {
      "type": "integer"
    },
    "effectiveDate": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "status",
    "version",
    "effectiveDate"
  ]
}
```
- **Validation Rules:** Amounts must be `≥ 0`; the new effective period must not overlap an existing structure version for the same scope/head ("no overlapping effective periods"); a change **always** creates a new version effective from a future date — already-issued invoices are never altered by this write ("a tuition increase effective next term leaves this term's issued invoices unchanged," FEE-001).
- **Error Codes:** `422 NEGATIVE_AMOUNT`; `409 OVERLAPPING_EFFECTIVE_PERIOD`.
- **Business Rules Applied:** FEE-001 (configurable, effective-dated, versioned), FEE-002 (heads & schedules), FEE-006 (category-based differentiation).
- **Pagination:** N/A.

### 4. Publish Fee Structure Version
- **Method:** `POST`
- **URL:** `/api/v1/fee-structures/{id}/versions/{version}/publish`
- **Description:** Publish a draft structure version as the active, immutable version (SCR-FEE-02).
- **Authentication (Role-Based):** Yes — `fee.config.manage`.
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
    "version": {
      "type": "integer"
    },
    "status": {
      "type": "string",
      "const": "ACTIVE"
    }
  },
  "required": [
    "id",
    "version",
    "status"
  ]
}
```
- **Validation Rules:** Once published, the version is immutable — further changes require a new version (CFG-004 pattern).
- **Error Codes:** `409 ALREADY_PUBLISHED`; `409 STALE_VERSION_CONFLICT`.
- **Business Rules Applied:** FEE-001, CFG-004.
- **Pagination:** N/A.

## Fee Heads, Schedules & Categories

### 5. List Fee Heads
- **Method:** `GET`
- **URL:** `/api/v1/fee-heads`
- **Description:** Browse fee heads/categories, including the non-waivable flag (SCR-FEE-03).
- **Authentication (Role-Based):** Yes — `fee.view`.
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
    "type": {
      "type": "string"
    },
    "nonWaivableboolean": {
      "type": "string",
      "description": "nonWaivable? (boolean)"
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
      "nonWaivable": {
        "type": "boolean"
      },
      "schedule": {
        "type": "string"
      },
      "status": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "name",
      "type",
      "nonWaivable",
      "schedule",
      "status"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** FEE-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 6. Create / Edit Fee Head
- **Method:** `POST`
- **URL:** `/api/v1/fee-heads`
- **Description:** Define or edit a fee head, including its **non-waivable** classification — the authoritative source consumed by `21-discount-api.md`'s waiver gate (SCR-FEE-03).
- **Authentication (Role-Based):** Yes — `fee.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "type": {
      "type": "string"
    },
    "nonWaivable": {
      "type": "boolean"
    },
    "schedule": {
      "type": "string"
    },
    "version": {
      "type": "number",
      "description": "for PUT"
    }
  },
  "required": [
    "name",
    "type",
    "nonWaivable",
    "schedule"
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
    "type": {
      "type": "string"
    },
    "nonWaivable": {
      "type": "string"
    },
    "schedule": {
      "type": "string"
    },
    "status": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "name",
    "type",
    "nonWaivable",
    "schedule",
    "status"
  ]
}
```
- **Validation Rules:** `name` required; flipping `nonWaivable` to `true` on a head with active waivers already applied does not retroactively unwind them (forward effect only).
- **Error Codes:** `422 NAME_REQUIRED`; `409 VERSION_CONFLICT` (PUT only).
- **Business Rules Applied:** FEE-002 (heads & non-waivable classification — "the authoritative source for Discount's waiver gate," C-03).
- **Pagination:** N/A.

## Invoices

### 7. Get Late-Fine Policy
- **Method:** `GET`
- **URL:** `/api/v1/fee-config/late-fine-policy`
- **Description:** View the configured late-fine amount/grace-period policy (SCR-FEE-11).
- **Authentication (Role-Based):** Yes — `fee.config.manage`.
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
      },
      "required": [
        "instituteId"
      ]
    }
  },
  "required": [
    "scope"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "amount": {
      "type": "number"
    },
    "gracePeriodDays": {
      "type": "number"
    },
    "cap": {
      "type": "number"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "amount",
    "gracePeriodDays",
    "cap",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** FEE-007.
- **Pagination:** N/A.

### 8. Set Late-Fine Policy
- **Method:** `PUT`
- **URL:** `/api/v1/fee-config/late-fine-policy`
- **Description:** Configure late-fine amount, grace period, and cap, versioned (SCR-FEE-11).
- **Authentication (Role-Based):** Yes — `fee.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "amount": {
      "type": "number"
    },
    "gracePeriodDays": {
      "type": "number"
    },
    "cap": {
      "type": "number"
    },
    "version": {
      "type": "number"
    }
  },
  "required": [
    "amount",
    "gracePeriodDays",
    "cap",
    "version"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "amount": {
      "type": "number"
    },
    "gracePeriodDays": {
      "type": "string"
    },
    "cap": {
      "type": "string"
    },
    "version": {
      "type": "number"
    }
  },
  "required": [
    "amount",
    "gracePeriodDays",
    "cap",
    "version"
  ]
}
```
- **Validation Rules:** `version` optimistic lock; the policy is itself versioned (CFG-004 pattern).
- **Error Codes:** `409 VERSION_CONFLICT`.
- **Business Rules Applied:** FEE-007, CFG-004.
- **Pagination:** N/A.

## Refunds (Governed Decision — Execution in Payment)

### 9. Import Fee Structure / Opening Balances
- **Method:** `POST`
- **URL:** `/api/v1/fee-structures/import`
- **Description:** Import a fee structure or opening dues balances (SCR-FEE-15).
- **Authentication (Role-Based):** Yes — `fee.import`.
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
            "type": "string",
            "description": "..."
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
            "type": "string",
            "description": "..."
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
- **Validation Rules:** Imported structures/heads are typed and validated as in #3; invalid rows excluded and always reported.
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** FEE-001; conventions §9.
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
