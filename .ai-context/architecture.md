# Architecture

## Default Architecture: Modular Monolith (Microservice Ready)

This project runs as a single deployable Spring Boot application in local development and initial production, while being structured so that any business module can later be extracted into an independent service without a rewrite.

Rules:
- The application runs as one deployable unit; do not introduce microservice deployment complexity (service discovery, distributed transactions, inter-service messaging infra) unless explicitly required later.
- Business modules must have clear boundaries and expose their functionality through well-defined interfaces (service classes / package boundaries), not through direct cross-module data access.
- Modules must minimize direct coupling with other modules — no reaching into another module's internal classes or repositories directly.
- Shared, cross-cutting functionality (logging, error handling, security/JWT filters, common utilities, database configuration) lives in isolated shared/infrastructure packages, not duplicated per module.
- A module should be structured so that, if extracted, its persistence (PostgreSQL schema/tables it owns) and API surface can move with it cleanly.

## Backend Structure Baseline (Java / Spring Boot)

```text
src/backend/
├── pom.xml                # Maven, spring-boot-starter-parent 4.1.1, Java 25
├── mvnw, mvnw.cmd, .mvn/   # Maven Wrapper (pinned to Maven 3.9.16)
├── Dockerfile, .dockerignore
└── src/main/
    ├── java/com/intglobal/eit/
    │   ├── EmployeeInternalTransferApplication.java
    │   ├── config/            # SecurityConfig (JWT filter chain, PasswordEncoder)
    │   ├── shared/
    │   │   ├── security/      # JwtService, JwtAuthenticationFilter
    │   │   └── exception/     # GlobalExceptionHandler, ApiError
    │   └── modules/           # One package per business module — empty until the first spec
    └── resources/application.yml

tests/backend/
└── src/test/
    ├── java/com/intglobal/eit/EmployeeInternalTransferApplicationTests.java
    └── resources/application.yml   # H2 override for offline context-load test
```

`pom.xml`'s `<testSourceDirectory>` points at `../../tests/backend/src/test/java` (relative to `src/backend/`) so Maven finds tests under the top-level `tests/backend/` tree per the INT standard, while the Maven module root stays `src/backend/`.

## Data & Persistence

- Database: PostgreSQL
- Data access: Hibernate ORM (JPA)
- Schema/migration strategy: to be defined per module spec (e.g. Flyway/Liquibase) — not yet decided.

## Authentication & Security

- Strategy: JWT (stateless, self-issued via `io.jsonwebtoken` 0.13.0 — not an OAuth2 resource server against an external IdP). `shared/security/JwtService` issues/validates HMAC-signed tokens; `shared/security/JwtAuthenticationFilter` populates the security context per request. `config/SecurityConfig` wires the stateless filter chain. Secret and expiry are environment-backed (`JWT_SECRET`, `JWT_EXPIRATION_MS`) — no business login/registration endpoint exists yet since that requires a User entity from a future spec.

## Deployment

- Target: Docker. `src/backend/Dockerfile` is a multi-stage build (`eclipse-temurin:25-jdk-alpine` → `eclipse-temurin:25-jre-alpine`), verified to build successfully. `docker-compose.yml` for local Postgres has not been added yet.

## Open Architectural Decisions

- Schema migration tool (Flyway vs Liquibase) — deferred until the first module requiring persistence is specced.

Record any architecture decision that changes the above via an ADR in `.ai-context/decisions/ADR-NNN.md`.
