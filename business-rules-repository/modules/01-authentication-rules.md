# 01 — Authentication Business Rules

## 1. Module Purpose
Establish and protect user **identity** within a client deployment: prove who a user is, issue and manage access credentials, and govern sessions, devices, MFA, and recovery. Authentication answers **"who are you?"** only — *what you may do and where* is Authorization (Doc 02). This separation is deliberate and load-bearing: a user may authenticate successfully yet have no accessible scope.

## 2. Actors
- **End User** — any human principal (admin, teacher, student, parent, accountant) authenticating to the deployment.
- **User Administrator** — a user with rights to invite, activate, suspend, or reset other users (scope-bounded; see Doc 02).
- **System** — the authentication service issuing/validating tokens, enforcing lockout, sending security notifications.
- **Identity Provider (future)** — an external OIDC/SAML provider; rules are written so federation is additive (D33).

## 3. Use Cases

**Use Case ID:** UC-AUTH-01 — Standard Login
**Actors:** End User, System
**Description:** A user signs in with credentials to obtain an authenticated session.
**Preconditions:** User account exists and is in `ACTIVE` state; deployment is not in maintenance lockout.
**Main Flow:** 1) User submits identifier + password. 2) System validates credentials. 3) If MFA required/enabled, system challenges for the second factor. 4) System issues a short-lived access token + rotating refresh token, creates a session bound to the device. 5) User is authenticated (scope resolved separately by Authorization).
**Alternative Flow:** A1) New device detected → system sends a new-device notification and, if client policy requires, forces step-up MFA.
**Exception Flow:** E1) Invalid credentials → generic failure, increment failed-attempt counter (AUTH-004). E2) Account locked/suspended → generic failure + (for locked) lockout notice. E3) MFA fails → deny, count as failed attempt.
**Post Conditions:** Active session + device record exist; login event audited.
**Business Rules Applied:** AUTH-001, AUTH-002, AUTH-004, AUTH-006, AUTH-007.

**Use Case ID:** UC-AUTH-02 — Token Refresh with Reuse Detection
**Actors:** End User (client app), System
**Description:** The client exchanges a refresh token for a new access token without re-login.
**Preconditions:** A valid, current refresh token exists in its family.
**Main Flow:** 1) Client presents refresh token. 2) System verifies it is the current token of its family. 3) System rotates: invalidates the presented token, issues a new access + refresh pair.
**Alternative Flow:** A1) Two near-simultaneous refreshes (multi-tab) → the grace window accepts the immediate predecessor once (AUTH-003).
**Exception Flow:** E1) A previously-rotated (stale) refresh token is presented → **reuse detected** → revoke the entire family, end all derived sessions, notify the user.
**Post Conditions:** Either a rotated pair is issued, or the family is revoked and audited.
**Business Rules Applied:** AUTH-002, AUTH-003.

**Use Case ID:** UC-AUTH-03 — Password Recovery
**Actors:** End User, System
**Description:** A user who forgot their password sets a new one.
**Preconditions:** None disclosed to the requester (non-enumerating).
**Main Flow:** 1) User requests recovery for an identifier. 2) System always responds identically. 3) If the identifier exists, a single-use, short-expiry, hashed recovery token is sent out-of-band. 4) User presents the token and sets a new password meeting policy. 5) System invalidates the token and **all active sessions**.
**Exception Flow:** E1) Expired/used/invalid token → generic failure. E2) Rate limit exceeded → throttle.
**Post Conditions:** Password updated; every session invalidated; event audited.
**Business Rules Applied:** AUTH-008, AUTH-009, AUTH-010.

## 4. Business Rules

**Rule ID:** AUTH-001
**Rule Name:** Short-Lived Access Token Issuance
**Description:** Successful authentication issues a short-lived, asymmetric-signed access token carrying identity, active scope, roles, and a permissions-version (not the full permission set).
**Priority:** Critical
**Category:** Credential issuance
**Preconditions:** Credentials (and MFA if required) validated; account `ACTIVE`.
**Business Rule:** Access-token lifetime is short (configurable, default 10–15 min). The token never carries secrets or the full permission list; permissions resolve from cache by version.
**System Action:** Sign token with the deployment's current asymmetric key; record session.
**Validation:** Account state = `ACTIVE`; signing key is current/non-expired; clock within skew tolerance.
**Failure Behavior:** No token issued; generic error; failed-attempt logic applies where credential-related.
**Audit Requirement:** Log `LOGIN_SUCCEEDED` with user, device, ip, timestamp, session id.
**Example Scenario:** A teacher logs in at 9:00; the access token expires by ~9:15 unless refreshed.
**Related Rules:** AUTH-002, AUTH-006, AUTHZ-006.

**Rule ID:** AUTH-002
**Rule Name:** Rotating Refresh Tokens with Family Reuse Detection
**Description:** Refresh tokens rotate on every use; presenting a rotated-out token revokes the whole family.
**Priority:** Critical
**Category:** Session security
**Preconditions:** A valid refresh token belonging to a known family.
**Business Rule:** Each refresh issues a new token and invalidates the old. Refresh tokens are stored **hashed**. Replay of a stale token is treated as theft → revoke family.
**System Action:** Rotate and persist hash; on reuse, revoke family and terminate sessions.
**Validation:** Token hash matches a current family member; family not already revoked.
**Failure Behavior:** On reuse → 401, full-family revocation, user notified.
**Audit Requirement:** Log `TOKEN_ROTATED` and, on reuse, `TOKEN_REUSE_DETECTED` + `SESSION_FAMILY_REVOKED`.
**Example Scenario:** A stolen refresh token is replayed an hour later; the legitimate user's next refresh exposes the reuse and all sessions die.
**Related Rules:** AUTH-003, AUTH-006.

**Rule ID:** AUTH-003
**Rule Name:** Concurrent-Refresh Grace Window
**Description:** Tolerate a brief race where two clients refresh near-simultaneously without false reuse-detection.
**Priority:** High
**Category:** Session security / reliability
**Preconditions:** Rotation in progress for a family.
**Business Rule:** The immediate predecessor token is accepted **once** within a short grace window (configurable, e.g., 10–30s) before being treated as reuse.
**System Action:** Serve the already-rotated successor for an in-window predecessor presentation; do not revoke.
**Validation:** Presented token is the exact immediate predecessor; within window; not already consumed twice.
**Failure Behavior:** Outside window or second consumption → treat as reuse (AUTH-002).
**Audit Requirement:** Log `REFRESH_GRACE_USED` (info level) for observability.
**Example Scenario:** A user with two browser tabs triggers two refreshes within 2 seconds; both succeed, no family revocation.
**Related Rules:** AUTH-002.

**Rule ID:** AUTH-004
**Rule Name:** Progressive Failed-Attempt Lockout
**Description:** Repeated failed authentications trigger throttling then temporary lockout, with notification.
**Priority:** Critical
**Category:** Brute-force defense
**Preconditions:** Authentication attempt fails.
**Business Rule:** Per-account and per-ip counters increment on failure. After a configurable threshold, progressive delay applies; beyond a second threshold, the account is temporarily `LOCKED` for a configurable cooldown. Counters reset on success or cooldown expiry.
**System Action:** Throttle, lock, notify the account owner of the lockout.
**Validation:** Thresholds and cooldown are client-configurable; counters are scoped per account and per ip.
**Failure Behavior:** Locked accounts reject login generically until cooldown or admin unlock.
**Audit Requirement:** Log `LOGIN_FAILED` (with reason category, not the attempted password) and `ACCOUNT_LOCKED`.
**Example Scenario:** 10 wrong passwords in 5 minutes lock the account for 15 minutes and email the user.
**Related Rules:** AUTH-001, AUTH-008.

**Rule ID:** AUTH-005
**Rule Name:** Immediate Revocation on State Change
**Description:** Suspending, locking, or deactivating an account, or an admin-forced logout, must take effect within minutes despite stateless access tokens.
**Priority:** Critical
**Category:** Session control
**Preconditions:** An account state change or forced revocation occurs while sessions are active.
**Business Rule:** A revoked session/family is denylisted; combined with short token life, access ends within the token TTL at most.
**System Action:** Add token/family identifiers to the denylist; terminate sessions.
**Validation:** Denylist checked on every authenticated request; entries expire after max token TTL.
**Failure Behavior:** If denylist is unreachable, fail closed for sensitive operations (deny) and alert.
**Audit Requirement:** Log `SESSION_REVOKED` with actor (self/admin/system) and reason.
**Example Scenario:** An admin deactivates a departing employee; their active session stops working within 15 minutes without waiting for natural expiry.
**Related Rules:** AUTH-002, AUTH-006, INST-007 (institute suspension cascades).

**Rule ID:** AUTH-006
**Rule Name:** Session and Device Visibility & Control
**Description:** Users and admins can list active sessions/devices and revoke them individually or in bulk.
**Priority:** High
**Category:** Session management
**Preconditions:** Authenticated user; or admin with user-management rights.
**Business Rule:** Concurrent sessions are permitted; each is bound to a device record and independently revocable. "Sign out all other devices" is supported.
**System Action:** Maintain session/device records (created, last-seen, ip, agent); revoke on request.
**Validation:** A user may only view/revoke their own sessions unless they hold user-management permission for the target's scope.
**Failure Behavior:** Unauthorized revocation attempt → 403, audited.
**Audit Requirement:** Log `SESSION_LISTED` (optional), `SESSION_REVOKED`, `DEVICE_REVOKED`.
**Example Scenario:** A parent who lost a phone signs out that device from another.
**Related Rules:** AUTH-005, AUTH-007.

**Rule ID:** AUTH-007
**Rule Name:** MFA Policy and Step-Up
**Description:** TOTP MFA is available per client, may be required per role, and is stepped-up for sensitive actions.
**Priority:** High
**Category:** Strong authentication
**Preconditions:** Client MFA policy configured; user enrolled if required.
**Business Rule:** MFA mode is configurable (optional / encouraged / required-per-role). Sensitive operations (e.g., result publishing, bulk financial actions, bulk export) may demand a fresh MFA challenge regardless of session age. Single-use recovery codes prevent lockout.
**System Action:** Challenge TOTP; verify within time-step + skew; consume recovery code if used.
**Validation:** TOTP window tolerance configurable; recovery codes are single-use and hashed.
**Failure Behavior:** Failed MFA counts as a failed attempt (AUTH-004); exhausted recovery codes require admin-assisted re-enrollment.
**Audit Requirement:** Log `MFA_ENROLLED`, `MFA_CHALLENGED`, `MFA_SUCCEEDED/FAILED`, `RECOVERY_CODE_USED`, `STEP_UP_REQUIRED`.
**Example Scenario:** An exam controller must pass a fresh TOTP check before publishing results even though logged in an hour ago.
**Related Rules:** AUTH-001, AUTH-004.

**Rule ID:** AUTH-008
**Rule Name:** Password Policy & Breached-Password Rejection
**Description:** Passwords must meet a configurable policy and are checked against known-breached corpora where feasible.
**Priority:** High
**Category:** Credential strength
**Preconditions:** Password set/change/recovery.
**Business Rule:** Minimum length enforced; trivial/breached passwords rejected. Passwords are stored with a memory-hard salted hash (Argon2id or equivalent), never reversibly.
**System Action:** Validate against policy + breach check; hash on store.
**Validation:** Policy values client-configurable within secure minimums; breach check non-blocking-on-outage (fail-open only for availability, log it).
**Failure Behavior:** Non-compliant password rejected with guidance (without revealing the breach list).
**Audit Requirement:** Log `PASSWORD_CHANGED` (never the value); `PASSWORD_POLICY_REJECTED` (reason category).
**Example Scenario:** A user tries "password123"; it is rejected as breached.
**Related Rules:** AUTH-009, AUTH-010.

**Rule ID:** AUTH-009
**Rule Name:** Non-Enumerating, Rate-Limited Recovery
**Description:** Password recovery never reveals whether an identifier exists and is rate-limited.
**Priority:** Critical
**Category:** Account privacy
**Preconditions:** Recovery requested.
**Business Rule:** Identical response regardless of account existence; recovery tokens are single-use, short-expiry, hashed; per-account and per-ip rate limits apply.
**System Action:** Always return the same message; issue token only if account exists.
**Validation:** Token expiry and rate limits configurable within secure bounds.
**Failure Behavior:** Over-limit requests throttled; expired/used tokens rejected generically.
**Audit Requirement:** Log `RECOVERY_REQUESTED` (without confirming existence in user-facing output), `RECOVERY_COMPLETED`.
**Example Scenario:** An attacker probing emails cannot distinguish real from fake accounts by the response.
**Related Rules:** AUTH-008, AUTH-010.

**Rule ID:** AUTH-010
**Rule Name:** Session Invalidation on Credential Change
**Description:** A password reset/change invalidates all of the user's active sessions.
**Priority:** High
**Category:** Session security
**Preconditions:** Password successfully changed or recovered.
**Business Rule:** All sessions and refresh-token families for the user are revoked, forcing re-login everywhere (in case of compromise).
**System Action:** Revoke all families; denylist tokens.
**Validation:** Applies to self-service change, recovery, and admin reset.
**Failure Behavior:** If revocation partially fails, retry and alert; never leave a session valid silently.
**Audit Requirement:** Log `ALL_SESSIONS_REVOKED` with cause = credential change.
**Example Scenario:** A user resets their password from a clean device; an attacker's parallel session is killed.
**Related Rules:** AUTH-002, AUTH-005.

**Rule ID:** AUTH-011
**Rule Name:** Invitation-Based Activation
**Description:** Admin-created users activate via a single-use, expiring invitation token; no admin sets end-user passwords.
**Priority:** High
**Category:** Onboarding
**Preconditions:** Admin creates a user in `INVITED` state.
**Business Rule:** The invitee sets their own initial password through an equivalent single-use token flow; the invitation expires if unused.
**System Action:** Issue invitation token; transition `INVITED → ACTIVE` on completion.
**Validation:** Token single-use, expiring; password meets policy.
**Failure Behavior:** Expired invitation requires re-issue by an admin.
**Audit Requirement:** Log `USER_INVITED`, `USER_ACTIVATED`, `INVITATION_EXPIRED`.
**Example Scenario:** A new accountant is invited; they activate and set their password within 72 hours.
**Related Rules:** AUTH-008, AUTHZ-007.

## 5. Validation Rules
- Identifier format validated (email/phone/username per client config) before any credential check.
- Password never logged, echoed, returned, or placed in a token.
- TOTP codes validated within a configurable time-step window accounting for clock skew.
- Refresh tokens and recovery codes compared by hash only.
- Denylist consulted on every authenticated request; entries auto-expire at max token TTL.
- All counters (failed attempts, rate limits) are per-account **and** per-ip.

## 6. State Machine

**State Name:** INVITED
**Description:** Account created by an admin; not yet activated.
**Allowed Transitions:** → ACTIVE (activation completed); → EXPIRED_INVITE (token lapses); → DEACTIVATED (admin cancels).
**Forbidden Transitions:** → LOCKED (cannot lock an un-activated account); → SUSPENDED.
**System Actions:** Issue invitation token; block login until activated.

**State Name:** ACTIVE
**Description:** Normal usable account.
**Allowed Transitions:** → LOCKED (failed attempts); → SUSPENDED (admin); → DEACTIVATED (admin/offboarding).
**Forbidden Transitions:** → INVITED.
**System Actions:** Permit authentication; enforce MFA policy.

**State Name:** LOCKED
**Description:** Temporarily blocked by failed-attempt lockout.
**Allowed Transitions:** → ACTIVE (cooldown elapses or admin unlock); → SUSPENDED; → DEACTIVATED.
**Forbidden Transitions:** → INVITED.
**System Actions:** Reject login generically; notify owner; auto-unlock after cooldown.

**State Name:** SUSPENDED
**Description:** Administratively disabled (e.g., investigation, leave, institute suspension cascade).
**Allowed Transitions:** → ACTIVE (admin reinstates); → DEACTIVATED.
**Forbidden Transitions:** → LOCKED.
**System Actions:** Reject login; revoke active sessions (AUTH-005).

**State Name:** DEACTIVATED
**Description:** Offboarded; retained for history, cannot authenticate.
**Allowed Transitions:** → ANONYMIZED (retention-driven only).
**Forbidden Transitions:** → ACTIVE (reactivation requires a new explicit admin action creating a fresh activation, audited) — direct silent reactivation forbidden.
**System Actions:** Block all authentication; preserve audit linkage.

**State Name:** ANONYMIZED
**Description:** Personal data removed per retention/erasure; terminal.
**Allowed Transitions:** none (terminal).
**Forbidden Transitions:** all.
**System Actions:** Strip PII; retain non-identifying audit references.

## 7. Status Definitions
`INVITED` (awaiting activation) · `ACTIVE` (usable) · `LOCKED` (temporary, auto-recoverable) · `SUSPENDED` (admin, indefinite) · `DEACTIVATED` (offboarded, non-auth) · `ANONYMIZED` (PII removed, terminal). MFA sub-status: `MFA_DISABLED` / `MFA_PENDING` / `MFA_ENABLED`.

## 8. Workflow Rules
- Account creation by an admin is immediate but yields `INVITED`; usability requires self-activation (no approval workflow for standard onboarding).
- Admin **unlock** of a `LOCKED` account is a single privileged action (no multi-step approval) but is audited and permission-gated.
- Bulk user import creates many `INVITED` accounts; each follows the individual activation flow.
- Reactivation of a `DEACTIVATED` account is an explicit, audited admin action, not a silent toggle.

## 9. Permission Rules
- Authenticating requires no permission (it establishes identity).
- Inviting/suspending/deactivating/unlocking/resetting another user requires `auth.user.manage` **within the target user's scope** (Doc 02); an actor cannot manage a user outside their scope.
- Viewing another user's sessions/devices requires `auth.session.manage` in scope; self-management needs none.
- Configuring MFA/password/lockout policy requires `auth.policy.manage` at the appropriate scope (org or institute).

## 10. Notification Rules
- `LOGIN_FROM_NEW_DEVICE` → notify the user (channel per preference; security notices override opt-out).
- `ACCOUNT_LOCKED` → notify the user.
- `PASSWORD_CHANGED` / `ALL_SESSIONS_REVOKED` → notify the user.
- `TOKEN_REUSE_DETECTED` → notify the user (possible compromise).
- `MFA_ENROLLED` / `MFA_DISABLED` → notify the user.
- Security notifications are **transactional** and bypass marketing-style preferences.

## 11. Audit Requirements
Mandatory immutable events: `LOGIN_SUCCEEDED`, `LOGIN_FAILED`, `LOGOUT`, `TOKEN_ROTATED`, `TOKEN_REUSE_DETECTED`, `SESSION_REVOKED`, `SESSION_FAMILY_REVOKED`, `ACCOUNT_LOCKED/UNLOCKED`, `ACCOUNT_SUSPENDED/REINSTATED/DEACTIVATED`, `USER_INVITED/ACTIVATED`, `PASSWORD_CHANGED`, `RECOVERY_REQUESTED/COMPLETED`, `MFA_*`, `STEP_UP_REQUIRED`. Each carries actor, target, device, ip, scope, timestamp. **Never** log password or token values.

## 12. Data Retention Rules
- Session/device records retained for a configurable window then purged; revoked sessions retained for the audit window.
- Failed-attempt and rate-limit counters are ephemeral (cooldown-bound).
- Authentication audit events follow the audit retention class (long, immutable).
- On account `ANONYMIZED`, identifying fields are stripped while non-identifying audit references persist for integrity.

## 13. Edge Cases
- **Concurrent refresh (multi-tab/app):** handled by the grace window (AUTH-003) — a frequent false-positive source if missed.
- **Clock skew on TOTP:** validation window must tolerate device clock drift; otherwise valid codes fail.
- **User with no active membership:** may authenticate but has zero scope → can log in yet see nothing (Authorization denies). This is intentional, not a bug.
- **User belonging to multiple institutes:** one identity, many memberships; authentication is identity-only, scope chosen post-login.
- **Password reset while sessions active:** all sessions die (AUTH-010), including the device performing the reset unless re-authenticated.
- **Lost MFA device:** recovery codes; if exhausted, admin-assisted re-enrollment (never admin-sets-TOTP).
- **Invitation token reused or shared:** single-use enforcement prevents a second activation.
- **Deployment maintenance window:** authentication may be globally paused; in-flight sessions handled per maintenance policy.
- **Institute suspension cascade:** users scoped only to a suspended institute lose access even while `ACTIVE` (handled via Authorization scope, see INST-007).

## 14. Failure Scenarios
- **Denylist (Redis) unavailable:** fail closed for sensitive operations; alert; do not silently allow revoked tokens.
- **Signing key rotation mid-flight:** old key accepted within overlap window; tokens signed with a retired key past overlap are rejected.
- **Notification channel down:** security event still audited; notification retried via queue; never block the security action on notification delivery.
- **Breach-check service down:** password change may proceed (availability) but the event is flagged for later review.
- **Partial session-revocation failure:** retry; alert; treat as a security incident if unresolved.

## 15. Exception Handling Rules
- All authentication failures return **generic** messages (no enumeration, no reason leakage to the client).
- Internal errors never expose stack traces or token material.
- Every exception path still emits the appropriate audit event.
- Sensitive-operation step-up failures degrade to denial, never to bypass.

## 16. Compliance Considerations
- **Minors:** student/guardian accounts follow heightened protection; parental-consent linkage governs guardian-of-minor access (detailed in Doc 09).
- **GDPR-aligned:** right-to-erasure maps to `ANONYMIZED`; authentication PII is minimized; lawful-basis/consent recorded where applicable.
- **Credential handling:** memory-hard hashing, no plaintext, no reversible storage; aligns with security baseline (Blueprint Part F).
- **Auditability:** complete, immutable authentication trail supports incident response and regulator inquiries.

## 17. Future Considerations
- WebAuthn/passkeys as a stronger factor (the MFA seam is already abstract).
- OIDC/SAML SSO with JIT provisioning and account linking (D33) — federation must not weaken any rule here.
- Risk-based/adaptive authentication (device reputation, geo-velocity) feeding step-up decisions.
- Passwordless flows for low-risk parent/student access where appropriate.
