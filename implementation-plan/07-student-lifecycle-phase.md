# 07 — Student Lifecycle Phase

**Phase type:** Feature (vertical slice) · **Release:** R2 · **Maps to:** Blueprint Part 8

Getting students into the system — the first feature a client sees as "the ERP," and the first consumer of the workflow engine (admission approval).

## Objectives
- Deliver admission (configurable form + documents + workflow approval), enrollment, guardians, profiles, and transfers.
- Harden the workflow and file engines through their first real consumer.

## Scope
**In:** admission (dynamic form, document upload, workflow approval, admission-fee hook); enrollment at the leaf; guardian/parent records + links; student profiles (core + dynamic custom fields); transfers (section/campus/institute, scoped + audited).
**Out:** parent/student portals (R4, phase 15); online admission-fee payment (R4).

## Deliverables
- A school admits a student through a configurable application with documents and an approval workflow, then enrolls them for the session.
- Build/DB/API/UI executed per blueprint Part 8.

## Dependencies
Academic (06) — enrollment leaf. Workflow (05) — approval. Configuration (04) — dynamic forms/fields. Foundation (02) — file service.

## Risks
- **First real workflow/file consumer surfaces engine gaps.** *Mitigation:* treat as the engines' hardening sprint.
- **Document-heavy uploads stress the file service.** *Mitigation:* signed-URL + scan path (D46).

## Acceptance Criteria
- An admission moves submit → review → approve with full audit and a version-pinned workflow.
- An enrolled student appears at the correct enrollment leaf.
- A guardian linked to a student is the basis for later parent access.
- Cross-institute isolation holds for all student data.

## Exit Criteria
- End-to-end admit→enroll demoed for a school; workflow + file engines confirmed hardened.
- Guardian-link model documented for the Portals phase.
- **Attendance Phase unblocked.**
