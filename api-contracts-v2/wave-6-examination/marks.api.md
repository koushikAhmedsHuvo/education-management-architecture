# Marks API

## 1. API Overview

**Purpose.** Capture, lock, moderate, and govern post-lock correction of marks — ownership-scoped entry, validation against maxima, distinct special outcomes, completeness-gated submit-and-lock immutability, bounded transparent moderation, and SoD-governed correction.

**Module Context.** Implements Business Rules Catalog Doc 14 (Examination) Rules EXM-004/005/006/007/008 and UI Screen Spec `15-examination-screens.md` SCR-EXM-04…08/12…14.

---

## 2. Endpoints

### 1. Get Mark-Entry Roster
- **Method:** `GET`
- **URL:** `/api/v1/exams/{id}/mark-entry/roster`
- **Description:** The dense, ownership-scoped, windowed roster a subject-teacher enters marks against (SCR-EXM-04, the screen's own "★ high-care" designation).
- **Authentication (Role-Based):** Yes — `exam.marks.enter`, **resolved to the caller's owned subject/section only** — a teacher sees only their own sheets; non-owned sheets are never returned, let alone selectable.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "subjectId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "componentKey": {
      "type": "string"
    }
  },
  "required": [
    "subjectId",
    "sectionId"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "windowOpen": {
      "type": "boolean"
    },
    "maxMarks": {
      "type": "number"
    },
    "students": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "studentId": {
            "type": "string"
          },
          "rollNumber": {
            "type": "integer"
          },
          "name": {
            "type": "string"
          },
          "mark": {
            "type": "number"
          },
          "specialOutcome": {
            "type": "string",
            "enum": [
              "AB",
              "EX",
              "MAL"
            ]
          },
          "status": {
            "type": "string",
            "enum": [
              "DRAFT",
              "SUBMITTED",
              "LOCKED"
            ]
          }
        },
        "required": [
          "studentId",
          "rollNumber",
          "name",
          "status"
        ]
      }
    }
  },
  "required": [
    "windowOpen",
    "maxMarks",
    "students"
  ]
}
```
- **Validation Rules:** The (subject, section) pair must be owned by the caller per the Subject-Teacher assignment (`11-subject-api.md` #8, EXM-005/SUB-004) or the caller must hold exam-controller authority entering on-behalf (attributed); only currently eligible students appear.
- **Error Codes:** `403 NOT_OWNED_SHEET`.
- **Business Rules Applied:** EXM-005 (ownership-scoped mark entry), EXM-009 (entry window).
- **Pagination:** N/A (virtualized roster, not server-paginated — the whole eligible roster is returned for client-side virtualization).

### 2. Save Marks Draft
- **Method:** `POST`
- **URL:** `/api/v1/exams/{id}/mark-entry/draft`
- **Description:** Save a batch of mark or special-outcome entries — idempotent, batched, autosave-friendly (SCR-EXM-04/05).
- **Authentication (Role-Based):** Yes — `exam.marks.enter`, owned sheet only (as #10).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "subjectId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "componentKey": {
      "type": "string"
    },
    "entries": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "studentId": {
            "type": "string"
          },
          "mark": {
            "type": "number"
          },
          "specialOutcome": {
            "type": "string",
            "enum": [
              "AB",
              "EX",
              "MAL"
            ]
          }
        },
        "required": [
          "studentId"
        ]
      }
    },
    "idempotencyKey": {
      "type": "string"
    }
  },
  "required": [
    "subjectId",
    "sectionId",
    "entries",
    "idempotencyKey"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "saved": {
      "type": "number"
    },
    "status": {
      "type": "string",
      "const": "DRAFT"
    }
  },
  "required": [
    "saved",
    "status"
  ]
}
```
- **Validation Rules:** Each entry has **either** `mark` **or** `specialOutcome`, never both ("Enter a mark or a special outcome, not both," EXM-006); `mark` must be within `[0, maxMarks]` ("Mark exceeds maximum (\<max\>)," EXM-004); only permitted while the entry window is open ("Entry window closed," EXM-009); `AB`/`EX`/`MAL` are recorded as distinct typed values, **never** coerced to a numeric zero (EXM-006); saving is idempotent — a retried/offline-queued submission with the same `idempotencyKey` never double-applies.
- **Error Codes:** `422 MARK_EXCEEDS_MAXIMUM`; `422 MARK_AND_OUTCOME_BOTH_SET`; `409 WINDOW_CLOSED`; `403 NOT_OWNED_SHEET`.
- **Business Rules Applied:** EXM-004 (mark validation against maxima), EXM-005 (ownership), EXM-006 (absence/exemption/malpractice are distinct, non-numeric outcomes), EXM-009 (entry window).
- **Pagination:** N/A.

### 3. Submit & Lock Marks
- **Method:** `POST`
- **URL:** `/api/v1/exams/{id}/mark-entry/submit-lock`
- **Description:** Finalize a mark sheet — completeness-gated and **irreversible** except via governed correction (SCR-EXM-06).
- **Authentication (Role-Based):** Yes — `exam.marks.submit`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "subjectId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "componentKey": {
      "type": "string"
    }
  },
  "required": [
    "subjectId",
    "sectionId"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "status": {
      "type": "string",
      "const": "LOCKED"
    },
    "lockedAt": {
      "type": "string"
    }
  },
  "required": [
    "status",
    "lockedAt"
  ]
}
```
- **Validation Rules:** **Every** eligible student must have a mark or a valid special-outcome marker — no silent gaps ("\<N\> students still unmarked — complete before locking," EXM-009/RES-005); once locked, marks become read-only and feed Result Processing; this action requires a fresh step-up MFA challenge regardless of session age (EXM-007).
- **Error Codes:** `422 INCOMPLETE_ENTRIES` (lists the missing students); `401 STEP_UP_REQUIRED`.
- **Business Rules Applied:** EXM-007 (submit-and-lock immutability), EXM-009 (completeness — no silent zeros).
- **Pagination:** N/A.

### 4. Apply Moderation / Grace
- **Method:** `POST`
- **URL:** `/api/v1/exams/{id}/mark-entry/moderation`
- **Description:** Apply a bounded, transparent adjustment (uniform moderation or grace-to-pass) as a distinct layer over the raw mark — the original is always preserved (SCR-EXM-07).
- **Authentication (Role-Based):** Yes — `exam.moderation.manage`; SoD may require a different actor than the entering teacher.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "subjectId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "rule": {
      "type": "string",
      "enum": [
        "GRACE_TO_PASS",
        "UNIFORM_SCALING"
      ]
    },
    "bound": {
      "type": "number"
    },
    "affectedStudentIds": {
      "type": "array",
      "items": {
        "type": "string"
      }
    }
  },
  "required": [
    "subjectId",
    "sectionId",
    "rule",
    "bound"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "status": {
      "type": "string",
      "enum": [
        "APPLIED",
        "PENDING_APPROVAL"
      ]
    },
    "adjustments": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "studentId": {
            "type": "string"
          },
          "rawMark": {
            "type": "number"
          },
          "adjustedMark": {
            "type": "number"
          },
          "basis": {
            "type": "string"
          }
        },
        "required": [
          "studentId",
          "rawMark",
          "adjustedMark",
          "basis"
        ]
      }
    }
  },
  "required": [
    "status",
    "adjustments"
  ]
}
```
- **Validation Rules:** The adjustment must stay within the configured policy bound; it is applied as a **separate, visible layer** — the raw mark is never overwritten, and both are shown in provenance; grace never pushes a mark above its maximum ("grace never exceeds the maximum mark"); high-impact moderation may route to approval rather than applying immediately.
- **Error Codes:** `422 ADJUSTMENT_EXCEEDS_BOUND`.
- **Business Rules Applied:** EXM-008 (bounded, transparent moderation & grace, AUTHZ-009 SoD).
- **Pagination:** N/A.

## Post-Lock Correction (Governed)

### 5. Propose Post-Lock Correction
- **Method:** `POST`
- **URL:** `/api/v1/exams/{id}/mark-entry/corrections`
- **Description:** Request a correction to a **locked** mark, preserving the original value (SCR-EXM-08).
- **Authentication (Role-Based):** Yes — `exam.marks.enter` (raises the request).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string"
    },
    "subjectId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "newMark": {
      "type": "number"
    },
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "studentId",
    "subjectId",
    "sectionId",
    "newMark",
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
- **Validation Rules:** Only permitted against a `LOCKED` mark (EXM-007); `reason` required; if the result has already been published downstream, approval cascades to trigger a Result revision (`17-result-processing-api.md` #12), not a silent recompute.
- **Error Codes:** `409 MARK_NOT_LOCKED`; `422 REASON_REQUIRED`.
- **Business Rules Applied:** EXM-007 (post-lock correction is governed; original preserved).
- **Pagination:** N/A.

### 6. List Correction Approvals
- **Method:** `GET`
- **URL:** `/api/v1/exams/correction-approvals`
- **Description:** Inbox for pending post-lock corrections (SCR-EXM-14).
- **Authentication (Role-Based):** Yes — `exam.correction.approve`, scoped.
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
      "studentId": {
        "type": "string"
      },
      "studentName": {
        "type": "string"
      },
      "subjectId": {
        "type": "string"
      },
      "currentMark": {
        "type": "number"
      },
      "proposedMark": {
        "type": "number"
      },
      "requestedBy": {
        "type": "string"
      },
      "reason": {
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
      "studentId",
      "studentName",
      "subjectId",
      "currentMark",
      "proposedMark",
      "requestedBy",
      "reason",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Requester ≠ approver (AUTHZ-009) — self-raised corrections are listed but the requester cannot decide them.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-009, EXM-007.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 7. Decide Correction
- **Method:** `POST`
- **URL:** `/api/v1/exams/correction-approvals/{id}/decide`
- **Description:** Approve or reject a pending correction; on approval, the corrected value applies, the original is retained in history, and affected results are flagged for recomputation.
- **Authentication (Role-Based):** Yes — `exam.correction.approve`; **blocked if the caller raised the request**; requires step-up MFA.
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
    "recomputeTriggered": {
      "type": "boolean"
    }
  },
  "required": [
    "id",
    "status",
    "recomputeTriggered"
  ]
}
```
- **Validation Rules:** Caller ≠ requester; step-up MFA satisfied; on approval the original raw mark is **never deleted**, only superseded with both values retained in provenance.
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED`; `401 STEP_UP_REQUIRED`.
- **Business Rules Applied:** EXM-007 (original preserved, audit-complete), AUTHZ-009.
- **Pagination:** N/A.

## Progress, Search & Reporting

### 8. Bulk Mark Entry / Import
- **Method:** `POST`
- **URL:** `/api/v1/exams/{id}/mark-entry/bulk-import`
- **Description:** Import marks from a file with full validation (SCR-EXM-12).
- **Authentication (Role-Based):** Yes — `exam.marks.enter`.
- **Request DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "subjectId": {
          "type": "string"
        },
        "sectionId": {
          "type": "string"
        },
        "rows": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "rollNumber": {
                "type": "integer"
              },
              "mark": {
                "type": "number"
              },
              "specialOutcome": {
                "type": "string"
              }
            },
            "required": [
              "rollNumber"
            ]
          }
        }
      },
      "required": [
        "subjectId",
        "sectionId",
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
              "studentId": {
                "type": "string"
              }
            },
            "required": [
              "row",
              "studentId"
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
- **Validation Rules:** Every row validated against the same rules as #11 ([0,max] EXM-004, ownership EXM-005, window EXM-009); out-of-range, out-of-window, or non-owned rows are excluded and always reported — never silently imported.
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** EXM-004, EXM-005, EXM-009; conventions §9.
- **Pagination:** N/A.

### 9. Export Marks
- **Method:** `GET`
- **URL:** `/api/v1/exams/marks/export`
- **Description:** Governed export of marks (SCR-EXM-13).
- **Authentication (Role-Based):** Yes — `report.export` (a shared, platform-level export permission used consistently across reporting screens in this module).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "examId": {
      "type": "string"
    },
    "subjectId": {
      "type": "string"
    },
    "classId": {
      "type": "string"
    }
  },
  "required": [
    "examId"
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
