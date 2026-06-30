# Auth API

## 1. API Overview

**Purpose.** Authenticate identities and manage credentials, sessions, MFA, and authentication policy for the deployment. Answers "who are you?" only — permission/scope resolution is the Role & Permission API.

**Module Context.** Implements Business Rules Catalog Doc 01 (Authentication) and UI Screen Spec `01-authentication-screens.md` (SCR-AUTH-01…11). No new business logic introduced; every rule traces to a numbered AUTH-### rule.

---

## 2. Endpoints

### 1. Login
- **Method:** `POST`
- **URL:** `/api/v1/auth/login`
- **Description:** Authenticate with credentials and obtain a session, or trigger an MFA challenge (UC-AUTH-01).
- **Authentication (Role-Based):** No (public; this endpoint establishes identity).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "identifier": {
      "type": "string"
    },
    "password": {
      "type": "string"
    },
    "deviceInfo": {
      "type": "object",
      "properties": {
        "userAgent": {
          "type": "string"
        },
        "platform": {
          "type": "string"
        }
      },
      "required": [
        "userAgent"
      ]
    }
  },
  "required": [
    "identifier",
    "password"
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
        "accessToken": {
          "type": "string"
        },
        "refreshToken": {
          "type": "string"
        },
        "expiresIn": {
          "type": "number"
        },
        "sessionId": {
          "type": "string"
        },
        "user": {
          "type": "object",
          "properties": {
            "id": {
              "type": "string"
            },
            "displayName": {
              "type": "string"
            },
            "email": {
              "type": "string"
            }
          },
          "required": [
            "id",
            "displayName",
            "email"
          ]
        }
      },
      "required": [
        "accessToken",
        "refreshToken",
        "expiresIn",
        "sessionId",
        "user"
      ]
    },
    {
      "type": "object",
      "properties": {
        "mfaRequired": {
          "type": "boolean",
          "const": true
        },
        "mfaToken": {
          "type": "string"
        },
        "expiresIn": {
          "type": "number"
        }
      },
      "required": [
        "mfaRequired",
        "mfaToken",
        "expiresIn"
      ]
    }
  ]
}
```
- **Validation Rules:** `identifier` format validated per client config (email/phone/username) before any credential check; `password` non-empty; account must be `ACTIVE` to issue a token (AUTH-001).
- **Error Codes:** `401 INVALID_CREDENTIALS` (generic — never reveals which field, AUTH-009 non-enumeration spirit applied to login); `403 ACCOUNT_LOCKED` (generic, with lockout notice); `403 ACCOUNT_SUSPENDED` (generic); `429 TOO_MANY_ATTEMPTS` (progressive lockout, AUTH-004).
- **Business Rules Applied:** AUTH-001 (short-lived access token issuance), AUTH-004 (progressive lockout), AUTH-006 (new device → session/device record created), AUTH-007 (MFA policy/step-up).
- **Pagination:** N/A.

### 2. Verify MFA Challenge
- **Method:** `POST`
- **URL:** `/api/v1/auth/mfa/challenge`
- **Description:** Complete login with a TOTP code or a recovery code after `mfaRequired` (UC-AUTH-01 step 3).
- **Authentication (Role-Based):** No (bound to the short-lived `mfaToken` from Login).
- **Request DTO (JSON Schema):**
```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "mfaToken": {
          "type": "string"
        },
        "code": {
          "type": "string"
        }
      },
      "required": [
        "mfaToken",
        "code"
      ]
    },
    {
      "type": "object",
      "properties": {
        "code": {
          "type": "string"
        }
      },
      "required": [
        "code"
      ]
    }
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "accessToken": {
      "type": "string"
    },
    "refreshToken": {
      "type": "string"
    },
    "expiresIn": {
      "type": "number"
    },
    "sessionId": {
      "type": "string"
    },
    "user": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "displayName": {
          "type": "string"
        },
        "email": {
          "type": "string"
        }
      },
      "required": [
        "id",
        "displayName",
        "email"
      ]
    }
  },
  "required": [
    "accessToken",
    "refreshToken",
    "expiresIn",
    "sessionId",
    "user"
  ]
}
```
- **Validation Rules:** `mfaToken` valid and unexpired; `code` is a 6-digit numeric TOTP within the configured time-step + clock-skew window, or a single-use recovery code not previously consumed.
- **Error Codes:** `401 INVALID_MFA_CODE`; `410 MFA_TOKEN_EXPIRED`; `429 TOO_MANY_ATTEMPTS` (failed MFA counts as a failed attempt, AUTH-004).
- **Business Rules Applied:** AUTH-007 (TOTP/recovery-code verification), AUTH-004 (failure counted toward lockout).
- **Pagination:** N/A.

### 3. Refresh Access Token
- **Method:** `POST`
- **URL:** `/api/v1/auth/refresh`
- **Description:** Exchange a refresh token for a new access/refresh pair without re-login (UC-AUTH-02).
- **Authentication (Role-Based):** No (bound to the presented `refreshToken`).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "refreshToken": {
      "type": "string"
    }
  },
  "required": [
    "refreshToken"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "accessToken": {
      "type": "string"
    },
    "refreshToken": {
      "type": "string"
    },
    "expiresIn": {
      "type": "number"
    }
  },
  "required": [
    "accessToken",
    "refreshToken",
    "expiresIn"
  ]
}
```
- **Validation Rules:** Token hash matches a current family member; family not already revoked; if the token is the immediate predecessor within the grace window, it is honored once (AUTH-003) rather than treated as reuse.
- **Error Codes:** `401 INVALID_REFRESH_TOKEN` (expired/unknown/already-consumed outside the grace window). On a stale (rotated-out) token presented **outside** the grace window, the response is still `401 TOKEN_REUSE_DETECTED`, and as a side effect the entire token family and all derived sessions are revoked and the user is notified (AUTH-002).
- **Business Rules Applied:** AUTH-002 (rotating refresh tokens with reuse detection), AUTH-003 (concurrent-refresh grace window).
- **Pagination:** N/A.

### 4. Logout
- **Method:** `POST`
- **URL:** `/api/v1/auth/logout`
- **Description:** End the caller's current session.
- **Authentication (Role-Based):** Yes — self only (no permission string; any authenticated session may end itself).
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
  "type": "null",
  "description": "204 No Content."
}
```
- **Validation Rules:** None beyond a valid current session.
- **Error Codes:** `401` if the access token is already invalid.
- **Business Rules Applied:** AUTH-005 (session revocation), AUTH-006 (session/device control).
- **Pagination:** N/A.

### 5. List My Sessions
- **Method:** `GET`
- **URL:** `/api/v1/auth/sessions`
- **Description:** List the caller's own active sessions/devices (SCR-AUTH-08).
- **Authentication (Role-Based):** Yes — self (no permission string needed for one's own sessions, AUTH-006).
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
      "id": {
        "type": "string"
      },
      "deviceLabel": {
        "type": "string"
      },
      "ipcoarse": {
        "type": "string",
        "description": "ip (coarse)"
      },
      "userAgent": {
        "type": "string"
      },
      "isCurrent": {
        "type": "boolean"
      },
      "createdAt": {
        "type": "string"
      },
      "lastActiveAt": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "deviceLabel",
      "userAgent",
      "isCurrent",
      "createdAt",
      "lastActiveAt"
    ]
  }
}
```
- **Validation Rules:** None.
- **Error Codes:** `401` invalid session.
- **Business Rules Applied:** AUTH-006 (session/device visibility & control).
- **Pagination:** Yes — default `page=1, pageSize=25`; sorted by `lastActiveAt desc`.

### 6. Revoke a Session
- **Method:** `DELETE`
- **URL:** `/api/v1/auth/sessions/{sessionId}`
- **Description:** Revoke one of the caller's own sessions (e.g., a lost device).
- **Authentication (Role-Based):** Yes — self-owned session only via this endpoint; revoking *another user's* session requires `auth.session.revoke` and is exposed in `02-user-management-api.md`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {},
  "description": "None (path param `sessionId`)."
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "null",
  "description": "204 No Content."
}
```
- **Validation Rules:** `sessionId` must belong to the caller.
- **Error Codes:** `403 FORBIDDEN` if the session belongs to another user (audited as `ACCESS_DENIED`); `404` if not found.
- **Business Rules Applied:** AUTH-005, AUTH-006.
- **Pagination:** N/A.

### 7. Sign Out All Other Devices
- **Method:** `POST`
- **URL:** `/api/v1/auth/sessions/revoke-others`
- **Description:** Revoke every session except the one making the request.
- **Authentication (Role-Based):** Yes — self.
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
    "revokedCount": {
      "type": "number"
    }
  },
  "required": [
    "revokedCount"
  ]
}
```
- **Validation Rules:** None.
- **Error Codes:** `401` invalid session.
- **Business Rules Applied:** AUTH-006.
- **Pagination:** N/A.

### 8. Request Password Recovery
- **Method:** `POST`
- **URL:** `/api/v1/auth/password/forgot`
- **Description:** Begin password recovery for an identifier (UC-AUTH-03 steps 1–3).
- **Authentication (Role-Based):** No (public).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "identifier": {
      "type": "string"
    }
  },
  "required": [
    "identifier"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "message": {
      "type": "string"
    }
  },
  "required": [
    "message"
  ]
}
```
- **Validation Rules:** `identifier` format only; no existence check is ever surfaced to the caller.
- **Error Codes:** `429 TOO_MANY_REQUESTS` if the per-account/per-ip rate limit is exceeded (the throttle itself is the only observable difference, by design).
- **Business Rules Applied:** AUTH-009 (non-enumerating, rate-limited recovery).
- **Pagination:** N/A.

### 9. Complete Password Reset
- **Method:** `POST`
- **URL:** `/api/v1/auth/password/reset`
- **Description:** Set a new password using a recovery token (UC-AUTH-03 steps 4–5).
- **Authentication (Role-Based):** No (bound to the single-use recovery token).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "token": {
      "type": "string"
    },
    "newPassword": {
      "type": "string"
    }
  },
  "required": [
    "token",
    "newPassword"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "message": {
      "type": "string",
      "const": "Password updated. Please sign in again."
    }
  },
  "required": [
    "message"
  ]
}
```
- **Validation Rules:** Token unexpired and unused; `newPassword` meets the configured policy (min length, complexity) and is rejected if found in the breach corpus (AUTH-008).
- **Error Codes:** `400 INVALID_OR_EXPIRED_TOKEN` (generic, AUTH-009); `422 PASSWORD_POLICY_REJECTED` (with a guidance message, never revealing the breach list, AUTH-008).
- **Business Rules Applied:** AUTH-008 (password policy & breach rejection), AUTH-009 (token single-use/expiry), AUTH-010 (all active sessions invalidated as a side effect).
- **Pagination:** N/A.

### 10. Change Password (Self-Service)
- **Method:** `POST`
- **URL:** `/api/v1/auth/password/change`
- **Description:** An authenticated user changes their own password (SCR-AUTH-07).
- **Authentication (Role-Based):** Yes — self.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "currentPassword": {
      "type": "string"
    },
    "newPassword": {
      "type": "string"
    },
    "signOutOtherDevices": {
      "type": "boolean",
      "description": "default true"
    }
  },
  "required": [
    "currentPassword",
    "newPassword"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "message": {
      "type": "string",
      "const": "Password changed."
    }
  },
  "required": [
    "message"
  ]
}
```
- **Validation Rules:** `currentPassword` must verify; `newPassword` meets policy and breach check (AUTH-008).
- **Error Codes:** `401 CURRENT_PASSWORD_INCORRECT`; `422 PASSWORD_POLICY_REJECTED`.
- **Business Rules Applied:** AUTH-008, AUTH-010 (all sessions invalidated; current session re-issued only if `signOutOtherDevices` semantics permit per client policy).
- **Pagination:** N/A.

### 11. Validate Invitation Token
- **Method:** `GET`
- **URL:** `/api/v1/auth/invitations/{token}/validate`
- **Description:** Check an invitation token before rendering the activation form (SCR-AUTH-06).
- **Authentication (Role-Based):** No (token-bound).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {},
  "description": "None (path param `token`)."
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
    "displayName": {
      "type": "string"
    },
    "emailMasked": {
      "type": "string"
    },
    "expiresAt": {
      "type": "string"
    }
  },
  "required": [
    "valid"
  ]
}
```
- **Validation Rules:** Token must be unexpired and unused.
- **Error Codes:** `400 INVALID_OR_EXPIRED_TOKEN` (generic).
- **Business Rules Applied:** AUTH-011 (invitation-based activation).
- **Pagination:** N/A.

### 12. Activate Invitation
- **Method:** `POST`
- **URL:** `/api/v1/auth/invitations/{token}/activate`
- **Description:** Complete activation by setting the initial password (UC; AUTH-011).
- **Authentication (Role-Based):** No (token-bound; no admin ever sets the end-user password).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "password": {
      "type": "string"
    }
  },
  "required": [
    "password"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "message": {
      "type": "string",
      "const": "Account activated. Please sign in."
    }
  },
  "required": [
    "message"
  ]
}
```
- **Validation Rules:** Token single-use and unexpired; `password` meets policy (AUTH-008).
- **Error Codes:** `410 INVITATION_EXPIRED` (re-issue required via `02-user-management-api.md` reactivate/reinvite); `422 PASSWORD_POLICY_REJECTED`.
- **Business Rules Applied:** AUTH-011, AUTH-008. State transition `INVITED → ACTIVE`.
- **Pagination:** N/A.

### 13. Begin MFA Enrollment
- **Method:** `POST`
- **URL:** `/api/v1/auth/mfa/enroll/begin`
- **Description:** Generate a new TOTP secret and provisioning payload (SCR-AUTH-03).
- **Authentication (Role-Based):** Yes — self only (an admin can never view or set another user's MFA secret, AUTH module note + SCR-AUTH-03 permission visibility).
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
    "secret": {
      "type": "string"
    },
    "otpauthUri": {
      "type": "string"
    }
  },
  "required": [
    "secret",
    "otpauthUri"
  ]
}
```
- **Validation Rules:** None beyond an authenticated self request.
- **Error Codes:** `409 MFA_ALREADY_ENROLLED` if already enrolled (must disable first).
- **Business Rules Applied:** AUTH-007.
- **Pagination:** N/A.

### 14. Confirm MFA Enrollment
- **Method:** `POST`
- **URL:** `/api/v1/auth/mfa/enroll/confirm`
- **Description:** Verify the first TOTP code and finalize enrollment, issuing one-time recovery codes.
- **Authentication (Role-Based):** Yes — self only.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "code": {
      "type": "string"
    }
  },
  "required": [
    "code"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "recoveryCodes": {
      "type": "array",
      "items": {
        "type": "string"
      }
    }
  },
  "required": [
    "recoveryCodes"
  ]
}
```
- **Validation Rules:** `code` valid within the time-step/skew window.
- **Error Codes:** `401 INVALID_MFA_CODE`.
- **Business Rules Applied:** AUTH-007 (single-use recovery codes prevent lockout).
- **Pagination:** N/A.

### 15. Disable MFA
- **Method:** `POST`
- **URL:** `/api/v1/auth/mfa/disable`
- **Description:** Turn off MFA for the caller's own account (where client policy allows; roles with required-MFA cannot self-disable).
- **Authentication (Role-Based):** Yes — self; blocked if the caller's role has MFA `required` per client policy (AUTH-007).
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "password": {
      "type": "string"
    }
  },
  "required": [
    "password"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "null",
  "description": "204 No Content."
}
```
- **Validation Rules:** `password` must verify; role's MFA mode must not be `required`.
- **Error Codes:** `401 CURRENT_PASSWORD_INCORRECT`; `403 MFA_REQUIRED_FOR_ROLE`.
- **Business Rules Applied:** AUTH-007.
- **Pagination:** N/A.

### 16. Regenerate Recovery Codes
- **Method:** `POST`
- **URL:** `/api/v1/auth/mfa/recovery-codes/regenerate`
- **Description:** Invalidate old recovery codes and issue a new set.
- **Authentication (Role-Based):** Yes — self; requires a fresh MFA challenge (step-up, see #17) since this is a sensitive operation.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "stepUpToken": {
      "type": "string"
    }
  },
  "required": [
    "stepUpToken"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "recoveryCodes": {
      "type": "array",
      "items": {
        "type": "string"
      }
    }
  },
  "required": [
    "recoveryCodes"
  ]
}
```
- **Validation Rules:** `stepUpToken` valid and unexpired.
- **Error Codes:** `401 STEP_UP_REQUIRED`.
- **Business Rules Applied:** AUTH-007.
- **Pagination:** N/A.

### 17. Step-Up Authentication
- **Method:** `POST`
- **URL:** `/api/v1/auth/step-up`
- **Description:** Obtain a short-lived step-up token by passing a fresh MFA challenge mid-session, required by sensitive operations elsewhere in the system (e.g., result publishing, bulk financial actions) regardless of session age.
- **Authentication (Role-Based):** Yes — self.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "code": {
      "type": "string"
    }
  },
  "required": [
    "code"
  ]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "stepUpToken": {
      "type": "string"
    },
    "expiresIn": {
      "type": "number"
    }
  },
  "required": [
    "stepUpToken",
    "expiresIn"
  ]
}
```
- **Validation Rules:** `code` valid TOTP/recovery code.
- **Error Codes:** `401 INVALID_MFA_CODE`.
- **Business Rules Applied:** AUTH-007 (step-up for sensitive operations).
- **Pagination:** N/A.

### 18. Get Authentication Policy
- **Method:** `GET`
- **URL:** `/api/v1/auth/policy`
- **Description:** Read the deployment/institute's password, lockout, MFA, and token-TTL configuration (SCR-AUTH-10).
- **Authentication (Role-Based):** Yes — `auth.policy.manage` (the UI spec gates both view and edit behind this single permission — "others hidden").
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
    "minLength": {
      "type": "string"
    },
    "complexityClasses": {
      "type": "string",
      "description": "complexityClasses[]"
    },
    "breachCheckEnabled": {
      "type": "string"
    },
    "maxFailedAttempts": {
      "type": "string"
    },
    "lockoutCooldownMinutes": {
      "type": "string"
    },
    "mfaRequiredRoles": {
      "type": "string",
      "description": "mfaRequiredRoles[]"
    },
    "accessTokenTtlMinutes": {
      "type": "string"
    },
    "refreshTokenTtlDays": {
      "type": "string"
    },
    "recoveryTokenTtlMinutes": {
      "type": "string"
    },
    "version": {
      "type": "integer"
    },
    "updatedAt": {
      "type": "string"
    },
    "updatedBy": {
      "type": "string"
    }
  },
  "required": [
    "minLength",
    "breachCheckEnabled",
    "maxFailedAttempts",
    "lockoutCooldownMinutes",
    "accessTokenTtlMinutes",
    "refreshTokenTtlDays",
    "recoveryTokenTtlMinutes",
    "version",
    "updatedAt",
    "updatedBy"
  ]
}
```
- **Validation Rules:** N/A (read).
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTH-004, AUTH-007, AUTH-008, AUTH-009 (the configurable values these rules reference).
- **Pagination:** N/A.

### 19. Update Authentication Policy
- **Method:** `PUT`
- **URL:** `/api/v1/auth/policy`
- **Description:** Update the configurable authentication policy values.
- **Authentication (Role-Based):** Yes — `auth.policy.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "version": {
      "type": "integer"
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
  "description": "Updated policy DTO (as #18).",
  "$ref": "#endpoint-18"
}
```
- **Validation Rules:** All values must be within the secure minimums/maximums the policy schema defines (e.g., `minLength` cannot be set below the platform floor); `version` must match current.
- **Error Codes:** `422 POLICY_VALUE_OUT_OF_BOUNDS`; `409 VERSION_CONFLICT`.
- **Business Rules Applied:** AUTH-004, AUTH-007, AUTH-008, AUTH-009 ("client-configurable within secure minimums").
- **Pagination:** N/A.

### 20. List Login & Security Events
- **Method:** `GET`
- **URL:** `/api/v1/auth/security-events`
- **Description:** Query the immutable authentication audit trail for the security report (SCR-AUTH-11).
- **Authentication (Role-Based):** Yes — `auth.audit.view` (named per the conventions' view/manage pairing note; the UI spec states "view requires security-report access").
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
    "dateFrom": {
      "type": "string"
    },
    "dateTo": {
      "type": "string"
    },
    "eventTypecommasepara": {
      "type": "string",
      "description": "eventType? (comma-separated)"
    },
    "userId": {
      "type": "string"
    },
    "sortBydefaulttimesta": {
      "type": "string",
      "description": "sortBy? (default timestamp)"
    },
    "sortOrderdefaultdesc": {
      "type": "string",
      "description": "sortOrder? (default desc)"
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
      "eventType": {
        "type": "string"
      },
      "actorUserId": {
        "type": "string"
      },
      "targetUserId": {
        "type": "string"
      },
      "ipcoarse": {
        "type": "string",
        "description": "ip (coarse)"
      },
      "scope": {
        "type": "object",
        "properties": {
          "instituteId": {
            "type": "string"
          },
          "campusId": {
            "type": "string"
          }
        },
        "required": [
          "instituteId"
        ]
      },
      "timestamp": {
        "type": "string"
      }
    },
    "required": [
      "id",
      "eventType",
      "actorUserId",
      "scope",
      "timestamp"
    ]
  }
}
```
- **Validation Rules:** `eventType` values restricted to the mandated audit event set (`LOGIN_SUCCEEDED`, `LOGIN_FAILED`, `LOGOUT`, `ACCOUNT_LOCKED`, `ACCOUNT_UNLOCKED`, `MFA_*`, `TOKEN_REUSE_DETECTED`, `SESSION_REVOKED`, `PASSWORD_CHANGED`, etc., per AUTH Doc §11); results are scope-limited to the caller's authority.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTH §11 (Audit Requirements — mandatory immutable events).
- **Pagination:** Yes — default `page=1, pageSize=25`, sorted `timestamp desc`.

### 21. Export Security Events
- **Method:** `GET`
- **URL:** `/api/v1/auth/security-events/export`
- **Description:** Governed export of the security report (signed URL).
- **Authentication (Role-Based):** Yes — `auth.audit.export`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "description": "Same filters as #20.",
  "$ref": "#endpoint-20"
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
- **Validation Rules:** Same as #20; the export action itself is audited.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** AUTH §11; conventions §10 (governed export).
- **Pagination:** N/A (export is the full filtered set, generated async if large per conventions §9).

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — destructive operations on academic/financial/configuration records are soft deletes (`deletedAt` timestamp) per the entity strategy in `00-api-conventions.md` §7; hard delete is exposed only where a specific business rule explicitly allows it for empty/never-used records.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId` (and `campusId`/`sessionId` where relevant), resolved from the session or the `X-Active-Institute-Id` override header; cross-institute access is structurally impossible (AUTHZ-002). `instituteId` is never required as a manual request field — it is derived from authenticated context, not client-supplied, to prevent tenant-boundary tampering.
