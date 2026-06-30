# Payment API

## 1. API Overview

**Purpose.** Record payments with gapless immutable receipts and deterministic allocation, manage clearance and reconciliation, track credit balances, and govern reversal/mis-post correction — payments are reversed, never deleted, and processing is idempotent throughout.

**Module Context.** Implements Business Rules Catalog Doc 18 (Payment) and UI Screen Spec `19-payment-screens.md` (SCR-PAY-01…16).

---

## 2. Endpoints

### 1. Get Student Dues
- **Method:** `GET`
- **URL:** `/api/v1/payments/student-dues`
- **Description:** Outstanding-dues summary for the payment counter screen (SCR-PAY-01).
- **Authentication (Role-Based):** Yes — `payment.record`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string"
    }
  },
  "required": [
    "studentId"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string"
    },
    "totalDue": {
      "type": "string"
    },
    "invoices": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "invoiceId": {
            "type": "string"
          },
          "head": {
            "type": "string"
          },
          "outstanding": {
            "type": "string"
          },
          "dueDate": {
            "type": "string"
          }
        },
        "required": [
          "invoiceId",
          "head",
          "outstanding",
          "dueDate"
        ]
      }
    }
  },
  "required": [
    "studentId",
    "totalDue",
    "invoices"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 STUDENT_NOT_FOUND`.
- **Business Rules Applied:** FEE-007 (dues basis).
- **Pagination:** N/A.

### 2. Preview Allocation
- **Method:** `POST`
- **URL:** `/api/v1/payments/preview-allocation`
- **Description:** Live preview of which dues a given amount would clear, before recording (SCR-PAY-01).
- **Authentication (Role-Based):** Yes — `payment.record`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string"
    },
    "amount": {
      "type": "number"
    }
  },
  "required": [
    "studentId",
    "amount"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "allocations": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "invoiceId": {
            "type": "string"
          },
          "outstanding": {
            "type": "string"
          },
          "allocated": {
            "type": "string"
          },
          "remaining": {
            "type": "string"
          }
        },
        "required": [
          "invoiceId",
          "outstanding",
          "allocated",
          "remaining"
        ]
      }
    },
    "unallocatedRemainder": {
      "type": "number"
    }
  },
  "required": [
    "allocations",
    "unallocatedRemainder"
  ]
}
```
- **Validation Rules:** Reflects the **same deterministic allocation policy** (oldest-due-first, or the configured policy, #20/#21) that the actual recording (#3) will apply.
- **Error Codes:** `422 AMOUNT_MUST_BE_POSITIVE`.
- **Business Rules Applied:** PAY-002 (deterministic payment allocation).
- **Pagination:** N/A.

### 3. Record Payment
- **Method:** `POST`
- **URL:** `/api/v1/payments`
- **Description:** Record a counter payment, issuing a gapless, immutable receipt, idempotently — the screen's own "★ money hot path" designation (UC-PAY-01, SCR-PAY-01).
- **Authentication (Role-Based):** Yes — `payment.record`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string"
    },
    "amount": {
      "type": "number"
    },
    "method": {
      "type": "string",
      "enum": [
        "CASH",
        "CHEQUE",
        "BANK_TRANSFER",
        "GATEWAY"
      ]
    },
    "reference": {
      "type": "string"
    },
    "idempotencyKey": {
      "type": "string"
    }
  },
  "required": [
    "studentId",
    "amount",
    "method",
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
    "receiptNumber": {
      "type": "string"
    },
    "allocations": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "invoiceId": {
            "type": "string"
          },
          "allocated": {
            "type": "string"
          }
        },
        "required": [
          "invoiceId",
          "allocated"
        ]
      }
    },
    "creditBalanceCreated": {
      "type": "number"
    },
    "status": {
      "type": "string",
      "enum": [
        "RECORDED",
        "PENDING_CLEARANCE"
      ]
    }
  },
  "required": [
    "id",
    "receiptNumber",
    "allocations",
    "status"
  ]
}
```
- **Validation Rules:** `amount > 0` ("Amount must be greater than zero"); **idempotent** — a replayed or double-submitted request with the same `idempotencyKey` is recognized and returns the **original** receipt, never a second credit ("This payment was already recorded," PAY-008); the receipt number is assigned from a **gapless, unique, immutable sequence** — never reused, never renumbered (PAY-001); allocation follows the deterministic policy (PAY-002); `CHEQUE`/`BANK_TRANSFER`/`GATEWAY` require a `reference` and enter `PENDING_CLEARANCE` rather than counting as settled (PAY-003/PAY-004); any amount beyond outstanding dues becomes a tracked credit balance, never silently absorbed (PAY-006).
- **Error Codes:** `422 AMOUNT_NOT_POSITIVE`; `422 REFERENCE_REQUIRED` (for non-cash methods); the idempotent-replay case is **not** an error — it returns `200` with the original receipt.
- **Business Rules Applied:** PAY-001 (gapless, immutable receipts), PAY-002 (deterministic allocation), PAY-003 (methods & reference capture), PAY-004 (clearance lifecycle for non-cash), PAY-006 (overpayment → credit balance), PAY-008 (idempotent processing — no double-credit).
- **Pagination:** N/A.

### 4. Record Online Payment (Guardian)
- **Method:** `POST`
- **URL:** `/api/v1/payments/online`
- **Description:** Let a financial-responsible guardian pay dues online for their own linked child (SCR-PAY-02).
- **Authentication (Role-Based):** Yes — self (guardian, financial-responsible role on the link, own child only — GRD-N-004/GRD-N-005).
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
    "amount": {
      "type": "number"
    },
    "idempotencyKey": {
      "type": "string"
    }
  },
  "required": [
    "studentId",
    "amount",
    "idempotencyKey"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "paymentSessionRef": {
      "type": "string"
    }
  },
  "required": [
    "paymentSessionRef"
  ]
}
```
- **Validation Rules:** `studentId` must be one of the caller's own linked children (GRD-N-004); the caller must hold the `isFinancialResponsible` role on that link (GRD-N-005); same idempotency guarantee as #3 — a gateway retry never double-charges.
- **Error Codes:** `403 NOT_FINANCIAL_RESPONSIBLE`; `404 NOT_LINKED_CHILD`.
- **Business Rules Applied:** PAY-008, PAY-001, GRD-N-004, GRD-N-005.
- **Pagination:** N/A.

## Receipts

### 5. Get Receipt
- **Method:** `GET`
- **URL:** `/api/v1/payments/{id}/receipt`
- **Description:** View an immutable receipt (SCR-PAY-03).
- **Authentication (Role-Based):** Yes — `payment.view`, scoped/ownership.
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
    "receiptNumber": {
      "type": "integer"
    },
    "studentId": {
      "type": "string"
    },
    "amount": {
      "type": "number"
    },
    "method": {
      "type": "string"
    },
    "allocations": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    },
    "status": {
      "type": "string",
      "enum": [
        "VALID",
        "REVERSED"
      ]
    },
    "issuedAt": {
      "type": "string"
    }
  },
  "required": [
    "receiptNumber",
    "studentId",
    "amount",
    "method",
    "allocations",
    "status",
    "issuedAt"
  ]
}
```
- **Validation Rules:** A reversed receipt's number and original content are still fully visible — reversal never deletes or hides the record, it is marked (PAY-001/PAY-007).
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** PAY-001, FILE-007.
- **Pagination:** N/A.

### 6. Get Receipt Download URL
- **Method:** `GET`
- **URL:** `/api/v1/payments/{id}/receipt/download-url`
- **Description:** Short-lived signed URL to download/print the receipt.
- **Authentication (Role-Based):** Yes — `payment.view`, scoped/ownership.
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
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_AVAILABLE`.
- **Business Rules Applied:** FILE-005.
- **Pagination:** N/A.

### 7. Reprint Receipt
- **Method:** `POST`
- **URL:** `/api/v1/payments/{id}/receipt/reprint`
- **Description:** Regenerate the receipt document — always the **identical** artifact (SCR-PAY-03).
- **Authentication (Role-Based):** Yes — `payment.view`.
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
- **Validation Rules:** A reprint never alters the underlying receipt record.
- **Error Codes:** `404 NOT_AVAILABLE`.
- **Business Rules Applied:** FILE-007 (immutable, reproducible).
- **Pagination:** N/A.

## Payment List, Detail & Allocation

### 8. List Payments
- **Method:** `GET`
- **URL:** `/api/v1/payments`
- **Description:** Browse/search payments (SCR-PAY-04).
- **Authentication (Role-Based):** Yes — `payment.view`, scoped/ownership.
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
    "statusRECORDEDPENDIN": {
      "type": "string",
      "description": "status? (RECORDED|PENDING_CLEARANCE|CLEARED|BOUNCED|REVERSED|ARCHIVED)"
    },
    "method": {
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
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "id": {
        "type": "string"
      },
      "receiptNumber": {
        "type": "integer"
      },
      "studentId": {
        "type": "string"
      },
      "amount": {
        "type": "number"
      },
      "method": {
        "type": "string"
      },
      "status": {
        "type": "string"
      },
      "recordedAt": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "receiptNumber",
      "studentId",
      "amount",
      "method",
      "status",
      "recordedAt"
    ]
  }
}
```
- **Validation Rules:** Scope-limited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-002, REP-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 9. Get Payment Detail
- **Method:** `GET`
- **URL:** `/api/v1/payments/{id}`
- **Description:** Full payment history — allocation, clearance, reversals (SCR-PAY-05).
- **Authentication (Role-Based):** Yes — `payment.view`.
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
    "receiptNumber": {
      "type": "integer"
    },
    "studentId": {
      "type": "string"
    },
    "amount": {
      "type": "number"
    },
    "method": {
      "type": "string"
    },
    "reference": {
      "type": "string"
    },
    "allocations": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    },
    "clearanceHistory": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    },
    "reversals": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    },
    "status": {
      "type": "string"
    },
    "recordedAt": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "receiptNumber",
    "studentId",
    "amount",
    "method",
    "allocations",
    "clearanceHistory",
    "reversals",
    "status",
    "recordedAt"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** PAY-002, PAY-004, PAY-007.
- **Pagination:** N/A.

### 10. Get Payment Allocation
- **Method:** `GET`
- **URL:** `/api/v1/payments/{id}/allocation`
- **Description:** View the deterministic allocation of a payment to dues (SCR-PAY-06).
- **Authentication (Role-Based):** Yes — `payment.view`.
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
    "allocations": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "invoiceId": {
            "type": "string"
          },
          "head": {
            "type": "string"
          },
          "allocated": {
            "type": "string"
          }
        },
        "required": [
          "invoiceId",
          "head",
          "allocated"
        ]
      }
    }
  },
  "required": [
    "allocations"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** PAY-002.
- **Pagination:** N/A.

### 11. Re-allocate Payment (Governed)
- **Method:** `POST`
- **URL:** `/api/v1/payments/{id}/reallocate`
- **Description:** Adjust a payment's allocation across dues, audited (SCR-PAY-06).
- **Authentication (Role-Based):** Yes — `payment.record` or `payment.correct`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "allocations": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "invoiceId": {
            "type": "string"
          },
          "amount": {
            "type": "number"
          }
        },
        "required": [
          "invoiceId",
          "amount"
        ]
      }
    },
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "allocations",
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
    "allocations": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    }
  },
  "required": [
    "id",
    "allocations"
  ]
}
```
- **Validation Rules:** `reason` required; the sum of allocations must not exceed the payment amount; this is an audited change to allocation only — it never alters the recorded payment amount or receipt.
- **Error Codes:** `422 ALLOCATION_EXCEEDS_PAYMENT`; `422 REASON_REQUIRED`.
- **Business Rules Applied:** PAY-002 (deterministic allocation, changes audited).
- **Pagination:** N/A.

## Clearance

### 12. Update Clearance Status
- **Method:** `POST`
- **URL:** `/api/v1/payments/{id}/clearance`
- **Description:** Confirm a non-cash payment cleared or mark it bounced (SCR-PAY-07).
- **Authentication (Role-Based):** Yes — `payment.record` or `payment.reconcile`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "status": {
      "type": "string",
      "enum": [
        "CLEARED",
        "BOUNCED"
      ]
    }
  },
  "required": [
    "status"
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
        "CLEARED",
        "BOUNCED"
      ]
    },
    "reversalTriggered": {
      "type": "boolean"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** Only from `PENDING_CLEARANCE`; `CLEARED` finalizes the allocation and settles the dues; `BOUNCED` automatically **triggers a reversal** (#16) — it never simply marks the payment failed and leaves dues in a stale provisional state ("bounce reverses, never deletes," PAY-004/PAY-007); a configured bounce fine may be added as a distinct charge.
- **Error Codes:** `409 NOT_PENDING_CLEARANCE`.
- **Business Rules Applied:** PAY-004 (clearance lifecycle), PAY-007 (bounce triggers reversal, not deletion).
- **Pagination:** N/A.

### 13. Bulk Clear Payments
- **Method:** `POST`
- **URL:** `/api/v1/payments/clearance/bulk-clear`
- **Description:** Mark many pending payments cleared at once (SCR-PAY-07).
- **Authentication (Role-Based):** Yes — `payment.reconcile`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "paymentIds": {
      "type": "array",
      "items": {
        "type": "string"
      }
    }
  },
  "required": [
    "paymentIds"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "cleared": {
      "type": "number"
    }
  },
  "required": [
    "cleared"
  ]
}
```
- **Validation Rules:** Same as #12 applied per payment.
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** PAY-004.
- **Pagination:** N/A.

## Reconciliation

### 14. Import Settlement Data
- **Method:** `POST`
- **URL:** `/api/v1/payments/reconciliation/import-settlement`
- **Description:** Import a bank/gateway settlement report for matching (SCR-PAY-08).
- **Authentication (Role-Based):** Yes — `payment.reconcile`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "multipartformdatafi": {
      "type": "string",
      "description": "multipart/form-data { file }"
    }
  }
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "null",
  "description": "202 Accepted { jobId } (large settlement files matched asynchronously per the approved screen's own deviation note)."
}
```
- **Validation Rules:** None beyond access; the import itself does not write any payment record — only the settlement reference data for subsequent matching.
- **Error Codes:** `422 INVALID_FILE_FORMAT`.
- **Business Rules Applied:** PAY-005 (reconciliation against settlements).
- **Pagination:** N/A.

### 15. Auto-Match Settlements
- **Method:** `POST`
- **URL:** `/api/v1/payments/reconciliation/auto-match`
- **Description:** Automatically match recorded payments to imported settlement transactions by reference/amount (SCR-PAY-08).
- **Authentication (Role-Based):** Yes — `payment.reconcile`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "settlementBatchId": {
      "type": "string"
    }
  },
  "required": [
    "settlementBatchId"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "matched": {
      "type": "number"
    },
    "unmatched": {
      "type": "number"
    },
    "discrepancies": {
      "type": "number"
    }
  },
  "required": [
    "matched",
    "unmatched",
    "discrepancies"
  ]
}
```
- **Validation Rules:** Only exact reference/amount matches are auto-resolved; anything ambiguous is left as a discrepancy for manual resolution (#17) — never silently force-matched.
- **Error Codes:** `404 BATCH_NOT_FOUND`.
- **Business Rules Applied:** PAY-005 (reconciliation; discrepancies flagged and governed, never silent).
- **Pagination:** N/A.

### 16. List Reconciliation Discrepancies
- **Method:** `GET`
- **URL:** `/api/v1/payments/reconciliation/discrepancies`
- **Description:** View flagged mismatches — missing, extra, or amount-differing transactions (SCR-PAY-08).
- **Authentication (Role-Based):** Yes — `payment.reconcile`.
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
    "typeMISSINGEXTRAAMO": {
      "type": "string",
      "description": "type? (MISSING|EXTRA|AMOUNT_MISMATCH)"
    },
    "statusOPENRESOLVED": {
      "type": "string",
      "description": "status? (OPEN|RESOLVED)"
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
        "type": "string"
      },
      "recordedPaymentId": {
        "type": "string"
      },
      "settlementRef": {
        "type": "string"
      },
      "recordedAmount": {
        "type": "number"
      },
      "settledAmount": {
        "type": "number"
      },
      "status": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "type",
      "status"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** PAY-005.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 17. Resolve Reconciliation Discrepancy
- **Method:** `POST`
- **URL:** `/api/v1/payments/reconciliation/discrepancies/{id}/resolve`
- **Description:** Resolve a flagged discrepancy via a governed adjustment (SCR-PAY-08).
- **Authentication (Role-Based):** Yes — `payment.reconcile`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "resolution": {
      "type": "string",
      "enum": [
        "MATCHED_MANUALLY",
        "INVESTIGATE",
        "ALLOCATE_AS_NEW_PAYMENT",
        "FLAG_FOR_REFUND"
      ]
    },
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "resolution",
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
      "const": "RESOLVED"
    },
    "resolution": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "status",
    "resolution"
  ]
}
```
- **Validation Rules:** `reason` required; resolution is itself audited — discrepancies are never silently auto-cleared (PAY-005).
- **Error Codes:** `422 REASON_REQUIRED`.
- **Business Rules Applied:** PAY-005 (governed resolution, never silent).
- **Pagination:** N/A.

## Overpayment / Credit Balance

### 18. List Credit Balances
- **Method:** `GET`
- **URL:** `/api/v1/payments/credit-balances`
- **Description:** View per-student credit balances arising from overpayment (SCR-PAY-09).
- **Authentication (Role-Based):** Yes — `payment.record`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
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
      "studentId": {
        "type": "string"
      },
      "balance": {
        "type": "number"
      },
      "history": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "source": {
              "type": "string",
              "enum": [
                "OVERPAYMENT",
                "REFUND_OFFSET"
              ]
            },
            "amount": {
              "type": "number"
            },
            "date": {
              "type": "string"
            }
          },
          "required": [
            "source",
            "amount",
            "date"
          ]
        }
      }
    },
    "required": [
      "studentId",
      "balance",
      "history"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** PAY-006 (overpayment becomes credit balance — never lost or silently absorbed).
- **Pagination:** N/A.

### 19. Apply Credit Balance to Dues
- **Method:** `POST`
- **URL:** `/api/v1/payments/credit-balances/{studentId}/apply`
- **Description:** Apply a student's credit balance to current/future dues, transparently (SCR-PAY-09).
- **Authentication (Role-Based):** Yes — `payment.record`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "invoiceId": {
      "type": "string",
      "description": "omit to auto-apply per allocation policy"
    },
    "amount": {
      "type": "number"
    }
  }
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
    "remainingBalance": {
      "type": "number"
    }
  },
  "required": [
    "applied",
    "remainingBalance"
  ]
}
```
- **Validation Rules:** `amount` cannot exceed the available balance.
- **Error Codes:** `422 INSUFFICIENT_BALANCE`.
- **Business Rules Applied:** PAY-006.
- **Pagination:** N/A.

## Reversal & Mis-Post Correction (Governed)

### 20. Reverse Payment
- **Method:** `POST`
- **URL:** `/api/v1/payments/{id}/reverse`
- **Description:** Reverse a payment that failed or was recorded in error — **never a deletion** (UC-PAY-02, SCR-PAY-10).
- **Authentication (Role-Based):** Yes — `payment.reverse` (elevated, request).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "amount": {
      "type": "number",
      "description": "full if omitted"
    },
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
      "const": "PENDING_APPROVAL"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** `reason` required; the original payment record and receipt are never deletable through this or any other endpoint — **no delete path exists** ("delete payment is never offered," UC-PAY-017); on eventual approval the reversal is created as a distinct, linked entry and the affected dues are restored.
- **Error Codes:** `422 REASON_REQUIRED`; `409 ALREADY_REVERSED`.
- **Business Rules Applied:** PAY-007 (payments are reversed, never deleted).
- **Pagination:** N/A.

### 21. Correct Mis-Posted Payment
- **Method:** `POST`
- **URL:** `/api/v1/payments/{id}/correct`
- **Description:** Correct a payment recorded against the wrong student/invoice via a governed transfer (UC PAY-09, SCR-PAY-11).
- **Authentication (Role-Based):** Yes — `payment.correct` (elevated, request).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "correctTargetStudentId": {
      "type": "string"
    },
    "correctTargetInvoiceId": {
      "type": "string"
    },
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "correctTargetStudentId",
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
- **Validation Rules:** `reason` required; this is **not** a silent re-pointing — on approval, both the source and target ledgers reflect the move transparently, with the original payment record preserved and a linked transfer entry created.
- **Error Codes:** `422 REASON_REQUIRED`.
- **Business Rules Applied:** PAY-009 (mis-posted payment correction is governed, not silent).
- **Pagination:** N/A.

### 22. List Reversal / Correction Approvals
- **Method:** `GET`
- **URL:** `/api/v1/payments/reversal-approvals`
- **Description:** Inbox for pending reversals and mis-post corrections (SCR-PAY-16).
- **Authentication (Role-Based):** Yes — `payment.reversal.approve`, scoped.
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
    "typeREVERSALMISPOST_": {
      "type": "string",
      "description": "type? (REVERSAL|MISPOST_CORRECTION)"
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
        "type": "string"
      },
      "paymentId": {
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
      "type",
      "paymentId",
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
- **Validation Rules:** Requester ≠ approver — self-raised requests are listed but disabled for decision by the requester (AUTHZ-009).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-009, PAY-007, PAY-009.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 23. Decide Reversal / Correction
- **Method:** `POST`
- **URL:** `/api/v1/payments/reversal-approvals/{id}/decide`
- **Description:** Approve or reject a pending reversal or mis-post correction; on approval, the linked reversal/transfer entry is committed, dues are restored, and the original is preserved.
- **Authentication (Role-Based):** Yes — `payment.reversal.approve`; **blocked if the caller raised the request**.
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
- **Validation Rules:** Caller ≠ requester.
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED`.
- **Business Rules Applied:** AUTHZ-009, PAY-007, PAY-009.
- **Pagination:** N/A.

## Configuration

### 24. Get Method / Allocation Configuration
- **Method:** `GET`
- **URL:** `/api/v1/payments/config/methods-allocation`
- **Description:** View configured payment methods and the allocation rule (SCR-PAY-12).
- **Authentication (Role-Based):** Yes — `payment.config.manage`.
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
    "enabledMethods": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "allocationRule": {
      "type": "string",
      "enum": [
        "OLDEST_FIRST",
        "SPECIFIC_INVOICE",
        "HEAD_PRIORITY"
      ]
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "enabledMethods",
    "allocationRule",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** PAY-002, PAY-003.
- **Pagination:** N/A.

### 25. Set Method / Allocation Configuration
- **Method:** `PUT`
- **URL:** `/api/v1/payments/config/methods-allocation`
- **Description:** Configure enabled payment methods and the deterministic allocation rule, versioned (SCR-PAY-12).
- **Authentication (Role-Based):** Yes — `payment.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "enabledMethods": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "allocationRule": {
      "type": "string"
    },
    "version": {
      "type": "number"
    }
  },
  "required": [
    "enabledMethods",
    "allocationRule",
    "version"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "enabledMethods": {
      "type": "string"
    },
    "allocationRule": {
      "type": "string"
    },
    "version": {
      "type": "number"
    }
  },
  "required": [
    "enabledMethods",
    "allocationRule",
    "version"
  ]
}
```
- **Validation Rules:** The allocation rule must be one of the defined, deterministic options — never left undefined (PAY-002); `version` optimistic lock.
- **Error Codes:** `422 ALLOCATION_RULE_UNDEFINED`; `409 VERSION_CONFLICT`.
- **Business Rules Applied:** PAY-002, CFG-004.
- **Pagination:** N/A.

## Reporting & Data Transfer

### 26. Collection & Reconciliation Report
- **Method:** `GET`
- **URL:** `/api/v1/payments/report/collection-reconciliation`
- **Description:** Collection by method and reconciliation status (SCR-PAY-13).
- **Authentication (Role-Based):** Yes — `payment.view` (report authority).
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
    "method": {
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
    "byMethod": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "method": {
            "type": "string"
          },
          "collected": {
            "type": "string"
          }
        },
        "required": [
          "method",
          "collected"
        ]
      }
    },
    "reconciliationStatus": {
      "type": "object",
      "properties": {
        "matched": {
          "type": "string"
        },
        "unmatched": {
          "type": "string"
        },
        "exceptions": {
          "type": "string"
        }
      },
      "required": [
        "matched",
        "unmatched",
        "exceptions"
      ]
    }
  },
  "required": [
    "byMethod",
    "reconciliationStatus"
  ]
}
```
- **Validation Rules:** Scope-limited; large reports run async per conventions §9.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** REP-002, PAY-005.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 27. Bulk Payment Import
- **Method:** `POST`
- **URL:** `/api/v1/payments/bulk-import`
- **Description:** Bulk-import payments with idempotency (SCR-PAY-14).
- **Authentication (Role-Based):** Yes — `payment.import`.
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
              "studentId": {
                "type": "string"
              },
              "amount": {
                "type": "number"
              },
              "method": {
                "type": "string"
              },
              "reference": {
                "type": "string"
              },
              "idempotencyKey": {
                "type": "string"
              }
            },
            "required": [
              "studentId",
              "amount",
              "method",
              "idempotencyKey"
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
              "paymentId": {
                "type": "string"
              }
            },
            "required": [
              "row",
              "paymentId"
            ]
          }
        },
        "skippedDuplicates": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "row": {
                "type": "string"
              },
              "existingPaymentId": {
                "type": "string"
              }
            },
            "required": [
              "row",
              "existingPaymentId"
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
        "skippedDuplicates",
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
- **Validation Rules:** Each row validated against #3's rules (`amount > 0`, allocation policy); duplicate rows (by idempotency key) are **idempotently skipped**, never double-credited — this is reported distinctly from a true rejection.
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** PAY-008 (no double-credit on import), PAY-002; conventions §9.
- **Pagination:** N/A.

### 28. Export Payment Data
- **Method:** `GET`
- **URL:** `/api/v1/payments/export`
- **Description:** Governed export of payment data (SCR-PAY-15).
- **Authentication (Role-Based):** Yes — `payment.export`.
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
    "status": {
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
