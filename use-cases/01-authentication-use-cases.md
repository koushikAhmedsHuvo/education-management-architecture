# 01 — Authentication Use Cases

Transforms the Authentication business rules (`AUTH-001`…`AUTH-011`) into implementation-ready use cases. Identity only — *who you are*; authorization (*what you may do*) is module 02.

## 1. Primary Actors
End User (any role: admin, teacher, student, guardian, accountant), User Administrator (invites/manages accounts within scope).

## 2. Secondary Actors
System (token issuance, lockout, denylist, notifications), Identity Provider (future OIDC/SAML, additive), Notification Service, Audit Service.

## 3. Goals
Prove identity securely; manage sessions/devices; recover access safely; protect against brute force, token theft, and account takeover; onboard users via self-activation; keep minors' accounts safe.

## 4. User Journeys
- **New user onboarding:** admin invites → invitee receives single-use link → sets a policy-compliant password → (optional MFA enrollment) → first login.
- **Daily access:** user logs in → short-lived token + rotating refresh → silent refresh during work → step-up MFA for a sensitive action → logout or natural expiry.
- **Account trouble:** forgot password → non-enumerating recovery → reset → all sessions invalidated → re-login. Or: lost device → revoke session / re-enroll MFA via recovery codes.
- **Admin response:** suspicious activity → admin force-logs-out / suspends → access ends within token TTL via denylist.

## 5. Use Case Inventory

| Category | ID | Use Case | Priority |
|---|---|---|---|
| Core | UC-AUTH-001 | Standard Login | Critical |
| Core | UC-AUTH-002 | Token Refresh with Reuse Detection | Critical |
| Core | UC-AUTH-003 | Logout / Sign-out All Devices | High |
| Core | UC-AUTH-004 | Password Recovery (non-enumerating) | Critical |
| Core | UC-AUTH-005 | MFA Enrollment | High |
| Core | UC-AUTH-006 | MFA Challenge & Step-Up | High |
| Core | UC-AUTH-007 | Invitation-Based Activation | High |
| CRUD | UC-AUTH-008 | Create User (invite) | High |
| CRUD | UC-AUTH-009 | View Own Sessions/Devices | Medium |
| CRUD | UC-AUTH-010 | Change Own Password | High |
| CRUD | UC-AUTH-011 | Deactivate User (soft) | High |
| Admin | UC-AUTH-012 | Unlock Locked Account | High |
| Admin | UC-AUTH-013 | Force Logout / Revoke Session | High |
| Admin | UC-AUTH-014 | Reset User MFA | Medium |
| Admin | UC-AUTH-015 | Configure Auth Policy (password/lockout/MFA) | High |
| Search | UC-AUTH-016 | Search Users / Sessions | Medium |
| Bulk/Import | UC-AUTH-017 | Bulk User Invite (CSV import) | Medium |
| Export | UC-AUTH-018 | Export Login/Security Audit | Medium |
| Reporting | UC-AUTH-019 | Login Activity & Security Report | Medium |
| Workflow | UC-AUTH-020 | Account Reactivation (governed) | Medium |
| Exception | UC-AUTH-021 | Locked-Out Login Attempt | High |
| Exception | UC-AUTH-022 | Token Reuse Detected | Critical |

*Approval use cases:* authentication is largely non-approval; the one governed flow is account **reactivation** (UC-AUTH-020). User invitation/activation is self-service, not an approval.

---

## 6. Detailed Specifications (high-value use cases)

### UC-AUTH-001 — Standard Login
- **Module:** Authentication · **Priority:** Critical
- **Actors:** End User (primary), System (secondary)
- **Goal:** Authenticate with credentials and obtain a session.
- **Description:** User submits identifier + password (and MFA if required) and receives a short-lived access token plus a rotating refresh token bound to a device.
- **Business Rules Applied:** AUTH-001, AUTH-002, AUTH-004, AUTH-006, AUTH-007.
- **Preconditions:** Account exists and is `ACTIVE`; deployment not in maintenance lockout.
- **Trigger:** User submits the login form.
- **Main Success Scenario:**
  1. User submits identifier + password.
  2. System validates credentials against the stored hash.
  3. If MFA required/enabled, System issues an MFA challenge; user provides TOTP.
  4. System issues a short-lived access token (with scope/permissions-version) + rotating refresh token; creates a session bound to the device.
  5. System records a new-device check; user is authenticated (scope resolved by module 02).
- **Alternative Flows:** A1) New device → send `LOGIN_FROM_NEW_DEVICE` notice; if policy requires, force step-up MFA. A2) "Remember device" sets a longer-lived (policy-bound) device trust.
- **Exception Flows:** E1) Invalid credentials → generic failure, increment failed-attempt counter (AUTH-004). E2) Account `LOCKED`/`SUSPENDED` → generic failure (+ lockout notice if locked). E3) MFA fails → deny, counts as failed attempt.
- **Validation Rules:** Identifier format valid; account `ACTIVE`; TOTP within time-step + skew; clock within tolerance (AUTH-001).
- **Permissions Required:** None (authentication establishes identity).
- **Notifications Triggered:** `LOGIN_FROM_NEW_DEVICE` (new device); none on routine success.
- **Audit Events Generated:** `LOGIN_SUCCEEDED` (user, device, ip, session id) or `LOGIN_FAILED` (reason category, never the password).
- **Data Created:** Session, device record, refresh-token family (hashed).
- **Data Updated:** Last-login timestamp; failed-attempt counter (reset on success).
- **Data Deleted:** None.
- **Post Conditions:** Active session + device exist; login audited.
- **Related Use Cases:** UC-AUTH-002, UC-AUTH-006, UC-AUTH-021.
- **Acceptance Criteria:**
  - Given an active account with correct credentials, When the user logs in (passing MFA if required), Then a short-lived access token and rotating refresh token are issued and the login is audited.
  - Given incorrect credentials, When the user attempts login, Then a generic error is returned, no token is issued, and the failed-attempt counter increments.
  - Given a new device, When login succeeds, Then a new-device notification is sent.
- **Edge Case Analysis:**
  - *Invalid Input:* malformed identifier rejected before credential check; empty password rejected.
  - *Permission Failure:* N/A (no permission needed); but a user with no membership authenticates yet sees nothing (by design).
  - *Concurrent Update:* simultaneous logins create independent sessions; no conflict.
  - *Duplicate Data:* repeated identical submissions do not create duplicate sessions beyond intended concurrency.
  - *System Failure:* signing-key unavailable → login fails closed with a generic error and alert; denylist down → fail closed for sensitive ops.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* valid login no-MFA; valid login with MFA; new-device login triggers notice.
  - *Negative:* wrong password; locked account; suspended account; failed MFA; expired/disabled account.
  - *Boundary:* attempt exactly at lockout threshold; TOTP at edge of skew window; token used at the last second before expiry.

### UC-AUTH-002 — Token Refresh with Reuse Detection
- **Module:** Authentication · **Priority:** Critical
- **Actors:** End User's client app (primary), System (secondary)
- **Goal:** Obtain a new access token without re-login, safely.
- **Description:** Client exchanges a refresh token; the engine rotates it and detects replay of a rotated-out token as theft.
- **Business Rules Applied:** AUTH-002, AUTH-003, AUTH-006.
- **Preconditions:** A valid, current refresh token in its family.
- **Trigger:** Access token nears expiry or a 401 prompts refresh.
- **Main Success Scenario:**
  1. Client presents the refresh token.
  2. System verifies it is the current token of its family (by hash).
  3. System rotates: invalidates the presented token, issues a new access + refresh pair.
  4. Client continues with the new tokens.
- **Alternative Flows:** A1) Two near-simultaneous refreshes (multi-tab) → the immediate predecessor is accepted once within the grace window (AUTH-003); no revocation.
- **Exception Flows:** E1) A stale (already-rotated) token is presented → **reuse detected** → revoke the entire family, end derived sessions, notify the user (UC-AUTH-022).
- **Validation Rules:** Token hash matches a current family member; family not revoked; grace window honored (AUTH-003).
- **Permissions Required:** None (valid refresh token is the credential).
- **Notifications Triggered:** `TOKEN_REUSE_DETECTED` (on reuse).
- **Audit Events Generated:** `TOKEN_ROTATED`; on reuse `TOKEN_REUSE_DETECTED` + `SESSION_FAMILY_REVOKED`; `REFRESH_GRACE_USED` (info).
- **Data Created:** New refresh token (hashed).
- **Data Updated:** Refresh-token family chain; session last-seen.
- **Data Deleted:** None (old token invalidated, not deleted; family revoked on reuse).
- **Post Conditions:** Rotated token pair issued, or family revoked and audited.
- **Related Use Cases:** UC-AUTH-001, UC-AUTH-022.
- **Acceptance Criteria:**
  - Given a current refresh token, When refreshed, Then a new pair is issued and the old token is invalidated.
  - Given a rotated-out token replayed outside the grace window, When presented, Then the whole family is revoked and the user is notified.
  - Given two refreshes within the grace window, When both presented, Then both succeed without revocation.
- **Edge Case Analysis:**
  - *Invalid Input:* malformed/garbage token → 401, no family action.
  - *Permission Failure:* N/A.
  - *Concurrent Update:* the multi-tab race is the core concern — grace window prevents false reuse (AUTH-003).
  - *Duplicate Data:* duplicate presentation of the same current token within grace is tolerated once.
  - *System Failure:* token store/denylist down → fail closed (deny refresh) and alert.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* normal rotation; multi-tab within grace.
  - *Negative:* replayed stale token → family revoked; revoked-family token rejected.
  - *Boundary:* predecessor presented at the exact edge of the grace window; second consumption of predecessor → reuse.

### UC-AUTH-004 — Password Recovery (non-enumerating)
- **Module:** Authentication · **Priority:** Critical
- **Actors:** End User (primary), System (secondary)
- **Goal:** Reset a forgotten password without leaking account existence.
- **Description:** User requests recovery; the system always responds identically; if the account exists, a single-use, short-expiry, hashed token is sent out-of-band; reset invalidates all sessions.
- **Business Rules Applied:** AUTH-008, AUTH-009, AUTH-010.
- **Preconditions:** None disclosed to the requester.
- **Trigger:** User submits "forgot password" with an identifier.
- **Main Success Scenario:**
  1. User requests recovery for an identifier.
  2. System returns an identical "if the account exists, instructions were sent" response.
  3. If the account exists, System issues a single-use, short-expiry, hashed recovery token out-of-band.
  4. User opens the link, sets a new password meeting policy (length + breach check).
  5. System invalidates the token and **all active sessions** (AUTH-010).
- **Alternative Flows:** A1) Account does not exist → same response, no token issued.
- **Exception Flows:** E1) Expired/used/invalid token → generic failure. E2) Rate limit exceeded → throttle (AUTH-009).
- **Validation Rules:** Identical response regardless of existence; token single-use/short-expiry/hashed; new password meets policy & breach check (AUTH-008); per-account & per-ip rate limits.
- **Permissions Required:** None.
- **Notifications Triggered:** Recovery instructions (if exists); `PASSWORD_CHANGED` + `ALL_SESSIONS_REVOKED` on completion.
- **Audit Events Generated:** `RECOVERY_REQUESTED` (without confirming existence externally), `RECOVERY_COMPLETED`, `PASSWORD_CHANGED`, `ALL_SESSIONS_REVOKED`.
- **Data Created:** Recovery token (hashed).
- **Data Updated:** Password hash; all sessions revoked.
- **Data Deleted:** None (sessions revoked/denylisted, token consumed).
- **Post Conditions:** Password updated; every session invalidated; audited.
- **Related Use Cases:** UC-AUTH-010, UC-AUTH-001.
- **Acceptance Criteria:**
  - Given any identifier, When recovery is requested, Then the response is identical whether or not the account exists.
  - Given a valid recovery token, When a policy-compliant password is set, Then it is saved and all sessions are invalidated.
  - Given an expired or reused token, When submitted, Then it is rejected generically.
- **Edge Case Analysis:**
  - *Invalid Input:* weak/breached new password rejected with guidance (not revealing the breach list).
  - *Permission Failure:* N/A.
  - *Concurrent Update:* two recovery requests → latest token valid; earlier single-use tokens still single-use.
  - *Duplicate Data:* repeated requests rate-limited (AUTH-009).
  - *System Failure:* breach-check service down → may proceed (availability) and flag; notification down → reset still applies, notice retried.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* existing account resets successfully; sessions die.
  - *Negative:* non-existent account (identical response); expired/used token; over-rate-limit.
  - *Boundary:* token used at expiry edge; password at minimum length; reset on the device performing it (must re-login).

### UC-AUTH-006 — MFA Challenge & Step-Up
- **Module:** Authentication · **Priority:** High
- **Actors:** End User (primary), System (secondary)
- **Goal:** Verify a second factor at login and for sensitive operations.
- **Description:** TOTP challenge per policy; sensitive actions (result publish, bulk financial/export) demand a fresh challenge regardless of session age; recovery codes prevent lockout.
- **Business Rules Applied:** AUTH-007, AUTH-004.
- **Preconditions:** MFA policy configured; user enrolled where required.
- **Trigger:** Login (when MFA required) or initiation of a sensitive operation.
- **Main Success Scenario:**
  1. System detects MFA required (login) or step-up required (sensitive op).
  2. System prompts for a TOTP code.
  3. User submits the code; System verifies within time-step + skew.
  4. The login/operation proceeds.
- **Alternative Flows:** A1) User uses a single-use recovery code instead of TOTP; the code is consumed.
- **Exception Flows:** E1) Wrong code → counts as a failed attempt (AUTH-004); repeated failures lock. E2) Recovery codes exhausted → admin-assisted re-enrollment (UC-AUTH-014).
- **Validation Rules:** TOTP within configurable window; recovery codes single-use/hashed; step-up demanded for sensitive ops regardless of session age (AUTH-007).
- **Permissions Required:** None for the challenge itself; the underlying sensitive op has its own permission.
- **Notifications Triggered:** `MFA_ENROLLED`/`MFA_DISABLED` (on changes); none on routine success.
- **Audit Events Generated:** `MFA_CHALLENGED`, `MFA_SUCCEEDED/FAILED`, `RECOVERY_CODE_USED`, `STEP_UP_REQUIRED`.
- **Data Created:** None (challenge transient).
- **Data Updated:** Recovery-code consumption; failed-attempt counter.
- **Data Deleted:** None.
- **Post Conditions:** Second factor verified; sensitive operation unblocked.
- **Related Use Cases:** UC-AUTH-001, UC-AUTH-005, UC-AUTH-014.
- **Acceptance Criteria:**
  - Given MFA-required login, When a valid TOTP is entered, Then authentication completes.
  - Given a sensitive operation, When the session is old, Then a fresh MFA challenge is required before it proceeds.
  - Given exhausted recovery codes, When the device is lost, Then the user cannot self-bypass and must use admin-assisted re-enrollment.
- **Edge Case Analysis:**
  - *Invalid Input:* non-numeric/expired code rejected.
  - *Permission Failure:* sensitive op without its permission denied before/after step-up (deny overrides).
  - *Concurrent Update:* parallel step-up prompts resolve independently.
  - *Duplicate Data:* reused recovery code rejected (single-use).
  - *System Failure:* TOTP service issue → fail closed for the sensitive op.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* TOTP success; recovery-code success; step-up on a sensitive op.
  - *Negative:* wrong/expired code; reused recovery code; step-up bypass attempt.
  - *Boundary:* code at skew-window edges; last remaining recovery code.

### UC-AUTH-007 — Invitation-Based Activation
- **Module:** Authentication · **Priority:** High
- **Actors:** Invitee (primary), System, User Administrator (initiator)
- **Goal:** Let an admin-created user activate by setting their own password.
- **Description:** Admin creates an `INVITED` account; the invitee activates via a single-use, expiring token and sets a policy-compliant password. Admins never set end-user passwords.
- **Business Rules Applied:** AUTH-011, AUTH-008.
- **Preconditions:** Admin created the account in `INVITED` state.
- **Trigger:** Invitee opens the invitation link.
- **Main Success Scenario:**
  1. Invitee opens the single-use invitation link.
  2. System validates the token (single-use, not expired).
  3. Invitee sets a password meeting policy + breach check.
  4. System transitions `INVITED → ACTIVE` and consumes the token.
- **Alternative Flows:** A1) Optional MFA enrollment immediately after activation (UC-AUTH-005).
- **Exception Flows:** E1) Expired/used invitation → must be re-issued by an admin (UC-AUTH-008).
- **Validation Rules:** Token single-use & expiring; password meets policy (AUTH-008).
- **Permissions Required:** None for the invitee; admin needs `auth.user.manage` to invite.
- **Notifications Triggered:** `USER_ACTIVATED` to admin/user; invitation email by the invite UC.
- **Audit Events Generated:** `USER_ACTIVATED`, `INVITATION_EXPIRED` (if lapsed).
- **Data Created:** Password hash; activation record.
- **Data Updated:** Account state `INVITED → ACTIVE`.
- **Data Deleted:** None (token consumed).
- **Post Conditions:** Account `ACTIVE`; user can log in.
- **Related Use Cases:** UC-AUTH-008, UC-AUTH-005.
- **Acceptance Criteria:**
  - Given a valid invitation, When the invitee sets a compliant password, Then the account becomes `ACTIVE`.
  - Given an expired invitation, When opened, Then activation is blocked and re-issue is required.
- **Edge Case Analysis:**
  - *Invalid Input:* weak password rejected.
  - *Permission Failure:* inviting admin out of scope cannot create the account (module 02).
  - *Concurrent Update:* token used twice → second attempt rejected (single-use).
  - *Duplicate Data:* re-inviting an existing active user is blocked/handled.
  - *System Failure:* email delivery failure → admin can re-issue.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* activation within validity; activation + MFA enroll.
  - *Negative:* expired token; reused token; weak password.
  - *Boundary:* activation at the expiry edge.

### UC-AUTH-013 — Force Logout / Revoke Session (Admin)
- **Module:** Authentication · **Priority:** High
- **Actors:** User Administrator (primary), System
- **Goal:** Immediately end a user's session(s) despite stateless tokens.
- **Description:** Admin (or the user themselves) revokes a session/family; the denylist + short token TTL ends access within minutes.
- **Business Rules Applied:** AUTH-005, AUTH-006.
- **Preconditions:** Admin holds `auth.session.manage` in the target's scope (or the user manages their own).
- **Trigger:** Admin revokes a session, or suspends/deactivates the account.
- **Main Success Scenario:**
  1. Admin selects the user's session/device (or "sign out all").
  2. System adds the session/family identifiers to the denylist and terminates the session(s).
  3. Within at most the token TTL, the revoked tokens are rejected.
- **Alternative Flows:** A1) Triggered automatically by account suspension/deactivation (AUTH-005) or institute suspension cascade (INST-007).
- **Exception Flows:** E1) Denylist (Redis) unreachable → fail closed for sensitive operations; alert.
- **Validation Rules:** Actor authorized in scope; denylist entries auto-expire at max token TTL (AUTH-005).
- **Permissions Required:** `auth.session.manage` (others' sessions) / none (own).
- **Notifications Triggered:** Optional security notice to the user.
- **Audit Events Generated:** `SESSION_REVOKED` / `SESSION_FAMILY_REVOKED` (actor, reason).
- **Data Created:** Denylist entries.
- **Data Updated:** Session status → revoked.
- **Data Deleted:** None (revoked, retained for audit window).
- **Post Conditions:** Target access ends within token TTL.
- **Related Use Cases:** UC-AUTH-011, UC-AUTH-003.
- **Acceptance Criteria:**
  - Given an active session, When an authorized admin revokes it, Then the token is rejected within the token TTL.
  - Given a deactivated account, When deactivation occurs, Then its active sessions stop working without waiting for natural expiry.
- **Edge Case Analysis:**
  - *Invalid Input:* revoking a non-existent session → no-op with clear message.
  - *Permission Failure:* admin out of the target's scope → 403, audited.
  - *Concurrent Update:* simultaneous revoke + refresh → revoke wins (denylist consulted on every request).
  - *Duplicate Data:* double revoke is idempotent.
  - *System Failure:* denylist down → fail closed + alert.
  - *Workflow Failure:* N/A.
- **QA Coverage:**
  - *Positive:* single-session revoke; sign-out-all; suspension cascade.
  - *Negative:* out-of-scope revoke denied; revoke of unknown session.
  - *Boundary:* access exactly at TTL edge after revoke; denylist entry expiry.

---

## 7. Compact Specifications (routine use cases)

Each follows standard patterns; cited rules are authoritative.

- **UC-AUTH-003 — Logout / Sign-out All Devices** · *High* · Rules: AUTH-006, AUTH-010. User ends own session(s); refresh family revoked, denylisted. *Permissions:* none (own). *Validation:* session belongs to user. *Audit:* `LOGOUT`, `SESSION_REVOKED`. *Edge:* logout of already-expired session is idempotent; offline logout reconciles on reconnect. *QA:* logout single/all; concurrent logout idempotent.
- **UC-AUTH-005 — MFA Enrollment** · *High* · Rules: AUTH-007. User enrolls TOTP, receives single-use recovery codes. *Validation:* verify a test code before enabling; recovery codes hashed. *Audit:* `MFA_ENROLLED`. *Edge:* re-enroll replaces prior secret; codes shown once. *QA:* enroll/verify; invalid test code; re-enroll.
- **UC-AUTH-008 — Create User (invite)** · *High* · Rules: AUTH-011, AUTHZ-007. Admin creates `INVITED` account in scope, assigns role/membership (module 02). *Permissions:* `auth.user.manage` in scope. *Validation:* identifier unique; scope authorized. *Audit:* `USER_INVITED`. *Edge:* duplicate identifier blocked; out-of-scope invite denied. *QA:* invite valid; duplicate; out-of-scope.
- **UC-AUTH-009 — View Own Sessions/Devices** · *Medium* · Rules: AUTH-006. Read list of active sessions (created, last-seen, ip, agent). *Permissions:* none (own). *Audit:* optional `SESSION_LISTED`. *QA:* list reflects active sessions; revoked sessions excluded.
- **UC-AUTH-010 — Change Own Password** · *High* · Rules: AUTH-008, AUTH-010. User changes password; all sessions invalidated. *Validation:* current password verified; new meets policy/breach. *Audit:* `PASSWORD_CHANGED`, `ALL_SESSIONS_REVOKED`. *Edge:* same-as-old rejected per policy; sessions die including current unless re-auth. *QA:* valid change; weak/breached new; wrong current.
- **UC-AUTH-011 — Deactivate User (soft)** · *High* · Rules: AUTH-005. Admin sets account `DEACTIVATED`; access revoked; record retained. *Permissions:* `auth.user.manage` in scope. *Audit:* `ACCOUNT_DEACTIVATED`. *Edge:* no hard delete; reactivation is governed (UC-AUTH-020). *QA:* deactivate ends access; cannot hard-delete.
- **UC-AUTH-012 — Unlock Locked Account** · *High* · Rules: AUTH-004. Admin unlocks before cooldown. *Permissions:* `auth.user.manage`. *Audit:* `ACCOUNT_UNLOCKED`. *QA:* unlock restores login; auto-unlock after cooldown.
- **UC-AUTH-014 — Reset User MFA** · *Medium* · Rules: AUTH-007. Admin resets MFA for a locked-out user (admin never sets TOTP). *Permissions:* `auth.user.manage`. *Audit:* `MFA_RESET`. *Edge:* forces re-enrollment on next login. *QA:* reset → re-enroll required.
- **UC-AUTH-015 — Configure Auth Policy** · *High* · Rules: AUTH-004/007/008, CFG-002/004. Admin sets password/lockout/MFA policy (versioned via Config Engine). *Permissions:* `auth.policy.manage`. *Audit:* policy change. *Edge:* values validated within secure minimums; effective forward. *QA:* valid policy; below-minimum rejected.
- **UC-AUTH-016 — Search Users / Sessions** · *Medium* · Rules: AUTHZ-002/003. Scoped search of users/sessions. *Permissions:* `auth.user.manage`/`auth.session.manage`. *Validation:* results scope-filtered. *Edge:* never returns out-of-scope users. *QA:* scoped results; out-of-scope excluded; pagination boundary.
- **UC-AUTH-017 — Bulk User Invite (CSV import)** · *Medium* · Rules: AUTH-011. Admin imports many users → many `INVITED` accounts, each activating individually. *Permissions:* `auth.user.manage`. *Validation:* per-row validation; valid rows succeed, invalid reported. *Edge:* partial-failure report; duplicates flagged. *QA:* clean import; mixed valid/invalid; duplicate rows.
- **UC-AUTH-018 — Export Login/Security Audit** · *Medium* · Rules: AUD, REP-005. Export authentication audit (governed, scoped). *Permissions:* `audit.export`/`auth.audit.view`. *Audit:* `REPORT_EXPORTED`. *Edge:* bulk export governed; never includes secrets. *QA:* scoped export; unauthorized denied.
- **UC-AUTH-019 — Login Activity & Security Report** · *Medium* · Rules: AUD, REP-002. Report on logins, failures, lockouts, MFA adoption. *Permissions:* `auth.audit.view`. *Edge:* scope-filtered; as-of freshness shown. *QA:* counts match audit; scope respected.
- **UC-AUTH-020 — Account Reactivation (governed)** · *Medium* · Rules: AUTH-005 (state machine). Admin reactivates a `DEACTIVATED` account via an explicit, audited action (not a silent toggle). *Permissions:* `auth.user.manage` (elevated). *Audit:* `ACCOUNT_REACTIVATED`. *Edge:* requires fresh activation; reuse history preserved. *QA:* governed reactivation; silent toggle blocked.
- **UC-AUTH-021 — Locked-Out Login Attempt** · *High* · Rules: AUTH-004. Login during lockout → generic denial until cooldown/admin unlock. *Audit:* `LOGIN_FAILED` (locked). *QA:* denial during lockout; success after cooldown.
- **UC-AUTH-022 — Token Reuse Detected** · *Critical* · Rules: AUTH-002. Replay of a rotated token → family revoked, user notified. *Audit:* `TOKEN_REUSE_DETECTED`. *QA:* reuse triggers revocation + notice; legitimate refresh unaffected.

## 8. Module-level QA & Edge Themes
- **Security-first negatives:** every auth use case must be tested for enumeration leaks, generic error messages, and audit-without-secrets.
- **Fail-closed:** denylist/key/breach-service outages must never silently grant access; test the degraded paths.
- **Minors:** student/guardian accounts follow the same protections; no weaker path for younger users.
