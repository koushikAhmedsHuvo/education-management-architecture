# Scholarships (Programs)

## 1. Table Purpose

Funded, slot-bounded scholarship programs (SCH-001/002/005) — backs `scholarship.api.md` #1–4, #12–13.

## 2. Columns

| Column Name | Data Type | Nullable | Default Value |
|---|---|---|---|
| `id` | `UUID` | No | uuidv7() |
| `institute_id` | `UUID` | No | — |
| `name` | `VARCHAR(255)` | No | — |
| `type` | `scholarship_type (ENUM: MERIT, NEED, SPORTS, SPONSORED, FULL, PARTIAL)` | No | — |
| `eligibility_criteria` | `JSONB` | No | — |
| `slots` | `INTEGER` | No | — |
| `slots_filled` | `INTEGER` | No | 0 |
| `coverage` | `JSONB` | No | — |
| `total_fund_amount` | `NUMERIC(12,2)` | Yes | — |
| `fund_source` | `VARCHAR(255)` | No | — |
| `sponsor` | `VARCHAR(255)` | Yes | — |
| `renewal_conditions` | `JSONB` | Yes | — |
| `status` | `scholarship_program_status (ENUM: ACTIVE, CLOSED)` | No | 'ACTIVE' |

## 3. Primary Key

`id`.

## 4. Foreign Keys

`institute_id → institutes(id)` RESTRICT.

## 5. Unique Constraints

None beyond the PK.

## 6. Indexes

`(institute_id, status)`.

## 7. Relationships

One program has many `scholarship_awards`.

## 8. Soft Delete Strategy

Standard; `status = 'CLOSED'` is the disclosed end state.

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

**Direct, mandatory.** `institute_id` is NOT NULL.

## 11. Notes for TypeORM Entity Design

Slot consumption on award (`scholarship.api.md` #7) must be a single atomic statement — `UPDATE scholarship_programs SET slots_filled = slots_filled + 1 WHERE id = $1 AND slots_filled < slots RETURNING slots_filled` — the identical concurrency-safe pattern as Wave 4's `SectionCapacityGuard` and Wave 2's `id_generation_sequences` UPSERT; a zero-row result means the program is full, surfaced as `409 SLOTS_EXCEEDED` without a separate read-then-write race window. Fund balance/committed figures are **computed**, not stored: `disbursed = SUM(scholarship_disbursements.amount WHERE status = 'DISBURSED')`, `committed = SUM(expected remaining coverage for ACTIVE awards)` — no redundant ledger table, consistent with §0.3's "balances are computed, not cached" principle applied here a third time.

---

## 12. Performance Considerations

**Very low volume (a handful of active programs per institute at any time).** The `CHECK (slots_filled <= slots)` constraint and its atomic `UPDATE ... WHERE slots_filled < slots` write pattern are the genuine performance/correctness concern here, not row count — monitor for contention specifically during a popular program's selection window, when many award attempts may race for the same limited slot pool concurrently.