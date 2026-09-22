# Business Requirements Document (BRD)
**Project**: Employee Internal Transfer Digital Journey
**Status**: Pending Gate 1

## 1. Objective
Enable an employee to initiate and track an internal transfer request through the One-Point Employee Portal as a single digital journey, orchestrating downstream activities across HR, Payroll, IT, and Facilities.

## 2. Actors
- **Employee**: Initiates the transfer request and views status.
- **Manager**: Confirms the transfer.
- **HR**: Validates eligibility and updates organisational information.
- **Payroll**: Updates payroll information.
- **IT**: Provisions/removes access.
- **Facilities**: Arranges the employee's new location.

## 3. Requirements

### Functional Requirements
- **BRD-001**: An employee must be able to initiate an Internal Transfer Request from the One-Point Employee Portal.
- **BRD-002**: An employee must be able to select the proposed new department/business unit.
- **BRD-003**: An employee must be able to select the proposed new location.
- **BRD-004**: An employee must be able to select the proposed role/job position.
- **BRD-005**: An employee must be able to provide an effective date.
- **BRD-006**: An employee must be able to provide an optional reason for the transfer.
- **BRD-007**: An employee must be able to submit the request.
- **BRD-008**: An employee must be able to view the current status of the request.
- **BRD-009**: An employee must be able to view actions that are pending with other stakeholders.
- **BRD-010**: The portal must orchestrate downstream activities across HR, IT, Payroll, and Facilities.

### Non-Functional Requirements
- **BRD-NFR-001**: Single view of progress provided to the employee.

## 4. Business Rules
- Transfer request must include department/BU, location, role, and effective date.
- Reason for transfer is optional.
- **Request Statuses**: A request will follow these lifecycle states: `DRAFT`, `PENDING_MANAGER_APPROVAL`, `PENDING_HR_VALIDATION`, `PROCESSING_IT`, `PROCESSING_PAYROLL`, `PROCESSING_FACILITIES`, `CHANGES_REQUESTED`, `COMPLETED`, `REJECTED`, and `CANCELLED`.
- **Rejections / Modifications**: If a stakeholder (e.g., Manager or HR) requires changes, the state becomes `CHANGES_REQUESTED` and is returned to the employee for editing and resubmission.

## 5. Assumptions
- Employee already has access to the One-Point Employee Portal.
- The Portal has integration capabilities with downstream systems (HR, Payroll, IT, Facilities).

## 6. Out-of-Scope
- The physical fulfillment of IT hardware or Facilities arrangements (the system only orchestrates the request and tracks status).

## 7. Service Level Agreements (SLAs) & Notifications
- **SLAs**: Best effort tracking will be used. There are no strict automated time limits, but timestamps must be logged to track time-in-status for visibility.
- **Notifications**: Notifications will be delivered via both In-portal notifications and Email alerts.

## 8. Open Questions for Gate 0 Review
- **Workflow Sequence**: Do `PROCESSING_IT`, `PROCESSING_PAYROLL`, and `PROCESSING_FACILITIES` occur sequentially or in parallel?
- **Manager Approval Hierarchy**: Does "Manager Approval" require the employee's current manager, future manager, or both?
- **Cancellation / Rollback**: Up to what state can an employee cancel their request? If cancelled during processing, do we send rollback signals?
- **Data Sourcing & Validation**: Are departments/locations/roles static or fetched dynamically from an external HRIS? Are there constraints on the effective date?
- **Audit Trail**: Does the employee need a detailed historical audit log of status changes or just the current status?
