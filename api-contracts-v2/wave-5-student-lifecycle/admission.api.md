# Admission API

## 1. API Overview

**Purpose.** Govern the admission process end-to-end — applicant self-service, officer evaluation, SoD-governed decisions, atomic capacity-and-fee-checked conversion to Student + Enrollment, waitlist management, and form/capacity/criteria configuration.

**Module Context.** Implements Business Rules Catalog Doc 07 (Admission) and UI Screen Spec `07-admission-screens.md` (SCR-ADM-01…14).

---

## 2. Endpoints

### 1. Get Application Schema
- **Method:** `GET`
- **URL:** `/api/v1/admissions/application-schema`
- **Description:** Fetch the currently published, dynamic application-form schema for a target session/level (SCR-ADM-01).
- **Authentication (Role-Based):** No (public).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "sessionId": {
      "type": "string"
    },
    "levelClassId": {
      "type": "string"
    }
  },
  "required": [
    "sessionId",
    "levelClassId"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "formVersion": {
      "type": "number"
    },
    "fields": {
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
          "type": {
            "type": "string"
          },
          "required": {
            "type": "string"
          },
          "options": {
            "type": "string"
          }
        },
        "required": [
          "key",
          "label",
          "type",
          "required"
        ]
      }
    },
    "requiredDocuments": {
      "type": "array",
      "items": {
        "type": "string"
      }
    }
  },
  "required": [
    "formVersion",
    "fields",
    "requiredDocuments"
  ]
}
```
- **Validation Rules:** Admissions must be open for the target session/level.
- **Error Codes:** `409 ADMISSIONS_CLOSED`.
- **Business Rules Applied:** ADM-001 (configurable application form), CFG-010 (engine-validated schema).
- **Pagination:** N/A.

### 2. Save Application Draft
- **Method:** `POST`
- **URL:** `/api/v1/admissions/applications/draft`
- **Description:** Save an in-progress application for later resumption (SCR-ADM-01).
- **Authentication (Role-Based):** No (public; the draft is bound to the submitting session/device, not a staff permission).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "sessionId": {
      "type": "string"
    },
    "levelClassId": {
      "type": "string"
    },
    "formVersion": {
      "type": "number"
    },
    "fieldValues": {
      "type": "object"
    },
    "uploadedDocumentIds": {
      "type": "array",
      "items": {
        "type": "string"
      }
    }
  },
  "required": [
    "sessionId",
    "levelClassId",
    "formVersion",
    "fieldValues",
    "uploadedDocumentIds"
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
      "const": "DRAFT"
    },
    "lastSavedAt": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "status",
    "lastSavedAt"
  ]
}
```
- **Validation Rules:** Field values validated against the published schema (`formVersion`) as far as completed; partial data is allowed in `DRAFT`.
- **Error Codes:** `422 SCHEMA_VALIDATION_FAILED` (per-field).
- **Business Rules Applied:** ADM-001.
- **Pagination:** N/A.

### 3. Submit Application
- **Method:** `POST`
- **URL:** `/api/v1/admissions/applications/{id}/submit`
- **Description:** Finalize and submit a draft — idempotent, assigns a unique application number (SCR-ADM-01, UC-ADM-01).
- **Authentication (Role-Based):** No (public; idempotent on `{id}`).
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
    "applicationNumber": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "SUBMITTED"
    },
    "submittedAt": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "applicationNumber",
    "status",
    "submittedAt"
  ]
}
```
- **Validation Rules:** All required fields and required documents per the schema must be complete ("Complete all required documents before submitting"); re-submitting an already-`SUBMITTED` draft is a safe no-op returning the same application number — never a duplicate ("a double-click does not create two applications," ADM-002).
- **Error Codes:** `422 INCOMPLETE_SUBMISSION` (lists missing fields/documents).
- **Business Rules Applied:** ADM-001 (form version stamped), ADM-002 (unique number, idempotent submission).
- **Pagination:** N/A.

### 4. Get Application Status (Tracker)
- **Method:** `GET`
- **URL:** `/api/v1/admissions/applications/{id}/status`
- **Description:** Track an application's stage and act on an offer (SCR-ADM-02).
- **Authentication (Role-Based):** No special permission — applicant/guardian acting on their **own** application(s) only.
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
    "applicationNumber": {
      "type": "integer"
    },
    "stage": {
      "type": "string",
      "enum": [
        "SUBMITTED",
        "UNDER_REVIEW",
        "RETURNED_FOR_CORRECTION",
        "WAITLISTED",
        "APPROVED",
        "ENROLLED",
        "REJECTED",
        "WITHDRAWN",
        "EXPIRED"
      ]
    },
    "decisionReason": {
      "type": "string"
    },
    "offer": {
      "type": "object",
      "properties": {
        "expiresAt": {
          "type": "string"
        }
      },
      "required": [
        "expiresAt"
      ]
    },
    "documents": {
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
    }
  },
  "required": [
    "id",
    "applicationNumber",
    "stage",
    "documents"
  ]
}
```
- **Validation Rules:** Caller must be the submitting applicant/guardian.
- **Error Codes:** `404 NOT_FOUND` (not own application).
- **Business Rules Applied:** Doc 07 §6 (state machine).
- **Pagination:** N/A.

### 5. Accept Offer
- **Method:** `POST`
- **URL:** `/api/v1/admissions/applications/{id}/accept-offer`
- **Description:** Accept an admission offer before it expires (SCR-ADM-02/06).
- **Authentication (Role-Based):** Own application only.
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
      "const": "APPROVED"
    },
    "offerAccepted": {
      "type": "boolean",
      "const": true
    }
  },
  "required": [
    "id",
    "status",
    "offerAccepted"
  ]
}
```
- **Validation Rules:** The offer must be unexpired; expired offers are rejected ("offer valid/unexpired").
- **Error Codes:** `409 OFFER_EXPIRED`.
- **Business Rules Applied:** ADM-004 (offer acceptance & fee gating downstream).
- **Pagination:** N/A.

### 6. Pay Admission Fee
- **Method:** `POST`
- **URL:** `/api/v1/admissions/applications/{id}/pay-admission-fee`
- **Description:** Initiate the admission-fee payment that gates conversion (SCR-ADM-06). No card data is handled here — this hands off to the Payment module (a later wave).
- **Authentication (Role-Based):** Own application only.
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
    "paymentSessionRef": {
      "type": "string"
    }
  },
  "required": [
    "paymentSessionRef"
  ]
}
```
- **Validation Rules:** Payment is idempotent — a retry never double-charges.
- **Error Codes:** `409 FEE_ALREADY_SETTLED`.
- **Business Rules Applied:** ADM-006 (admission fee gating), PAY-008 (idempotent payment, Payment module).
- **Pagination:** N/A.

### 7. Withdraw Application
- **Method:** `POST`
- **URL:** `/api/v1/admissions/applications/{id}/withdraw`
- **Description:** The applicant/guardian withdraws before a terminal decision (SCR-ADM-02).
- **Authentication (Role-Based):** Own application only.
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
      "const": "WITHDRAWN"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** Only permitted while not yet `ENROLLED`/`REJECTED`/already terminal.
- **Error Codes:** `409 ALREADY_TERMINAL`.
- **Business Rules Applied:** Doc 07 §6 (state machine).
- **Pagination:** N/A.

## Officer Review & Evaluation

### 8. List Applications (Officer)
- **Method:** `GET`
- **URL:** `/api/v1/admissions/applications`
- **Description:** Triage applications within scope (SCR-ADM-03).
- **Authentication (Role-Based):** Yes — `admission.application.view`, scoped.
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
    "stagecommaseparated": {
      "type": "string",
      "description": "stage? (comma-separated)"
    },
    "levelClassId": {
      "type": "string"
    },
    "sessionId": {
      "type": "string"
    },
    "dateFrom": {
      "type": "string"
    },
    "dateTo": {
      "type": "string"
    },
    "officerId": {
      "type": "string"
    },
    "sortBysubmittedAtsco": {
      "type": "string",
      "description": "sortBy? (submittedAt|score|stage, default submittedAt desc)"
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
      "applicationNumber": {
        "type": "integer"
      },
      "applicantName": {
        "type": "string"
      },
      "levelClassId": {
        "type": "string"
      },
      "sessionId": {
        "type": "string"
      },
      "stage": {
        "type": "string"
      },
      "score": {
        "type": "number"
      },
      "submittedAt": {
        "type": "string"
      },
      "assignedOfficerId": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "applicationNumber",
      "applicantName",
      "levelClassId",
      "sessionId",
      "stage",
      "submittedAt",
      "assignedOfficerId"
    ]
  }
}
```
- **Validation Rules:** Scope-limited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** ADM-003 (version-pinned decisions), AUTHZ-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 9. Get Application Detail (Officer)
- **Method:** `GET`
- **URL:** `/api/v1/admissions/applications/{id}`
- **Description:** Full applicant data, documents (signed URLs), evaluation, and history (SCR-ADM-04).
- **Authentication (Role-Based):** Yes — `admission.application.view`.
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
    "applicationNumber": {
      "type": "integer"
    },
    "formVersion": {
      "type": "string"
    },
    "fieldValues": {
      "type": "string"
    },
    "documents": {
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
          "signedUrl": {
            "type": "string"
          },
          "expiresAt": {
            "type": "string"
          }
        },
        "required": [
          "id",
          "type",
          "signedUrl",
          "expiresAt"
        ]
      }
    },
    "evaluation": {
      "type": "object",
      "properties": {
        "score": {
          "type": "number"
        },
        "criteriaResult": {
          "type": "string"
        },
        "recommendation": {
          "type": "string"
        },
        "notes": {
          "type": "string"
        }
      },
      "required": [
        "score",
        "criteriaResult",
        "recommendation",
        "notes"
      ]
    },
    "returnCount": {
      "type": "integer"
    },
    "history": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    }
  },
  "required": [
    "id",
    "applicationNumber",
    "formVersion",
    "fieldValues",
    "documents",
    "returnCount",
    "history"
  ]
}
```
- **Validation Rules:** Document URLs are short-lived signed links (FILE-005), never raw public URLs.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** ADM-003 (pinned criteria version), FILE-004/005 (subject-scoped, signed-URL document access).
- **Pagination:** N/A.

### 10. Save Evaluation
- **Method:** `PUT`
- **URL:** `/api/v1/admissions/applications/{id}/evaluation`
- **Description:** Score the application against the configured, version-pinned rubric (SCR-ADM-04).
- **Authentication (Role-Based):** Yes — `admission.application.evaluate`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "score": {
      "type": "number"
    },
    "criteriaResult": {
      "type": "object",
      "description": "per the pinned rubric"
    },
    "recommendation": {
      "type": "string"
    },
    "officerNotes": {
      "type": "string"
    }
  },
  "required": [
    "criteriaResult",
    "recommendation"
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
    "evaluation": {
      "type": "object",
      "properties": {
        "note": {
          "type": "string",
          "description": "..."
        }
      }
    }
  },
  "required": [
    "id",
    "evaluation"
  ]
}
```
- **Validation Rules:** Criteria evaluated must match the **pinned** rubric version for this application (ADM-003), not whatever is currently configured.
- **Error Codes:** `409 STALE_RUBRIC_VERSION` (reload).
- **Business Rules Applied:** ADM-003 (version-pinned criteria).
- **Pagination:** N/A.

### 11. Forward Application to Approver
- **Method:** `POST`
- **URL:** `/api/v1/admissions/applications/{id}/forward`
- **Description:** Move a `SUBMITTED`/`UNDER_REVIEW` application into the decision queue.
- **Authentication (Role-Based):** Yes — `admission.application.evaluate`.
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
      "const": "UNDER_REVIEW"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** Required evaluation criteria must be complete before forwarding ("Complete required criteria before forwarding").
- **Error Codes:** `422 EVALUATION_INCOMPLETE`.
- **Business Rules Applied:** ADM-003, WFL-001/003 (FSM-driven progression).
- **Pagination:** N/A.

### 12. Return Application for Correction
- **Method:** `POST`
- **URL:** `/api/v1/admissions/applications/{id}/return`
- **Description:** Send the application back to the applicant for fixes (SCR-ADM-04).
- **Authentication (Role-Based):** Yes — `admission.application.evaluate`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "requestedChanges": {
      "type": "string"
    }
  },
  "required": [
    "requestedChanges"
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
      "const": "RETURNED_FOR_CORRECTION"
    },
    "returnCount": {
      "type": "integer"
    },
    "roundsRemaining": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "status",
    "returnCount",
    "roundsRemaining"
  ]
}
```
- **Validation Rules:** Each return increments a bounded round counter — once the configured bound is reached, returns are no longer available and the application escalates instead (WFL-011).
- **Error Codes:** `409 RETURN_BOUND_EXCEEDED` (auto-escalates).
- **Business Rules Applied:** WFL-011 (bounded return-for-correction loops).
- **Pagination:** N/A.

## Decision (Approval)

### 13. List Decision Queue
- **Method:** `GET`
- **URL:** `/api/v1/admissions/decision-queue`
- **Description:** Inbox of applications awaiting an admission decision (SCR-ADM-05/14).
- **Authentication (Role-Based):** Yes — `admission.decision.approve`, scoped.
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
    "levelClassId": {
      "type": "string"
    },
    "sessionId": {
      "type": "string"
    },
    "stage": {
      "type": "string"
    },
    "sortByslascoredefa": {
      "type": "string",
      "description": "sortBy? (sla|score, default sla)"
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
      "applicationNumber": {
        "type": "integer"
      },
      "applicantName": {
        "type": "string"
      },
      "levelClassId": {
        "type": "string"
      },
      "officerRecommendation": {
        "type": "string"
      },
      "score": {
        "type": "number"
      },
      "slaDueAt": {
        "type": "string"
      },
      "stage": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "applicationNumber",
      "applicantName",
      "levelClassId",
      "officerRecommendation",
      "score",
      "slaDueAt",
      "stage"
    ]
  }
}
```
- **Validation Rules:** Applications the caller themselves evaluated as the officer are still listed but cannot be decided by them (#14).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-009 (SoD — officer ≠ approver), WFL-004.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 14. Record Admission Decision
- **Method:** `POST`
- **URL:** `/api/v1/admissions/applications/{id}/decide`
- **Description:** Approve, reject, return, or waitlist an application under governance (SCR-ADM-05).
- **Authentication (Role-Based):** Yes — `admission.decision.approve`; **blocked if the caller is this application's evaluating officer or submitter** — self-decision is disabled regardless of role.
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
        "RETURN",
        "WAITLIST"
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
          "type": "string"
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
- **Validation Rules:** Caller ≠ the application's officer/submitter ("You evaluated this application — a different approver is required," AUTHZ-009); `reason` required on `REJECT`/`RETURN`; the decision is made against the **pinned** workflow version from submission (ADM-003); `RETURN` is subject to the same bounded-round rule as #12.
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED`; `422 REASON_REQUIRED`.
- **Business Rules Applied:** ADM-003 (version-pinned decision), AUTHZ-009, WFL-011.
- **Pagination:** N/A.

### 15. Bulk Decide Applications
- **Method:** `POST`
- **URL:** `/api/v1/admissions/decisions/bulk-decide`
- **Description:** Approve or reject many applications at once (SCR-ADM-05).
- **Authentication (Role-Based):** Yes — `admission.decision.approve`.
- **Request DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "applicationIds": {
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
        "applicationIds",
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
  "description": "202 Accepted { jobId } (per-application results via GET /api/v1/jobs/{jobId})."
}
```
- **Validation Rules:** Same SoD rule as #14 applied per application — any application where the caller is the officer is excluded from the bulk batch and reported, not silently force-decided.
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** AUTHZ-009; conventions §9.
- **Pagination:** N/A.

## Conversion

### 16. Convert Application to Student + Enrollment
- **Method:** `POST`
- **URL:** `/api/v1/admissions/applications/{id}/convert`
- **Description:** Atomically materialize a `Student` record and an `Enrollment` from an approved, fee-cleared application (SCR-ADM-07, UC-ADM-03).
- **Authentication (Role-Based):** Yes — `admission.convert` (carries implicit `student.create`/`enrollment.create` authority within the same scope).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "targetClassId": {
      "type": "string"
    },
    "targetSectionId": {
      "type": "string"
    },
    "idempotencyKey": {
      "type": "string"
    }
  },
  "required": [
    "targetClassId",
    "targetSectionId",
    "idempotencyKey"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "applicationId": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "ENROLLED"
    },
    "studentId": {
      "type": "string"
    },
    "enrollmentId": {
      "type": "string"
    },
    "rollNumber": {
      "type": "integer"
    }
  },
  "required": [
    "applicationId",
    "status",
    "studentId",
    "enrollmentId",
    "rollNumber"
  ]
}
```
- **Validation Rules:** All conversion preconditions must hold **transactionally** — application `APPROVED`, admission fee settled or waived (ADM-006), a guardian link available if the applicant is a minor (STU-006), and the target section has an available seat counted against **confirmed enrollments**, not mere approvals (ADM-004/ADM-005, C-01 hard cap); if any precondition fails, the whole operation aborts cleanly and the application **stays `APPROVED`** — nothing is half-created; re-invoking with the same `idempotencyKey` never creates a duplicate student or enrollment.
- **Error Codes:** `409 FEE_NOT_SETTLED`; `409 GUARDIAN_LINK_REQUIRED` (minor); `409 SECTION_CAPACITY_EXCEEDED` (suggests waitlist); each returned with the specific unmet precondition.
- **Business Rules Applied:** ADM-007 (transactional conversion integrity), ADM-005 (capacity-checked, confirmed-enrollment basis), ADM-006 (fee gating), STU-001/STU-006, ENR-001/ENR-003.
- **Pagination:** N/A.

### 17. Bulk Convert Applications
- **Method:** `POST`
- **URL:** `/api/v1/admissions/applications/bulk-convert`
- **Description:** Convert many approved applications at once (SCR-ADM-03/07).
- **Authentication (Role-Based):** Yes — `admission.convert`.
- **Request DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "applicationIds": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "id": {
                "type": "string"
              },
              "targetClassId": {
                "type": "string"
              },
              "targetSectionId": {
                "type": "string"
              }
            },
            "required": [
              "id",
              "targetClassId",
              "targetSectionId"
            ]
          }
        }
      },
      "required": [
        "applicationIds"
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
  "description": "202 Accepted { jobId } (per-application results, with capacity-exceeded rows flagged individually)."
}
```
- **Validation Rules:** Same per-application preconditions as #16; capacity is re-checked per row in submission order so a batch never over-commits a section.
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** ADM-007, ADM-005; conventions §9.
- **Pagination:** N/A.

## Waitlist

### 18. List Waitlist
- **Method:** `GET`
- **URL:** `/api/v1/admissions/waitlist`
- **Description:** View the ranked waitlist with capacity context (SCR-ADM-08).
- **Authentication (Role-Based):** Yes — `admission.waitlist.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "levelClassId": {
      "type": "string"
    },
    "sessionId": {
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
      "id": {
        "type": "string"
      },
      "rank": {
        "type": "integer"
      },
      "applicantName": {
        "type": "string"
      },
      "score": {
        "type": "number"
      },
      "addedAt": {
        "type": "string"
      },
      "status": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "rank",
      "applicantName",
      "score",
      "addedAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Scope-limited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** ADM-005 (waitlist management).
- **Pagination:** Yes — default `page=1, pageSize=25`, sorted by `rank asc`.

### 19. Reorder Waitlist
- **Method:** `POST`
- **URL:** `/api/v1/admissions/waitlist/reorder`
- **Description:** Adjust the waitlist's ordered ranking per the institute's configured, transparent criterion.
- **Authentication (Role-Based):** Yes — `admission.waitlist.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "ordering": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "applicationId": {
            "type": "string"
          },
          "rank": {
            "type": "integer"
          }
        },
        "required": [
          "applicationId",
          "rank"
        ]
      }
    }
  },
  "required": [
    "ordering"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "updated": {
      "type": "number"
    }
  },
  "required": [
    "updated"
  ]
}
```
- **Validation Rules:** Out-of-order promotions later are blocked unless justified (#20's promotion respects this order).
- **Error Codes:** `422 INVALID_ORDERING`.
- **Business Rules Applied:** ADM-005 (fairness — ordered, transparent waitlist).
- **Pagination:** N/A.

### 20. Promote from Waitlist
- **Method:** `POST`
- **URL:** `/api/v1/admissions/waitlist/{id}/promote`
- **Description:** Promote the next-in-order waitlisted applicant as a seat frees, issuing a fresh offer.
- **Authentication (Role-Based):** Yes — `admission.waitlist.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "overrideOrderJustification": {
      "type": "string",
      "description": "required only for an out-of-order promotion"
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
      "const": "APPROVED"
    },
    "offerIssued": {
      "type": "boolean",
      "const": true
    }
  },
  "required": [
    "id",
    "status",
    "offerIssued"
  ]
}
```
- **Validation Rules:** Promotion respects the current waitlist order unless explicitly justified and audited; promotion is blocked if it would exceed capacity (C-01).
- **Error Codes:** `409 CAPACITY_FULL`; `422 OUT_OF_ORDER_JUSTIFICATION_REQUIRED`.
- **Business Rules Applied:** ADM-005 (waitlist promotion fairness), C-01 (capacity hard cap).
- **Pagination:** N/A.

## Configuration

### 21. Get Application Form Configuration
- **Method:** `GET`
- **URL:** `/api/v1/admissions/form-config`
- **Description:** View the current versioned application-form schema for editing (SCR-ADM-09).
- **Authentication (Role-Based):** Yes — `admission.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "levelClassId": {
      "type": "string"
    },
    "sessionId": {
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
    "formVersion": {
      "type": "string"
    },
    "fields": {
      "type": "string",
      "description": "fields[]"
    },
    "requiredDocuments": {
      "type": "string",
      "description": "requiredDocuments[]"
    },
    "updatedAt": {
      "type": "string"
    }
  },
  "required": [
    "formVersion",
    "updatedAt"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** ADM-001, CFG-004 (versioned).
- **Pagination:** N/A.

### 22. Update Application Form Configuration
- **Method:** `PUT`
- **URL:** `/api/v1/admissions/form-config`
- **Description:** Edit and publish a new version of the application-form schema; the published version drives `1. Get Application Schema` (SCR-ADM-09).
- **Authentication (Role-Based):** Yes — `admission.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "fields": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    },
    "requiredDocuments": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "version": {
      "type": "number"
    }
  },
  "required": [
    "fields",
    "requiredDocuments",
    "version"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "formVersion": {
      "type": "number"
    },
    "status": {
      "type": "string",
      "const": "PUBLISHED"
    }
  },
  "required": [
    "formVersion",
    "status"
  ]
}
```
- **Validation Rules:** Typed/structurally valid per CFG-002; `version` optimistic lock; in-flight applications keep the form version they were submitted under — publishing a new version never alters them (ADM-001).
- **Error Codes:** `409 VERSION_CONFLICT`; `422 INVALID_SCHEMA`.
- **Business Rules Applied:** ADM-001, CFG-002, CFG-004.
- **Pagination:** N/A.

### 23. Get Capacity / Criteria / Fee Configuration
- **Method:** `GET`
- **URL:** `/api/v1/admissions/capacity-config`
- **Description:** View intake capacity, evaluation criteria, and fee-gating configuration (SCR-ADM-10).
- **Authentication (Role-Based):** Yes — `admission.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "levelClassId": {
      "type": "string"
    },
    "sessionId": {
      "type": "string"
    }
  },
  "required": [
    "levelClassId",
    "sessionId"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "capacity": {
      "type": "number"
    },
    "criteria": {
      "type": "object",
      "description": "rubric"
    },
    "feeGating": {
      "type": "object",
      "properties": {
        "required": {
          "type": "boolean"
        },
        "amountRef": {
          "type": "number"
        }
      },
      "required": [
        "required"
      ]
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "capacity",
    "criteria",
    "feeGating",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** ADM-004, ADM-006.
- **Pagination:** N/A.

### 24. Set Capacity / Criteria / Fee Configuration
- **Method:** `PUT`
- **URL:** `/api/v1/admissions/capacity-config`
- **Description:** Set capacity, evaluation rubric, and fee gating for a level/session (SCR-ADM-10).
- **Authentication (Role-Based):** Yes — `admission.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "capacity": {
      "type": "number",
      "description": "\u22650"
    },
    "criteria": {
      "type": "object"
    },
    "feeGating": {
      "type": "object",
      "properties": {
        "required": {
          "type": "string"
        },
        "amountRef": {
          "type": "number"
        }
      },
      "required": [
        "required"
      ]
    },
    "version": {
      "type": "number"
    }
  },
  "required": [
    "capacity",
    "criteria",
    "feeGating",
    "version"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "capacity": {
      "type": "number"
    },
    "criteria": {
      "type": "string"
    },
    "feeGating": {
      "type": "string"
    },
    "version": {
      "type": "number"
    },
    "status": {
      "type": "string",
      "const": "PUBLISHED"
    }
  },
  "required": [
    "capacity",
    "criteria",
    "feeGating",
    "version",
    "status"
  ]
}
```
- **Validation Rules:** `capacity ≥ 0`; `version` optimistic lock; changes are versioned (CFG-004) and apply to subsequent evaluation/conversion only.
- **Error Codes:** `422 INVALID_CAPACITY`; `409 VERSION_CONFLICT`.
- **Business Rules Applied:** ADM-004, ADM-006, CFG-002/004.
- **Pagination:** N/A.

## Reporting & Data Transfer

### 25. Admission Funnel & Intake Report
- **Method:** `GET`
- **URL:** `/api/v1/admissions/report/funnel`
- **Description:** Submitted → offered → converted funnel and intake KPIs (SCR-ADM-11).
- **Authentication (Role-Based):** Yes — `admission.application.view` (the approved screen names a generic "report view," scoped; this contract reuses the resource's own view permission, consistent with the convention applied across report screens).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "levelClassId": {
      "type": "string"
    },
    "sessionId": {
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
  "oneOf": [
    {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "stage": {
            "type": "string"
          },
          "count": {
            "type": "number"
          }
        },
        "required": [
          "stage",
          "count"
        ]
      }
    },
    {
      "type": "object",
      "properties": {
        "conversionRate": {
          "type": "string"
        },
        "capacityVsOffers": {
          "type": "number"
        }
      },
      "required": [
        "conversionRate",
        "capacityVsOffers"
      ]
    }
  ]
}
```
- **Validation Rules:** Scope-limited; large date ranges run async per conventions §9.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** REP-002 (authorization-preserving reporting), REP-005 (consistent definitions).
- **Pagination:** N/A (aggregate).

### 26. Import Applications
- **Method:** `POST`
- **URL:** `/api/v1/admissions/applications/import`
- **Description:** Bulk-import applications from a file (SCR-ADM-12).
- **Authentication (Role-Based):** Yes — `admission.import`.
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
              "schemadefinedfields": {
                "type": "string",
                "description": "...schema-defined fields"
              }
            }
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
              "applicationId": {
                "type": "string"
              }
            },
            "required": [
              "row",
              "applicationId"
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
- **Validation Rules:** Each row validated against the published schema; duplicate applicants are flagged (ADM-002) rather than silently re-submitted; invalid/duplicate rows excluded and always reported.
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** ADM-001, ADM-002, CFG-010; conventions §9.
- **Pagination:** N/A.

### 27. Export Applicant Data
- **Method:** `GET`
- **URL:** `/api/v1/admissions/applicants/export`
- **Description:** Governed, masked export of applicant data (SCR-ADM-13).
- **Authentication (Role-Based):** Yes — `admission.export`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "levelClassId": {
      "type": "string"
    },
    "sessionId": {
      "type": "string"
    },
    "stage": {
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
- **Business Rules Applied:** REP-002, REP-008 (export audited), conventions §10.
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
