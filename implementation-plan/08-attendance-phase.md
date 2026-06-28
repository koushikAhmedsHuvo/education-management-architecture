# 08 — Attendance Phase

**Phase type:** Feature (vertical slice) · **Release:** R2 · **Maps to:** Blueprint Part 9

Daily operational data capture at scale — a high-volume, partitioned table and the first real test of write-volume handling.

## Objectives
- Let teachers mark daily/period attendance for their assigned sections, fast and at scale.
- Establish the partitioning + batched-write pattern for high-volume tables.

## Scope
**In:** attendance configuration (modes, statuses — config-driven); marking (section/period); bulk/fast entry; corrections (audited); attendance summaries (read-model seed).
**Out:** attendance-based notifications (R4); biometric/device integrations (R5).

## Deliverables
- Teachers mark attendance for their sections; summaries are queryable.
- `attendance` table range-partitioned by date (D16).

## Dependencies
Academic (06) — sections. Student Lifecycle (07) — enrolled students. Configuration (04) — modes/statuses.

## Risks
- **Write-volume hotspots on the primary during peak marking.** *Mitigation:* batching + partitioning.
- **N+1 when loading rosters.** *Mitigation:* explicit batched loads + query-count tests (D71).

## Acceptance Criteria
- Marking a 60-student section is one batched operation (query-count asserted).
- A teacher can mark **only** their assigned sections (ownership enforced).
- Corrections are audited.
- Shared Definition of Done met.

## Exit Criteria
- Attendance demoed for a school section; query-count and ownership tests green.
- Summary read-model documented for the Reporting phase.
- **Examination Phase unblocked.**
