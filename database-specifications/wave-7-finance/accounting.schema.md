# Accounting (Chart of Accounts)

## 1. Table Purpose

Hierarchical chart of accounts for the proposed double-entry ledger extension.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `code` | `VARCHAR(16)` | No | — |
| `name` | `VARCHAR(255)` | No | — |
| `type` | `account_type (ENUM: ASSET, LIABILITY, EQUITY, INCOME, EXPENSE)` | No | — |
| `parent_account_id` | `UUID` | Yes | — |
| `status` | `account_status (ENUM: ACTIVE, INACTIVE)` | No | 'ACTIVE' |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT; `parent_account_id → chart_of_accounts(id)` RESTRICT, self-referencing.

## 5. Unique Constraints

`(institute_id, code) WHERE deleted_at IS NULL`.

## 6. Indexes

`(institute_id, type)`; `(parent_account_id)` for hierarchy traversal.

## 7. Relationships

Self-referencing hierarchy; referenced by `journal_entry_lines` (§22).

## 8. Soft Delete Strategy

None operative — an account referenced by any posted journal line is never hard-deleted, only deactivated (`status = 'INACTIVE'`), mirroring every other "consumers depend on this forever" registry table in this schema.

## 9. Audit Fields

| Column | Type | Nullable | Default Value |
|---|---|---|---|
| `created_at` | `TIMESTAMPTZ` | No | `now()` |
| `created_by` | `UUID` | Yes | — |
| `updated_at` | `TIMESTAMPTZ` | No | `now()` |
| `updated_by` | `UUID` | Yes | — |
| `deleted_at` | `TIMESTAMPTZ` | Yes | — |
| `deleted_by` | `UUID` | Yes | — |

## 10. Multi-Institute Isolation Strategy

**Direct, mandatory.** `institute_id` is NOT NULL. *(Beyond approved repo — see the file's own header flag.)*

## 11. Notes for TypeORM Entity Design

Seed via a standard chart-of-accounts template per institute on creation, mirroring `config_type_templates`' (Wave 2) seed-on-creation pattern.

---

## 12. Performance Considerations

**Trivially low volume (a standard chart template, typically well under a few hundred accounts per institute).** Cache at the application layer; read on every journal-entry-line write and every report query.