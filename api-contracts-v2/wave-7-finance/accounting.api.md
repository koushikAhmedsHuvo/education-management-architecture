# Accounting API

> ⚠ **[Beyond approved repo — standard accounting conventions]**
> The approved Business Rules Catalog has exactly four Finance modules: Fee Management (Doc 17), Payment (Doc 18), Discount (Doc 19), Scholarship (Doc 20). **There is no separate double-entry "Accounting" business-rules module** — this ERP's finance is fee-collection-centric (invoices, payments, dues), not a general-ledger system. This file is generated to **standard accounting conventions** (a conventional Chart of Accounts / Journal / Ledger / Trial Balance / Financial Statements model, in the spirit of Xero, QuickBooks, and Odoo Accounting), as a **proposed extension** to satisfy the requested repository structure. It is **not validated against the Business Rules Catalog** and should be reconciled with the architecture team before implementation. Every endpoint below is written to plug into the same underlying fee/payment data as its source of truth (`fee-collection.api.md`, `payment.api.md`) rather than introducing a parallel, disconnected ledger.

## 1. API Overview

**Purpose.** Maintain a conventional double-entry general ledger — a Chart of Accounts, journal entries, account balances, a trial balance, and standard financial statements (Income Statement, Balance Sheet, Cash Flow Summary) — derived from and reconciled against the fee/payment transaction data already governed by `fee-collection.api.md` and `payment.api.md`.

**Module Context.** Not grounded in the approved Business Rules Catalog (see the flag above). Conventions follow standard double-entry bookkeeping: every journal entry's debits must equal its credits; published/posted entries are immutable (mirroring the immutability discipline already established for invoices, FEE-003, and receipts, PAY-001, in the approved modules); corrections are reversing entries, never edits, for the same reason an issued invoice is never edited.

---

## 2. Endpoints

### 1. List Chart of Accounts
- **Method:** `GET`
- **URL:** `/api/v1/accounting/accounts`
- **Description:** Browse the hierarchical chart of accounts (Assets / Liabilities / Equity / Income / Expense).
- **Authentication (Role-Based):** Yes — `accounting.view`, scoped.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "type": { "type": "string", "enum": ["ASSET", "LIABILITY", "EQUITY", "INCOME", "EXPENSE"] },
    "page": { "type": "integer" },
    "pageSize": { "type": "integer" }
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
      "id": { "type": "string" },
      "code": { "type": "string" },
      "name": { "type": "string" },
      "type": { "type": "string", "enum": ["ASSET", "LIABILITY", "EQUITY", "INCOME", "EXPENSE"] },
      "parentAccountId": { "type": "string" },
      "balance": { "type": "number" },
      "status": { "type": "string", "enum": ["ACTIVE", "INACTIVE"] }
    },
    "required": ["id", "code", "name", "type", "status"]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** None (beyond approved repo — see file-level flag).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 2. Create / Edit Account
- **Method:** `POST` / `PUT /api/v1/accounting/accounts/{id}`
- **URL:** `/api/v1/accounting/accounts`
- **Description:** Define a new account in the chart, or edit an existing one. A referenced account is **deactivated, never deleted**.
- **Authentication (Role-Based):** Yes — `accounting.config.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "code": { "type": "string" },
    "name": { "type": "string" },
    "type": { "type": "string", "enum": ["ASSET", "LIABILITY", "EQUITY", "INCOME", "EXPENSE"] },
    "parentAccountId": { "type": "string" },
    "openingBalance": { "type": "number" },
    "version": { "type": "integer" }
  },
  "required": ["code", "name", "type"]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "id": { "type": "string" },
    "code": { "type": "string" },
    "name": { "type": "string" },
    "status": { "type": "string", "const": "ACTIVE" },
    "version": { "type": "integer" }
  },
  "required": ["id", "code", "name", "status", "version"]
}
```
- **Validation Rules:** `code` unique within the chart; an account already referenced by a posted journal entry cannot be deleted — only deactivated (mirroring the deprecate-not-delete discipline used throughout the approved Configuration Engine and Permission Registry).
- **Error Codes:** `409 ACCOUNT_CODE_EXISTS`; `409 CANNOT_DELETE_REFERENCED_ACCOUNT`.
- **Business Rules Applied:** None (beyond approved repo).
- **Pagination:** N/A.

### 3. List Journal Entries
- **Method:** `GET`
- **URL:** `/api/v1/accounting/journal-entries`
- **Description:** Browse journal entries — draft and posted.
- **Authentication (Role-Based):** Yes — `accounting.view`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "status": { "type": "string", "enum": ["DRAFT", "POSTED"] },
    "dateFrom": { "type": "string" },
    "dateTo": { "type": "string" },
    "page": { "type": "integer" },
    "pageSize": { "type": "integer" }
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
      "id": { "type": "string" },
      "entryNumber": { "type": "string" },
      "date": { "type": "string" },
      "description": { "type": "string" },
      "totalDebit": { "type": "number" },
      "totalCredit": { "type": "number" },
      "source": { "type": "string", "enum": ["MANUAL", "AUTO_FROM_COLLECTION"] },
      "status": { "type": "string", "enum": ["DRAFT", "POSTED"] }
    },
    "required": ["id", "entryNumber", "date", "totalDebit", "totalCredit", "status"]
  }
}
```
- **Validation Rules:** None beyond access.
- **Error Codes:** `403 FORBIDDEN`.
- **Business Rules Applied:** None (beyond approved repo).
- **Pagination:** Yes — default `page=1, pageSize=25`.

### 4. Create Journal Entry (Draft)
- **Method:** `POST`
- **URL:** `/api/v1/accounting/journal-entries`
- **Description:** Create a draft double-entry journal entry — debits and credits must balance before it can be posted.
- **Authentication (Role-Based):** Yes — `accounting.entry.manage`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "date": { "type": "string" },
    "description": { "type": "string" },
    "lines": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "accountId": { "type": "string" },
          "debit": { "type": "number" },
          "credit": { "type": "number" }
        },
        "required": ["accountId"]
      }
    }
  },
  "required": ["date", "description", "lines"]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "id": { "type": "string" },
    "entryNumber": { "type": "string" },
    "totalDebit": { "type": "number" },
    "totalCredit": { "type": "number" },
    "balanced": { "type": "boolean" },
    "status": { "type": "string", "const": "DRAFT" }
  },
  "required": ["id", "entryNumber", "totalDebit", "totalCredit", "balanced", "status"]
}
```
- **Validation Rules:** Each line carries either a `debit` or a `credit`, not both; a draft may be saved unbalanced (`balanced: false`) while being composed — balance is enforced only at posting (#5).
- **Error Codes:** `422 LINE_HAS_BOTH_DEBIT_AND_CREDIT`.
- **Business Rules Applied:** None (beyond approved repo).
- **Pagination:** N/A.

### 5. Post Journal Entry
- **Method:** `POST`
- **URL:** `/api/v1/accounting/journal-entries/{id}/post`
- **Description:** Post a draft journal entry, making it immutable. **Posting is blocked unless debits exactly equal credits.**
- **Authentication (Role-Based):** Yes — `accounting.entry.post` (may be separated by SoD from `accounting.entry.manage` for entries above a configured threshold).
- **Request DTO (JSON Schema):**
```json
{ "type": "object", "properties": {}, "description": "No request body." }
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "id": { "type": "string" },
    "status": { "type": "string", "const": "POSTED" },
    "postedAt": { "type": "string" }
  },
  "required": ["id", "status", "postedAt"]
}
```
- **Validation Rules:** Σdebits must exactly equal Σcredits ("Debits (৳X) must equal credits (৳Y)"); once posted, the entry is **immutable** — corrections are made via a new, reversing entry, never an edit, mirroring `fee-collection.api.md` #11 (credit notes never edit an issued invoice) and `payment.api.md` #20 (payments are reversed, never deleted).
- **Error Codes:** `422 ENTRY_NOT_BALANCED`.
- **Business Rules Applied:** None (beyond approved repo) — pattern mirrors FEE-003 / PAY-007's immutability discipline.
- **Pagination:** N/A.

### 6. Get General Ledger
- **Method:** `GET`
- **URL:** `/api/v1/accounting/ledger/{accountId}`
- **Description:** Per-account ledger activity with a running balance over a period — the drill-down behind every account figure.
- **Authentication (Role-Based):** Yes — `accounting.view`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "dateFrom": { "type": "string" },
    "dateTo": { "type": "string" }
  }
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "accountId": { "type": "string" },
    "openingBalance": { "type": "number" },
    "lines": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "date": { "type": "string" },
          "entryId": { "type": "string" },
          "description": { "type": "string" },
          "debit": { "type": "number" },
          "credit": { "type": "number" },
          "runningBalance": { "type": "number" }
        }
      }
    },
    "closingBalance": { "type": "number" }
  },
  "required": ["accountId", "openingBalance", "lines", "closingBalance"]
}
```
- **Validation Rules:** Read-only; computed entirely from posted journal entries.
- **Error Codes:** `404 ACCOUNT_NOT_FOUND`.
- **Business Rules Applied:** None (beyond approved repo).
- **Pagination:** N/A.

### 7. Get Trial Balance
- **Method:** `GET`
- **URL:** `/api/v1/accounting/trial-balance`
- **Description:** All accounts with their debit/credit balances as of a date — the standard pre-statement integrity check.
- **Authentication (Role-Based):** Yes — `accounting.view`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "asOfDate": { "type": "string" }
  },
  "required": ["asOfDate"]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "asOfDate": { "type": "string" },
    "lines": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "accountCode": { "type": "string" },
          "accountName": { "type": "string" },
          "debit": { "type": "number" },
          "credit": { "type": "number" }
        }
      }
    },
    "totalDebit": { "type": "number" },
    "totalCredit": { "type": "number" },
    "inBalance": { "type": "boolean" }
  },
  "required": ["asOfDate", "lines", "totalDebit", "totalCredit", "inBalance"]
}
```
- **Validation Rules:** `totalDebit` must equal `totalCredit` for `inBalance: true`; an out-of-balance result is surfaced prominently, never silently hidden.
- **Error Codes:** None — an out-of-balance trial balance is a valid (if alarming) response, not an error.
- **Business Rules Applied:** None (beyond approved repo).
- **Pagination:** N/A.

### 8. Get Income Statement
- **Method:** `GET`
- **URL:** `/api/v1/accounting/reports/income-statement`
- **Description:** Income less expenses over a period — surplus/deficit, grouped by category with optional comparative.
- **Authentication (Role-Based):** Yes — `accounting.view`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "periodFrom": { "type": "string" },
    "periodTo": { "type": "string" },
    "comparative": { "type": "boolean" }
  },
  "required": ["periodFrom", "periodTo"]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "income": { "type": "array", "items": { "type": "object", "properties": { "category": { "type": "string" }, "amount": { "type": "number" } } } },
    "totalIncome": { "type": "number" },
    "expenses": { "type": "array", "items": { "type": "object", "properties": { "category": { "type": "string" }, "amount": { "type": "number" } } } },
    "totalExpense": { "type": "number" },
    "netSurplusOrDeficit": { "type": "number" }
  },
  "required": ["income", "totalIncome", "expenses", "totalExpense", "netSurplusOrDeficit"]
}
```
- **Validation Rules:** `netSurplusOrDeficit = totalIncome - totalExpense`, computed from posted entries only.
- **Error Codes:** None.
- **Business Rules Applied:** None (beyond approved repo).
- **Pagination:** N/A.

### 9. Get Balance Sheet
- **Method:** `GET`
- **URL:** `/api/v1/accounting/reports/balance-sheet`
- **Description:** Assets, liabilities, and equity as of a date, where Assets = Liabilities + Equity.
- **Authentication (Role-Based):** Yes — `accounting.view`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "asOfDate": { "type": "string" },
    "comparative": { "type": "boolean" }
  },
  "required": ["asOfDate"]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "assets": { "type": "array", "items": { "type": "object" } },
    "totalAssets": { "type": "number" },
    "liabilities": { "type": "array", "items": { "type": "object" } },
    "totalLiabilities": { "type": "number" },
    "equity": { "type": "array", "items": { "type": "object" } },
    "totalEquity": { "type": "number" },
    "equationBalanced": { "type": "boolean" }
  },
  "required": ["assets", "totalAssets", "liabilities", "totalLiabilities", "equity", "totalEquity", "equationBalanced"]
}
```
- **Validation Rules:** `equationBalanced = (totalAssets == totalLiabilities + totalEquity)`; surfaced prominently if false, never hidden.
- **Error Codes:** None.
- **Business Rules Applied:** None (beyond approved repo).
- **Pagination:** N/A.

### 10. Get Cash Flow Summary
- **Method:** `GET`
- **URL:** `/api/v1/accounting/reports/cash-flow`
- **Description:** Cash inflows and outflows over a period (operating / investing / financing) and the net change in cash.
- **Authentication (Role-Based):** Yes — `accounting.view`.
- **Request DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "periodFrom": { "type": "string" },
    "periodTo": { "type": "string" }
  },
  "required": ["periodFrom", "periodTo"]
}
```
- **Response DTO (JSON Schema):**
```json
{
  "type": "object",
  "properties": {
    "operating": { "type": "number" },
    "investing": { "type": "number" },
    "financing": { "type": "number" },
    "netChange": { "type": "number" },
    "openingCash": { "type": "number" },
    "closingCash": { "type": "number" }
  },
  "required": ["operating", "investing", "financing", "netChange", "openingCash", "closingCash"]
}
```
- **Validation Rules:** `closingCash = openingCash + netChange`.
- **Error Codes:** None.
- **Business Rules Applied:** None (beyond approved repo).
- **Pagination:** N/A.

---

## 3. Standards

- **RESTful design** — resource-oriented URLs, standard HTTP verbs, standard status codes (see `00-api-conventions.md` §5 at the repository root).
- **Consistent naming** — camelCase JSON fields; kebab-case URL segments; plural resource collections.
- **Versioning** — all endpoints are rooted at `/api/v1/`; breaking changes ship as a new version per resource group, never in place.
- **Soft delete** — accounts are deactivated, never deleted, once referenced by a posted entry; posted journal entries are immutable and corrected only via reversing entries, never edited or deleted.
- **Audit fields** — every persisted resource's response DTO carries `createdAt`, `updatedAt`, `createdBy`, `updatedBy`, and an optimistic-lock `version`, per `00-api-conventions.md` §7–8.
- **Multi-tenant support** — every scoped resource is implicitly filtered by the caller's active `instituteId`, resolved from authenticated context; cross-institute access is structurally impossible (AUTHZ-002).

> **Recommendation:** before implementation, reconcile this file with the architecture team to decide whether a full double-entry GL is in scope for this release, or whether `fee-collection.api.md` and `payment.api.md`'s existing invoice/payment ledger already satisfies the institution's financial reporting needs without a parallel chart of accounts.
