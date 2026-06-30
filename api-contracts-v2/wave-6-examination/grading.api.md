# Grading API

## 1. API Overview

**Purpose.** Govern the grade scale — the configuration that converts marks into grades, grade points, and GPA/CGPA. Versioned and effective-dated so historical results never silently change, with a live coverage/boundary validator and governed overrides.

**Module Context.** Implements Business Rules Catalog Doc 16 (Grading) and UI Screen Spec `17-grading-screens.md` (SCR-GRD-01…11).

---

## 2. Endpoints

### 1. List Grade Scales
- **Method:** `GET`
- **URL:** `/api/v1/grading/scales`
- **Description:** Browse grade scales and their versions/scope (SCR-GRD-01).
- **Authentication (Role-Based):** Yes — `grading.view`, institute-scoped.
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
    "scopeinstituteclass": {
      "type": "string",
      "description": "scope? (institute|class|exam)"
    },
    "schemeABSOLUTERELATI": {
      "type": "string",
      "description": "scheme? (ABSOLUTE|RELATIVE)"
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
      "scheme": {
        "type": "string"
      },
      "scope": {
        "type": "object",
        "properties": {
          "level": {
            "type": "string"
          },
          "refId": {
            "type": "string"
          }
        },
        "required": [
          "level"
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
      "scheme",
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
- **Business Rules Applied:** GRD-001 (configurable, versioned scale), GRD-005 (scheme scope), AUTHZ-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Grade Scale Detail
- **Method:** `GET`
- **URL:** `/api/v1/grading/scales/{id}`
- **Description:** Full scale definition — bands, scheme, GPA rule, special grades, version history (SCR-GRD-02).
- **Authentication (Role-Based):** Yes — `grading.view`.
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
    "scheme": {
      "type": "string"
    },
    "scope": {
      "type": "string"
    },
    "bands": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "rangeFrom": {
            "type": "string"
          },
          "rangeTo": {
            "type": "string"
          },
          "gradeLabel": {
            "type": "string"
          },
          "points": {
            "type": "string"
          },
          "boundaryOwner": {
            "type": "string"
          }
        },
        "required": [
          "rangeFrom",
          "rangeTo",
          "gradeLabel",
          "points",
          "boundaryOwner"
        ]
      }
    },
    "gpaRule": {
      "type": "object"
    },
    "specialGrades": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "code": {
            "type": "string"
          },
          "label": {
            "type": "string"
          },
          "excludedFromAggregate": {
            "type": "boolean"
          }
        },
        "required": [
          "code",
          "label",
          "excludedFromAggregate"
        ]
      }
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
    "scheme",
    "scope",
    "bands",
    "gpaRule",
    "specialGrades",
    "version",
    "effectiveDate",
    "status",
    "versionHistory"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** GRD-001 through GRD-006.
- **Pagination:** N/A.

### 3. Define / Draft Grade Scale (Band Editor)
- **Method:** `POST`
- **URL:** `/api/v1/grading/scales`
- **Description:** Define a new grade scale's bands — the screen's own "★ coverage & boundary determinism" designation (SCR-GRD-03).
- **Authentication (Role-Based):** Yes — `grading.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "scope": {
      "type": "object",
      "properties": {
        "level": {
          "type": "string",
          "enum": [
            "institute",
            "class",
            "exam"
          ]
        },
        "refId": {
          "type": "string"
        }
      },
      "required": [
        "level"
      ]
    },
    "bands": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "rangeFrom": {
            "type": "number"
          },
          "rangeTo": {
            "type": "number"
          },
          "gradeLabel": {
            "type": "string"
          },
          "points": {
            "type": "number"
          }
        },
        "required": [
          "rangeFrom",
          "rangeTo",
          "gradeLabel",
          "points"
        ]
      }
    }
  },
  "required": [
    "name",
    "scope",
    "bands"
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
    "bands": {
      "type": "string",
      "description": "bands[]"
    },
    "version": {
      "type": "string",
      "description": "0"
    }
  },
  "required": [
    "id",
    "status",
    "version"
  ]
}
```
- **Validation Rules:** Drafts may be saved incomplete while being authored — full coverage/boundary validation runs at publish (#5), mirroring the Configuration Engine and Workflow Engine's draft/publish pattern.
- **Error Codes:** `422 INVALID_BAND_SHAPE`.
- **Business Rules Applied:** GRD-001 (scheme drafted before publish).
- **Pagination:** N/A.

### 4. Validate Band Coverage
- **Method:** `POST`
- **URL:** `/api/v1/grading/scales/{id}/validate-coverage`
- **Description:** Run the live coverage/overlap/boundary validator against the current draft bands, without publishing (SCR-GRD-03).
- **Authentication (Role-Based):** Yes — `grading.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "bands": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    }
  },
  "required": [
    "bands"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "valid": {
      "type": "boolean"
    },
    "gaps": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "from": {
            "type": "string"
          },
          "to": {
            "type": "string"
          }
        },
        "required": [
          "from",
          "to"
        ]
      }
    },
    "overlaps": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "at": {
            "type": "string"
          }
        },
        "required": [
          "at"
        ]
      }
    },
    "ambiguousBoundaries": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "value": {
            "type": "string"
          },
          "candidateBands": {
            "type": "array",
            "items": {
              "type": "string"
            }
          }
        },
        "required": [
          "value",
          "candidateBands"
        ]
      }
    }
  },
  "required": [
    "valid",
    "gaps",
    "overlaps",
    "ambiguousBoundaries"
  ]
}
```
- **Validation Rules:** Bands must cover **[0, max] contiguously with no overlaps or gaps** ("Gap between \<x\> and \<y\>," "Overlap at \<value\>," GRD-002); every exact boundary value must belong to **exactly one** band, with explicit inclusive/exclusive ownership ("Boundary \<value\> is ambiguous — assign it to one band," GRD-003) — this endpoint is purely diagnostic and never blocks the draft from being saved, only from being published.
- **Error Codes:** None (the response body itself carries the validation result, not an error).
- **Business Rules Applied:** GRD-002 (complete, non-overlapping band coverage), GRD-003 (explicit, deterministic boundary ownership).
- **Pagination:** N/A.

### 5. Publish Grade Scale Version
- **Method:** `POST`
- **URL:** `/api/v1/grading/scales/{id}/versions/{version}/publish`
- **Description:** Validate and publish an immutable, effective-dated scale version (UC-GRD-01, SCR-GRD-03).
- **Authentication (Role-Based):** Yes — `grading.config.manage`; publishing a scale already in use by results may route to approval (`grading.change.approve`, #10).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "effectiveDate": {
      "type": "string"
    }
  },
  "required": [
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
    "version": {
      "type": "integer"
    },
    "status": {
      "type": "string",
      "enum": [
        "ACTIVE",
        "PENDING_APPROVAL"
      ]
    },
    "effectiveDate": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "version",
    "status",
    "effectiveDate"
  ]
}
```
- **Validation Rules:** **Identical** to #4's checks, but here they **block publish outright** rather than just being reported — coverage gaps, overlaps, or ambiguous boundaries make the scale impossible to publish, protecting reproducible grading at the source; once published, the version is **immutable** — in-place edits are forbidden, a further change requires a new version (GRD-001/CFG-004); superseding a scale already referenced by in-use results triggers a warning surfaced to the publisher about stamping impact, but never alters those historical results.
- **Error Codes:** `422 GAP_DETECTED`; `422 OVERLAP_DETECTED`; `422 AMBIGUOUS_BOUNDARY`.
- **Business Rules Applied:** GRD-001 (immutable version on publish), GRD-002, GRD-003, CFG-004.
- **Pagination:** N/A.

## GPA, Scope & Special Grades Configuration

### 6. Set GPA / CGPA Rule
- **Method:** `PUT`
- **URL:** `/api/v1/grading/scales/{id}/gpa-rule`
- **Description:** Configure grade points per band and credit-weighted aggregation (SCR-GRD-04).
- **Authentication (Role-Based):** Yes — `grading.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "gpaScale": {
      "type": "string",
      "enum": [
        "4.0",
        "5.0",
        "10.0"
      ]
    },
    "aggregation": {
      "type": "string",
      "enum": [
        "CREDIT_WEIGHTED",
        "SIMPLE_AVERAGE"
      ]
    },
    "cgpaMethod": {
      "type": "string",
      "enum": [
        "CREDIT_WEIGHTED_ACROSS_TERMS",
        "AVERAGE_OF_GPAS"
      ]
    },
    "failedSubjectPolicy": {
      "type": "string",
      "enum": [
        "INCLUDE_ZERO_POINT",
        "EXCLUDE"
      ]
    }
  },
  "required": [
    "gpaScale",
    "aggregation",
    "cgpaMethod",
    "failedSubjectPolicy"
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
    "gpaRule": {
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
    "gpaRule"
  ]
}
```
- **Validation Rules:** GPA = sum(grade_point × credit) / sum(credit) over **graded** subjects only — non-graded/excluded subjects (per Subject's own `nonGraded` flag, `11-subject-api.md`) are excluded from the denominator, never counted as zero (GRD-004); special grades are excluded from the GPA calculation per their own configuration (#8, GRD-006).
- **Error Codes:** `422 ZERO_GRADED_CREDIT` (would divide by zero).
- **Business Rules Applied:** GRD-004 (GPA/CGPA computation rule), GRD-006 (special-grade exclusion).
- **Pagination:** N/A.

### 7. Set Scheme Scope
- **Method:** `PUT`
- **URL:** `/api/v1/grading/scales/{id}/scheme-scope`
- **Description:** Set absolute-vs-relative scheme and its applicable scope (SCR-GRD-05).
- **Authentication (Role-Based):** Yes — `grading.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "scheme": {
      "type": "string",
      "enum": [
        "ABSOLUTE",
        "RELATIVE"
      ]
    },
    "scope": {
      "type": "object",
      "properties": {
        "level": {
          "type": "string",
          "enum": [
            "institute",
            "class",
            "exam"
          ]
        },
        "refId": {
          "type": "string"
        }
      },
      "required": [
        "level"
      ]
    },
    "relativeConfig": {
      "type": "object",
      "properties": {
        "cohortDefinition": {
          "type": "string"
        },
        "minimumCohortSize": {
          "type": "number"
        }
      },
      "required": [
        "cohortDefinition",
        "minimumCohortSize"
      ]
    }
  },
  "required": [
    "scheme",
    "scope"
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
    "scheme": {
      "type": "string"
    },
    "scope": {
      "type": "string"
    },
    "relativeConfig": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "scheme",
    "scope"
  ]
}
```
- **Validation Rules:** Schemes resolve by scope using **most-specific-wins** (subject → class/stream → institute), the same precedence pattern as the Configuration Engine (CFG-003); `RELATIVE` (curve/percentile) grading requires `relativeConfig.minimumCohortSize` to be defined — a cohort below that minimum **automatically falls back to absolute grading**, and the fallback is itself audited, never silent.
- **Error Codes:** `422 RELATIVE_CONFIG_REQUIRED`.
- **Business Rules Applied:** GRD-005 (scope-specific schemes, absolute vs. relative, minimum-cohort guard).
- **Pagination:** N/A.

### 8. Set Special Grades
- **Method:** `PUT`
- **URL:** `/api/v1/grading/scales/{id}/special-grades`
- **Description:** Define non-numeric outcomes (Absent, Incomplete, Exempt, Withdrawn, Withheld) and their aggregate treatment (SCR-GRD-06).
- **Authentication (Role-Based):** Yes — `grading.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "specialGrades": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "code": {
            "type": "string",
            "enum": [
              "AB",
              "I",
              "EX",
              "W",
              "WH"
            ]
          },
          "label": {
            "type": "string"
          },
          "excludedFromAggregate": {
            "type": "boolean"
          }
        },
        "required": [
          "code",
          "label",
          "excludedFromAggregate"
        ]
      }
    }
  },
  "required": [
    "specialGrades"
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
    "specialGrades": {
      "type": "string",
      "description": "specialGrades[]"
    }
  },
  "required": [
    "id"
  ]
}
```
- **Validation Rules:** Every special grade's aggregate treatment must be **explicit** — an exempted subject is excluded from GPA, not silently dragged to zero; absent/incomplete typically block a final result until resolved, per the configured policy here, never coerced to a numeric grade point (GRD-006).
- **Error Codes:** `422 TREATMENT_UNDEFINED`.
- **Business Rules Applied:** GRD-006 (special grades defined and excluded correctly).
- **Pagination:** N/A.

## Re-versioning (Effective-Dated)

### 9. Re-version Grade Scale
- **Method:** `POST`
- **URL:** `/api/v1/grading/scales/{id}/reversion`
- **Description:** Create a new, effective-dated scale version under governance — past results remain bound to their originally stamped version (SCR-GRD-08).
- **Authentication (Role-Based):** Yes — `grading.config.manage`; high-impact re-versioning routes to approval (`grading.change.approve`, #10).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "bands": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    },
    "effectiveDate": {
      "type": "string"
    },
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "bands",
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
    "newVersion": {
      "type": "number"
    },
    "status": {
      "type": "string",
      "enum": [
        "PENDING_APPROVAL",
        "ACTIVE"
      ]
    },
    "effectiveDate": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "newVersion",
    "status",
    "effectiveDate"
  ]
}
```
- **Validation Rules:** Same coverage/boundary validation as #4/#5; the new version applies **forward only** — every result already stamped with a prior `gradeVer` continues to resolve against that prior version exactly as it did when computed, never retroactively altered ("past results remain bound to their stamped version," GRD-007).
- **Error Codes:** `422 GAP_DETECTED`; `422 OVERLAP_DETECTED`; `422 AMBIGUOUS_BOUNDARY`.
- **Business Rules Applied:** GRD-007 (effective-dated resolution & result stamping — the core historical-fidelity guarantee), GRD-001, GRD-002, GRD-003.
- **Pagination:** N/A.

## Governed Override

### 10. Propose Grade Override
- **Method:** `POST`
- **URL:** `/api/v1/grading/overrides`
- **Description:** Manually override a computed grade for a justified exceptional case — the computed value is always preserved, never erased (UC-GRD-03, SCR-GRD-07).
- **Authentication (Role-Based):** Yes — `grading.override` (elevated, request).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "resultId": {
      "type": "string"
    },
    "subjectId": {
      "type": "string"
    },
    "overrideGrade": {
      "type": "string"
    },
    "reason": {
      "type": "string"
    }
  },
  "required": [
    "resultId",
    "overrideGrade",
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
    "computedGrade": {
      "type": "string"
    },
    "proposedOverride": {
      "type": "string"
    }
  },
  "required": [
    "id",
    "status",
    "computedGrade",
    "proposedOverride"
  ]
}
```
- **Validation Rules:** `reason` required; the override is recorded as a **layer over** the computed grade — the original computed value is stored and remains visible in provenance, never destroyed ("computed value preserved").
- **Error Codes:** `422 REASON_REQUIRED`.
- **Business Rules Applied:** GRD-008 (governed, transparent grade overrides), AUTHZ-009 (SoD — not the entering teacher).
- **Pagination:** N/A.

### 11. List Scale-Change / Override Approvals
- **Method:** `GET`
- **URL:** `/api/v1/grading/approvals`
- **Description:** Inbox for pending scale publications, re-versions, and grade overrides (SCR-GRD-11).
- **Authentication (Role-Based):** Yes — `grading.change.approve`, scoped.
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
    "typeSCALE_PUBLISHSCA": {
      "type": "string",
      "description": "type? (SCALE_PUBLISH|SCALE_REVERSION|GRADE_OVERRIDE)"
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
      "summary",
      "requestedBy",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Requester ≠ approver (AUTHZ-009) — self-raised requests are listed but cannot be decided by their own requester.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTHZ-009, GRD-008.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 12. Decide Scale-Change / Override Request
- **Method:** `POST`
- **URL:** `/api/v1/grading/approvals/{id}/decide`
- **Description:** Approve or reject a pending scale publish, re-version, or grade override; on approval, the underlying action (#5/#9/#10) is committed.
- **Authentication (Role-Based):** Yes — `grading.change.approve`; **blocked if the caller raised the request**.
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
- **Validation Rules:** Caller ≠ requester; `reason` required on `REJECT`.
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED`; `422 REASON_REQUIRED`.
- **Business Rules Applied:** AUTHZ-009, GRD-008, GRD-007.
- **Pagination:** N/A.

## Reporting & Data Transfer

### 13. Grade Distribution Report
- **Method:** `GET`
- **URL:** `/api/v1/grading/report/distribution`
- **Description:** Grade distribution across a scope, scheme-aware (SCR-GRD-09).
- **Authentication (Role-Based):** Yes — `grading.view` (report authority).
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
    "scaleId": {
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
      "gradeLabel": {
        "type": "string"
      },
      "count": {
        "type": "integer"
      },
      "percent": {
        "type": "number"
      }
    },
    "required": [
      "gradeLabel",
      "count",
      "percent"
    ]
  }
}
```
- **Validation Rules:** Scope-limited; reflects the scheme (absolute/relative) actually in effect for the queried scope (GRD-005).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** REP-002, GRD-005.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 14. Import Grade Scale
- **Method:** `POST`
- **URL:** `/api/v1/grading/scales/import`
- **Description:** Import a grade scale from a file, coverage-validated before acceptance (SCR-GRD-10).
- **Authentication (Role-Based):** Yes — `grading.import`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "scope": {
      "type": "object",
      "properties": {
        "note": {
          "type": "string",
          "description": "..."
        }
      }
    },
    "bands": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
    }
  },
  "required": [
    "name",
    "scope",
    "bands"
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
    "validation": {
      "type": "object",
      "properties": {
        "valid": {
          "type": "boolean"
        },
        "gaps": {
          "type": "string"
        },
        "overlaps": {
          "type": "string"
        },
        "ambiguousBoundaries": {
          "type": "string"
        }
      },
      "required": [
        "valid"
      ]
    }
  },
  "required": [
    "id",
    "status",
    "validation"
  ]
}
```
- **Validation Rules:** The imported bands run through the **same** coverage/boundary validator as #4; an invalid import is held as a `DRAFT` with the validation errors attached — it is **never** silently published or treated as valid.
- **Error Codes:** `422 INVALID_SCALE_FILE`.
- **Business Rules Applied:** GRD-002, GRD-003.
- **Pagination:** N/A.

### 15. Export Grade Scale
- **Method:** `GET`
- **URL:** `/api/v1/grading/scales/export`
- **Description:** Governed export of a grade scale definition (SCR-GRD-10).
- **Authentication (Role-Based):** Yes — `grading.export`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "scaleId": {
      "type": "string"
    }
  },
  "required": [
    "scaleId"
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
