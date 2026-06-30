# Journal Entry Lines

## 1. Table Purpose

Individual debit/credit lines — normalized, not JSONB, because cross-cutting account-level aggregation (the general ledger, trial balance) is a genuine, frequent, independent query.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `journal_entry_id` | `UUID` | No | — |
| `account_id` | `UUID` | No | — |
| `debit` | `NUMERIC(12,2)` | Yes | — |
| `credit` | `NUMERIC(12,2)` | Yes | — |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`journal_entry_id → journal_entries(id)` ON DELETE CASCADE — a true dependent of its entry, the same reasoning as every other line-item-of-a-parent table in this schema; `account_id → chart_of_accounts(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK.

## 6. Indexes

`(account_id, journal_entry_id)` — the general-ledger drill-down's primary access path.

## 7. Relationships

Many lines per entry; many lines per account (its full ledger, queried via this index).

## 8. Soft Delete Strategy

Not soft-deleted independently of its parent entry — a line is removed only by removing (or never posting) its `DRAFT` parent.

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

**Derived, via `journal_entry_id → journal_entries → institute_id`.** No direct institute column. *(Beyond approved repo — see the file's own header flag.)*

## 11. Notes for TypeORM Entity Design

The General Ledger, Trial Balance, Income Statement, Balance Sheet, and Cash Flow Summary (`accounting.api.md` #6–10) are **all computed reports** — `SUM(debit), SUM(credit) GROUP BY account_id` and its variants — with zero additional storage, the same "reports are queries, not tables" discipline this schema has held to in every prior wave.

---

## 12. Performance Considerations

**Volume scales directly with `journal_entries`** (typically 2+ lines per entry, by double-entry construction). The `(account_id, journal_entry_id)` index is the General Ledger/Trial Balance report's entire performance story — every financial-statement query is fundamentally a `GROUP BY account_id` aggregate over this table, so this index's health directly determines report-generation latency at scale.