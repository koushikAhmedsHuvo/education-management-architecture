# Transcripts

## 1. Table Purpose

**This is not a separate table.** Per the already-approved API Contract (`wave-6-examination/transcript.api.md`), a transcript is an **immutable, reproducible generated document** — a cumulative, multi-session aggregation of a student's already-published `results` rows, reproducing (never recomputing) the provenance each constituent result already carries. The approved API contract's own Module Context is explicit on this point: *"the transcript never recomputes a result, it reproduces what Result Processing already finalized."* A transcript therefore has no independent business state of its own beyond the generated artifact itself, and is realized as a row in the shared, polymorphic `documents` table (introduced in Wave 5, `document_type = 'TRANSCRIPT'`, `entity_type = 'student'`, `immutable = true`) — see [`../wave-5-student-lifecycle/documents.schema.md`](../wave-5-student-lifecycle/documents.schema.md).

This reuses, rather than duplicates, the exact same mechanism already used for admit cards (`document_type = 'ADMIT_CARD'`) and single-exam marksheets (`document_type = 'MARKSHEET'`) — all three are generated, immutable documents aggregating data from elsewhere in the schema (`exam_eligibility`, `results`, and `results` again respectively), differing only in *which* source rows they aggregate and *how broadly* (one exam vs. cumulative across many). Introducing a dedicated `transcripts` table here would duplicate `documents`' storage-key, versioning, and immutability columns for no benefit, and would fragment "generated student documents" across two tables for a distinction (single-exam vs. cumulative) that is a *generation-time* concern, not a *storage-shape* one.

**Where this data actually lives:** the `documents` table (Wave 5) stores the generated artifact's metadata (`storage_key`, `version`, `immutable`). The *content* of a transcript — which `results` rows it aggregates, and their carried-forward provenance (`rule_ver`/`config_ver`/`grade_ver`) — is computed at generation time directly from [`results.schema.md`](./results.schema.md) and is not separately persisted; regenerating a transcript for the same student/session set is guaranteed to be byte-identical precisely because every input (`results`) is itself immutable, append-only data.

## 2–9. (Not applicable)

No columns, keys, constraints, indexes, relationships, soft-delete strategy, or audit fields exist for this file, since it does not back a physical table. See `documents.schema.md` (Wave 5) for the actual storage schema, and `results.schema.md` for the data a transcript aggregates.

## 10. Multi-Institute Isolation Strategy

Not applicable as an independent concern — covered transitively by `documents.schema.md` §10 (polymorphic, resolved via the referenced `entity_id`) and `results.schema.md` §10 (derived via `student_id → students → institute_id`), since a transcript's isolation is simply the isolation of the student and results it aggregates.

## 11. Notes for TypeORM Entity Design

Do not create a `Transcript` entity. Transcript generation is a service-layer operation (`TranscriptGenerationService`) that queries `results` for a student's eligible (published, non-withheld) sessions, renders the cumulative document, and writes the resulting file's metadata as a single `Document` entity row with `documentType: 'TRANSCRIPT'`. See `transcript.api.md` #4–7 in the approved API Contract for the exact generation, download, and reprint operation contracts this service implements.

## 12. Performance Considerations

Not applicable as an independent concern. Transcript *generation* is read-heavy against `results` (one query per student aggregating potentially many sessions' worth of rows) but write-light against `documents` (a single new row per generation); see `documents.schema.md` §12 and `results.schema.md` §12 for the performance characteristics of the tables a transcript actually touches. No independent indexing strategy is needed for "transcripts" as a concept, since no table exists to index.
