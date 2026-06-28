# 12 — HR Phase

**Phase type:** Feature (vertical slice) · **Release:** R3 · **Maps to:** Blueprint Part 13

Staff records — the people side of the institution. A lighter domain that reuses config and workflow heavily and provides the assignment data that drives teacher ownership.

## Objectives
- Manage staff, assign teaching duties (the source of ownership truth for Attendance/Exam), and process leave via workflow.
- Lay basic payroll configuration groundwork (full payroll deferred).

## Scope
**In:** staff profiles (core + dynamic fields); staff roles/assignments (teacher→subject/section); leave management (via **workflow**); basic payroll configuration; staff documents.
**Out:** full payroll run/calculation (later in R3); biometric attendance for staff (R5).

## Deliverables
- Staff managed, assigned to teaching duties, and requesting leave through approval.
- Assignment model feeding ownership in Attendance/Exam.

## Dependencies
Core Platform (03) — users/roles. Workflow (05) — leave. Configuration (04) — dynamic fields. Academic (06) — assignment targets.

## Risks
- **Assignment gaps weaken access control (ownership).** *Mitigation:* treat assignment as the source of ownership truth; test it.
- **Payroll scope creep.** *Mitigation:* defer full payroll; deliver config groundwork only.

## Acceptance Criteria
- A teacher's assignments correctly drive their attendance/exam ownership.
- Leave routes through workflow with audit.
- Staff custom fields work via config.

## Exit Criteria
- Staff + assignment + leave demoed; assignment→ownership link verified against Attendance/Exam.
- Payroll groundwork documented for its later R3 build-out.
- **Reporting Phase unblocked** (staff data available).
