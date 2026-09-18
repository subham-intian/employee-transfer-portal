# Project Status Board

_Last updated: 2026-09-18_

## Project
employee-internal-transfer (Backend Only — Java Spring Boot / PostgreSQL + Hibernate / JWT / Docker / Modular Monolith)

## BRD Status
Ingested from the source `.docx` document under `docs/` on 2026-09-18. **Approved (Gate 0)** by Subham Bhattacharyya (subham.bhattacharyya@intglobal.com) on 2026-09-18 — see `.ai-context/BRD.md` and `.ai-context/pr_reviews/BRD-20260918-013000.md`.

## Active Specs

| Spec ID | Title | Status | Owner | Last Updated | Notes |
|---|---|---|---|---|---|
| internal-transfer-request | Employee Internal Transfer Request | In Peer Review | Subham Bhattacharyya | 2026-09-18 | Awaiting Gate 1 review by soumyadeep.adhikary@intglobal.com. See `.ai-context/specs/internal-transfer-request.spec.md`. |

## Daily Execution Log

### 2026-09-18
- **internal-transfer-request**: Authored from approved BRD (BRD-001–BRD-009) plus a folded-in Identity & Access technical prerequisite (TR-001–TR-003, login/JWT claims) since no login/User model exists yet. Status set to `In Peer Review`. Submitted for Gate 1.
- **internal-transfer-request**: Pre-Gate-1 gap review surfaced 8 ambiguities; closed 5 of them via stakeholder Q&A (kept 3 — migration tool, seed data specifics, cross-module access — correctly deferred to post-Gate-1 Plan phase). Added a Per-Stage Authorization Model (Manager/HR are 1:1 assigned individuals; Accounts/IT/Facilities are role-based/shared-team), renamed "Payroll" to "Accounts" (spec/implementation-level only, BRD.md left as-is), added 3 catalog endpoints (API06–API08) and 4 new ACs/6 new test cases (concurrency block, past-effective-date validation, required reject comment). Still `In Peer Review`, still awaiting Gate 1 from soumyadeep.adhikary@intglobal.com.

## Open Incidents
_(none)_

## Pending Hotfixes
_(none)_

## Releases
_(none)_
