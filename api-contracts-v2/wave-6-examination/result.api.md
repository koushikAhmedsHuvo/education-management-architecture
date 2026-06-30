# Result API

## 1. API Overview

**Purpose.** Transform locked marks into finalized results — provenance-stamped deterministic computation, review, async withholding-aware publish, governed version-pinned revision, withhold/release, computation configuration, and reporting. Marksheet/transcript viewing is split out to `transcript.api.md` in this same wave.

**Module Context.** Implements Business Rules Catalog Doc 15 (Result Processing) and UI Screen Spec `16-result-processing-screens.md` SCR-RES-01…03/05…12.

---

## 2. Endpoints

### 1. Get Pre-Compute Checklist
- **Method:** `GET`
- **URL:** `/api/v1/results/precompute-checklist`
- **Description:** Verify marks are locked, complete, and an effective grade scale resolves, before allowing compute (SCR-RES-01).
- **Authentication (Role-Based):** Yes — `result.compute`, scoped.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "examId": {
      "type": "string"
    },
    "classId": {
      "type": "string"
    },
    "sectionId": {
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
    "marksLocked": {
      "type": "boolean"
    },
    "unlockedSubjects": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "complete": {
      "type": "boolean"
    },
    "incompleteStudents": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "studentId": {
            "type": "string"
          },
          "missingSubjects": {
            "type": "array",
            "items": {
              "type": "string"
            }
          }
        },
        "required": [
          "studentId",
          "missingSubjects"
        ]
      }
    },
    "effectiveGradeScaleResolved": {
      "type": "boolean"
    }
  },
  "required": [
    "marksLocked",
    "unlockedSubjects",
    "complete",
    "incompleteStudents",
    "effectiveGradeScaleResolved"
  ]
}
```
- **Validation Rules:** None beyond access (read-only diagnostic).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** EXM-007 (locked marks required), RES-005 (completeness), GRD-007 (effective scale resolution).
- **Pagination:** N/A.

### 2. Compute Result
- **Method:** `POST`
- **URL:** `/api/v1/results/compute`
- **Description:** Deterministically compute results for a scope from locked marks, stamping the provenance triple (UC-RES-01, SCR-RES-01, the screen's own "★ provenance" designation).
- **Authentication (Role-Based):** Yes — `result.compute`, scoped.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "examId": {
      "type": "string"
    },
    "classId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "idempotencyKey": {
      "type": "string"
    }
  },
  "required": [
    "examId",
    "idempotencyKey"
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
        "computed": {
          "type": "number"
        },
        "held": {
          "type": "number"
        },
        "provenance": {
          "type": "object",
          "properties": {
            "ruleVer": {
              "type": "string"
            },
            "configVer": {
              "type": "string"
            },
            "gradeVer": {
              "type": "string"
            },
            "computedAt": {
              "type": "string"
            }
          },
          "required": [
            "ruleVer",
            "configVer",
            "gradeVer",
            "computedAt"
          ]
        }
      },
      "required": [
        "computed",
        "held",
        "provenance"
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
- **Validation Rules:** Every subject in scope must have **locked** marks ("Marks not locked for \<subject\>," EXM-004/EXM-007); every student must have complete data — no required mark missing — or that student is **held**, not computed with an implicit zero ("Incomplete marks — cannot compute," RES-005, the completeness gate, no silent zeros); the effective grading scheme is resolved by effective date for the result's context (GRD-007); pass/fail is evaluated **layered** — component → subject → aggregate, so a student can pass every subject total yet fail overall on a component rule, or vice versa (RES-002); percentages/aggregates are rounded **exactly once**, at the policy-defined point, never at intermediate steps (RES-003); rank/position uses a defined cohort and a deterministic, configured tie-break — never an arbitrary ordering (RES-008); given the same locked marks and the same resolved versions, computation is **always reproducible** (RES-001).
- **Error Codes:** `409 MARKS_NOT_LOCKED` (per subject); `422 INCOMPLETE_DATA` (affected students held, not a hard failure of the whole batch); `422 ROUNDING_OR_TIEBREAK_UNDEFINED`.
- **Business Rules Applied:** RES-001 (deterministic, provenance-stamped computation), RES-002 (layered pass/fail), RES-003 (single-point rounding), RES-005 (completeness gate), RES-008 (deterministic rank/tie-break), RES-009 (async, performance-safe), GRD-007 (effective-scale resolution & stamping), EXM-007.
- **Pagination:** N/A.

### 3. Get Result Provenance
- **Method:** `GET`
- **URL:** `/api/v1/results/{id}/provenance`
- **Description:** View the exact `{ruleVer, configVer, gradeVer}` stamp and computed-at timestamp that produced this result.
- **Authentication (Role-Based):** Yes — `result.compute` or `result.view`, scoped.
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
    "resultId": {
      "type": "string"
    },
    "ruleVer": {
      "type": "string"
    },
    "configVer": {
      "type": "string"
    },
    "gradeVer": {
      "type": "string"
    },
    "computedAt": {
      "type": "string"
    },
    "recomputedFrom": {
      "type": "object",
      "properties": {
        "previousProvenance": {
          "type": "string"
        }
      },
      "required": [
        "previousProvenance"
      ]
    }
  },
  "required": [
    "resultId",
    "ruleVer",
    "configVer",
    "gradeVer",
    "computedAt"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** RES-001 (provenance is the core reproducibility guarantee).
- **Pagination:** N/A.

## Review & Completeness

### 4. List Computed Results for Review
- **Method:** `GET`
- **URL:** `/api/v1/results`
- **Description:** Review computed, unpublished results, with anomaly flags (SCR-RES-02).
- **Authentication (Role-Based):** Yes — `result.compute` (review authority).
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
    "examId": {
      "type": "string"
    },
    "statusDRAFTCALCULATE": {
      "type": "string",
      "description": "status? (DRAFT|CALCULATED|VERIFIED|PUBLISHED|REVISED|WITHHELD|INCOMPLETE)"
    },
    "classId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "anomaliesOnlyboolean": {
      "type": "string",
      "description": "anomaliesOnly? (boolean)"
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
      "aggregate": {
        "type": "string"
      },
      "division": {
        "type": "string"
      },
      "gpa": {
        "type": "string"
      },
      "rank": {
        "type": "integer"
      },
      "status": {
        "type": "string"
      },
      "anomalyFlags": {
        "type": "array",
        "items": {
          "type": "string"
        }
      }
    },
    "required": [
      "id",
      "studentId",
      "studentName",
      "aggregate",
      "division",
      "gpa",
      "status",
      "anomalyFlags"
    ]
  }
}
```
- **Validation Rules:** Scope-limited; `anomalyFlags` surfaces conditions like "all-subjects-failed" or "rank tie resolved by fallback" for human attention, never auto-resolved.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** RES-002, RES-005, RES-008.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 5. Approve for Publish
- **Method:** `POST`
- **URL:** `/api/v1/results/{id}/approve-for-publish`
- **Description:** Mark a reviewed result `VERIFIED` — only verified results may be published (SCR-RES-02).
- **Authentication (Role-Based):** Yes — `result.compute` (review authority).
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
      "const": "VERIFIED"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** Only from `CALCULATED`.
- **Error Codes:** `409 INVALID_STATE_TRANSITION`.
- **Business Rules Applied:** Doc 15 §6 (state machine — `CALCULATED → VERIFIED → PUBLISHED`).
- **Pagination:** N/A.

### 6. Get Completeness Gate Gaps
- **Method:** `GET`
- **URL:** `/api/v1/results/completeness-gate`
- **Description:** View missing marks/outcomes by subject/student that are blocking compute or publish (SCR-RES-08).
- **Authentication (Role-Based):** Yes — `result.compute`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "examId": {
      "type": "string"
    },
    "classId": {
      "type": "string"
    },
    "sectionId": {
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
      "missingSubjects": {
        "type": "array",
        "items": {
          "type": "string"
        }
      }
    },
    "required": [
      "studentId",
      "studentName",
      "missingSubjects"
    ]
  }
}
```
- **Validation Rules:** This is purely diagnostic — resolving a gap requires returning to `16-examination-api.md` #11 (mark entry), not an edit here.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** RES-005 (completeness gate — no silent zeros, every gap surfaced not guessed).
- **Pagination:** N/A.

## Publish

### 7. Publish Results
- **Method:** `POST`
- **URL:** `/api/v1/results/publish`
- **Description:** Publish verified results to students/guardians at scale — **always asynchronous** at cohort scope, the peak-event-safe path (UC-RES-02, SCR-RES-03).
- **Authentication (Role-Based):** Yes — `result.publish` (step-up MFA required, narrowly granted).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "examId": {
      "type": "string"
    },
    "classId": {
      "type": "string"
    },
    "sectionId": {
      "type": "string"
    },
    "staged": {
      "type": "boolean"
    },
    "idempotencyKey": {
      "type": "string"
    }
  },
  "required": [
    "examId",
    "idempotencyKey"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "null",
  "description": "202 Accepted { jobId }."
}
```
- **Validation Rules:** Only `VERIFIED` results are eligible; results flagged `WITHHELD` (#15) are **excluded from publication** automatically, never accidentally surfaced (RES-007); the operation is idempotent — re-running a publish job never double-issues a marksheet (RES-009); requires a fresh step-up MFA challenge.
- **Error Codes:** `401 STEP_UP_REQUIRED`; `409 RESULTS_NOT_VERIFIED`.
- **Business Rules Applied:** RES-009 (asynchronous, performance-safe, idempotent publishing), RES-007 (withheld results excluded), FILE-007 (immutable marksheets generated as part of publish).
- **Pagination:** N/A.

### 8. Get Publish Status
- **Method:** `GET`
- **URL:** `/api/v1/results/publish-status`
- **Description:** Live per-cohort publish progress — queued/running/published (SCR-RES-03).
- **Authentication (Role-Based):** Yes — `result.publish`.
- **Request DTO (JSON Schema):**
```json
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
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "jobId": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "enum": [
        "QUEUED",
        "RUNNING",
        "COMPLETED",
        "FAILED"
      ]
    },
    "perCohortProgress": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "classId": {
            "type": "string"
          },
          "sectionId": {
            "type": "string"
          },
          "published": {
            "type": "string"
          },
          "total": {
            "type": "number"
          }
        },
        "required": [
          "classId",
          "sectionId",
          "published",
          "total"
        ]
      }
    }
  },
  "required": [
    "jobId",
    "status",
    "perCohortProgress"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 JOB_NOT_FOUND`.
- **Business Rules Applied:** RES-009.
- **Pagination:** N/A.

## Result/Marksheet View

### 9. Submit Result Revision
- **Method:** `POST`
- **URL:** `/api/v1/results/revisions`
- **Description:** Propose a governed, version-pinned correction to a **published** result, with a provenance diff (UC-RES-03, SCR-RES-05, the screen's own "★ (C-02)" designation).
- **Authentication (Role-Based):** Yes — `result.revise` (elevated, request).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "resultId": {
      "type": "string"
    },
    "reason": {
      "type": "string"
    },
    "reEvaluationContext": {
      "type": "object"
    }
  },
  "required": [
    "resultId",
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
    },
    "provenanceDiff": {
      "type": "object",
      "properties": {
        "before": {
          "type": "object",
          "properties": {
            "note": {
              "type": "string",
              "description": "..."
            }
          }
        },
        "after": {
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
        "before",
        "after"
      ]
    }
  },
  "required": [
    "id",
    "status",
    "provenanceDiff"
  ]
}
```
- **Validation Rules:** Only a `PUBLISHED` result may be revised ("Only published results can be revised," RES-006); `reason` required; the proposed revision must follow the institute's **disclosed re-evaluation direction policy** — e.g., marks-can-only-increase versus can-change-either-way, as configured and shown to both requester and approver ("Revision direction violates the disclosed policy," C-02) — a violation blocks submission outright.
- **Error Codes:** `409 RESULT_NOT_PUBLISHED`; `422 DIRECTION_POLICY_VIOLATION`; `422 REASON_REQUIRED`.
- **Business Rules Applied:** RES-006 (governed post-publish revision), C-02 (disclosed re-evaluation direction policy).
- **Pagination:** N/A.

### 10. List Revision Approvals
- **Method:** `GET`
- **URL:** `/api/v1/results/revision-approvals`
- **Description:** Inbox for pending result revisions (SCR-RES-12).
- **Authentication (Role-Based):** Yes — `result.revision.approve` (SoD), scoped.
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
      "reason": {
        "type": "string"
      },
      "direction": {
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
      "studentId",
      "studentName",
      "reason",
      "direction",
      "requestedBy",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Requester ≠ approver — self-raised revisions are listed but the requester's decide action is disabled (AUTHZ-009).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-009, RES-006.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 11. Decide Result Revision
- **Method:** `POST`
- **URL:** `/api/v1/results/revision-approvals/{id}/decide`
- **Description:** Approve or reject a pending revision; on approval, a **new marksheet version** is issued, the prior version is retained, and the student/guardian is explicitly notified that the result changed.
- **Authentication (Role-Based):** Yes — `result.revision.approve`; **blocked if the caller raised the request**; step-up MFA required.
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
            "REJECTED"
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
        "PUBLISHEDREVISEDPU": {
          "type": "string",
          "description": "PUBLISHED \u2192 REVISED \u2192 PUBLISHED"
        }
      }
    }
  ]
}
```
- **Validation Rules:** Caller ≠ requester; step-up MFA satisfied; recompute (where the revision requires it) is **version-pinned** to the same provenance versions unless the revision itself explicitly changes them, in which case the new versions are stamped and disclosed.
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED`; `401 STEP_UP_REQUIRED`.
- **Business Rules Applied:** RES-006 (prior version retained, notification mandatory), AUTHZ-009, WFL-002/004.
- **Pagination:** N/A.

## Withholding

### 12. Withhold Result
- **Method:** `POST`
- **URL:** `/api/v1/results/{id}/withhold`
- **Description:** Withhold a result for a defined, fair, time-bound reason — dues, pending malpractice review, or incompleteness (SCR-RES-06).
- **Authentication (Role-Based):** Yes — `result.withhold.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "reason": {
      "type": "string",
      "enum": [
        "DUES",
        "MALPRACTICE_REVIEW",
        "OTHER"
      ]
    },
    "reasonDetail": {
      "type": "string"
    },
    "until": {
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
      "const": "WITHHELD"
    },
    "reason": {
      "type": "string"
    },
    "until": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "status",
    "reason"
  ]
}
```
- **Validation Rules:** `reason` always required — withholding without a recorded, transparent reason is not permitted ("withholding always carries a reason and an end condition"); statutory results may be exempt from fee-based withholding per applicable policy/law.
- **Error Codes:** `422 REASON_REQUIRED`; `422 STATUTORY_EXEMPT`.
- **Business Rules Applied:** RES-007 (withholding is defined, fair, time-bound, never indefinite or unaudited).
- **Pagination:** N/A.

### 13. Release Result
- **Method:** `POST`
- **URL:** `/api/v1/results/{id}/release`
- **Description:** Release a withheld result once its condition is resolved (e.g., dues paid).
- **Authentication (Role-Based):** Yes — `result.withhold.manage`.
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
      "const": "PUBLISHED"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** Release is immediate on resolution and does **not** require a full publish re-run ("released and published on resolution without a full re-run").
- **Error Codes:** `409 NOT_WITHHELD`.
- **Business Rules Applied:** RES-007.
- **Pagination:** N/A.

### 14. Bulk Withhold
- **Method:** `POST`
- **URL:** `/api/v1/results/withhold/bulk`
- **Description:** Withhold many results at once (e.g., a dues sweep before publish) (SCR-RES-06).
- **Authentication (Role-Based):** Yes — `result.withhold.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "resultIds": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "reason": {
      "type": "string"
    },
    "reasonDetail": {
      "type": "string"
    }
  },
  "required": [
    "resultIds",
    "reason"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "withheld": {
      "type": "number"
    }
  },
  "required": [
    "withheld"
  ]
}
```
- **Validation Rules:** Same per-result requirements as #15.
- **Error Codes:** `422 REASON_REQUIRED`.
- **Business Rules Applied:** RES-007.
- **Pagination:** N/A.

## Computation Configuration

### 15. Get Computation Configuration
- **Method:** `GET`
- **URL:** `/api/v1/results/computation-config`
- **Description:** View the current rounding, division/classification, tie-break, and GPA rule configuration — one leg of every result's provenance stamp (SCR-RES-07).
- **Authentication (Role-Based):** Yes — `result.config.manage`.
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
        "classId": {
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
    "roundingPoint": {
      "type": "string",
      "const": "FINAL_PERCENTAGE"
    },
    "precision": {
      "type": "number"
    },
    "method": {
      "type": "string",
      "enum": [
        "HALF_UP",
        "HALF_EVEN"
      ]
    },
    "divisionBands": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "label": {
            "type": "string"
          },
          "minPercent": {
            "type": "number"
          }
        },
        "required": [
          "label",
          "minPercent"
        ]
      }
    },
    "tieBreak": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "priority": {
            "type": "string"
          },
          "metric": {
            "type": "string"
          }
        },
        "required": [
          "priority",
          "metric"
        ]
      }
    },
    "gpaRule": {
      "type": "object"
    },
    "version": {
      "type": "integer"
    }
  },
  "required": [
    "roundingPoint",
    "precision",
    "method",
    "divisionBands",
    "tieBreak",
    "gpaRule",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** RES-003, RES-004, RES-008, CFG-004.
- **Pagination:** N/A.

### 16. Set Computation Configuration
- **Method:** `PUT`
- **URL:** `/api/v1/results/computation-config`
- **Description:** Configure the single rounding point, division bands, deterministic tie-break, and GPA rule, versioned (SCR-RES-07).
- **Authentication (Role-Based):** Yes — `result.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "roundingPoint": {
      "type": "string"
    },
    "precision": {
      "type": "string"
    },
    "method": {
      "type": "string"
    },
    "divisionBands": {
      "type": "string"
    },
    "tieBreak": {
      "type": "string"
    },
    "gpaRule": {
      "type": "string"
    },
    "version": {
      "type": "number"
    }
  },
  "required": [
    "roundingPoint",
    "precision",
    "method",
    "divisionBands",
    "tieBreak",
    "gpaRule",
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
    },
    "status": {
      "type": "string",
      "const": "PUBLISHED"
    }
  },
  "required": [
    "version",
    "status"
  ]
}
```
- **Validation Rules:** Exactly **one** rounding point is defined — never applied at multiple intermediate steps ("single rounding point," RES-003); `tieBreak` must be a fully deterministic, ordered rule — an undefined tie-break blocks computation rather than producing an arbitrary order (RES-008); `divisionBands`/`gpaRule` consistent with pass/fail (RES-004); this version becomes the `configVer` leg of subsequent results' provenance stamp; `version` optimistic lock.
- **Error Codes:** `422 ROUNDING_POLICY_UNDEFINED`; `422 TIEBREAK_UNDEFINED`; `409 VERSION_CONFLICT`.
- **Business Rules Applied:** RES-003, RES-004, RES-008, CFG-004.
- **Pagination:** N/A.

## Reporting, Bulk Compute & Export

### 17. Result Analysis & Tabulation Report
- **Method:** `GET`
- **URL:** `/api/v1/results/report/analysis-tabulation`
- **Description:** Pass-rate, division spread, and tabulation analysis (SCR-RES-09).
- **Authentication (Role-Based):** Yes — `result.view` (report authority).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "examId": {
      "type": "string"
    },
    "classId": {
      "type": "string"
    },
    "sectionId": {
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
    "passRate": {
      "type": "string"
    },
    "divisionSpread": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "label": {
            "type": "string"
          },
          "count": {
            "type": "integer"
          }
        },
        "required": [
          "label",
          "count"
        ]
      }
    },
    "byStudent": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "studentId": {
            "type": "string"
          },
          "aggregate": {
            "type": "string"
          },
          "division": {
            "type": "string"
          },
          "gpa": {
            "type": "string"
          },
          "rank": {
            "type": "integer"
          }
        },
        "required": [
          "studentId",
          "aggregate",
          "division",
          "gpa"
        ]
      }
    }
  },
  "required": [
    "passRate",
    "divisionSpread",
    "byStudent"
  ]
}
```
- **Validation Rules:** Scope-limited; uses the same consistent metric definitions as the rest of the module (REP-005); large tabulations run async per conventions §9.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** REP-002, REP-005, RES-002, RES-004.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 18. Bulk Compute / Recompute
- **Method:** `POST`
- **URL:** `/api/v1/results/bulk-compute`
- **Description:** Compute or recompute results at scale (SCR-RES-10).
- **Authentication (Role-Based):** Yes — `result.compute`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "examId": {
      "type": "string"
    },
    "classIds": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "idempotencyKey": {
      "type": "string"
    }
  },
  "required": [
    "examId",
    "idempotencyKey"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "null",
  "description": "202 Accepted { jobId } (per-student results, each individually provenance-stamped)."
}
```
- **Validation Rules:** Same gates as #2 applied per student (locked, complete, resolved versions); a recompute re-stamps the provenance triple — a deliberate, visible change, not a silent overwrite.
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** RES-001, RES-005, RES-009; conventions §9.
- **Pagination:** N/A.

### 19. Export Results / Tabulation
- **Method:** `GET`
- **URL:** `/api/v1/results/export`
- **Description:** Governed export of results/tabulation (SCR-RES-11).
- **Authentication (Role-Based):** Yes — `report.export` (the shared, platform-level export permission used consistently across reporting screens).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "examId": {
      "type": "string"
    },
    "classId": {
      "type": "string"
    },
    "sectionId": {
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
- **Validation Rules:** Scope-limited; export is audited; withheld results are excluded from the export by default.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** REP-002, REP-008, RES-007, conventions §10.
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
