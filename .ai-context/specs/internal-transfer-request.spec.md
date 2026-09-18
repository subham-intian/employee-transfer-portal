# Spec: Employee Internal Transfer Request

## Spec ID
internal-transfer-request

## Status
In Peer Review

## Roles & Assignments
- **Developer:** Subham Bhattacharyya (subham.bhattacharyya@intglobal.com)
- **Gate 1 Reviewer(s):** Soumyadeep Adhikary (soumyadeep.adhikary@intglobal.com)
- **Gate 2 Reviewer(s):** TBD — assign before first Gate 2 review

## Linked BRD
.ai-context/BRD.md#BRD-001, #BRD-002, #BRD-003, #BRD-004, #BRD-005, #BRD-006, #BRD-007, #BRD-008, #BRD-009

## Gate Approvals & History
| Gate | Approver Name | Approver Email/ID | Date/Time | Outcome | Approval Comment / Summary |
|---|---|---|---|---|---|
| Gate 1 (Spec Review) | Soumyadeep Adhikary | soumyadeep.adhikary@intglobal.com | — | Pending | — |
| Gate 2 (Code Review) | _(pending)_ | TBD | — | Pending | — |

## Intent
Allow an employee to initiate an Internal Transfer Request from the One-Point Employee Portal by proposing a new department/business unit, location, and/or role, and have the portal orchestrate the resulting Manager → HR → Accounts → IT → Facilities workflow, giving the employee a single, always-current view of the request's status and full decision history.

## Context
- Builds on: `.ai-context/architecture.md` (Modular Monolith backend structure) — introduces the first two business modules in this codebase: `modules/identity` (employee identity, login, roles) and `modules/transfer` (the transfer request workflow itself), kept as separate modules per the module-boundary rule in `architecture.md` even though both ship in this one spec.
- Related: none (first feature spec in this repository).
- API contract: internal only (no external system integration in this phase — see A-002 in the BRD).
- Business Rules and Assumptions this spec implements: BRD.md's BR-001 through BR-009, A-001, A-002, NFR-001.
- **Naming note:** the BRD's "Payroll" actor is called **Accounts** throughout this spec and its implementation (same stakeholder — a naming preference decided during spec authoring). `.ai-context/BRD.md` itself is left unchanged, since it is already Gate 0-approved; this is a label mapping, not a scope change.

### Identity & Access — technical prerequisite (not a BRD-numbered requirement)
The BRD's business intent (BRD-001–BRD-009) presupposes knowing who the employee is, who their manager and HR representative are, and which access role(s) a caller holds (Accounts/IT/Facilities). No BRD requirement covers login or identity — `architecture.md` already committed this project to JWT authentication during project setup but deferred the actual login endpoint and User/Employee entity to "a future spec." Per the developer's decision, that prerequisite is folded into this spec rather than split into a separate one, scoped to the minimum needed to unblock BRD-001–BRD-009:

| ID | Requirement |
|---|---|
| TR-001 | Employee identity is represented as data: department, location, job role, `managerId` (the employee's specifically assigned Manager — a relational reference to another Employee), `assignedHrId` (the employee's specifically assigned HR representative — a relational reference to another Employee), and `accessRoles` (a set drawn from `ACCOUNTS`, `IT`, `FACILITIES`, denoting shared-service team membership). Provisioned via seed/migration data for this phase — there is no self-service registration or employee-management API in scope (see Explicitly Out of Scope). |
| TR-002 | An employee authenticates with email + password against a login endpoint and receives a JWT access token (reusing the existing `JwtService`) whose claims include `employeeId` and the employee's `accessRoles`. |
| TR-003 | `JwtAuthenticationFilter` / `SecurityConfig` populate the request's security context from those JWT claims so API01–API05 can authorize by request ownership, by relational assignment (`managerId`/`assignedHrId`, looked up against the target request — not derivable from the JWT claims alone), and by shared-service role membership (`accessRoles`). |

### Per-Stage Authorization Model
Manager and HR are specific individuals assigned to the employee; Accounts, IT, and Facilities are shared-service teams where any member holding that access role may act. This distinction governs both who may *decide* at a stage (API04) and who may *view* the request (API02):

| Stage | Who may decide / view | Mechanism |
|---|---|---|
| `MANAGER_REVIEW` | The employee's assigned Manager | `caller.employeeId == request.employee.managerId` |
| `HR_REVIEW` | The employee's assigned HR representative | `caller.employeeId == request.employee.assignedHrId` |
| `ACCOUNTS` | Any employee holding `accessRoles` containing `ACCOUNTS` | `ACCOUNTS ∈ caller.accessRoles` |
| `IT` | Any employee holding `accessRoles` containing `IT` | `IT ∈ caller.accessRoles` |
| `FACILITIES` | Any employee holding `accessRoles` containing `FACILITIES` | `FACILITIES ∈ caller.accessRoles` |

For visibility (API02), a stakeholder retains access to a request once it has reached (or passed) their stage — the Manager/HR check is evaluated via the relational fields above (a fixed individual, regardless of later stage changes), while Accounts/IT/Facilities visibility is granted to any current role-holder once the request has reached or passed their respective stage.

## API Contract

### internal-transfer-request.API00 — POST /api/v1/auth/login
An employee logs in with email + password and receives a JWT access token. (TR-002)

**Request payload:**
```json
{
  "email": "string",
  "password": "string"
}
```

**Success response (200):**
```json
{
  "accessToken": "string",
  "expiresAt": "datetime"
}
```

**Exceptions:**
| Code | Condition | Response body |
|---|---|---|
| 401 | Email not found, or password does not match | `{ "error": "INVALID_CREDENTIALS" }` |

### internal-transfer-request.API01 — POST /api/v1/transfer-requests
Employee submits a new transfer request. Valid `departmentId`/`locationId`/`roleId` values can be discovered via API06–API08. (BRD-001–BRD-006)

**Request payload:**
```json
{
  "departmentId": "string",
  "locationId": "string",
  "roleId": "string",
  "effectiveDate": "date (ISO-8601)",
  "reason": "string (optional)"
}
```

**Success response (201):**
```json
{
  "id": "string",
  "status": "SUBMITTED",
  "departmentId": "string",
  "locationId": "string",
  "roleId": "string",
  "effectiveDate": "date",
  "reason": "string | null",
  "submittedAt": "datetime",
  "employeeId": "string"
}
```

**Exceptions:**
| Code | Condition | Response body |
|---|---|---|
| 400 | Missing/invalid `departmentId`, `locationId`, `roleId`, or `effectiveDate` (reason is optional and may be absent); or `effectiveDate` is earlier than today | `{ "error": "VALIDATION_ERROR", "fields": [...] }` |
| 401 | No authenticated employee | `{ "error": "UNAUTHENTICATED" }` |
| 409 | Caller already has another transfer request in a non-terminal state (`SUBMITTED`, `MANAGER_REVIEW`, `HR_REVIEW`, `ACCOUNTS`, `IT`, or `FACILITIES`) | `{ "error": "ACTIVE_REQUEST_EXISTS" }` |

### internal-transfer-request.API02 — GET /api/v1/transfer-requests/{id}
View a single request's current status, current pending stakeholder, and full upstream decision history. (BRD-007, BRD-008, BR-009)

**Success response (200):**
```json
{
  "id": "string",
  "status": "SUBMITTED | MANAGER_REVIEW | HR_REVIEW | ACCOUNTS | IT | FACILITIES | COMPLETED | REJECTED | CANCELLED",
  "pendingStakeholder": { "stage": "MANAGER | HR | ACCOUNTS | IT | FACILITIES | null", "assignedTo": "employeeId | null" },
  "departmentId": "string",
  "locationId": "string",
  "roleId": "string",
  "effectiveDate": "date",
  "reason": "string | null",
  "submittedAt": "datetime",
  "history": [
    { "stage": "MANAGER", "actorId": "string", "decision": "APPROVE | REJECT", "comment": "string | null", "decidedAt": "datetime" }
  ]
}
```
`pendingStakeholder.assignedTo` is populated with a specific `employeeId` when the pending stage is `MANAGER` or `HR` (a specific assigned individual), and is `null` for `ACCOUNTS`/`IT`/`FACILITIES` (a shared-team queue, not one fixed individual) or when the request is in a terminal state.

**Exceptions:**
| Code | Condition | Response body |
|---|---|---|
| 403 | Caller is neither the owning employee nor a stakeholder authorized per the Per-Stage Authorization Model above (currently or previously) | `{ "error": "FORBIDDEN" }` |
| 404 | No request with that id | `{ "error": "NOT_FOUND" }` |

### internal-transfer-request.API03 — GET /api/v1/transfer-requests
List the authenticated employee's own transfer requests. (BRD-007)

**Success response (200):**
```json
[
  { "id": "string", "status": "string", "departmentId": "string", "locationId": "string", "roleId": "string", "effectiveDate": "date", "submittedAt": "datetime" }
]
```

**Exceptions:**
| Code | Condition | Response body |
|---|---|---|
| 401 | No authenticated employee | `{ "error": "UNAUTHENTICATED" }` |

### internal-transfer-request.API04 — POST /api/v1/transfer-requests/{id}/decisions
A stakeholder (Manager, HR, Accounts, IT, or Facilities) records their decision on a request. The server resolves which stage is being decided from the request's current status, then authorizes the caller per the Per-Stage Authorization Model above. (BR-001–BR-005, A-002)

**Request payload:**
```json
{
  "decision": "APPROVE | REJECT",
  "comment": "string (optional; required when decision is REJECT)"
}
```

**Success response (200):** same shape as API02's response, reflecting the new status.

**Exceptions:**
| Code | Condition | Response body |
|---|---|---|
| 400 | `decision` missing or not one of `APPROVE`/`REJECT`; or `decision` is `REJECT` and `comment` is missing/empty | `{ "error": "VALIDATION_ERROR" }` |
| 403 | Caller is not authorized for the request's current stage per the Per-Stage Authorization Model | `{ "error": "FORBIDDEN" }` |
| 404 | No request with that id | `{ "error": "NOT_FOUND" }` |
| 409 | Request is already in a terminal state (`COMPLETED`, `REJECTED`, `CANCELLED`) | `{ "error": "INVALID_STATE" }` |

### internal-transfer-request.API05 — POST /api/v1/transfer-requests/{id}/cancel
The owning employee cancels their own request, only while it has not yet begun execution. (BR-006)

**Success response (200):** same shape as API02's response, with `status: "CANCELLED"`.

**Exceptions:**
| Code | Condition | Response body |
|---|---|---|
| 403 | Caller is not the owning employee | `{ "error": "FORBIDDEN" }` |
| 404 | No request with that id | `{ "error": "NOT_FOUND" }` |
| 409 | Request status is `ACCOUNTS`, `IT`, `FACILITIES`, `COMPLETED`, `REJECTED`, or already `CANCELLED` | `{ "error": "INVALID_STATE" }` |

### internal-transfer-request.API06 — GET /api/v1/departments
List valid department/business-unit catalog values. (A-001)

**Success response (200):**
```json
[ { "id": "string", "name": "string" } ]
```

**Exceptions:**
| Code | Condition | Response body |
|---|---|---|
| 401 | No authenticated employee | `{ "error": "UNAUTHENTICATED" }` |

### internal-transfer-request.API07 — GET /api/v1/locations
List valid location catalog values. Same shape and exceptions as API06. (A-001)

### internal-transfer-request.API08 — GET /api/v1/roles
List valid role/job-position catalog values. Same shape and exceptions as API06. (A-001)

## Acceptance Criteria

1. `internal-transfer-request.AC1` — Given an authenticated employee provides `departmentId`, `locationId`, `roleId`, and `effectiveDate` (with or without `reason`), when they submit via API01, then a request is created with status `SUBMITTED` and returned to them. (BRD-001–BRD-006)
2. `internal-transfer-request.AC2` — Given an employee omits `departmentId`, `locationId`, `roleId`, or `effectiveDate`, when they attempt to submit via API01, then the request is rejected with 400 and no record is created. (BRD-004)
3. `internal-transfer-request.AC3` — Given a previously submitted request, when the owning employee requests it via API02, then the current status and the full upstream stakeholder decision history are returned. (BRD-007)
4. `internal-transfer-request.AC4` — Given a previously submitted request that is not yet in a terminal state, when the owning employee requests it via API02, then the currently pending stage is included, with the specific assigned individual when the stage is `MANAGER` or `HR`. (BRD-008)
5. `internal-transfer-request.AC5` — Given a request in `MANAGER_REVIEW`, when the employee's assigned Manager approves it via API04, then the request transitions to `HR_REVIEW`; when that Manager rejects it (with a comment), then the request transitions to `REJECTED`. (BR-002, BR-003)
6. `internal-transfer-request.AC6` — Given a request in `HR_REVIEW`, when the employee's assigned HR representative approves it via API04, then the request transitions to `ACCOUNTS`; when they reject it (with a comment), then the request transitions to `REJECTED`. (BR-001, BR-004)
7. `internal-transfer-request.AC7` — Given a request in `ACCOUNTS`, `IT`, or `FACILITIES`, when a caller holding the matching `accessRole` approves it via API04, then the request advances to the next stage in the sequence Accounts → IT → Facilities → `COMPLETED`; when they reject it (with a comment), then the request transitions to `REJECTED`. (BR-004, BR-005, A-002)
8. `internal-transfer-request.AC8` — Given a request that has one or more recorded stakeholder decisions, when a stakeholder authorized per the Per-Stage Authorization Model requests it via API02, then the response includes the full upstream decision history with actor, decision, and comment. (BR-009)
9. `internal-transfer-request.AC9` — Given a user who is neither the owning employee nor a stakeholder authorized per the Per-Stage Authorization Model (currently or previously), when they request the request via API02, then access is denied with 403. (BR-009)
10. `internal-transfer-request.AC10` — Given a request in `SUBMITTED`, `MANAGER_REVIEW`, or `HR_REVIEW`, when the owning employee cancels it via API05, then the request transitions to `CANCELLED`. (BR-006)
11. `internal-transfer-request.AC11` — Given a request in `ACCOUNTS`, `IT`, or `FACILITIES`, when the owning employee attempts to cancel it via API05, then the cancellation is rejected with 409 and the request's status is unchanged. (BR-006)
12. `internal-transfer-request.AC12` — Given a caller who is not authorized for the request's current stage per the Per-Stage Authorization Model attempts a decision via API04, then it is rejected with 403 and the request's status is unchanged. (BR-002–BR-004)
13. `internal-transfer-request.AC13` — Given a provisioned employee with valid credentials, when they POST to API00, then they receive a JWT access token whose claims include their `employeeId` and `accessRoles`. (TR-002)
14. `internal-transfer-request.AC14` — Given invalid credentials (unknown email or wrong password), when posting to API00, then 401 is returned and no token is issued. (TR-002)
15. `internal-transfer-request.AC15` — Given an employee already has a transfer request in a non-terminal state, when they attempt to submit another via API01, then the submission is rejected with 409 and no new record is created. (Concurrency decision, 2026-09-18)
16. `internal-transfer-request.AC16` — Given an employee provides an `effectiveDate` earlier than today, when they attempt to submit via API01, then the request is rejected with 400. (Effective-date decision, 2026-09-18)
17. `internal-transfer-request.AC17` — Given a stakeholder submits `decision: REJECT` without a `comment` via API04, then the request is rejected with 400 and the underlying transfer request's status is unchanged. (Reject-comment decision, 2026-09-18)
18. `internal-transfer-request.AC18` — Given an authenticated employee requests API06, API07, or API08, then the full catalog list for that resource (`id`, `name` pairs) is returned. (A-001; catalog-endpoint decision, 2026-09-18)

## Unit Test Cases (spec-derived)

| Test ID | Maps to AC | Scenario | Expected |
|---|---|---|---|
| internal-transfer-request.UT01 | AC1 | Valid submission with all required fields, no reason | 201, status `SUBMITTED` |
| internal-transfer-request.UT02 | AC1 | Valid submission with optional `reason` provided | 201, `reason` echoed back |
| internal-transfer-request.UT03 | AC2 | Submission missing `departmentId` | 400, `VALIDATION_ERROR` |
| internal-transfer-request.UT04 | AC2 | Submission missing `effectiveDate` | 400, `VALIDATION_ERROR` |
| internal-transfer-request.UT05 | AC3 | Owner fetches a request with prior decisions | 200, history array populated |
| internal-transfer-request.UT06 | AC4 | Owner fetches a request currently in `HR_REVIEW` | 200, `pendingStakeholder: { "stage": "HR", "assignedTo": "<hrEmployeeId>" }` |
| internal-transfer-request.UT07 | AC5 | Assigned Manager approves a `MANAGER_REVIEW` request | 200, status `HR_REVIEW` |
| internal-transfer-request.UT08 | AC5 | Assigned Manager rejects a `MANAGER_REVIEW` request with a comment | 200, status `REJECTED` |
| internal-transfer-request.UT09 | AC6 | Assigned HR rep approves an `HR_REVIEW` request | 200, status `ACCOUNTS` |
| internal-transfer-request.UT10 | AC6 | Assigned HR rep rejects an `HR_REVIEW` request with a comment | 200, status `REJECTED` |
| internal-transfer-request.UT11 | AC7 | Accounts-role holder approves an `ACCOUNTS` request | 200, status `IT` |
| internal-transfer-request.UT12 | AC7 | Facilities-role holder approves a `FACILITIES` request | 200, status `COMPLETED` |
| internal-transfer-request.UT13 | AC8 | HR rep fetches a request after Manager's decision | 200, history includes Manager's decision + comment |
| internal-transfer-request.UT14 | AC9 | Unrelated employee fetches another employee's request | 403, `FORBIDDEN` |
| internal-transfer-request.UT15 | AC10 | Owner cancels a `SUBMITTED` request | 200, status `CANCELLED` |
| internal-transfer-request.UT16 | AC11 | Owner attempts to cancel an `ACCOUNTS`-stage request | 409, `INVALID_STATE` |
| internal-transfer-request.UT17 | AC12 | Accounts-role holder attempts a decision while request is still in `MANAGER_REVIEW` | 403, `FORBIDDEN` |
| internal-transfer-request.UT18 | AC13 | Login with valid email/password | 200, token issued with `employeeId`/`accessRoles` claims |
| internal-transfer-request.UT19 | AC14 | Login with wrong password | 401, `INVALID_CREDENTIALS` |
| internal-transfer-request.UT20 | AC15 | Employee with an existing `SUBMITTED` request submits another | 409, `ACTIVE_REQUEST_EXISTS` |
| internal-transfer-request.UT21 | AC16 | Submission with `effectiveDate` set to yesterday | 400, `VALIDATION_ERROR` |
| internal-transfer-request.UT22 | AC17 | Assigned Manager rejects without a `comment` | 400, `VALIDATION_ERROR` |
| internal-transfer-request.UT23 | AC18 | Fetch each of API06/API07/API08 | 200, seeded catalog entries returned |

## Explicitly Out of Scope
- Real-time/system integration with external Accounts, IT, or Facilities systems — these steps are manual/tracked-only within the portal (A-002).
- Reassignment of the employee's reporting manager as part of this journey (BR-008).
- Automatic escalation of stale pending actions (NFR-001 covers only a reminder, no escalation).
- The reminder notification itself (NFR-001) — its exact SLA duration is an unresolved BRD Open Question; implementing the reminder is deferred to a follow-up increment/spec once that duration is decided.
- Self-service employee registration, an employee-management admin UI/API, and password-reset flows — employee, manager/HR assignment, and access-role data is provisioned via seed/migration for this phase (TR-001).
- Refresh tokens — API00 issues only a short-lived access token; no refresh/rotation endpoint is in scope.
- Create/update/delete for the department/location/role catalog — API06–API08 are read-only; catalog data is provisioned via seed/migration for this phase (A-001).
- Any UI/frontend surface — this project is Backend Only.

## Non-Functional Constraints (from constitution.md)
`.ai-context/constitution.md` has no populated Non-Functional Baselines yet (the ingested BRD source document contained no Constitution section). This spec therefore only inherits the project-wide baseline already established elsewhere:
- Modular Monolith backend structure; module boundaries per `.ai-context/architecture.md` (no direct cross-module data access) — the `transfer` module's need to read `identity` module data (manager/HR assignment, access roles) must be resolved via an explicit service interface at Plan time, not direct repository access across modules.
- JWT-based authentication (stateless) per `.ai-context/architecture.md`; access token expiry is environment-backed (`JWT_EXPIRATION_MS`).
- Passwords are stored hashed (adaptive hash, e.g. BCrypt) — never in plaintext or logged.
- PostgreSQL + Hibernate ORM for persistence; schema/migration tool (Flyway vs Liquibase) is an open architectural decision (`architecture.md`) to be resolved at Plan time, before this spec's data model can be implemented.
- BRD NFR-001 (reminder notification policy) — duration not yet finalized; not implemented in this spec (see Explicitly Out of Scope).
