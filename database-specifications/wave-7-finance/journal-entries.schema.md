# Journal Entries

## 1. Table Purpose

Double-entry journal entries — draft until balanced and posted.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `entry_number` | `VARCHAR(32)` | No | — |
| `entry_date` | `DATE` | No | — |
| `description` | `TEXT` | No | — |
| `source` | `journal_source (ENUM: MANUAL, AUTO_FROM_COLLECTION)` | No | 'MANUAL' |
| `status` | `journal_entry_status (ENUM: DRAFT, POSTED)` | No | 'DRAFT' |
| `posted_by` | `UUID` | Yes | — |
| `posted_at` | `TIMESTAMPTZ` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT; `posted_by → users(id)` RESTRICT.

## 5. Unique Constraints

`(institute_id, entry_number) WHERE deleted_at IS NULL`.

## 6. Indexes

`(institute_id, status, entry_date)`.

## 7. Relationships

One entry has many `journal_entry_lines` (§22).

## 8. Soft Delete Strategy

A `DRAFT` entry may be soft-deleted; a `POSTED` entry **never** is — blocked by trigger, identical reasoning to `invoices`/`payments`' immutability.

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

The trigger is `DEFERRABLE INITIALLY DEFERRED` specifically so the entry header and its lines can be inserted in any order within one transaction and the balance is only checked at commit — without deferral, inserting the header before its lines would fail the check prematurely.

---

## 12. Performance Considerations

**Moderate-to-high volume if auto-posting from collection is enabled** (`source = 'AUTO_FROM_COLLECTION'` could generate one entry per payment/invoice event, scaling with `payments`/`invoices` directly). The `DEFERRABLE` balance-check trigger adds a small, transaction-commit-time cost per posted entry — acceptable given posting is a deliberate, infrequent-relative-to-collection-volume action even in the auto-posting case.