# Fee Collection API

## 1. API Overview

**Purpose.** Generate immutable, idempotent invoices from the effective fee structure (with pro-rata), correct them only via linked credit notes, track dues/aging/fines, carry forward arrears, and process governed refund decisions (executed as a Payment reversal).

**Module Context.** Implements Business Rules Catalog Doc 17 (Fee Management) Rules FEE-003/004/005/008 and UI Screen Spec `18-fee-management-screens.md` SCR-FEE-04…10/12…14/16.

---

## 2. Endpoints

### 1. List Invoices
- **Method:** `GET`
- **URL:** `/api/v1/invoices`
- **Description:** Browse/search invoices and dues (SCR-FEE-04).
- **Authentication (Role-Based):** Yes — `fee.view`, scoped + ownership (student/guardian own).
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
    "statusDRAFTISSUEDPA": {
      "type": "string",
      "description": "status? (DRAFT|ISSUED|PARTIALLY_PAID|PAID|ADJUSTED|VOIDED|ARCHIVED)"
    },
    "overdueboolean": {
      "type": "string",
      "description": "overdue? (boolean)"
    },
    "classId": {
      "type": "string"
    },
    "period": {
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
      "invoiceNumber": {
        "type": "integer"
      },
      "studentId": {
        "type": "string"
      },
      "studentName": {
        "type": "string"
      },
      "amount": {
        "type": "number"
      },
      "paid": {
        "type": "string"
      },
      "due": {
        "type": "string"
      },
      "status": {
        "type": "string"
      },
      "dueDate": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "invoiceNumber",
      "studentId",
      "studentName",
      "amount",
      "paid",
      "due",
      "status",
      "dueDate"
    ]
  }
}
```
- **Validation Rules:** Scope-limited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-002, REP-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Invoice Detail
- **Method:** `GET`
- **URL:** `/api/v1/invoices/{id}`
- **Description:** View an invoice with heads, payments, and linked credit notes (SCR-FEE-05).
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
    "invoiceNumber": {
      "type": "integer"
    },
    "studentId": {
      "type": "string"
    },
    "structureVersionStamped": {
      "type": "number"
    },
    "lines": {
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
          "discountApplied": {
            "type": "integer"
          },
          "net": {
            "type": "string"
          }
        },
        "required": [
          "feeHeadId",
          "amount",
          "net"
        ]
      }
    },
    "payments": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "paymentId": {
            "type": "string"
          },
          "amount": {
            "type": "number"
          },
          "date": {
            "type": "string"
          }
        },
        "required": [
          "paymentId",
          "amount",
          "date"
        ]
      }
    },
    "creditNotes": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": {
            "type": "string"
          },
          "amount": {
            "type": "number"
          },
          "reason": {
            "type": "string"
          }
        },
        "required": [
          "id",
          "amount",
          "reason"
        ]
      }
    },
    "status": {
      "type": "string"
    },
    "issuedAt": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "invoiceNumber",
    "studentId",
    "structureVersionStamped",
    "lines",
    "payments",
    "creditNotes",
    "status",
    "issuedAt"
  ]
}
```
- **Validation Rules:** This endpoint is read-only by construction — there is no edit path for an issued invoice's lines/amounts; any attempted direct edit is rejected at the data layer ("any edit attempt on an issued invoice is blocked, credit-note path offered").
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** FEE-003 (issued invoices are immutable), FILE-007.
- **Pagination:** N/A.

### 3. Generate Invoice
- **Method:** `POST`
- **URL:** `/api/v1/invoices/generate`
- **Description:** Generate an immutable invoice for a student/period from the **effective** fee structure, preventing duplicates — the screen's own "★ money-integrity" designation (SCR-FEE-06).
- **Authentication (Role-Based):** Yes — `fee.invoice.issue`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string"
    },
    "period": {
      "type": "string"
    },
    "idempotencyKey": {
      "type": "string"
    }
  },
  "required": [
    "studentId",
    "period",
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
    "invoiceNumber": {
      "type": "integer"
    },
    "lines": {
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
          "discountApplied": {
            "type": "integer"
          },
          "scholarshipApplied": {
            "type": "string"
          },
          "net": {
            "type": "string"
          }
        },
        "required": [
          "feeHeadId",
          "amount",
          "net"
        ]
      }
    },
    "totalNet": {
      "type": "string"
    },
    "structureVersionStamped": {
      "type": "number"
    },
    "status": {
      "type": "string",
      "const": "ISSUED"
    }
  },
  "required": [
    "id",
    "invoiceNumber",
    "lines",
    "totalNet",
    "structureVersionStamped",
    "status"
  ]
}
```
- **Validation Rules:** The system resolves the **effective, version-stamped** structure for the student's scope/category and stamps that exact version on the invoice (FEE-001); generation is **idempotent** by `(student, period, head)` — replaying the same request never produces a duplicate invoice ("an invoice already exists for this student/period," FEE-004); pro-rata is applied automatically for mid-period join/leave per the configured method (FEE-005); applicable discounts (`21-discount-api.md`) and scholarship coverage (`22-scholarship-api.md`) apply as transparent reduction lines; the net amount can **never go negative** ("Net cannot be negative," DSC-004); once issued, the invoice is permanently immutable (FEE-003).
- **Error Codes:** `409 DUPLICATE_INVOICE` (returns a link to the existing invoice, not an error-only response); `422 NO_EFFECTIVE_STRUCTURE` ("Define/publish a fee structure first"); `422 NET_BELOW_ZERO`.
- **Business Rules Applied:** FEE-001 (resolved, stamped structure version), FEE-003 (immutable on issue), FEE-004 (duplicate prevention, idempotent), FEE-005 (pro-rata), DSC-004 (net ≥ 0).
- **Pagination:** N/A.

### 4. Bulk Generate Invoices
- **Method:** `POST`
- **URL:** `/api/v1/invoices/bulk-generate`
- **Description:** Generate invoices for a cohort — always asynchronous, idempotent at scale (SCR-FEE-07).
- **Authentication (Role-Based):** Yes — `fee.invoice.issue`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "cohort": {
      "type": "object",
      "properties": {
        "classId": {
          "type": "string"
        },
        "sectionId": {
          "type": "string"
        },
        "campusId": {
          "type": "string"
        }
      }
    },
    "period": {
      "type": "string"
    },
    "idempotencyKey": {
      "type": "string"
    }
  },
  "required": [
    "cohort",
    "period",
    "idempotencyKey"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "null",
  "description": "202 Accepted { jobId } (per-student results via GET /api/v1/jobs/{jobId}: { accepted: [{ studentId, invoiceId }], skippedDuplicates: [{ studentId, existingInvoiceId }], rejected: [{ studentId, reason }] })."
}
```
- **Validation Rules:** Same per-student rules as #9 (duplicate-skip is idempotent, never an error — "re-running a batch does not duplicate already-issued invoices," FEE-004); invalid rows (no effective structure, etc.) are excluded and reported.
- **Error Codes:** `400 MALFORMED_REQUEST`.
- **Business Rules Applied:** FEE-003, FEE-004, FEE-005; conventions §9.
- **Pagination:** N/A.

### 5. Issue Credit Note
- **Method:** `POST`
- **URL:** `/api/v1/invoices/{id}/credit-notes`
- **Description:** Correct or reduce an issued invoice **without editing it** — the only correction path (UC-FEE-03, SCR-FEE-08).
- **Authentication (Role-Based):** Yes — `fee.creditnote.issue`; large amounts may route to approval per Doc 17 §8.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "amount": {
      "type": "number"
    },
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "amount",
    "reason"
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
        "invoiceId": {
          "type": "string"
        },
        "amount": {
          "type": "number"
        },
        "reason": {
          "type": "string"
        },
        "status": {
          "type": "string",
          "enum": [
            "ISSUED",
            "PENDING_APPROVAL"
          ]
        }
      },
      "required": [
        "id",
        "invoiceId",
        "amount",
        "reason",
        "status"
      ]
    },
    {
      "type": "object",
      "properties": {
        "status": {
          "type": "string"
        }
      },
      "required": [
        "status"
      ]
    },
    {
      "type": "object",
      "properties": {
        "ADJUSTED": {
          "type": "string"
        }
      },
      "required": [
        "ADJUSTED"
      ]
    }
  ]
}
```
- **Validation Rules:** `reason` required; the credit note amount is capped so it cannot exceed the invoice's remaining amount; the original invoice's line items and figures remain permanently unchanged — only a new, distinctly linked document is created ("an over-billed invoice is corrected by a credit note, not by changing the original figure," FEE-003).
- **Error Codes:** `422 EXCEEDS_INVOICE_AMOUNT`; `422 REASON_REQUIRED`.
- **Business Rules Applied:** FEE-003 (issued invoices are immutable), FEE-008 (governed adjustments).
- **Pagination:** N/A.

### 6. Compute Pro-Rata
- **Method:** `POST`
- **URL:** `/api/v1/invoices/pro-rata/compute`
- **Description:** Compute the pro-rated amount for a mid-period join/leave, for preview before invoice generation (SCR-FEE-09).
- **Authentication (Role-Based):** Yes — `fee.invoice.issue`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string"
    },
    "feeHeadId": {
      "type": "string"
    },
    "joinDate": {
      "type": "string"
    },
    "leaveDate": {
      "type": "string"
    }
  },
  "required": [
    "studentId",
    "feeHeadId"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "fullAmount": {
      "type": "number"
    },
    "proRatedAmount": {
      "type": "number"
    },
    "method": {
      "type": "string",
      "enum": [
        "DAILY",
        "WEEKLY"
      ]
    },
    "basis": {
      "type": "string"
    }
  },
  "required": [
    "fullAmount",
    "proRatedAmount",
    "method",
    "basis"
  ]
}
```
- **Validation Rules:** The pro-rata method (daily/weekly) is resolved from configuration; if undefined for the scope, the system defaults to full-period and flags it rather than guessing (FEE-005 failure behavior).
- **Error Codes:** `422 PRORATA_METHOD_UNDEFINED` (returns the full-period fallback with a flag, not a hard failure).
- **Business Rules Applied:** FEE-005 (pro-rata for mid-period join/leave).
- **Pagination:** N/A.

## Dues, Fines & Aging

### 7. List Dues & Aging
- **Method:** `GET`
- **URL:** `/api/v1/dues`
- **Description:** View outstanding dues with aging buckets (SCR-FEE-10).
- **Authentication (Role-Based):** Yes — `fee.view`, scoped + ownership.
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
    "classId": {
      "type": "string"
    },
    "agingBucket03031": {
      "type": "string",
      "description": "agingBucket? (\"0-30\"|\"31-60\"|\"61-90\"|\"90+\")"
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
      "studentId": {
        "type": "string"
      },
      "studentName": {
        "type": "string"
      },
      "totalDue": {
        "type": "string"
      },
      "buckets": {
        "type": "object",
        "properties": {
          "current": {
            "type": "string"
          },
          "030": {
            "type": "string",
            "description": "\"0-30\""
          },
          "3160": {
            "type": "string",
            "description": "\"31-60\""
          },
          "6190": {
            "type": "string",
            "description": "\"61-90\""
          },
          "90": {
            "type": "string",
            "description": "\"90+\""
          }
        },
        "required": [
          "current"
        ]
      }
    },
    "required": [
      "studentId",
      "studentName",
      "totalDue",
      "buckets"
    ]
  }
}
```
- **Validation Rules:** Net due = issued − adjustments − payments − approved reductions, computed deterministically (FEE-007).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** FEE-007 (dues, due dates & late fines).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 8. Apply Late Fine
- **Method:** `POST`
- **URL:** `/api/v1/dues/{studentId}/fines/apply`
- **Description:** Apply a configured late fine as a **new, distinct charge** — never a silent edit to the original invoice (SCR-FEE-10).
- **Authentication (Role-Based):** Yes — `fee.fine.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "invoiceId": {
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
    "fineId": {
      "type": "string"
    },
    "amount": {
      "type": "number"
    },
    "basis": {
      "type": "string"
    }
  },
  "required": [
    "fineId",
    "amount",
    "basis"
  ]
}
```
- **Validation Rules:** Only applies after the configured grace period; the fine amount is bounded by the configured cap — never exceeds it ("a fine is added as a new line after a 7-day grace, capped at the configured maximum," FEE-007).
- **Error Codes:** `422 GRACE_PERIOD_ACTIVE`; `422 FINE_CAP_EXCEEDED`.
- **Business Rules Applied:** FEE-007 (late fines are distinct charges, bounded, never silent edits).
- **Pagination:** N/A.

### 9. Waive Fine (Governed)
- **Method:** `POST`
- **URL:** `/api/v1/dues/{studentId}/fines/{fineId}/waive`
- **Description:** Waive an applied late fine, routed to approval (SCR-FEE-10).
- **Authentication (Role-Based):** Yes — `fee.fine.manage` (request).
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
    "fineId": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "PENDING_APPROVAL"
    }
  },
  "required": [
    "fineId",
    "status"
  ]
}
```
- **Validation Rules:** `reason` required; a fine on a head flagged `nonWaivable` cannot be waived (C-03, enforced consistently with `21-discount-api.md`'s waiver gate).
- **Error Codes:** `409 NON_WAIVABLE_HEAD`; `422 REASON_REQUIRED`.
- **Business Rules Applied:** FEE-007, FEE-002 (non-waivable boundary), C-03.
- **Pagination:** N/A.

### 10. Request Refund
- **Method:** `POST`
- **URL:** `/api/v1/refunds`
- **Description:** Process a governed refund **decision** — Fee decides, Payment executes (SCR-FEE-12, SCR-FEE-16).
- **Authentication (Role-Based):** Yes — `fee.refund.process` (request).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "paymentId": {
      "type": "string"
    },
    "amount": {
      "type": "number"
    },
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "paymentId",
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
- **Validation Rules:** `amount` must be `≤` the amount actually paid ("Refund exceeds amount paid"); `reason` required.
- **Error Codes:** `422 EXCEEDS_PAID_AMOUNT`; `422 REASON_REQUIRED`.
- **Business Rules Applied:** FEE-008 (governed refunds), AUTHZ-009.
- **Pagination:** N/A.

### 11. List Refund / Waiver Approvals
- **Method:** `GET`
- **URL:** `/api/v1/refunds/approvals`
- **Description:** Inbox for pending refund decisions (SCR-FEE-16).
- **Authentication (Role-Based):** Yes — `fee.refund.approve`, scoped.
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
      "invoiceOrPaymentRef": {
        "type": "string"
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
      "slaDueAt": {
        "type": "string"
      },
      "status": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "invoiceOrPaymentRef",
      "studentId",
      "amount",
      "reason",
      "requestedBy",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Requester ≠ approver (AUTHZ-009) — self-raised refunds are listed but cannot be decided by the requester.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** FEE-008, AUTHZ-009.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 12. Decide Refund
- **Method:** `POST`
- **URL:** `/api/v1/refunds/approvals/{id}/decide`
- **Description:** Approve or reject a pending refund; on approval, the refund is **executed as a `20-payment-api.md` reversal** — never as an invoice or payment deletion.
- **Authentication (Role-Based):** Yes — `fee.refund.approve`; **blocked if the caller raised the request**; executing the resulting reversal additionally requires `payment.reverse` authority on the executing call.
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
    },
    "paymentReversalId": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** Caller ≠ requester; `reason` required on `REJECT`.
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED`; `422 REASON_REQUIRED`.
- **Business Rules Applied:** FEE-008, PAY-007 (reversal-not-delete execution), AUTHZ-009.
- **Pagination:** N/A.

## Arrears

### 13. Carry Forward Arrears
- **Method:** `POST`
- **URL:** `/api/v1/arrears/carry-forward`
- **Description:** Explicitly carry outstanding arrears into the next session, tied to session rollover (SCR-FEE-13).
- **Authentication (Role-Based):** Yes — `fee.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "fromSessionId": {
      "type": "string"
    },
    "toSessionId": {
      "type": "string"
    },
    "scope": {
      "type": "object",
      "properties": {
        "classId": {
          "type": "string"
        }
      }
    }
  },
  "required": [
    "fromSessionId",
    "toSessionId"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentsAffected": {
      "type": "number"
    },
    "totalArrearsCarried": {
      "type": "number"
    }
  },
  "required": [
    "studentsAffected",
    "totalArrearsCarried"
  ]
}
```
- **Validation Rules:** Carry-forward is an **explicit, recorded decision**, never silent — it is naturally invoked alongside `06-academic-session-api.md` #7's rollover, not a standalone background process.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** FEE-008 (explicit, recorded arrears carry-forward), SESS-007.
- **Pagination:** N/A.

## Reporting & Data Transfer

### 14. Dues / Collection / Aging Report
- **Method:** `GET`
- **URL:** `/api/v1/fee/report/dues-collection-aging`
- **Description:** Dues, collection, and aging report (SCR-FEE-14).
- **Authentication (Role-Based):** Yes — `fee.view` (report authority).
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
    }
  }
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "totalDue": {
      "type": "string"
    },
    "totalCollected": {
      "type": "string"
    },
    "agingBreakdown": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "bucket": {
            "type": "string"
          },
          "amount": {
            "type": "number"
          }
        },
        "required": [
          "bucket",
          "amount"
        ]
      }
    }
  },
  "required": [
    "totalDue",
    "totalCollected",
    "agingBreakdown"
  ]
}
```
- **Validation Rules:** Scope-limited; large reports run async per conventions §9.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** REP-002, FEE-007.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 15. Export Fee / Dues Data
- **Method:** `GET`
- **URL:** `/api/v1/fee/export`
- **Description:** Governed export of fee structures or dues data (SCR-FEE-15).
- **Authentication (Role-Based):** Yes — `fee.export`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "type": {
      "type": "string",
      "enum": [
        "STRUCTURES",
        "DUES"
      ]
    }
  },
  "required": [
    "type"
  ]
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
- **Validation Rules:** Scope-limited; export is audited.
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
