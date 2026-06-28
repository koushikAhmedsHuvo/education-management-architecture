# 19 — Sprint Plan

**Document type:** Planning (2-week sprints) · **Maps to:** Blueprint Part 20

Releases 1–2 planned sprint-by-sprint (the path to first client, where precision is achievable). Releases 3–5 are epic-level — sprint-planning 12+ months out is false precision and is decomposed at each release start.

## Objectives
- Provide an executable sprint sequence to the first paying client.
- Make dependencies between sprints explicit so parallel BE/FE work is coordinated.

## Scope
**In:** sprint-by-sprint for R1–R2 (Sprints 0–18); epic-level backlog for R3–R5.
**Out:** detailed R3–R5 sprints (decomposed later).

## Deliverables
| Sprint | Goal | Key Deliverable | Depends on |
|---|---|---|---|
| 0 | Rails ready | Stack runs; CI gates; staging stub | — |
| 1 | Skeleton walks | Walking skeleton on staging | S0 |
| 2 | Reliability spine | Audited write proven at-least-once | S1 |
| 3 | Identity | Login + refresh + sessions | S2 |
| 4 | Access control | **Cross-institute isolation proven** | S3 |
| 5 | Org backbone | Multi-institute org mgmt | S4 |
| 6 | Config engine I | Effective-value resolution | S5 |
| 7 | Config engine II | Add-a-field-no-code; rollback | S6 |
| 8 | Workflow engine I | A multi-step approval runs | S7 |
| 9 | Workflow engine II + wizard | Full approval engine + setup wizard | S8 |
| 10 | Academic structure | School structure for a session | S5,S7 |
| 11 | Admission | Student admitted via approval | S9,S10 |
| 12 | Enrollment + guardians | Students enrolled; guardians linked | S11 |
| 13 | Attendance | Teachers mark attendance (scoped) | S12 |
| 14 | Examination | Marks entered and locked | S12 |
| 15 | Results | Results published; marksheets | S14 |
| 16 | Fees | School billing end-to-end | S12,S9 |
| 17 | Hardening + readiness | Release-2 gate passed | S13–S16 |
| 18 | First client onboarding | **First paying client live** | S17 |

*With ~8 engineers, Sprints 10–16 parallelize across BE/FE via contract-first; the table shows dependency order, not strictly serial execution.*

**R3–R5 epics:** HR/payroll/leave · Reporting framework + datasets + builder · institution-type templates · routine — then Notifications · Portals · payment gateway · certificates · MFA · compliance — then Public API · webhooks · mobile · SSO · analytics · LMS · library/transport/hostel · AI.

## Dependencies
Sprint order encodes engine-before-consumer and the academic chain.

## Risks
- **Foundation sprints (1–9) overrun.** *Mitigation:* thin walking skeleton; timebox.
- **False precision in far sprints.** *Mitigation:* epic-level beyond R2; decompose at release start.

## Acceptance Criteria
- Each sprint ends with a demoable increment meeting the shared Definition of Done.

## Exit Criteria
- Sprint 18 closes with a live paying client; R3 sprints decomposed before R3 begins.
