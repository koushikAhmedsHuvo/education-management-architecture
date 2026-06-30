# Scholarship API

## 1. API Overview

**Purpose.** Govern funded, slot-bounded scholarship programs — conflict-of-interest-controlled selection separated from approval, fund-checked coverage/stipend disbursement with a no-negative-fee rule, sponsor-consent-scoped accountability, and forward-only renewal/revocation.

**Module Context.** Implements Business Rules Catalog Doc 20 (Scholarship) and UI Screen Spec `21-scholarship-screens.md` (SCR-SCH-01…11).

---

## 2. Endpoints

### 1. List Scholarship Programs
- **Method:** `GET`
- **URL:** `/api/v1/scholarships/programs`
- **Description:** Browse scholarship programs with slots, fund, and sponsor (SCR-SCH-01).
- **Authentication (Role-Based):** Yes — `scholarship.view`, scoped.
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
    "typeMERITNEEDSPORTS": {
      "type": "string",
      "description": "type? (MERIT|NEED|SPORTS|SPONSORED|FULL|PARTIAL)"
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
      "name": {
        "type": "string"
      },
      "type": {
        "type": "string"
      },
      "slotsFilled": {
        "type": "string"
      },
      "slotsTotal": {
        "type": "string"
      },
      "fundSource": {
        "type": "string"
      },
      "sponsor": {
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
      "type",
      "slotsFilled",
      "slotsTotal",
      "fundSource",
      "status",
      "updatedAt"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** SCH-001 (structured, funded award, distinct from discount), AUTHZ-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Program Detail
- **Method:** `GET`
- **URL:** `/api/v1/scholarships/programs/{id}`
- **Description:** Full program definition — criteria, coverage, fund, renewal conditions (SCR-SCH-02).
- **Authentication (Role-Based):** Yes — `scholarship.view`.
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
    "type": {
      "type": "string"
    },
    "eligibilityCriteria": {
      "type": "object"
    },
    "slots": {
      "type": "number"
    },
    "coverage": {
      "type": "object",
      "properties": {
        "type": {
          "type": "string",
          "enum": [
            "PERCENTAGE",
            "AMOUNT",
            "STIPEND"
          ]
        },
        "value": {
          "type": "string"
        },
        "coveredHeads": {
          "type": "array",
          "items": {
            "type": "string"
          }
        }
      },
      "required": [
        "type",
        "value"
      ]
    },
    "fundSource": {
      "type": "string"
    },
    "sponsor": {
      "type": "string"
    },
    "renewalConditions": {
      "type": "object"
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
    "id",
    "name",
    "type",
    "eligibilityCriteria",
    "slots",
    "coverage",
    "fundSource",
    "renewalConditions",
    "status",
    "createdAt",
    "updatedAt",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** SCH-001, SCH-002, SCH-005.
- **Pagination:** N/A.

### 3. Define Scholarship Program
- **Method:** `POST`
- **URL:** `/api/v1/scholarships/programs`
- **Description:** Define a funded, slot-bounded scholarship program (UC-SCH-01, SCR-SCH-02).
- **Authentication (Role-Based):** Yes — `scholarship.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "type": {
      "type": "string",
      "enum": [
        "MERIT",
        "NEED",
        "SPORTS",
        "SPONSORED",
        "FULL",
        "PARTIAL"
      ]
    },
    "eligibilityCriteria": {
      "type": "object"
    },
    "slots": {
      "type": "number"
    },
    "coverage": {
      "type": "object",
      "properties": {
        "type": {
          "type": "string"
        },
        "value": {
          "type": "string"
        },
        "coveredHeads": {
          "type": "array",
          "items": {
            "type": "string"
          }
        }
      },
      "required": [
        "type",
        "value"
      ]
    },
    "fundSource": {
      "type": "string"
    },
    "sponsor": {
      "type": "string"
    },
    "renewalConditions": {
      "type": "object"
    }
  },
  "required": [
    "name",
    "type",
    "eligibilityCriteria",
    "slots",
    "coverage",
    "fundSource"
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
      "const": "ACTIVE"
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
    "status",
    "createdAt",
    "version"
  ]
}
```
- **Validation Rules:** All program fields must be complete before activation — type, criteria, coverage, funding source, slots, and renewal conditions (SCH-001); `slots` required and `≥ 0`; the program is modeled and classified as its own entity, **distinct** from a discount or waiver in records and reporting even though fee-coverage mechanics resemble a funded discount (SCH-001); `fundSource` is recorded and required (SCH-005).
- **Error Codes:** `422 INCOMPLETE_PROGRAM`.
- **Business Rules Applied:** SCH-001 (structured, funded award), SCH-002 (slots), SCH-005 (fund/sponsor linkage).
- **Pagination:** N/A.

### 4. Edit Scholarship Program
- **Method:** `PUT`
- **URL:** `/api/v1/scholarships/programs/{id}`
- **Description:** Edit program criteria, slots, coverage, or renewal conditions (SCR-SCH-02).
- **Authentication (Role-Based):** Yes — `scholarship.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "eligibilityCriteria": {
      "type": "string"
    },
    "slots": {
      "type": "string"
    },
    "coverage": {
      "type": "string"
    },
    "renewalConditions": {
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
  "type": "object",
  "description": "Updated detail (as #2).",
  "$ref": "#endpoint-2"
}
```
- **Validation Rules:** Reducing `slots` below the current `slotsFilled` is blocked; `version` optimistic lock.
- **Error Codes:** `409 SLOTS_BELOW_FILLED`; `409 VERSION_CONFLICT`.
- **Business Rules Applied:** SCH-002.
- **Pagination:** N/A.

## Eligibility & Shortlist

### 5. Get Eligibility Shortlist
- **Method:** `GET`
- **URL:** `/api/v1/scholarships/programs/{id}/shortlist`
- **Description:** View the ranked shortlist of eligible candidates (SCR-SCH-03).
- **Authentication (Role-Based):** Yes — `scholarship.select`.
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
      "rank": {
        "type": "integer"
      },
      "studentId": {
        "type": "string"
      },
      "studentName": {
        "type": "string"
      },
      "eligibilityScore": {
        "type": "number"
      },
      "selected": {
        "type": "boolean"
      },
      "coiFlag": {
        "type": "boolean"
      }
    },
    "required": [
      "rank",
      "studentId",
      "studentName",
      "eligibilityScore",
      "selected",
      "coiFlag"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** SCH-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 6. Evaluate & Rank Shortlist
- **Method:** `POST`
- **URL:** `/api/v1/scholarships/programs/{id}/shortlist/evaluate`
- **Description:** Evaluate eligibility against the program's defined, transparent criteria and produce a fair, ranked shortlist (SCR-SCH-03).
- **Authentication (Role-Based):** Yes — `scholarship.select`.
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
    "evaluated": {
      "type": "number"
    },
    "shortlisted": {
      "type": "number"
    }
  },
  "required": [
    "evaluated",
    "shortlisted"
  ]
}
```
- **Validation Rules:** Eligibility criteria are defined and applied consistently — never an ad hoc judgment call at this stage (SCH-002); ranking basis is recorded for audit.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** SCH-002 (defined eligibility & fair, slot-bounded selection).
- **Pagination:** N/A.

## Governed Selection

### 7. Submit Selection
- **Method:** `POST`
- **URL:** `/api/v1/scholarships/selections`
- **Description:** Select awardees from the shortlist within slot bounds, with conflict-of-interest recusal enforced — selection ≠ approval (UC-SCH-01, SCR-SCH-04, the screen's own "★" designation).
- **Authentication (Role-Based):** Yes — `scholarship.select` (committee member).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "programId": {
      "type": "string"
    },
    "selectedStudentIds": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "recusals": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "committeeMemberId": {
            "type": "string"
          },
          "reason": {
            "type": "string"
          }
        },
        "required": [
          "committeeMemberId",
          "reason"
        ]
      }
    }
  },
  "required": [
    "programId",
    "selectedStudentIds"
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
    },
    "slotsUsed": {
      "type": "number"
    },
    "slotsAvailable": {
      "type": "number"
    }
  },
  "required": [
    "id",
    "status",
    "slotsUsed",
    "slotsAvailable"
  ]
}
```
- **Validation Rules:** The number of `selectedStudentIds` cannot exceed the program's available slots — over-slot selection is hard-blocked, never partially honored ("All slots are filled," SCH-002); a committee member with a declared conflict of interest **must recuse** and cannot participate in that selection decision ("You have a declared conflict and must recuse," SCH-003); selection is always a distinct step from approval — even an unconflicted selector cannot also be the approver (#9, SoD, AUTHZ-009).
- **Error Codes:** `409 SLOTS_EXCEEDED`; `403 COI_RECUSAL_REQUIRED`.
- **Business Rules Applied:** SCH-002 (slot-bounded fair selection), SCH-003 (governed selection & conflict-of-interest controls), AUTHZ-009 (SoD — selection ≠ approval).
- **Pagination:** N/A.

### 8. List Selection / Renewal Approvals
- **Method:** `GET`
- **URL:** `/api/v1/scholarships/selection-approvals`
- **Description:** Inbox for pending selections, renewals, and disbursement decisions (SCR-SCH-11).
- **Authentication (Role-Based):** Yes — `scholarship.approve`, scoped.
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
    "typeSELECTIONRENEWAL": {
      "type": "string",
      "description": "type? (SELECTION|RENEWAL|DISBURSEMENT)"
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
      "programId": {
        "type": "string"
      },
      "summary": {
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
      "programId",
      "summary",
      "requestedBy",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** A selecting or conflicted committee member sees the request listed but cannot decide it themselves (SCH-003/AUTHZ-009).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** SCH-003, AUTHZ-009.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 9. Decide Selection / Renewal Request
- **Method:** `POST`
- **URL:** `/api/v1/scholarships/selection-approvals/{id}/decide`
- **Description:** Approve or reject a pending selection, renewal, or disbursement; on approval of a selection, awards transition to `AWARDED`/`ACTIVE`.
- **Authentication (Role-Based):** Yes — `scholarship.approve`; **blocked if the caller participated in the selection or is otherwise conflicted on this decision**.
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
- **Validation Rules:** Caller is not a selector and is not COI-flagged for this request.
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED`; `403 COI_BLOCKED`.
- **Business Rules Applied:** SCH-003, AUTHZ-009.
- **Pagination:** N/A.

## Coverage & Disbursement

### 10. Apply Fee Coverage
- **Method:** `POST`
- **URL:** `/api/v1/scholarships/awards/{id}/coverage`
- **Description:** Apply the awarded fee coverage as a transparent reduction (functions like a funded discount) (UC-SCH-02, SCR-SCH-05).
- **Authentication (Role-Based):** Yes — `scholarship.disburse` (separated by SoD from selection, SCH-003).
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
    "awardId": {
      "type": "string"
    },
    "invoiceId": {
      "type": "string"
    },
    "coverageApplied": {
      "type": "number"
    },
    "net": {
      "type": "number"
    }
  },
  "required": [
    "awardId",
    "invoiceId",
    "coverageApplied",
    "net"
  ]
}
```
- **Validation Rules:** Coverage applies as a **transparent reduction** up to the covered heads, never silently; the net fee can **never go negative** — any benefit beyond the fee amount is routed to an explicit stipend disbursement instead, not a negative charge (SCH-004); interaction with any existing discount on the same charge follows the configured interaction policy and combined cap (`21-discount-api.md` #6, SCH-006).
- **Error Codes:** `422 NET_BELOW_ZERO` (excess routed to stipend instead).
- **Business Rules Applied:** SCH-004 (coverage, stipend & no-negative-fee rule), SCH-006 (scholarship + discount interaction).
- **Pagination:** N/A.

### 11. Disburse Stipend
- **Method:** `POST`
- **URL:** `/api/v1/scholarships/awards/{id}/stipend`
- **Description:** Schedule or execute a stipend payout, drawing on the program's tracked fund (UC-SCH-02, SCR-SCH-05).
- **Authentication (Role-Based):** Yes — `scholarship.disburse` (request; high-value disbursements route to approval).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "amount": {
      "type": "number"
    },
    "period": {
      "type": "string"
    }
  },
  "required": [
    "amount"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "awardId": {
      "type": "string"
    },
    "amount": {
      "type": "number"
    },
    "status": {
      "type": "string",
      "enum": [
        "DISBURSED",
        "PENDING_APPROVAL"
      ]
    },
    "fundBalanceAfter": {
      "type": "number"
    }
  },
  "required": [
    "awardId",
    "amount",
    "status",
    "fundBalanceAfter"
  ]
}
```
- **Validation Rules:** The program's fund balance must be `≥` the disbursement amount — over-fund disbursement is **blocked and flagged before it happens**, never allowed to overdraw ("a fund shortfall blocks disbursement before over-spend," SCH-005); the fund ledger is decremented and tracked against the sponsor on every disbursement.
- **Error Codes:** `409 INSUFFICIENT_FUND_BALANCE`.
- **Business Rules Applied:** SCH-004, SCH-005 (fund tracking & sponsor accountability).
- **Pagination:** N/A.

## Fund & Sponsor Accountability

### 12. Get Fund & Commitments
- **Method:** `GET`
- **URL:** `/api/v1/scholarships/programs/{id}/fund`
- **Description:** View fund balance, commitments, and disbursement history for a program (SCR-SCH-07).
- **Authentication (Role-Based):** Yes — `scholarship.config.manage`.
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
    "balance": {
      "type": "number"
    },
    "committed": {
      "type": "number"
    },
    "disbursed": {
      "type": "number"
    },
    "commitments": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "awardId": {
            "type": "string"
          },
          "studentId": {
            "type": "string"
          },
          "amount": {
            "type": "number"
          }
        },
        "required": [
          "awardId",
          "studentId",
          "amount"
        ]
      }
    }
  },
  "required": [
    "balance",
    "committed",
    "disbursed",
    "commitments"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** SCH-005 (fund tracking).
- **Pagination:** N/A.

### 13. Get Sponsor Report
- **Method:** `GET`
- **URL:** `/api/v1/scholarships/programs/{id}/sponsor-report`
- **Description:** A sponsor-scoped accountability report summarizing awards and spend (SCR-SCH-07).
- **Authentication (Role-Based):** Yes — `scholarship.sponsor.view` (the sponsor's own scoped view, strictly limited to **their** funded awards).
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
    "programId": {
      "type": "string"
    },
    "totalCommitted": {
      "type": "string"
    },
    "totalDisbursed": {
      "type": "string"
    },
    "awardSummaries": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "anonymizedorconsent": {
            "type": "string",
            "description": "/* anonymized or consented-scope student reference */"
          }
        }
      }
    }
  },
  "required": [
    "programId",
    "totalCommitted",
    "totalDisbursed",
    "awardSummaries"
  ]
}
```
- **Validation Rules:** Sponsor-facing data is shared **only within the consented scope** — minors' identifying data is never exposed beyond what consent permits ("a donor receives a report... with student data shared only per consent," SCH-005).
- **Error Codes:** `403 FORBIDDEN` (sponsor scoped to their own programs only).
- **Business Rules Applied:** SCH-005 (sponsor accountability respecting consent), compliance (minors' data).
- **Pagination:** N/A.

## Renewal & Revocation

### 14. Renew Award
- **Method:** `POST`
- **URL:** `/api/v1/scholarships/awards/{id}/renew`
- **Description:** Continue an award after its performance conditions (GPA/attendance/conduct) are confirmed met at a checkpoint (UC-SCH-03, SCR-SCH-06).
- **Authentication (Role-Based):** Yes — `scholarship.lifecycle.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "checkpointBasis": {
      "type": "object"
    }
  },
  "required": [
    "checkpointBasis"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "awardId": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "RENEWED"
    }
  },
  "required": [
    "awardId",
    "status"
  ]
}
```
- **Validation Rules:** Renewal is **condition-based** — evaluated against the program's defined renewal criteria at the checkpoint (SCH-007); the action **appends to** the award's history, it never rewrites or removes a past disbursement record (SCH-008).
- **Error Codes:** `422 CONDITIONS_NOT_MET`.
- **Business Rules Applied:** SCH-007 (condition-based, governed renewal), SCH-008 (history never rewritten).
- **Pagination:** N/A.

### 15. Revoke Award
- **Method:** `POST`
- **URL:** `/api/v1/scholarships/awards/{id}/revoke`
- **Description:** End an award for unmet conditions or sponsor withdrawal, with due process (UC-SCH-03, SCR-SCH-06).
- **Authentication (Role-Based):** Yes — `scholarship.lifecycle.manage`; high-impact revocations route to approval.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "reason": {
      "type": "string"
    },
    "basis": {
      "type": "object"
    }
  },
  "required": [
    "reason",
    "basis"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "awardId": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "enum": [
        "REVOKED",
        "PROBATION",
        "PENDING_APPROVAL"
      ]
    }
  },
  "required": [
    "awardId",
    "status"
  ]
}
```
- **Validation Rules:** `reason` and `basis` required — there is **no silent revocation path** ("no silent revocation; notice/appeal per policy," SCH-007); revocation stops **future** disbursement only — it is forward-looking and **never retroactively claws back** a legitimately disbursed past benefit, except via a separate, explicitly governed fraud process ("a revoked scholarship stops next term's coverage but the current term already disbursed stands," SCH-008).
- **Error Codes:** `422 REASON_REQUIRED`.
- **Business Rules Applied:** SCH-007 (governed renewal/revocation, due process), SCH-008 (forward-looking, history preserved, no silent clawback).
- **Pagination:** N/A.

## Reporting, Bulk & Data Transfer

### 16. Award & Fund Utilization Report
- **Method:** `GET`
- **URL:** `/api/v1/scholarships/report/award-fund-utilization`
- **Description:** Report award and fund utilization, sponsor-aware (SCR-SCH-08).
- **Authentication (Role-Based):** Yes — `scholarship.view` or `scholarship.sponsor.view` (scoped accordingly).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "programId": {
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
    "slotsFilled": {
      "type": "string"
    },
    "slotsAvailable": {
      "type": "string"
    },
    "fundCommitted": {
      "type": "string"
    },
    "fundDisbursed": {
      "type": "string"
    },
    "byType": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "type": {
            "type": "string"
          },
          "count": {
            "type": "integer"
          },
          "amount": {
            "type": "number"
          }
        },
        "required": [
          "type",
          "count",
          "amount"
        ]
      }
    }
  },
  "required": [
    "slotsFilled",
    "slotsAvailable",
    "fundCommitted",
    "fundDisbursed",
    "byType"
  ]
}
```
- **Validation Rules:** A sponsor-scoped caller sees only their own funded programs' data.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** REP-002, SCH-005.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 17. Bulk Award / Disbursement
- **Method:** `POST`
- **URL:** `/api/v1/scholarships/bulk-award`
- **Description:** Award or disburse to a cohort at once, within slot and fund bounds (SCR-SCH-09).
- **Authentication (Role-Based):** Yes — `scholarship.select` (for award) or `scholarship.disburse` (for disbursement).
- **Request DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "programId": {
          "type": "string"
        },
        "studentIds": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "action": {
          "type": "string",
          "enum": [
            "AWARD",
            "DISBURSE"
          ]
        }
      },
      "required": [
        "programId",
        "studentIds",
        "action"
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
- **Validation Rules:** Per-row slot bound (SCH-002), fund sufficiency (SCH-005), and net-≥-0 (SCH-004) are enforced individually — a batch never silently over-commits slots or fund; results report each student's outcome, including any blocked by these bounds.
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** SCH-002, SCH-004, SCH-005; conventions §9.
- **Pagination:** N/A.

### 18. Import Program Data
- **Method:** `POST`
- **URL:** `/api/v1/scholarships/programs/import`
- **Description:** Import scholarship program data from a file (SCR-SCH-10).
- **Authentication (Role-Based):** Yes — `scholarship.import`.
- **Request DTO (JSON Schema):**
```json
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
- **Validation Rules:** Imported programs validated for completeness as in #3; invalid rows reported, never silently imported.
- **Error Codes:** `422 INCOMPLETE_PROGRAM`.
- **Business Rules Applied:** SCH-001.
- **Pagination:** N/A.

### 19. Export Program Data
- **Method:** `GET`
- **URL:** `/api/v1/scholarships/programs/export`
- **Description:** Governed export of scholarship program data (SCR-SCH-10).
- **Authentication (Role-Based):** Yes — `scholarship.export`.
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
- **Validation Rules:** Export is audited; sponsor-sensitive data respects the same consent scope as #13.
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
