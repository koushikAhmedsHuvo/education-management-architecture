# Discount API

## 1. API Overview

**Purpose.** Apply transparent, capped, stacked discounts and waivers (with the non-waivable-head gate enforced at the point of waiver), under tiered separation-of-duties approval.

**Module Context.** Implements Business Rules Catalog Doc 19 (Discount) and UI Screen Spec `20-discount-screens.md` (SCR-DSC-01…11).

---

## 2. Endpoints

### 1. List Discounts
- **Method:** `GET`
- **URL:** `/api/v1/discounts`
- **Description:** Browse applied discounts/waivers — transparent records (SCR-DSC-01).
- **Authentication (Role-Based):** Yes — `discount.view`, scoped + ownership.
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
    "classificationDISCOUN": {
      "type": "string",
      "description": "classification? (DISCOUNT|WAIVER)"
    },
    "statusELIGIBLEREQUES": {
      "type": "string",
      "description": "status? (ELIGIBLE|REQUESTED|APPROVED|APPLIED|REVERSED|EXPIRED|REJECTED)"
    },
    "studentId": {
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
      "studentId": {
        "type": "string"
      },
      "studentName": {
        "type": "string"
      },
      "charge": {
        "type": "object",
        "properties": {
          "invoiceId": {
            "type": "string"
          },
          "head": {
            "type": "string"
          }
        },
        "required": [
          "invoiceId",
          "head"
        ]
      },
      "classification": {
        "type": "string"
      },
      "type": {
        "type": "string",
        "enum": [
          "PERCENTAGE",
          "FIXED"
        ]
      },
      "amount": {
        "type": "number"
      },
      "status": {
        "type": "string"
      },
      "appliedAt": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "studentId",
      "studentName",
      "charge",
      "classification",
      "type",
      "amount",
      "status"
    ]
  }
}
```
- **Validation Rules:** Scope-limited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-002, REP-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Apply Discount
- **Method:** `POST`
- **URL:** `/api/v1/discounts`
- **Description:** Apply a typed discount to a student/charge transparently, within caps and stacking rules — the screen's own "★" designation (SCR-DSC-02).
- **Authentication (Role-Based):** Yes — `discount.apply`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string"
    },
    "chargeRef": {
      "type": "object",
      "properties": {
        "invoiceId": {
          "type": "string"
        },
        "feeHeadId": {
          "type": "string"
        }
      },
      "required": [
        "invoiceId"
      ]
    },
    "type": {
      "type": "string",
      "enum": [
        "PERCENTAGE",
        "FIXED"
      ]
    },
    "amount": {
      "type": "number"
    }
  },
  "required": [
    "studentId",
    "chargeRef",
    "type",
    "amount"
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
    "type": {
      "type": "string"
    },
    "amount": {
      "type": "number"
    },
    "netPreview": {
      "type": "object",
      "properties": {
        "gross": {
          "type": "string"
        },
        "discount": {
          "type": "integer"
        },
        "net": {
          "type": "string"
        }
      },
      "required": [
        "gross",
        "discount",
        "net"
      ]
    },
    "status": {
      "type": "string",
      "enum": [
        "APPLIED",
        "PENDING_APPROVAL"
      ]
    }
  },
  "required": [
    "id",
    "type",
    "amount",
    "netPreview",
    "status"
  ]
}
```
- **Validation Rules:** The discount appears as a **distinct, visible reduction line** alongside the preserved gross charge — never a silent reduction of the original figure ("a 10% sibling discount shows as a clear reduction line, not a quietly lowered tuition figure," DSC-001); the discount is **capped at the charge** and the net **floors at zero**, never negative ("Discount would push net below zero," DSC-004); combining with any existing reduction on the same charge respects the configured stacking policy and combined cap (DSC-005, see #6); discretionary (non-rule) amounts above the configured threshold route to tiered approval (#10) rather than applying immediately (DSC-003).
- **Error Codes:** `422 NET_BELOW_ZERO`; `422 EXCEEDS_TYPE_CAP`; `409 STACKING_CAP_EXCEEDED`.
- **Business Rules Applied:** DSC-001 (transparent typed application), DSC-003 (discretionary approval authority/SoD), DSC-004 (caps, net ≥ 0), DSC-005 (stacking).
- **Pagination:** N/A.

### 3. Preview Auto-Eligibility Discounts
- **Method:** `GET`
- **URL:** `/api/v1/discounts/auto-eligibility/preview`
- **Description:** Preview students who qualify for a configured eligibility-rule discount (e.g., sibling, early-payment) before auto-applying (SCR-DSC-03).
- **Authentication (Role-Based):** Yes — `discount.apply`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "discountTypeId": {
      "type": "string"
    }
  },
  "required": [
    "discountTypeId"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "eligibleStudents": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "studentId": {
            "type": "string"
          },
          "studentName": {
            "type": "string"
          },
          "basis": {
            "type": "string"
          },
          "previewAmount": {
            "type": "number"
          }
        },
        "required": [
          "studentId",
          "studentName",
          "basis",
          "previewAmount"
        ]
      }
    }
  },
  "required": [
    "eligibleStudents"
  ]
}
```
- **Validation Rules:** Eligibility criteria are evaluated against **current** data at the point of preview (DSC-002).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** DSC-002 (eligibility-rule discounts).
- **Pagination:** N/A.

### 4. Auto-Apply Eligibility Discounts
- **Method:** `POST`
- **URL:** `/api/v1/discounts/auto-eligibility/apply`
- **Description:** Apply the configured eligibility-rule discount to all currently-qualifying students (SCR-DSC-03).
- **Authentication (Role-Based):** Yes — `discount.apply`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "discountTypeId": {
      "type": "string"
    },
    "studentIds": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "omit for all previewed"
    }
  },
  "required": [
    "discountTypeId"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "applied": {
      "type": "number"
    },
    "skipped": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "studentId": {
            "type": "string"
          },
          "reason": {
            "type": "string"
          }
        },
        "required": [
          "studentId",
          "reason"
        ]
      }
    }
  },
  "required": [
    "applied",
    "skipped"
  ]
}
```
- **Validation Rules:** Conditional discounts (e.g., early-payment) apply **provisionally** and are subject to automatic reversal if their condition later fails (#7, DSC-007); each application is capped and transparent as in #2.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** DSC-002, DSC-001, DSC-004.
- **Pagination:** N/A.

## Waivers

### 5. Apply Waiver
- **Method:** `POST`
- **URL:** `/api/v1/discounts/waivers`
- **Description:** Apply a waiver to a specific charge — blocked on non-waivable heads, under SoD approval. The screen's own "★ C-03 boundary" designation (SCR-DSC-04).
- **Authentication (Role-Based):** Yes — `discount.waiver.apply` (request).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string"
    },
    "invoiceId": {
      "type": "string"
    },
    "feeHeadId": {
      "type": "string"
    },
    "type": {
      "type": "string",
      "enum": [
        "PERCENTAGE",
        "FIXED"
      ]
    },
    "amount": {
      "type": "number"
    },
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "studentId",
    "invoiceId",
    "feeHeadId",
    "type",
    "amount",
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
      "const": "PENDING_APPROVAL"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** The target `feeHeadId` is checked against `19-fee-management-api.md`'s `nonWaivable` flag — a non-waivable head is rejected outright at the point of selection, not merely warned about ("This is a non-waivable charge and cannot be waived," FEE-002/DSC-006/C-03); the resulting net must be `≥ 0` (DSC-004); `reason` required; every waiver routes to approval — there is no immediate-apply path for a waiver, unlike a small rule-based discount (DSC-003).
- **Error Codes:** `409 NON_WAIVABLE_HEAD`; `422 NET_BELOW_ZERO`; `422 REASON_REQUIRED`.
- **Business Rules Applied:** DSC-006 (waiver sub-type semantics), FEE-002 (non-waivable heads, classified by Fee), C-03 (Discount owns waiver / Fee classifies non-waivable), DSC-004, DSC-003 (SoD), AUTHZ-009.
- **Pagination:** N/A.

## Caps & Stacking

### 6. Preview Stacking Resolution
- **Method:** `GET`
- **URL:** `/api/v1/discounts/stacking-preview`
- **Description:** View how multiple discounts/waivers on a charge stack against caps, and the resulting net (SCR-DSC-05).
- **Authentication (Role-Based):** Yes — `discount.view` or `discount.apply`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "invoiceId": {
      "type": "string"
    },
    "feeHeadId": {
      "type": "string"
    }
  },
  "required": [
    "invoiceId"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "gross": {
      "type": "number"
    },
    "appliedReductions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": {
            "type": "string"
          },
          "type": {
            "type": "string"
          },
          "amount": {
            "type": "number"
          },
          "order": {
            "type": "string"
          }
        },
        "required": [
          "id",
          "type",
          "amount",
          "order"
        ]
      }
    },
    "combinedReduction": {
      "type": "number"
    },
    "net": {
      "type": "number"
    },
    "capReached": {
      "type": "boolean"
    }
  },
  "required": [
    "gross",
    "appliedReductions",
    "combinedReduction",
    "net",
    "capReached"
  ]
}
```
- **Validation Rules:** Stacking order (additive vs. sequential-percentage) and the combined ceiling are applied per the configured policy (#9) — the resolution is always **deterministic and explainable**, never ad hoc ("net is always explainable," DSC-005); `net` never goes below zero (DSC-004); scholarship coverage interaction (`22-scholarship-api.md`, SCH-006) is reflected here too where applicable.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** DSC-004 (caps, net ≥ 0), DSC-005 (stacking), SCH-006 (scholarship interaction).
- **Pagination:** N/A.

## Reversal & Expiry

### 7. Reverse / Expire Discount
- **Method:** `POST`
- **URL:** `/api/v1/discounts/{id}/reverse`
- **Description:** Reverse a discount whose condition failed or which was applied in error, transparently recomputing dues (UC-DSC-03, SCR-DSC-06).
- **Authentication (Role-Based):** Yes — `discount.reverse` (elevated).
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
      "enum": [
        "REVERSED",
        "EXPIRED"
      ]
    },
    "dueRestored": {
      "type": "number"
    }
  },
  "required": [
    "id",
    "status",
    "dueRestored"
  ]
}
```
- **Validation Rules:** The reduction is removed via a **transparent reversing entry**, never a silent removal — the due is restored and recomputed visibly ("reversal recomputes dues transparently, not a silent removal"); if the underlying invoice was already paid, the reversal triggers credit/refund handling (`19-fee-management-api.md` #18, `20-payment-api.md`) rather than an impossible negative payment.
- **Error Codes:** `409 ALREADY_REVERSED`.
- **Business Rules Applied:** DSC-007 (conditional reversal & expiry, transparent and audited).
- **Pagination:** N/A.

## Configuration

### 8. Get Discount Type & Eligibility Configuration
- **Method:** `GET`
- **URL:** `/api/v1/discounts/config`
- **Description:** View configured discount types, eligibility rules, stacking policy, caps, and approval tiers (SCR-DSC-07).
- **Authentication (Role-Based):** Yes — `discount.config.manage`.
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
    "discountTypes": {
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
          "eligibilityRule": {
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
    "stackingPolicy": {
      "type": "object",
      "properties": {
        "order": {
          "type": "string"
        },
        "combinedCap": {
          "type": "string"
        }
      },
      "required": [
        "order",
        "combinedCap"
      ]
    },
    "approvalTiers": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "maxAmount": {
            "type": "number"
          },
          "requiredRole": {
            "type": "string"
          }
        },
        "required": [
          "maxAmount",
          "requiredRole"
        ]
      }
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "discountTypes",
    "stackingPolicy",
    "approvalTiers",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** DSC-001, DSC-003, DSC-004, DSC-005.
- **Pagination:** N/A.

### 9. Set Discount Type & Eligibility Configuration
- **Method:** `PUT`
- **URL:** `/api/v1/discounts/config`
- **Description:** Configure discount types, eligibility, stacking policy, caps, and tiered approval authority, versioned (SCR-DSC-07).
- **Authentication (Role-Based):** Yes — `discount.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "discountTypes": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    },
    "stackingPolicy": {
      "type": "object",
      "properties": {
        "order": {
          "type": "string",
          "enum": [
            "ADDITIVE",
            "SEQUENTIAL_PERCENTAGE"
          ]
        },
        "combinedCap": {
          "type": "number"
        }
      },
      "required": [
        "order",
        "combinedCap"
      ]
    },
    "approvalTiers": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "maxAmount": {
            "type": "number"
          },
          "requiredRole": {
            "type": "string"
          }
        },
        "required": [
          "maxAmount",
          "requiredRole"
        ]
      }
    },
    "version": {
      "type": "number"
    }
  },
  "required": [
    "discountTypes",
    "stackingPolicy",
    "approvalTiers",
    "version"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "config": {
      "type": "string",
      "description": "...config"
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
- **Validation Rules:** Stacking order and combined cap must both be explicitly defined — never left ambiguous (DSC-005); approval tiers must cover the full amount range with no gaps (so every discretionary amount resolves to exactly one required authority level); `version` optimistic lock.
- **Error Codes:** `422 STACKING_POLICY_INCOMPLETE`; `422 APPROVAL_TIER_GAP`; `409 VERSION_CONFLICT`.
- **Business Rules Applied:** DSC-001, DSC-003 (tiered authority), DSC-004, DSC-005, CFG-002/004.
- **Pagination:** N/A.

## Approvals

### 10. List Discount / Waiver Approvals
- **Method:** `GET`
- **URL:** `/api/v1/discounts/approvals`
- **Description:** Inbox for pending discretionary discount/waiver requests, tiered by amount (SCR-DSC-11).
- **Authentication (Role-Based):** Yes — `discount.approve`, scoped to the caller's authority tier.
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
      "type": {
        "type": "string",
        "enum": [
          "DISCOUNT",
          "WAIVER"
        ]
      },
      "studentId": {
        "type": "string"
      },
      "amount": {
        "type": "number"
      },
      "reason": {
        "type": "string"
      },
      "requestedBy": {
        "type": "string"
      },
      "requiredTier": {
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
      "studentId",
      "amount",
      "reason",
      "requestedBy",
      "requiredTier",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Requester ≠ approver — self-raised requests are listed but cannot be decided by the requester (DSC-003/AUTHZ-009); requests beyond the caller's own authority tier are visible but not decidable by them either, surfacing the need to escalate.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** DSC-003, AUTHZ-009.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 11. Decide Discount / Waiver Request
- **Method:** `POST`
- **URL:** `/api/v1/discounts/approvals/{id}/decide`
- **Description:** Approve or reject a pending discretionary discount or waiver; on approval, the reduction (#2/#5) applies with full provenance.
- **Authentication (Role-Based):** Yes — `discount.approve`, with authority covering the requested amount; **blocked if the caller raised the request — self-approval is impossible by construction**.
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
- **Validation Rules:** Caller ≠ requester; caller's authority tier ≥ the requested amount — an under-authorized approver cannot decide, only escalate (DSC-003).
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED`; `403 INSUFFICIENT_APPROVAL_AUTHORITY`.
- **Business Rules Applied:** DSC-003 (approval authority & SoD), AUTHZ-009.
- **Pagination:** N/A.

## Reporting, Bulk & Data Transfer

### 12. Discount / Waiver Utilization Report
- **Method:** `GET`
- **URL:** `/api/v1/discounts/report/utilization`
- **Description:** Report discount/waiver utilization (SCR-DSC-08).
- **Authentication (Role-Based):** Yes — `discount.view` (report authority).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "classId": {
      "type": "string"
    },
    "dateFrom": {
      "type": "string"
    },
    "dateTo": {
      "type": "string"
    },
    "classification": {
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
    "totalDiscounted": {
      "type": "integer"
    },
    "totalWaived": {
      "type": "string"
    },
    "byType": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "typeId": {
            "type": "string"
          },
          "name": {
            "type": "string"
          },
          "amount": {
            "type": "number"
          },
          "count": {
            "type": "integer"
          }
        },
        "required": [
          "typeId",
          "name",
          "amount",
          "count"
        ]
      }
    }
  },
  "required": [
    "totalDiscounted",
    "totalWaived",
    "byType"
  ]
}
```
- **Validation Rules:** Scope-limited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** REP-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 13. Bulk Apply Discount
- **Method:** `POST`
- **URL:** `/api/v1/discounts/bulk-apply`
- **Description:** Apply a discount to a cohort at once (SCR-DSC-09).
- **Authentication (Role-Based):** Yes — `discount.apply`.
- **Request DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "studentIds": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "discountTypeId": {
          "type": "string"
        },
        "amount": {
          "type": "number"
        }
      },
      "required": [
        "studentIds",
        "discountTypeId"
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
  "description": "202 Accepted { jobId } (per-student results via GET /api/v1/jobs/{jobId})."
}
```
- **Validation Rules:** Per-student caps and `net ≥ 0` enforced individually (DSC-004); any student whose amount is discretionary-tier is routed to approval rather than blocking the rest of the batch.
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** DSC-001, DSC-004; conventions §9.
- **Pagination:** N/A.

### 14. Import Discount Rules
- **Method:** `POST`
- **URL:** `/api/v1/discounts/import`
- **Description:** Import discount type/eligibility rules from a file (SCR-DSC-10).
- **Authentication (Role-Based):** Yes — `discount.import`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "rules": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    }
  },
  "required": [
    "rules"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
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
}
```
- **Validation Rules:** Imported rules are typed and structurally validated as in #9 before acceptance; invalid rules are rejected, never silently accepted.
- **Error Codes:** `422 INVALID_RULE_DEFINITION`.
- **Business Rules Applied:** CFG-002.
- **Pagination:** N/A.

### 15. Export Discount Rules
- **Method:** `GET`
- **URL:** `/api/v1/discounts/export`
- **Description:** Governed export of discount rule configuration (SCR-DSC-10).
- **Authentication (Role-Based):** Yes — `discount.export`.
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
- **Validation Rules:** Export is audited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** REP-002, REP-008, conventions §10.
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
