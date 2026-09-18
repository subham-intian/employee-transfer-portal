# Business Requirements Document (BRD)

**Status:** Approved (Gate 0)
**Source Document:** single `.docx` file under `docs/`
**Ingested:** 2026-09-18
**Gate 0 Review:** Approved by Subham Bhattacharyya (subham.bhattacharyya@intglobal.com) on 2026-09-18 — see `.ai-context/pr_reviews/BRD-20260918-013000.md`

> This BRD was ingested from a single provided BRD source document. No other candidate BRD document exists under `docs/`, so authority is unambiguous. Some sections of the source document describe process/evaluation context rather than product requirements — they are not reproduced as BRD fields but remain available in the source document for traceability. The document contains no explicit "Constitution" section, so `.ai-context/constitution.md` has not been modified by this ingestion.

## Objective

Enable an employee to initiate, track, and complete an **Internal Transfer Request** entirely through the One-Point Employee Portal, replacing the current process where the employee must coordinate manually across their manager, HR, Payroll, IT, and Facilities. The portal should orchestrate the downstream activities across these stakeholders and give the employee a single consolidated view of progress.

## Scope

- Self-service initiation of an internal transfer request by an employee from the One-Point Employee Portal.
- Capture of transfer details: proposed department/business unit, proposed location, proposed role/job position, effective date, optional reason.
- Orchestration of the downstream activities currently performed manually: manager confirmation, HR eligibility validation, organisational-information update, payroll update, IT access provisioning/removal, facilities relocation arrangement.
- Employee-facing visibility into the request's current status and which stakeholder(s) currently hold a pending action.

## Actors

| Actor | Role in the journey |
|---|---|
| Employee | Initiates the transfer request; selects department, location, role, effective date; provides optional reason; submits; views status and pending actions. |
| Manager | Discusses the transfer with the employee; confirms the transfer. |
| HR | Validates transfer eligibility; updates the employee's organisational information. |
| Payroll | Updates payroll records as needed for the transfer. |
| IT | Provisions or removes system/access rights as needed for the new role/location. |
| Facilities | Arranges the employee's new physical location as needed. |

## Functional Requirements

| ID | Requirement |
|---|---|
| BRD-001 | The employee shall be able to select the proposed new department/business unit for a transfer request. |
| BRD-002 | The employee shall be able to select the proposed new location for a transfer request. |
| BRD-003 | The employee shall be able to select the proposed new role/job position for a transfer request. |
| BRD-004 | The employee shall be able to provide an effective date for the transfer. |
| BRD-005 | The employee shall be able to provide an optional reason for the transfer. |
| BRD-006 | The employee shall be able to submit the internal transfer request. |
| BRD-007 | The employee shall be able to view the current status of a submitted transfer request. |
| BRD-008 | The employee shall be able to view which actions are currently pending with other stakeholders (manager, HR, payroll, IT, facilities) for their request. |
| BRD-009 | The portal shall orchestrate the downstream activities of the transfer (manager confirmation, HR eligibility validation, org-info update, payroll update, IT provisioning/removal, facilities relocation) and present the employee a single, consolidated view of progress across them. |

## Non-Functional Requirements

| ID | Requirement |
|---|---|
| NFR-001 | Each pending stakeholder step (Manager/HR/Payroll/IT/Facilities) triggers a reminder notification after a configurable SLA window; there is no automatic escalation to another approver on breach. Exact SLA duration is not yet defined — see Open Questions. |

## Business Rules

None of the rules below are stated explicitly in the source document; they were not specified there and were resolved as decisions by the requirements owner on 2026-09-18 to close the ambiguities the source document leaves open.

| ID | Rule |
|---|---|
| BR-001 | Transfer eligibility is determined via HR manual review; no automated/system-encoded eligibility rules exist in this phase. |
| BR-002 | The employee's current Manager must confirm the request before it proceeds to HR review — sequential, not parallel. |
| BR-003 | A Manager may reject a request; a Manager rejection moves the request to a terminal Rejected state. |
| BR-004 | Any stakeholder step (Manager Review, HR Review, Payroll, IT, Facilities) may end the request in a terminal Rejected state per that stakeholder's own decision authority. |
| BR-005 | A request progresses linearly: Submitted → Manager Review → HR Review → Payroll → IT → Facilities → Completed. Rejected/Cancelled are terminal exits reachable from any non-terminal state. |
| BR-006 | An employee may cancel their own request only while it is in Submitted, Manager Review, or HR Review — i.e. before Payroll/IT/Facilities execution has started. No cancellation once execution has begun. |
| BR-007 | A request qualifies as an "internal transfer" if it changes at least one of department/business unit, location, or role/job position — a single-attribute change is sufficient. |
| BR-008 | Reassignment of the employee's reporting manager, if any results from the transfer, is out of scope for this journey (see Out of Scope). |
| BR-009 | Each stakeholder can view the full upstream decision history of a request once it reaches (or has reached) their step — e.g. HR can see the Manager's approval and any comment; Payroll can see both the Manager's and HR's decisions. A stakeholder's action queue (what's pending on *them*) remains scoped to requests where they currently or previously held a pending action. |

## Assumptions

None of the assumptions below are stated in the source document; they were resolved as decisions by the requirements owner on 2026-09-18.

| ID | Assumption |
|---|---|
| A-001 | Department, location, and role selection lists are backed by new, portal-local reference data seeded for this feature, not an existing external HR/org master-data service (none is described in the source document). |
| A-002 | Payroll, IT, and Facilities steps are manual/tracked-only in this phase — each team marks its own step complete within the portal; no outbound integration with external Payroll/IT/Facilities systems is implemented. |

## Out of Scope

Resolved as decisions by the requirements owner on 2026-09-18 (not stated explicitly in the source document):

- Real-time/system integration with external Payroll, IT, or Facilities systems (tracked as manual checklist steps instead — see A-002).
- Reassignment of the employee's reporting manager as part of this journey (see BR-008).
- Automatic escalation of stale pending actions (see NFR-001).

## Open Questions

1. What is the specific SLA duration/threshold before a reminder notification fires for a pending stakeholder step (NFR-001)?
2. The source document's Deliverable list (Section 4) skips from "Deliverable 5" to "Deliverable 7" — Deliverable 6 is undefined in the source. Flagged as a document gap, not a product requirement.

## Acceptance Criteria

High-level, BRD-level scenarios; granular, individually identifiable acceptance criteria belong in the feature spec's AC section once drafted.

- **AC-1:** Given an employee provides a proposed department, location, role, and effective date, when they submit, then an Internal Transfer Request is created and immediately visible to the employee with an initial status.
- **AC-2:** Given the "reason for transfer" field is optional, submission succeeds whether or not it is provided.
- **AC-3:** Given a previously submitted request, the employee can view its current status at any time.
- **AC-4:** Given a previously submitted request, the employee can view which stakeholder(s) currently hold a pending action on it.
