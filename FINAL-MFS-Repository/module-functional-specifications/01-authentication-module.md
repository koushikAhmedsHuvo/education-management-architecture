# 01 — Authentication Module — Functional Specification

> Implementation-ready specification for engineering, QA, and design. Derives from the approved Architecture Blueprint, Business Rules Catalog (`AUTH-001…011`), and Use Case Repository (`UC-AUTH-001…022`). No new architecture or business rules are introduced here.

---

## 1. Module Overview

**Purpose.** Establish and maintain trusted user identity: credential verification, token issuance/rotation, session and device lifecycle, MFA, account recovery, and invitation-based activation.

**Business Goal.** Prevent unauthorized access to minors' and institutional data while keeping legitimate sign-in frictionless; provide the verifiable identity layer every other module's authorization depends on.

**Scope.** Login/logout; access + refresh token issuance and rotation with reuse detection; failed-attempt lockout; revocation on state change; session/device visibility; MFA enrollment/challenge/step-up; password policy and recovery; invitation-based activation; auth policy configuration; login/security auditing and reporting.

**Out of Scope.** Permission evaluation and role/scope resolution (Authorization module). Staff/student profile data (HR/Student modules). Notification delivery mechanics (Notification module — this module only raises events). Identity-provider federation/SSO (future enhancement).

---

## 2. Actors

**Primary Actors.** End User (staff/teacher/guardian/student), Security/IT Administrator, System (token rotation, lockout, revocation jobs).

**Secondary Actors.** Authorization module (consumes identity + permissions-version), Notification module (delivers invites/recovery/security alerts), Audit module (immutable security events), Configuration Engine (auth policy values).

---

## 3. Functional Requirements

| ID | Requirement | Description | Priority | Dependencies |
|---|---|---|---|---|
| FR-001 | Standard login | Verify credentials, issue a short-lived access token + rotating refresh token on success. | Critical | Config (policy), Authorization (scope/roles claims) |
| FR-002 | Token refresh with reuse detection | Rotate refresh tokens per use; detect reuse of a consumed token and revoke the whole family. | Critical | FR-001 |
| FR-003 | Concurrent-refresh grace window | Tolerate near-simultaneous legitimate refreshes within a small window without false family revocation. | High | FR-002 |
| FR-004 | Progressive lockout | Apply escalating delays/lockout on repeated failed attempts; never reveal account existence. | Critical | Config (lockout policy) |
| FR-005 | Immediate revocation on state change | Revoke active sessions/tokens immediately on deactivation, suspension, role/scope change, or credential change. | Critical | Authorization, HR/Staff status |
| FR-006 | Session & device visibility/control | Let a user view active sessions/devices and sign out individually or all. | High | FR-001 |
| FR-007 | MFA policy & step-up | Enforce MFA per policy/role and require step-up for sensitive operations. | High | Config (MFA policy) |
| FR-008 | Password policy & breached-password rejection | Enforce strength and reject known-breached passwords. | High | Config (password policy) |
| FR-009 | Non-enumerating, rate-limited recovery | Provide password recovery that never reveals whether an account exists and is rate-limited. | Critical | Notification |
| FR-010 | Session invalidation on credential change | Invalidate other sessions on password/credential change (configurable keep-current). | High | FR-005 |
| FR-011 | Invitation-based activation | Create users in a pending state activated via a secure, expiring invitation; user sets own credentials. | High | Notification, Authorization (membership) |
| FR-012 | Administered account lifecycle | Admin deactivate (soft), unlock, reactivate (governed), reset MFA, force-logout. | High | FR-005, Authorization |
| FR-013 | Auth policy configuration | Configure password/lockout/MFA/session policies via the Configuration Engine (versioned). | Medium | Configuration Engine |
| FR-014 | Security auditing & reporting | Record and report login/security events; export security audit. | Medium | Audit, Reporting |

---

## 4. Features

| Feature | Description | Business Value |
|---|---|---|
| Secure sign-in | Credential + optional MFA login issuing short-lived tokens. | Protects sensitive data; baseline trust. |
| Refresh-token rotation w/ reuse detection | Per-use rotation; family revocation on replay. | Contains token theft automatically. |
| Session & device management | Self-service visibility and remote sign-out. | User control; fast compromise response. |
| MFA & step-up | Policy-driven MFA and per-operation step-up. | Strong protection for privileged actions. |
| Invitation activation | No admin-set passwords; user-set credentials via invite. | Eliminates shared/known initial passwords. |
| Non-enumerating recovery | Identical responses regardless of account existence. | Prevents account discovery. |
| Account lifecycle controls | Deactivate/unlock/reactivate/reset-MFA/force-logout. | Operational security administration. |
| Auth policy engine | Versioned, configurable policies per deployment. | Per-client compliance without code change. |

---

## 5. Screens

Login; MFA Challenge; MFA Enrollment; Forgot Password (request); Reset Password (token landing); Set Password / Activate Invitation; Change Password (self-service); My Sessions & Devices; Admin — User List; Admin — User Detail (security tab: lock state, sessions, MFA); Admin — Invite User; Admin — Bulk Invite (CSV); Admin — Auth Policy Configuration; Admin — Login & Security Report; Admin — Security Audit Export; Session-Expired / Re-auth interstitial.

---

## 6. Screen Actions

| Screen | Actions / Buttons | Bulk Actions |
|---|---|---|
| Login | Sign In, Forgot Password, (SSO future) | — |
| MFA Challenge | Verify Code, Use Backup Code, Resend | — |
| MFA Enrollment | Show QR/Secret, Verify, Generate Backup Codes, Download Codes | — |
| Forgot / Reset Password | Submit Request, Set New Password | — |
| Set Password / Activate | Set Password, Activate | — |
| Change Password | Update Password, (toggle) Sign out other devices | — |
| My Sessions & Devices | Sign Out (per session), Sign Out All, Refresh List | Sign Out All |
| Admin User List | Invite User, Open Detail, Search/Filter, Export | Bulk Invite, Bulk Deactivate |
| Admin User Detail | Deactivate, Reactivate, Unlock, Reset MFA, Force Logout, View Sessions | Revoke All Sessions |
| Admin Invite / Bulk Invite | Send Invite, Upload CSV, Validate, Dispatch, Download Error Report | Dispatch Invites |
| Auth Policy Config | Edit, Save (new version), View History, Rollback | — |
| Login & Security Report | Run, Filter, Export | Export |

---

## 7. Forms

**Login** — `identifier` (email/username, text, required), `password` (password, required). Validation: non-empty; generic failure message (no enumeration).

**Set Password / Change Password** — `newPassword` (password, required), `confirmPassword` (password, required, must match). Validation: meets configured policy (length/complexity); not a known-breached password (AUTH-008); confirm matches.

**MFA Enrollment** — `verificationCode` (text, required, 6 digits). Validation: valid TOTP for the enrollment secret.

**Forgot Password** — `identifier` (email, required). Validation: format only; identical success response regardless of existence (AUTH-009).

**Invite User** — `email` (email, required), `displayName` (text, required), `initialScope` (select, required — institute/campus), `initialRole` (select, required). Defaults: status `PENDING`, invite TTL from policy. Validation: email unique among active users; inviter authorized for the chosen scope (AUTHZ-007).

**Bulk Invite (CSV)** — columns `email, displayName, scope, role`. Validation: per-row schema, duplicate detection, scope authority; produces a downloadable error report for rejected rows.

**Auth Policy** — `minLength` (number), `complexityClasses` (multi-select), `breachCheck` (toggle), `maxFailedAttempts` (number), `lockoutSchedule` (json/structured), `mfaRequiredRoles` (multi-select), `accessTokenTTL` (number), `refreshTokenTTL` (number), `graceWindowSeconds` (number). Validation: typed/range-checked by the Configuration Engine (CFG-002); publish creates a new version (CFG-004).

---

## 8. Search & Filter Requirements

**Users:** search by email/displayName; filter by status (pending/active/locked/deactivated), scope, role, MFA-enabled, last-login range. **Sessions:** filter by user, device, IP, active/expired, created range. Sorting: last-login, created, status. Pagination: server-side, default 25/page, max 100; cursor or offset per the shared table engine.

---

## 9. Table Requirements

**User table columns:** Display Name, Email, Status, Scope(s)/Role(s) summary, MFA, Last Login, Created. Sorting on Name/Status/Last Login/Created. Filtering as above. Export (governed) for admins. Bulk actions: Invite, Deactivate. **Sessions table columns:** Device/Agent, IP, Created, Last Active, Status, action Sign-Out. **Security audit table:** Timestamp, Actor, Event, Target, Result, IP — read-only, exportable.

---

## 10. Workflow Requirements

**Trigger events:** login success/failure, refresh, reuse-detection, lockout, invite sent/accepted, password/MFA change, admin lifecycle action. **Status changes:** User `PENDING → ACTIVE → LOCKED ⇄ ACTIVE → DEACTIVATED → (governed) REACTIVATED`. **Approvals:** governed reactivation may require approval (Workflow Engine). **Notifications:** invite, recovery link, new-device/login alert, lockout, credential-change confirmation. **Audit events:** all of the above as immutable security events (AUD-001 via outbox).

---

## 11. Permission Requirements

| Capability | Permission |
|---|---|
| View users/sessions | `auth.user.view`, `auth.session.view` |
| Invite/create user | `auth.user.invite` |
| Deactivate/reactivate | `auth.user.deactivate`, `auth.user.reactivate` |
| Unlock account | `auth.user.unlock` |
| Reset MFA | `auth.user.mfa.reset` |
| Force logout / revoke session | `auth.session.revoke` |
| Configure auth policy | `auth.policy.manage` |
| Export security audit | `auth.audit.export` |

Self-service actions (own password, own MFA, own sessions) require only an authenticated session for the acting user. All admin actions are scope-bound (AUTHZ-002) and authority-checked (AUTHZ-007).

---

## 12. Business Rule References

AUTH-001 (access token issuance), AUTH-002 (rotating refresh + family reuse detection), AUTH-003 (concurrent-refresh grace), AUTH-004 (progressive lockout), AUTH-005 (immediate revocation on state change), AUTH-006 (session/device control), AUTH-007 (MFA policy/step-up), AUTH-008 (password policy/breached rejection), AUTH-009 (non-enumerating recovery), AUTH-010 (session invalidation on credential change), AUTH-011 (invitation activation). Cross-cutting: AUTHZ-002/007 (scope authority for admin actions), CFG-002/004 (typed, versioned policy), AUD-001 (immutable audit).

## 13. Use Case References

UC-AUTH-001…022 — notably UC-AUTH-001 (Standard Login), UC-AUTH-002 (Token Refresh with Reuse Detection), UC-AUTH-003 (Logout/Sign-out All), UC-AUTH-004 (Password Recovery), UC-AUTH-005/006 (MFA Enrollment/Challenge & Step-Up), UC-AUTH-007/008 (Invitation Activation/Create User), UC-AUTH-011 (Deactivate), UC-AUTH-012 (Unlock), UC-AUTH-013 (Force Logout), UC-AUTH-014 (Reset MFA), UC-AUTH-015 (Configure Policy), UC-AUTH-009 (View Own Sessions/Devices), UC-AUTH-010 (Change Own Password), UC-AUTH-016 (Search Users/Sessions), UC-AUTH-017 (Bulk Invite), UC-AUTH-018 (Export Login/Security Audit), UC-AUTH-019 (Login Activity & Security Report), UC-AUTH-020 (Reactivation), UC-AUTH-021 (Locked-Out Attempt), UC-AUTH-022 (Token Reuse Detected).

---

## 14. API Requirements

| Endpoint Purpose | Method | Actor |
|---|---|---|
| Authenticate (login) | POST | End User |
| Refresh access token (rotate) | POST | End User (client) |
| Logout current session | POST | End User |
| Logout all sessions | POST | End User |
| Initiate password recovery | POST | End User |
| Complete password reset | POST | End User |
| Activate invitation / set password | POST | Invited User |
| Enroll MFA / verify | POST | End User |
| MFA challenge / step-up verify | POST | End User |
| Change own password | POST | End User |
| List own sessions/devices | GET | End User |
| Revoke a session | DELETE | End User / Admin |
| Invite user | POST | Admin |
| Bulk invite (CSV) | POST | Admin |
| Deactivate / reactivate user | POST | Admin |
| Unlock account | POST | Admin |
| Reset user MFA | POST | Admin |
| Force logout user | POST | Admin |
| Get/Update auth policy | GET/PUT | Admin |
| List users / sessions (search) | GET | Admin |
| Export security audit | GET | Admin |

All calls pass Authorization-layer checks (deny-by-default, scope, ownership). Bearer-token calls are CSRF-immune; the cookie refresh path uses SameSite + anti-CSRF (per blueprint D56).

---

## 15. Database Requirements

**Entities:** `User` (identity, status, MFA flags), `Credential` (hashed password, algorithm/params), `RefreshTokenFamily` + `RefreshToken` (hashed, rotation lineage, reuse flag), `Session`/`Device`, `MfaEnrollment` (+ backup codes hashed), `Invitation` (token hash, TTL, status), `FailedAttempt`/`LockoutState`, `AuthPolicyVersion` (via Config). **Relationships:** User 1—* Session; User 1—* RefreshTokenFamily 1—* RefreshToken; User 1—1 Credential; User 1—* MfaEnrollment; User 1—* Invitation. **Indexes:** unique(User.email where active), index(Session.userId, status), index(RefreshToken.familyId), unique(RefreshToken.hash), index(Invitation.tokenHash), index(FailedAttempt.userId, ts). Tokens and codes stored hashed only (never plaintext).

---

## 16. Notification Requirements

| Channel | Notifications |
|---|---|
| Email | Invitation, password-recovery link, credential-change confirmation, new-device/login alert, account lockout, reactivation. |
| SMS | MFA codes (if SMS MFA configured), high-risk login alert (policy). |
| Push | New-device login alert (if app installed). |
| In-App | Active-session changes, forced-logout notice, MFA-reset notice. |

Security/safeguarding notifications are mandatory-category (cannot be opted out — NOT-003). Content is minimized on low-assurance channels (NOT-004).

---

## 17. Audit Requirements

Log every: login success/failure (with reason class, not credential), refresh + rotation, reuse-detection + family revocation, lockout/unlock, invite sent/accepted, password/MFA enrollment/reset/change, admin lifecycle actions, policy changes. Record **who** (actor user/admin or system), **when** (UTC timestamp), **before/after** for state changes (status, MFA-enabled, policy values). **Never** store passwords, tokens, codes, or secrets — only change-occurred metadata (AUD-004). Emitted via outbox for reliable, immutable capture (AUD-001/002).

---

## 18. Reporting Requirements

**Reports:** Login Activity (success/failure trends, by scope/role), Security Events (lockouts, reuse-detections, MFA resets), Dormant/never-activated accounts, MFA adoption. **Exports:** governed security-audit export (signed URL, audited — UC-AUTH-018). **Dashboards:** admin security overview (active sessions, recent lockouts, pending invites, failed-login spikes).

---

## 19. Error Handling

**Validation errors:** weak/breached password, mismatched confirmation, invalid MFA code, expired/invalid invite or reset token → specific, non-enumerating messages. **Permission errors:** unauthorized admin action → 403; out-of-scope target → 403/not-found (no leak). **Workflow errors:** governed reactivation pending approval → clear pending state. **System errors:** token-store/Redis unavailable → fail closed for issuance; Config unavailable → last-known policy with alert; Audit unavailable → security actions fail closed (AUD-007).

---

## 20. Edge Cases

**Concurrent updates:** simultaneous legitimate refreshes handled by the grace window (AUTH-003); double login → independent sessions. **Duplicate data:** re-invite an existing active email → blocked/linked. **Partial failures:** invite created but email send fails → invite valid, delivery retried (outbox); never blocks creation. **Rollback scenarios:** policy publish failure → prior version remains active (no partial policy); lockout job interrupted → idempotent re-run. **Reuse race:** old + new refresh tokens presented near-simultaneously → grace tolerates legitimate, reuse outside window revokes family.

---

## 21. Acceptance Criteria

**Functional.** Valid credentials (+MFA if required) yield a short-lived access token and a rotating refresh token; each refresh rotates; replay of a consumed token revokes the family; lockout escalates and recovery/lookups never reveal account existence; state changes revoke access immediately; invited users activate by setting their own password; admins can unlock/deactivate/reactivate/reset-MFA/force-logout within scope.

**Business.** No admin ever knows a user's password; minors'/institutional data is unreachable without a valid authenticated, authorized session; security policies are configurable per deployment and versioned; every security-consequential action is immutably audited without storing secrets.

---

## 22. Future Enhancements

SSO/OIDC and SAML federation; passkeys/WebAuthn passwordless; risk-based/adaptive MFA (geo/velocity); device fingerprinting and trust; configurable session-timeout by sensitivity; anomaly-detection alerting; admin-delegated MFA recovery workflows.
