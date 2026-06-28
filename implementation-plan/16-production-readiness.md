# 16 — Production Readiness

**Phase type:** Gate (not a feature) · **Release:** before each client-facing release · **Maps to:** Blueprint Part 17

Prove the system is safe, fast, and recoverable **before** a real client's data is on it. A mandatory release gate with named owners.

## Objectives
- Pass security, performance, load, backup, and disaster-recovery reviews against defined bars before go-live.
- Surface the single-primary write ceiling and any access-control gaps before production, not after.

## Scope
**In:** security review (authz/scope/ownership suite + SAST/DAST/dep/secret scans + manual access-control & SSRF review); performance review (four peak events vs budgets); load testing (large-tier peak, e.g. 3,000 concurrent on result-publish); backup testing (PITR take + restore + integrity); DR testing (failover drill + cross-region restore).
**Out:** new feature work; per-client tuning (happens at deployment).

## Deliverables
- Signed-off security, performance, backup, and DR reports; a load-test result against the peak profile.

## Dependencies
All feature modules in the target release complete.

## Risks
- **"We'll harden later" skips the gate.** *Mitigation:* mandatory gate with named owners and ADR sign-off.
- **Load tests reveal the write ceiling late.** *Mitigation:* load-test result-publish as early as the Result phase (10), not first here.

## Acceptance Criteria
- Zero high/critical security findings open; isolation suite green.
- Each peak event meets its latency/throughput budget under target concurrency.
- Interactive latency holds while bulk jobs run; no primary write collapse.
- A PITR restore succeeds within the tier's RTO; DR failover meets RPO/RTO.

## Exit Criteria
- Every review passes its bar; sign-off recorded as an ADR.
- Open findings triaged with owners and timelines (none high/critical outstanding).
- **Client Deployment Phase unblocked** for the target release.
