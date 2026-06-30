# 00 — API Design Conventions (Wave 1: Platform Foundation)

> Shared conventions referenced by every file in `/api-contracts`. These are not new business logic — they operationalize decisions already made in the approved **Architecture Blueprint** (Part B: DTO strategy D10, entity strategy, exception filter, common module) for a NestJS + TypeORM + PostgreSQL backend consumed by a Next.js + RTK Query frontend. Every endpoint in every module file inherits this document; module files do not repeat it.

## 1. Versioning & Base URL
All endpoints are rooted at `/api/v1/`. Breaking changes ship as `/api/v2/` per resource group, never as an in-place change to `/v1`.

## 2. Authentication & Active Scope
- Every protected endpoint requires `Authorization: Bearer <accessToken>` (short-lived access token, AUTH-001).
- The **active scope** (institute, and campus where relevant) is resolved server-side from the user's session by default. A caller holding more than one membership may override it per-request via `X-Active-Institute-Id` and `X-Active-Campus-Id` headers (UC-AUTHZ-03, Architecture Part C "active institute/campus scope header"); the override is validated against the caller's memberships before use (AUTHZ-002).
- Endpoints marked **Auth: No** are either public (login, forgot-password) or bound to a single-use token presented in the request body/path (refresh token, password-reset token, invitation token) rather than a session — see AUTH-002, AUTH-009, AUTH-011.

## 3. Authorization Model (applies to every protected endpoint)
Every protected operation is evaluated in three independent layers, per AUTHZ-001–004: **(a) permission** — the caller holds the named permission string; **(b) scope** — the resource's `instituteId`/`campusId` matches the caller's active, membership-backed scope; **(c) ownership** — row-level predicates (e.g., "my own session", "a user within my scope") are applied in the data layer. All three must pass; any single failure is a denial (deny-by-default, AUTHZ-004). Each API below names the governing permission string and the scope it is evaluated at; these strings are taken verbatim from the approved UI Screen Specifications' "Permission Visibility" sections so the API surface matches what the UI already encodes.

**Naming note:** where the Business Rules Catalog names an umbrella manage-permission (e.g., Doc 03 Institute's `org.institute.manage`) but the approved UI Screen Specs name more granular, per-action permissions (e.g., `institute.create`, `institute.activate`, `institute.suspend`), this contract uses the granular UI-approved strings, since they map one-to-one onto REST endpoints. Where neither document names a **view** counterpart to a stated **manage** permission, this contract follows the view/manage naming pattern already established elsewhere in the approved rules (e.g., `authz.policy.view` beside `authz.role.manage`, `institute.view` beside `institute.update`) and introduces the corresponding `<domain>.view` string for read endpoints, scoped identically to its manage counterpart. This is REST contract scaffolding required to expose any read endpoint at all, not new business logic.

## 4. Response Envelope
**Single resource (200/201):**
```json
{ "data": { /* response DTO */ }, "meta": { "correlationId": "c5e1..." } }
```
**List (200):**
```json
{ "data": [ /* response DTOs */ ], "meta": { "page": 1, "pageSize": 25, "totalItems": 134, "totalPages": 6, "correlationId": "c5e1..." } }
```
**Error (4xx/5xx)** — produced by the single global exception filter (Architecture Part B):
```json
{ "error": { "code": "VALIDATION_FAILED", "message": "displayName is required", "field": "displayName", "correlationId": "c5e1..." } }
```
`code` is a stable, machine-readable string (e.g., `VALIDATION_FAILED`, `INVALID_CREDENTIALS`, `OUT_OF_SCOPE`, `VERSION_CONFLICT`, `PRIVILEGE_ESCALATION_BLOCKED`); `field` is present only for field-level validation errors; `correlationId` is always present and is the same id surfaced to the user-facing error screen so support can trace it end-to-end (Architecture Part C).

## 5. HTTP Status Mapping
| Status | Meaning | Typical cause |
|---|---|---|
| 200 | OK | successful read/update |
| 201 | Created | successful resource creation |
| 202 | Accepted | request enqueued as a deferred/async job (see §9) |
| 204 | No Content | successful action with no response body (revoke, delete) |
| 400 | Bad Request | malformed request (unparseable body, bad query param) |
| 401 | Unauthorized | missing/invalid/expired access token, invalid credentials |
| 403 | Forbidden | permission, scope, or ownership denial (AUTHZ-001–004); SoD/escalation block |
| 404 | Not Found | resource does not exist **or** exists outside the caller's scope (AUTHZ-002 — out-of-scope reads return 404, never reveal existence) |
| 409 | Conflict | optimistic-lock version mismatch, duplicate unique field, invalid state transition, single-current-session violation |
| 422 | Unprocessable Entity | request is well-formed but fails domain validation (policy rules, business invariants) |
| 429 | Too Many Requests | rate limiting / progressive lockout (AUTH-004, AUTH-009) |
| 500 | Internal Server Error | unexpected failure; never leaks stack traces or internals (Architecture Part B) |

## 6. Pagination, Filtering, Sorting
- Pagination is page-based: `page` (1-based, default `1`), `pageSize` (default `25`, max `100`) — the `25` default matches the approved UI's table default page size for consistency end-to-end.
- Filters are documented per endpoint as query parameters; multi-value filters accept comma-separated values (e.g., `status=ACTIVE,LOCKED`).
- Sorting uses `sortBy` (a documented field name) and `sortOrder` (`asc`|`desc`); each endpoint states its default sort.
- List responses always include the `meta.page/pageSize/totalItems/totalPages` block from §4.

## 7. Soft Delete & Audit Fields
- Per the Architecture's entity strategy, every persistence entity carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, a soft-delete marker (`deletedAt`), and an optimistic-lock `version`. Every response DTO in this contract exposes these fields.
- Where a business rule permits removal (e.g., an empty `DRAFT` institute, a never-used campus, INST-008/CAMP-007), the operation is a **soft delete** by default: `deletedAt` is stamped and the row is excluded from ordinary list/get queries. A small number of endpoints expose a distinct, separately-permissioned **hard delete** only where the business rules explicitly allow it for empty/never-used records — each such case is called out in its module file.
- List endpoints exclude soft-deleted rows unless the caller supplies `includeDeleted=true` **and** holds the relevant view/audit permission; this is an explicit, audited recovery path, not a default.

## 8. Optimistic Concurrency
Every mutable resource's response DTO includes a `version` integer. `PUT`/`PATCH` requests that modify a versioned resource must include the `version` they last read in the request body. A mismatch returns `409 Conflict` with `code: "VERSION_CONFLICT"` and the current `version`, so the client can reload and retry — this never silently overwrites a concurrent change.

## 9. Async / Bulk Operations
Bulk operations (e.g., bulk invite, bulk transfer, bulk promotion) follow the Architecture's documented deferred-job pattern (Part B "Background Jobs"): small batches (≤50 rows, per-endpoint threshold noted where it applies) execute synchronously and return inline per-row results; larger batches return `202 Accepted` with a `jobId` and are processed as an idempotent, retry-safe background job. Idempotency is supplied by the caller via an `Idempotency-Key` request header; replaying the same key returns the original result without reprocessing. Job status is polled via the shared platform endpoint below (not duplicated per module):

**GET /api/v1/jobs/{jobId}**
- **Purpose:** Poll the status/progress/result of an async job created by a bulk operation.
- **Auth:** Yes — caller must be the job's original requester, or hold the same permission the originating bulk operation required, in the same scope.
- **Response DTO:** `{ jobId, type, status: "QUEUED"|"RUNNING"|"COMPLETED"|"FAILED", progress: { processed, total }, result?: { accepted: [...], rejected: [{ row, reason }] }, createdAt, completedAt? }`
- **Errors:** `404` job not found / not visible to caller.
- **Pagination:** N/A.

## 10. Governed Exports
Where a screen offers a governed export (e.g., Access Audit Export, Login & Security Report export), the export endpoint never streams a file directly. It returns a short-lived, signed download URL backed by the platform's object-storage adapter (Architecture Part B "platform/storage"), and the export action itself is audited as a sensitive operation: `{ downloadUrl, expiresAt }`, `200 OK`.

## 11. Data Shapes
- **IDs:** UUIDv7 strings (time-ordered, per the Architecture's entity strategy).
- **Timestamps:** ISO 8601 UTC strings.
- **JSON field casing:** camelCase in all request/response bodies (database columns are snake_case internally per the Architecture; the API boundary is camelCase, consistent with the NestJS/TypeORM + Next.js/RTK Query stack).
- **Scope fields:** scoped resources carry `instituteId` and, where relevant, `campusId`/`sessionId` (INST-006, CAMP-004).

## 12. Module Map (Wave 1 → Business Rule Modules)
This wave implements exactly the five approved Business Rule modules already in scope for Wave 1 — no new modules are introduced:

| File | Business Rule Module | UI Screen Spec |
|---|---|---|
| `01-authentication-api.md` | Doc 01 — Authentication | `01-authentication-screens.md` (11 screens) |
| `02-user-management-api.md` | Doc 01 — Authentication (User account lifecycle) | `02-user-management-screens.md` (4 screens) |
| `03-role-permission-api.md` | Doc 02 — Authorization | `03-role-permission-screens.md` (13 screens) |
| `04-institute-api.md` | Doc 03 — Institute | `04-institute-management-screens.md` (13 screens) |
| `05-campus-api.md` | Doc 04 — Campus | `05-campus-management-screens.md` (12 screens) |
| `06-academic-session-api.md` | Doc 05 — Academic Session | `06-academic-session-screens.md` (13 screens) |

User account CRUD is split from Role/Permission/Membership because the Business Rules Catalog itself splits them this way: Doc 01 (Authentication) owns the **User** entity and its lifecycle states (`INVITED`/`ACTIVE`/`LOCKED`/`SUSPENDED`/`DEACTIVATED`/`ANONYMIZED`); Doc 02 (Authorization) owns **Role**, **Permission Registry**, and **Membership** (the user↔scope↔role link) as distinct aggregates. This contract mirrors that boundary rather than collapsing it.
