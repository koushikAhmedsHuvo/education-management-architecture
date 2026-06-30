# Enrollment API

## 1. API Overview

**Purpose.** Govern enrollment — placing a student into a specific session's class/section leaf and maintaining that placement across transfer, withdrawal, and promotion, with capacity as a hard cap throughout.

**Module Context.** Implements Business Rules Catalog Doc 08 (Enrollment) and UI Screen Spec `10-enrollment-screens.md` (SCR-ENR-01…14).

---

## 2. Endpoints

### 1. List Enrollments
- **Method:** `GET`
- **URL:** `/api/v1/enrollments`
- **Description:** Browse enrollments within scope with capacity context (SCR-ENR-01).
- **Authentication (Role-Based):** Yes — `enrollment.view`, scoped + ownership (teacher → own sections; student/guardian → own).
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
    "sessionId": {
      "type": "string"
    },
    "classId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "statuscommaseparated": {
      "type": "string",
      "description": "status? (comma-separated)"
    },
    "sortByrollNumberstud": {
      "type": "string",
      "description": "sortBy? (rollNumber|studentName|enrolledAt, default rollNumber)"
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
      "sessionId": {
        "type": "string"
      },
      "classId": {
        "type": "string"
      },
      "sectionId": {
        "type": "string"
      },
      "rollNumber": {
        "type": "integer"
      },
      "status": {
        "type": "string"
      },
      "enrolledAt": {
        "type": "string"
      },
      "version": {
        "type": "integer"
      }
    },
    "required": [
      "id",
      "studentId",
      "studentName",
      "sessionId",
      "classId",
      "sectionId",
      "rollNumber",
      "status",
      "enrolledAt",
      "version"
    ]
  }
}
```
- **Validation Rules:** Scope-limited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** ENR-002 (one active enrollment per student per session, reflected in results), AUTHZ-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Enrollment Detail
- **Method:** `GET`
- **URL:** `/api/v1/enrollments/{id}`
- **Description:** Placement, roll number, and lifecycle status (SCR-ENR-02).
- **Authentication (Role-Based):** Yes — `enrollment.view`.
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
    "studentId": {
      "type": "string"
    },
    "sessionId": {
      "type": "string"
    },
    "classId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "rollNumber": {
      "type": "integer"
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
    "studentId",
    "sessionId",
    "classId",
    "sectionId",
    "rollNumber",
    "status",
    "createdAt",
    "updatedAt",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** ENR-001 (binds to a session-instance leaf), ENR-008 (status drives operational eligibility).
- **Pagination:** N/A.

### 3. Get Enrollment History
- **Method:** `GET`
- **URL:** `/api/v1/enrollments/{id}/history`
- **Description:** Period-accurate placement history across transfers (SCR-ENR-02).
- **Authentication (Role-Based):** Yes — `enrollment.view`.
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
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "periodFrom": {
        "type": "string"
      },
      "periodTo": {
        "type": "string"
      },
      "sectionId": {
        "type": "string"
      },
      "rollNumber": {
        "type": "integer"
      },
      "changeType": {
        "type": "string",
        "enum": [
          "INITIAL",
          "TRANSFER",
          "REASSIGNED_ROLL"
        ]
      },
      "actor": {
        "type": "string"
      },
      "at": {
        "type": "string"
      }
    },
    "required": [
      "periodFrom",
      "sectionId",
      "rollNumber",
      "changeType",
      "actor",
      "at"
    ]
  }
}
```
- **Validation Rules:** This history is **append-only** — a transfer never rewrites a prior period's row, it closes it and opens a new one (ENR-005).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** ENR-005 (transfer preserves period-accurate history).
- **Pagination:** N/A.

## Create

### 4. Resolve Sections with Live Capacity
- **Method:** `GET`
- **URL:** `/api/v1/enrollments/resolve-sections`
- **Description:** Look up the candidate class/section instances for a session with their live fill, to drive the Create Enrollment form (SCR-ENR-03).
- **Authentication (Role-Based):** Yes — `enrollment.create`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "sessionId": {
      "type": "string"
    },
    "classId": {
      "type": "string"
    }
  },
  "required": [
    "sessionId"
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
      "sectionId": {
        "type": "string"
      },
      "sectionName": {
        "type": "string"
      },
      "classId": {
        "type": "string"
      },
      "capacity": {
        "type": "number"
      },
      "activeEnrollmentCount": {
        "type": "integer"
      },
      "available": {
        "type": "number"
      }
    },
    "required": [
      "sectionId",
      "sectionName",
      "classId",
      "capacity",
      "activeEnrollmentCount",
      "available"
    ]
  }
}
```
- **Validation Rules:** Only sections belonging to an active session-instance are returned (ENR-001).
- **Error Codes:** `404 SESSION_OR_CLASS_NOT_FOUND`.
- **Business Rules Applied:** ENR-001, ENR-003 (capacity informs the picker before submission).
- **Pagination:** N/A.

### 5. Create Enrollment
- **Method:** `POST`
- **URL:** `/api/v1/enrollments`
- **Description:** Place a student at the enrollment leaf — session/class/section — for a session (SCR-ENR-03, UC-ENR-01).
- **Authentication (Role-Based):** Yes — `enrollment.create`; roll-number assignment additionally requires `enrollment.rollnumber.manage` if a manual number is supplied.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string"
    },
    "sessionId": {
      "type": "string"
    },
    "classId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "rollNumber": {
      "type": "number",
      "description": "auto if omitted"
    }
  },
  "required": [
    "studentId",
    "sessionId",
    "classId",
    "sectionId"
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
    "studentId": {
      "type": "string"
    },
    "sessionId": {
      "type": "string"
    },
    "classId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "rollNumber": {
      "type": "integer"
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
    "studentId",
    "sessionId",
    "classId",
    "sectionId",
    "rollNumber",
    "status",
    "createdAt",
    "version"
  ]
}
```
- **Validation Rules:** The student must not already hold an active enrollment in this session ("Student already enrolled this session," ENR-002); the target section's capacity is a **hard cap** — counted against active enrollments, never advisory ("Section is full," ENR-003, C-01); the assigned `rollNumber` must be unique within the section for the session ("Roll number already used in this section," ENR-004); the target must reference a valid, active session-instance leaf (ENR-001).
- **Error Codes:** `409 ALREADY_ACTIVELY_ENROLLED`; `409 SECTION_CAPACITY_EXCEEDED`; `409 DUPLICATE_ROLL_NUMBER`.
- **Business Rules Applied:** ENR-001 (bind to session instance), ENR-002 (single active enrollment), ENR-003 (capacity hard cap), ENR-004 (unique roll number).
- **Pagination:** N/A.

## Edit (Non-Structural)

### 6. Edit Enrollment
- **Method:** `PUT`
- **URL:** `/api/v1/enrollments/{id}`
- **Description:** Edit non-structural enrollment details only — **section changes are never made here**, they go through Transfer (#7) so capacity and history rules are enforced (SCR-ENR-04).
- **Authentication (Role-Based):** Yes — `enrollment.update`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "notes": {
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
- **Validation Rules:** This endpoint rejects any attempt to change `sectionId`/`classId` directly ("section change is not done here — use Transfer"); `version` optimistic lock.
- **Error Codes:** `422 USE_TRANSFER_FOR_SECTION_CHANGE`; `409 VERSION_CONFLICT`.
- **Business Rules Applied:** ENR-004 (roll-number uniqueness preserved), ENR-005 (structural moves redirected to the governed Transfer path).
- **Pagination:** N/A.

## Transfer (Governed)

### 7. Submit Inter-Section Transfer
- **Method:** `POST`
- **URL:** `/api/v1/enrollments/transfers`
- **Description:** Move a student to another section within the institute, capacity-checked, history-preserving, and routed to approval (SCR-ENR-05, UC-ENR-02).
- **Authentication (Role-Based):** Yes — `enrollment.transfer.execute` (request).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "enrollmentId": {
      "type": "string"
    },
    "targetSectionId": {
      "type": "string"
    },
    "effectiveDate": {
      "type": "string"
    },
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "enrollmentId",
    "targetSectionId",
    "effectiveDate",
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
- **Validation Rules:** `targetSectionId` must differ from the current section ("Target must differ from current section"); the target must have available capacity at submission, **re-checked again at approval time** ("Target section is full," ENR-003, C-01); on eventual approval, the prior placement is closed as-of `effectiveDate` and a new one opened — the prior period's attendance/marks remain attributed to the old section (ENR-005).
- **Error Codes:** `422 SAME_SECTION`; `409 TARGET_SECTION_FULL`.
- **Business Rules Applied:** ENR-003 (capacity hard cap), ENR-005 (period-accurate history), C-01.
- **Pagination:** N/A.

### 8. List Transfer/Withdrawal Approvals
- **Method:** `GET`
- **URL:** `/api/v1/enrollments/transfer-approvals`
- **Description:** Inbox for pending transfers and withdrawals (SCR-ENR-05/14).
- **Authentication (Role-Based):** Yes — `enrollment.transfer.approve`, scoped.
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
    "typeTRANSFERWITHDRAW": {
      "type": "string",
      "description": "type? (TRANSFER|WITHDRAWAL)"
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
      "studentName": {
        "type": "string"
      },
      "fromSection": {
        "type": "string"
      },
      "toSection": {
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
      "studentName",
      "requestedBy",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Requester ≠ approver — self-raised requests are listed but the actions are disabled for the requester (AUTHZ-009).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-009, WFL-004.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 9. Decide Transfer/Withdrawal Request
- **Method:** `POST`
- **URL:** `/api/v1/enrollments/transfer-approvals/{id}/decide`
- **Description:** Approve or reject a pending transfer or withdrawal; on approval, capacity is re-validated and the change committed with preserved history.
- **Authentication (Role-Based):** Yes — `enrollment.transfer.approve`; **blocked if the caller raised the request**.
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
- **Validation Rules:** Caller ≠ requester; target-section capacity (for a transfer) is re-checked at this point, since it may have changed since submission.
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED`; `409 TARGET_CAPACITY_CHANGED`.
- **Business Rules Applied:** AUTHZ-009, ENR-003, ENR-005.
- **Pagination:** N/A.

## Withdrawal & Cross-Institution Exit

### 10. Submit Withdrawal
- **Method:** `POST`
- **URL:** `/api/v1/enrollments/withdrawals`
- **Description:** Withdraw a student with a clearance checklist, routed to approval (SCR-ENR-06).
- **Authentication (Role-Based):** Yes — `enrollment.withdraw`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "enrollmentId": {
      "type": "string"
    },
    "reason": {
      "type": "string"
    },
    "effectiveDate": {
      "type": "string"
    },
    "isCrossInstitutionExit": {
      "type": "boolean"
    }
  },
  "required": [
    "enrollmentId",
    "reason",
    "effectiveDate",
    "isCrossInstitutionExit"
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
    "clearanceComplete": {
      "type": "boolean"
    }
  },
  "required": [
    "id",
    "status",
    "clearanceComplete"
  ]
}
```
- **Validation Rules:** Submission is blocked until the clearance checklist is complete per institute policy ("Complete clearance before withdrawal"); on eventual approval, status transitions to `WITHDRAWN`/`TRANSFERRED_OUT` and the section's capacity is freed (ENR-006/ENR-008).
- **Error Codes:** `422 CLEARANCE_INCOMPLETE`.
- **Business Rules Applied:** ENR-006 (withdrawal + clearance), ENR-008 (status drives eligibility, capacity freed).
- **Pagination:** N/A.

### 11. Get Clearance Checklist
- **Method:** `GET`
- **URL:** `/api/v1/enrollments/{id}/clearance-checklist`
- **Description:** View the configured clearance items (dues, library, assets) and their completion state (SCR-ENR-06).
- **Authentication (Role-Based):** Yes — `enrollment.view`.
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
    "items": {
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
          "complete": {
            "type": "boolean"
          }
        },
        "required": [
          "key",
          "label",
          "complete"
        ]
      }
    }
  },
  "required": [
    "items"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** ENR-006.
- **Pagination:** N/A.

### 12. Generate Transfer Certificate
- **Method:** `POST`
- **URL:** `/api/v1/enrollments/{id}/transfer-certificate`
- **Description:** Generate a transfer certificate (TC) for a cross-institution exit — an immutable, reproducible document (SCR-ENR-07, ENR-006).
- **Authentication (Role-Based):** Yes — `enrollment.withdraw` (same governed flow as #10).
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
    "certificateId": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "GENERATED"
    }
  },
  "required": [
    "certificateId",
    "status"
  ]
}
```
- **Validation Rules:** Only generated for an approved `TRANSFERRED_OUT`/exit withdrawal; once generated, the certificate content is immutable and reproducible — regenerating produces the identical document (FILE-007).
- **Error Codes:** `409 WITHDRAWAL_NOT_APPROVED`.
- **Business Rules Applied:** ENR-006, FILE-007 (immutable, reproducible document).
- **Pagination:** N/A.

### 13. Get Transfer Certificate Download URL
- **Method:** `GET`
- **URL:** `/api/v1/enrollments/{id}/transfer-certificate/download-url`
- **Description:** Short-lived signed URL to download the generated TC (SCR-ENR-07).
- **Authentication (Role-Based):** Yes — `enrollment.view`, or the student/guardian's own subject-scoped access.
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
- **Error Codes:** `404 CERTIFICATE_NOT_GENERATED`.
- **Business Rules Applied:** FILE-005 (short-lived signed URLs).
- **Pagination:** N/A.

## Roll Number & Promotion

### 14. Assign / Reassign Roll Number
- **Method:** `PUT`
- **URL:** `/api/v1/enrollments/{id}/roll-number`
- **Description:** Assign or re-sequence a roll number, unique within section/session (SCR-ENR-08).
- **Authentication (Role-Based):** Yes — `enrollment.rollnumber.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "rollNumber": {
      "type": "number"
    }
  },
  "required": [
    "rollNumber"
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
    "rollNumber": {
      "type": "integer"
    }
  },
  "required": [
    "id",
    "rollNumber"
  ]
}
```
- **Validation Rules:** Must be unique within the (section, session) pair; reassignment is an explicit, audited action, never a silent renumbering.
- **Error Codes:** `409 DUPLICATE_ROLL_NUMBER`.
- **Business Rules Applied:** ENR-004 (roll number assignment, audited resequencing).
- **Pagination:** N/A.

### 15. Promotion to Next Session — *see `06-academic-session-api.md`*
- **Method:** `GET`
- **URL:** `/api/v1/unknown`
- **Description:** 
- **Authentication (Role-Based):** 
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
  "properties": {},
  "description": "No request body."
}
```
- **Validation Rules:** N/A.
- **Error Codes:** N/A.
- **Business Rules Applied:** ENR-007, ENR-003, C-01, SESS-006.

## Bulk Operations & Reporting
- **Pagination:** N/A.

### 16. Bulk-Create Enrollments
- **Method:** `POST`
- **URL:** `/api/v1/enrollments/bulk-create`
- **Description:** Bulk-enroll a cohort at once (SCR-ENR-11).
- **Authentication (Role-Based):** Yes — `enrollment.create`.
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
              "sessionId": {
                "type": "string"
              },
              "classId": {
                "type": "string"
              },
              "sectionId": {
                "type": "string"
              },
              "rollNumber": {
                "type": "integer"
              }
            },
            "required": [
              "studentId",
              "sessionId",
              "classId",
              "sectionId"
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
              "enrollmentId": {
                "type": "string"
              }
            },
            "required": [
              "row",
              "enrollmentId"
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
- **Validation Rules:** Each row validated against #5's rules (no double-active enrollment, capacity hard cap, unique roll number); invalid rows excluded and always reported — never silently dropped.
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** ENR-002, ENR-003, ENR-004, C-01; conventions §9.
- **Pagination:** N/A.

### 17. Rebalance Sections
- **Method:** `POST`
- **URL:** `/api/v1/enrollments/rebalance`
- **Description:** Redistribute students across under/over-filled sections of the same class within capacity (SCR-ENR-11).
- **Authentication (Role-Based):** Yes — `enrollment.create` (the approved screen groups this bulk action under the same list-screen permission as bulk enroll).
- **Request DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "classId": {
          "type": "string"
        },
        "sessionId": {
          "type": "string"
        },
        "strategy": {
          "type": "string",
          "enum": [
            "BALANCED_FILL",
            "MANUAL"
          ]
        },
        "manualMoves": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "enrollmentId": {
                "type": "string"
              },
              "targetSectionId": {
                "type": "string"
              }
            },
            "required": [
              "enrollmentId",
              "targetSectionId"
            ]
          }
        }
      },
      "required": [
        "classId",
        "sessionId",
        "strategy"
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
  "description": "202 Accepted { jobId } (per-student rebalance results, each effectively a transfer with preserved history per ENR-005)."
}
```
- **Validation Rules:** No target section may exceed capacity as a result of the rebalance (ENR-003/C-01); every move is recorded with the same period-accurate history semantics as a regular transfer (ENR-005).
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** ENR-003, ENR-005, C-01; conventions §9.
- **Pagination:** N/A.

### 18. Enrollment & Capacity Report
- **Method:** `GET`
- **URL:** `/api/v1/enrollments/report/capacity`
- **Description:** Enrollment counts and section capacity utilization (SCR-ENR-10).
- **Authentication (Role-Based):** Yes — `enrollment.view` (the approved screen names a generic "report view"; reused here per the established convention).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "sessionId": {
      "type": "string"
    },
    "classId": {
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
      "sectionId": {
        "type": "string"
      },
      "sectionName": {
        "type": "string"
      },
      "capacity": {
        "type": "number"
      },
      "enrolled": {
        "type": "string"
      },
      "utilizationPercent": {
        "type": "number"
      }
    },
    "required": [
      "sectionId",
      "sectionName",
      "capacity",
      "enrolled",
      "utilizationPercent"
    ]
  }
}
```
- **Validation Rules:** Scope-limited; based on **active** enrollments, the operative truth for capacity (ENR-003).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** ENR-003, REP-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 19. Import Enrollments
- **Method:** `POST`
- **URL:** `/api/v1/enrollments/import`
- **Description:** Bulk-import enrollments with full validation (SCR-ENR-12).
- **Authentication (Role-Based):** Yes — `enrollment.import`.
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
              "sessionId": {
                "type": "string"
              },
              "classId": {
                "type": "string"
              },
              "sectionId": {
                "type": "string"
              },
              "rollNumber": {
                "type": "integer"
              }
            },
            "required": [
              "studentId",
              "sessionId",
              "classId",
              "sectionId"
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
              "enrollmentId": {
                "type": "string"
              }
            },
            "required": [
              "row",
              "enrollmentId"
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
- **Validation Rules:** Same full rule set as #5/#16 (ENR-001 through ENR-004, C-01); invalid rows excluded and always reported.
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** ENR-001, ENR-002, ENR-003, ENR-004, C-01; conventions §9.
- **Pagination:** N/A.

### 20. Export Enrollments
- **Method:** `GET`
- **URL:** `/api/v1/enrollments/export`
- **Description:** Governed export of enrollment data (SCR-ENR-13).
- **Authentication (Role-Based):** Yes — `enrollment.export`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "sessionId": {
      "type": "string"
    },
    "classId": {
      "type": "string"
    },
    "sectionId": {
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
