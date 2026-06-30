# Guardian API

## 1. API Overview

**Purpose.** Govern the guardian — a child-safety and access-control boundary, not contact data. Children-only portal scope, custody/contact restriction enforcement, parental consent, and governed guardianship changes that can never orphan a minor.

**Module Context.** Implements Business Rules Catalog Doc 09 (Guardian) and UI Screen Spec `09-guardian-management-screens.md` (SCR-GRDN-01…13).

---

## 2. Endpoints

### 1. List Guardians
- **Method:** `GET`
- **URL:** `/api/v1/guardians`
- **Description:** Browse guardians within scope, masked by default (SCR-GRDN-01).
- **Authentication (Role-Based):** Yes — `guardian.view`, scoped to the children's institute/campus.
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
    "relationship": {
      "type": "string"
    },
    "rolefinancialemergen": {
      "type": "string",
      "description": "role? (financial|emergency|portal)"
    },
    "hasRestrictionboolean": {
      "type": "string",
      "description": "hasRestriction? (boolean)"
    },
    "campusId": {
      "type": "string"
    },
    "sortBynamelinkedStud": {
      "type": "string",
      "description": "sortBy? (name|linkedStudents|updatedAt, default name)"
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
      "relationships": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "linkedStudentCount": {
        "type": "integer"
      },
      "isFinancialResponsibleAny": {
        "type": "boolean"
      },
      "restrictionFlags": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "contact": {
        "type": "string",
        "description": "\"MASKED\"|value"
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
      "relationships",
      "linkedStudentCount",
      "isFinancialResponsibleAny",
      "restrictionFlags",
      "contact",
      "updatedAt",
      "version"
    ]
  }
}
```
- **Validation Rules:** No free-text search against masked contact/national-ID fields without `student.sensitive.view`-equivalent authority.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** GRD-N-001 (guardian-of-record), GRD-N-006 (restriction visibility), STU-005 (masking), AUTHZ-002.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Get Guardian Detail
- **Method:** `GET`
- **URL:** `/api/v1/guardians/{id}`
- **Description:** Profile, linked students, roles, restrictions, and consent history (SCR-GRDN-02).
- **Authentication (Role-Based):** Yes — `guardian.view`; a guardian may always view their own profile.
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
    "relationship": {
      "type": "string"
    },
    "contact": {
      "type": "string",
      "description": "\"MASKED\"|value"
    },
    "nationalId": {
      "type": "string",
      "description": "\"MASKED\"|value"
    },
    "linkedStudents": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "studentId": {
            "type": "string"
          },
          "name": {
            "type": "string"
          },
          "relationship": {
            "type": "string"
          },
          "isFinancialResponsible": {
            "type": "boolean"
          },
          "emergencyPriority": {
            "type": "string"
          },
          "portalAccess": {
            "type": "string"
          },
          "restrictions": {
            "type": "array",
            "items": {
              "type": "string"
            }
          }
        },
        "required": [
          "studentId",
          "name",
          "relationship",
          "isFinancialResponsible",
          "emergencyPriority",
          "portalAccess",
          "restrictions"
        ]
      }
    },
    "consentHistory": {
      "type": "array",
      "items": {
        "type": "string",
        "description": "..."
      }
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
    "relationship",
    "contact",
    "nationalId",
    "linkedStudents",
    "consentHistory",
    "createdAt",
    "updatedAt",
    "version"
  ]
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `404 NOT_FOUND`.
- **Business Rules Applied:** GRD-N-001, GRD-N-006, GRD-N-008.
- **Pagination:** N/A.

### 3. Check for Duplicate Guardian
- **Method:** `GET`
- **URL:** `/api/v1/guardians/dedup-check`
- **Description:** Surface a likely-existing guardian before creating a new record — siblings should reuse one guardian, not duplicate (SCR-GRDN-03).
- **Authentication (Role-Based):** Yes — `guardian.create`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "contact": {
      "type": "string"
    },
    "nationalId": {
      "type": "string"
    }
  },
  "required": [
    "name",
    "contact"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "candidates": {
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
          "contact": {
            "type": "string",
            "const": "MASKED"
          },
          "matchedOn": {
            "type": "array",
            "items": {
              "type": "string"
            }
          }
        },
        "required": [
          "id",
          "name",
          "contact",
          "matchedOn"
        ]
      }
    }
  },
  "required": [
    "candidates"
  ]
}
```
- **Validation Rules:** Presented for human review; the create flow nudges "link instead of creating" on a strong match but does not auto-block.
- **Error Codes:** None.
- **Business Rules Applied:** GRD-N-003 (one guardian, many students — sibling reuse).
- **Pagination:** N/A.

### 4. Create / Designate Guardian
- **Method:** `POST`
- **URL:** `/api/v1/guardians`
- **Description:** Create a new guardian record (SCR-GRDN-03, UC-GRD-01).
- **Authentication (Role-Based):** Yes — `guardian.create`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "relationship": {
      "type": "string"
    },
    "contact": {
      "type": "string"
    },
    "nationalId": {
      "type": "string"
    },
    "dedupConfirmation": {
      "type": "string",
      "description": "\"CONFIRMED_NEW\"|{ useExistingId: string }"
    }
  },
  "required": [
    "name",
    "relationship",
    "contact",
    "dedupConfirmation"
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
    "relationship": {
      "type": "string"
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
    "name",
    "relationship",
    "createdAt",
    "version"
  ]
}
```
- **Validation Rules:** The dedup result (#3) must be explicitly confirmed before creation; `nationalId` is stored masked-by-default in all subsequent reads (STU-005).
- **Error Codes:** `422 DUPLICATE_NOT_CONFIRMED`.
- **Business Rules Applied:** GRD-N-001 (guardian-of-record designation), GRD-N-003 (dedup/sibling reuse).
- **Pagination:** N/A.

## Linking & Roles

### 5. Link Guardian to Student(s)
- **Method:** `POST`
- **URL:** `/api/v1/guardians/{id}/students`
- **Description:** Link an existing guardian to one or more students with relationship and roles (SCR-GRDN-04).
- **Authentication (Role-Based):** Yes — `guardian.link.manage` (+ `guardian.role.manage` for setting roles in the same call).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "links": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "studentId": {
            "type": "string"
          },
          "relationship": {
            "type": "string"
          },
          "isFinancialResponsible": {
            "type": "boolean"
          },
          "emergencyPriority": {
            "type": "number"
          },
          "portalAccess": {
            "type": "boolean"
          }
        },
        "required": [
          "studentId",
          "relationship"
        ]
      }
    }
  },
  "required": [
    "links"
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
        "guardianId": {
          "type": "string"
        },
        "links": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "studentId": {
                "type": "string"
              },
              "relationship": {
                "type": "string"
              },
              "linkedAt": {
                "type": "string"
              }
            },
            "required": [
              "studentId",
              "relationship",
              "linkedAt"
            ]
          }
        }
      },
      "required": [
        "guardianId",
        "links"
      ]
    },
    {
      "type": "object",
      "properties": {
        "portalAccess": {
          "type": "string"
        }
      },
      "required": [
        "portalAccess"
      ]
    }
  ]
}
```
- **Validation Rules:** Exactly one active **primary** guardian-of-record is enforced per student — linking a second primary requires first demoting the existing one (GRD-N-001); `emergencyPriority`, where used, must be unique per student if the institute enforces strict ordering.
- **Error Codes:** `409 PRIMARY_ALREADY_DESIGNATED`; `422 DUPLICATE_EMERGENCY_PRIORITY`.
- **Business Rules Applied:** GRD-N-001, GRD-N-002 (≥1 active guardian for a minor, checked on the student side), GRD-N-004 (children-only scope), GRD-N-008 (financial/emergency roles).
- **Pagination:** N/A.

### 6. Set Guardian Roles for a Student Link
- **Method:** `PUT`
- **URL:** `/api/v1/guardians/{id}/students/{studentId}/roles`
- **Description:** Update financial-responsible, emergency-priority, and portal-access flags on an existing link (SCR-GRDN-04/08).
- **Authentication (Role-Based):** Yes — `guardian.role.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "isFinancialResponsible": {
      "type": "boolean"
    },
    "emergencyPriority": {
      "type": "number"
    },
    "portalAccess": {
      "type": "boolean"
    }
  }
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "guardianId": {
      "type": "string"
    },
    "studentId": {
      "type": "string"
    },
    "isFinancialResponsible": {
      "type": "boolean"
    },
    "emergencyPriority": {
      "type": "string"
    },
    "portalAccess": {
      "type": "string"
    }
  },
  "required": [
    "guardianId",
    "studentId",
    "isFinancialResponsible",
    "emergencyPriority",
    "portalAccess"
  ]
}
```
- **Validation Rules:** Capabilities derive strictly from these role flags — only a `isFinancialResponsible` guardian sees/pays fees by default; `portalAccess` controls login eligibility (GRD-N-005); where the institute requires at least one financial-responsible guardian per fee-bearing student, removing the last one is blocked.
- **Error Codes:** `409 NO_FINANCIAL_RESPONSIBLE_REMAINING` (where required).
- **Business Rules Applied:** GRD-N-005 (role-scoped capabilities), GRD-N-008.
- **Pagination:** N/A.

## Guardian Portal (Self-Service, Children-Only)

### 7. Get My Children
- **Method:** `GET`
- **URL:** `/api/v1/guardians/me/children`
- **Description:** The portal's entry point — list only the caller's own linked, non-restricted children (SCR-GRDN-05).
- **Authentication (Role-Based):** Yes — self (guardian).
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
  "oneOf": [
    {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "studentId": {
            "type": "string"
          },
          "name": {
            "type": "string"
          },
          "classId": {
            "type": "string"
          },
          "sectionId": {
            "type": "string"
          },
          "photoUrl": {
            "type": "string"
          }
        },
        "required": [
          "studentId",
          "name",
          "classId",
          "sectionId"
        ]
      }
    },
    {
      "type": "object",
      "properties": {
        "NO_ACCESS": {
          "type": "string"
        }
      },
      "required": [
        "NO_ACCESS"
      ]
    }
  ]
}
```
- **Validation Rules:** The set is resolved strictly from active, non-restricted links (GRD-N-004); a revoked or restricted link ends visibility immediately on the very next call, never cached stale.
- **Error Codes:** None (an empty list is a valid, neutral result — "No students linked to your account").
- **Business Rules Applied:** GRD-N-004 (strict children-only ownership scope), GRD-N-006 (restriction enforcement, fail-closed).
- **Pagination:** N/A.

### 8. Get My Child Summary
- **Method:** `GET`
- **URL:** `/api/v1/guardians/me/children/{studentId}/summary`
- **Description:** A role/consent-gated summary panel (fees due, results, attendance) for one linked child (SCR-GRDN-05). *Detailed fee, result, and attendance data are owned by their respective modules (Finance, Examination, Attendance — later waves); this endpoint returns only the gated summary tiles this screen needs, and links out to those modules' own detail endpoints once specified.*
- **Authentication (Role-Based):** Yes — self (guardian); `studentId` must be in the caller's `7. Get My Children` result.
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
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "studentId": {
          "type": "string"
        },
        "feesDueVisible": {
          "type": "boolean"
        },
        "feesDueSummary": {
          "type": "object",
          "properties": {
            "amount": {
              "type": "number"
            }
          },
          "required": [
            "amount"
          ]
        },
        "resultsVisible": {
          "type": "boolean"
        },
        "attendanceVisible": {
          "type": "boolean"
        }
      },
      "required": [
        "studentId",
        "feesDueVisible",
        "resultsVisible",
        "attendanceVisible"
      ]
    },
    {
      "type": "object",
      "properties": {
        "feesDueSummary": {
          "type": "string"
        }
      },
      "required": [
        "feesDueSummary"
      ]
    },
    {
      "type": "object",
      "properties": {
        "isFinancialResponsible": {
          "type": "boolean"
        }
      },
      "required": [
        "isFinancialResponsible"
      ]
    }
  ]
}
```
- **Validation Rules:** A non-linked or restricted `studentId` returns the same neutral not-found response as any other out-of-scope resource — never a distinguishable "restricted" error.
- **Error Codes:** `404 NOT_FOUND` (not linked, or restricted — indistinguishable).
- **Business Rules Applied:** GRD-N-004, GRD-N-005, GRD-N-006, GRD-N-008.
- **Pagination:** N/A.

## Custody / Contact Restrictions

### 9. List Restrictions for a Guardian
- **Method:** `GET`
- **URL:** `/api/v1/guardians/{id}/restrictions`
- **Description:** View all active and historical restrictions recorded against this guardian's links (SCR-GRDN-06).
- **Authentication (Role-Based):** Yes — `guardian.restriction.manage`.
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
      "id": {
        "type": "string"
      },
      "type": {
        "type": "string",
        "enum": [
          "NO_ACCESS",
          "NO_CONTACT",
          "SUPERVISED"
        ]
      },
      "studentId": {
        "type": "string"
      },
      "effectiveFrom": {
        "type": "string"
      },
      "reason": {
        "type": "string"
      },
      "documentRef": {
        "type": "string"
      },
      "status": {
        "type": "string",
        "enum": [
          "ACTIVE",
          "REMOVED"
        ]
      }
    },
    "required": [
      "id",
      "type",
      "studentId",
      "effectiveFrom",
      "reason",
      "status"
    ]
  }
}
```
- **Validation Rules:** None beyond access (this is highly sensitive data — confidentiality is enforced by the permission gate itself).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** GRD-N-006.
- **Pagination:** N/A.

### 10. Create Restriction
- **Method:** `POST`
- **URL:** `/api/v1/guardians/{id}/restrictions`
- **Description:** Record a court-ordered or institution-recorded custody/contact restriction — enforced immediately and system-wide (SCR-GRDN-06). *(This is the same underlying write as `13-student-api.md` #13, exposed here for the guardian-anchored entry point; both converge on one restriction record.)*
- **Authentication (Role-Based):** Yes — `guardian.restriction.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "type": {
      "type": "string",
      "enum": [
        "NO_ACCESS",
        "NO_CONTACT",
        "SUPERVISED"
      ]
    },
    "studentId": {
      "type": "string"
    },
    "effectiveDate": {
      "type": "string"
    },
    "reason": {
      "type": "string"
    },
    "documentRef": {
      "type": "string"
    }
  },
  "required": [
    "type",
    "studentId",
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
    "type": {
      "type": "string"
    },
    "studentId": {
      "type": "string"
    },
    "effectiveFrom": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "const": "ACTIVE"
    }
  },
  "required": [
    "id",
    "type",
    "studentId",
    "effectiveFrom",
    "status"
  ]
}
```
- **Validation Rules:** `reason` required ("Reason required for a restriction"); takes effect immediately across portal access (#7's list), notification routing, and file access (`13-student-api.md` #18) — there is no propagation delay.
- **Error Codes:** `422 REASON_REQUIRED`.
- **Business Rules Applied:** GRD-N-006 (system-wide restriction enforcement).
- **Pagination:** N/A.

### 11. Remove Restriction (Governed)
- **Method:** `DELETE`
- **URL:** `/api/v1/guardians/{id}/restrictions/{restrictionId}`
- **Description:** Lift a restriction — itself a governed, audited, potentially elevated-authority action (SCR-GRDN-06).
- **Authentication (Role-Based):** Yes — `guardian.restriction.manage`; removal may require elevated/safeguarding authority per institute policy.
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
      "const": "REMOVED"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** `reason` required; this is never a casual toggle — full audit trail retained on both creation and removal.
- **Error Codes:** `422 REASON_REQUIRED`.
- **Business Rules Applied:** GRD-N-006.
- **Pagination:** N/A.

## Parental Consent

### 12. Get Consent History
- **Method:** `GET`
- **URL:** `/api/v1/guardians/{id}/consent`
- **Description:** View consent grant/revoke history for a guardian's linked minor(s) (SCR-GRDN-07).
- **Authentication (Role-Based):** Yes — `guardian.consent.manage` (admin), or self (own consent record).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string"
    },
    "consentType": {
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
      "consentType": {
        "type": "string"
      },
      "studentId": {
        "type": "string"
      },
      "status": {
        "type": "string",
        "enum": [
          "GRANTED",
          "REVOKED"
        ]
      },
      "date": {
        "type": "string"
      },
      "scope": {
        "type": "string"
      }
    },
    "required": [
      "consentType",
      "studentId",
      "status",
      "date",
      "scope"
    ]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** GRD-N-008 (parental consent for minors).
- **Pagination:** N/A.

### 13. Grant or Revoke Consent
- **Method:** `POST`
- **URL:** `/api/v1/guardians/{id}/consent`
- **Description:** Record a consent grant or withdrawal for a specific, configured scope (e.g., media, optional data use) (SCR-GRDN-07).
- **Authentication (Role-Based):** Yes — self (the guardian-of-record, for their own linked children) or `guardian.consent.manage` (admin, on the guardian's behalf where supported).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string"
    },
    "consentType": {
      "type": "string",
      "description": "config-defined: data|media|activity"
    },
    "action": {
      "type": "string",
      "enum": [
        "GRANT",
        "REVOKE"
      ]
    }
  },
  "required": [
    "studentId",
    "consentType",
    "action"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "consentType": {
      "type": "string"
    },
    "studentId": {
      "type": "string"
    },
    "status": {
      "type": "string",
      "enum": [
        "GRANTED",
        "REVOKED"
      ]
    },
    "recordedAt": {
      "type": "string"
    }
  },
  "required": [
    "consentType",
    "studentId",
    "status",
    "recordedAt"
  ]
}
```
- **Validation Rules:** Revoking consent stops the **gated, optional** processing going forward only — it never blocks **mandatory** safety/legal notices, which ride a separate, non-consent lawful basis ("revoking media consent never blocks mandatory notices," C-10).
- **Error Codes:** None beyond standard validation.
- **Business Rules Applied:** GRD-N-008 (consent recorded, scoped, withdrawable), C-10 (mandatory-notice lawful basis independent of consent).
- **Pagination:** N/A.

## Guardianship Changes (Governed)

### 14. Submit Guardianship Change
- **Method:** `POST`
- **URL:** `/api/v1/guardians/{id}/guardianship-changes`
- **Description:** Add, remove, or replace a guardianship relationship under governance — reasoned, dated, and approved (SCR-GRDN-09, UC-GRD-03).
- **Authentication (Role-Based):** Yes — `guardian.guardianship.update`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "studentId": {
      "type": "string"
    },
    "changeType": {
      "type": "string",
      "enum": [
        "ADD",
        "REMOVE",
        "REPLACE"
      ]
    },
    "newGuardianId": {
      "type": "string",
      "description": "for ADD/REPLACE"
    },
    "reason": {
      "type": "string"
    },
    "effectiveDate": {
      "type": "string"
    }
  },
  "required": [
    "studentId",
    "changeType",
    "reason",
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
      "const": "PENDING_APPROVAL"
    }
  },
  "required": [
    "id",
    "status"
  ]
}
```
- **Validation Rules:** `reason` and `effectiveDate` required (GRD-N-007); a `REMOVE` or `REPLACE` that would leave a minor with zero active guardians is **hard-blocked before submission**, not just at approval ("This is the minor's only guardian — add a replacement first," GRD-N-002).
- **Error Codes:** `409 LAST_GUARDIAN_OF_MINOR`; `422 REASON_OR_DATE_REQUIRED`.
- **Business Rules Applied:** GRD-N-002, GRD-N-007 (reasoned, dated, audited changes).
- **Pagination:** N/A.

### 15. List Guardianship-Change Approvals
- **Method:** `GET`
- **URL:** `/api/v1/guardians/guardianship-approvals`
- **Description:** Inbox for pending guardianship changes (SCR-GRDN-13).
- **Authentication (Role-Based):** Yes — `guardian.change.approve`, scoped.
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
      "changeType": {
        "type": "string"
      },
      "before": {
        "type": "string"
      },
      "after": {
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
      "changeType",
      "before",
      "after",
      "requestedBy",
      "slaDueAt",
      "status"
    ]
  }
}
```
- **Validation Rules:** Requester ≠ approver (AUTHZ-009).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** GRD-N-007, AUTHZ-009.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 16. Decide Guardianship Change
- **Method:** `POST`
- **URL:** `/api/v1/guardians/guardianship-approvals/{id}/decide`
- **Description:** Approve or reject a pending guardianship change; on approval, the change in #14 is applied effective-dated, and access/notifications re-resolve immediately.
- **Authentication (Role-Based):** Yes — `guardian.change.approve`; **blocked if the caller raised the request**.
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
- **Validation Rules:** Caller ≠ requester; the last-guardian-of-minor check (#14) is **re-validated** at approval time, not just at submission.
- **Error Codes:** `403 SOD_VIOLATION_BLOCKED`; `409 LAST_GUARDIAN_OF_MINOR` (re-checked).
- **Business Rules Applied:** AUTHZ-009, GRD-N-002, GRD-N-007.
- **Pagination:** N/A.

## Reporting & Data Transfer

### 17. Contact & Consent Report
- **Method:** `GET`
- **URL:** `/api/v1/guardians/report/contact-consent`
- **Description:** Report guardian contactability and consent status, restriction-aware (SCR-GRDN-10).
- **Authentication (Role-Based):** Yes — `guardian.view` (the approved screen names a generic "report view"; reused here per the established convention).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "campusId": {
      "type": "string"
    },
    "hasConsentType": {
      "type": "boolean"
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
      "guardianId": {
        "type": "string"
      },
      "name": {
        "type": "string"
      },
      "contactable": {
        "type": "boolean"
      },
      "consentSummary": {
        "type": "object"
      },
      "restricted": {
        "type": "boolean"
      }
    },
    "required": [
      "guardianId",
      "name",
      "contactable",
      "consentSummary",
      "restricted"
    ]
  }
}
```
- **Validation Rules:** Scope-limited; masked per STU-005.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** REP-002, GRD-N-006.
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 18. Bulk Link / Import Guardians
- **Method:** `POST`
- **URL:** `/api/v1/guardians/bulk-link`
- **Description:** Bulk-link or import guardians with dedup and minor-guardian validation (SCR-GRDN-11).
- **Authentication (Role-Based):** Yes — `guardian.import`.
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
              "name": {
                "type": "string"
              },
              "relationship": {
                "type": "string"
              },
              "contact": {
                "type": "string"
              },
              "isFinancialResponsible": {
                "type": "boolean"
              }
            },
            "required": [
              "studentId",
              "name",
              "relationship",
              "contact"
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
              "guardianId": {
                "type": "string"
              }
            },
            "required": [
              "row",
              "guardianId"
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
- **Validation Rules:** Each row run through the dedup heuristic (#3, GRD-N-003); rows that would leave a referenced minor with zero guardians are flagged (GRD-N-002); invalid rows excluded and always reported.
- **Error Codes:** `400 MALFORMED_BATCH`.
- **Business Rules Applied:** GRD-N-002, GRD-N-003; conventions §9.
- **Pagination:** N/A.

### 19. Export Guardian Data
- **Method:** `GET`
- **URL:** `/api/v1/guardians/export`
- **Description:** Governed, masked, restriction-aware export of guardian data (SCR-GRDN-12).
- **Authentication (Role-Based):** Yes — `guardian.export`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "campusId": {
      "type": "string"
    },
    "relationship": {
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
- **Business Rules Applied:** STU-005, GRD-N-006, REP-002, REP-008, conventions §10.
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
