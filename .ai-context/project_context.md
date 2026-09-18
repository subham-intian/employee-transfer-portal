# Project Context

## Baseline Parameters

| Parameter | Value |
|---|---|
| Project Name | employee-internal-transfer |
| Project Type | Backend Only |
| Architecture Style | Modular Monolith (Microservice Ready) |
| Frontend Technology & Styling | N/A (Backend Only) |
| Backend Technology & Framework | Java Spring Boot 4.1.1 |
| Build Tool | Maven (with Maven Wrapper, pinned to Maven 3.9.16) |
| Java Version | 25 (LTS) |
| Maven groupId / artifactId | `com.intglobal.eit` / `employee-internal-transfer` |
| Database & Data Access / ORM Layer | PostgreSQL + Hibernate ORM |
| Authentication & Security Strategy | JWT (self-issued, via jjwt 0.13.0 — not an OAuth2/external IdP resource server) |
| Deployment Target | Docker |

## Reviewer Assignments

| Gate | Reviewer(s) |
|---|---|
| Gate 0 Reviewer(s) — BRD Review | subham.bhattacharyya@intglobal.com |
| Gate 1 Reviewer(s) — Spec Peer Review | soumyadeep.adhikary@intglobal.com |
| Gate 2 Reviewer(s) — Code Review | TBD — assign before first Gate 2 review |

## BRD Status
Ingested and Gate 0-approved on 2026-09-18 — see `.ai-context/BRD.md` and `.ai-context/pr_reviews/BRD-20260918-013000.md`. Next: run `int-sdd-lifecycle` to draft the first feature spec (Employee Internal Transfer).

## Notes
- Maven build and Spring Boot bootstrap are complete: `src/backend/pom.xml`, the app entrypoint, shared JWT security infrastructure, and global exception handling exist. Verified locally: `./mvnw compile` and `./mvnw test` (1/1 tests passing against an in-memory H2 override) both succeed.
- No business modules exist yet (`src/backend/src/main/java/com/intglobal/eit/modules/` is an empty package pending the first spec).
- Schema migration tool (Flyway vs Liquibase) remains an open decision — see `.ai-context/architecture.md`.
