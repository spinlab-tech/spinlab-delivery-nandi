# Feature Specification: DP Runner Monitoring Wrapper

**Feature Branch**: `[008-dp-runner-monitoring-wrapper]`  
**Created**: 2026-02-21  
**Status**: Implemented  
**Input**: User description: "add `monitoring.js` to record timings of functions and leverage existing console.log wrappers for each alert."

## Multi-Repo Traceability *(mandatory)*

**Affected Repositories**:

- `nandi_dp_runner`
- `spinlab-delivery-nandi`

**Change Type**: meta

**Linked Work Items**:

- `TBD`

## Summary

Add reusable monitoring utilities to the DP runner and integrate them with existing alert logger wrappers so each run records:

- run-level timing
- phase/function timing
- result counters
- structured logs for start/end/failure

without changing alert business logic.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Observe cron performance per mode (Priority: P1)

As an operator, I need each alert mode to emit consistent timing and counter logs so I can diagnose slow runs and regressions.

**Why this priority**: Existing logs show status and counts but not consistent phase durations or machine-readable run summaries.

**Independent Test**: Run DP runner mode(s) and verify start/end structured logs include run ID, total duration, phase durations, and counters.

**Acceptance Scenarios**:

1. **Given** a mode run starts, **When** logger `startSession` is called, **Then** a monitoring start event with run metadata is emitted.
2. **Given** a phase executes, **When** wrapped by monitoring timer helper, **Then** phase duration is emitted and accumulated in session summary.
3. **Given** a mode run finishes successfully, **When** run completes, **Then** a monitoring end event is emitted with total duration and counters.
4. **Given** a mode run fails, **When** exception is caught, **Then** failure event is emitted with error details and duration until failure.

### Edge Cases

- Monitoring should not throw if a phase ends without a start marker.
- If a run finishes without explicit phase metrics, end event should still include total duration.
- Existing human-readable `console.log` messages should remain available.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Add a common monitoring helper module that supports run lifecycle (`start`, `finish`, `fail`) and phase timing (`measureAsync`).
- **FR-002**: Existing mode logger wrappers MUST leverage the monitoring helper instead of ad-hoc timing code.
- **FR-003**: Monitoring logs MUST include a run identifier to correlate phase logs.
- **FR-004**: Monitoring integration MUST preserve existing alert processing logic and status transitions.
- **FR-005**: Monitoring integration MUST expose basic run counters (`created`, `renewed`, `resolved`, and stage counts when available).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Each monitored mode emits one run start and one run end/fail structured log event.
- **SC-002**: Each monitored mode emits phase-level duration logs for core stages (data load, compute, and handlers).
- **SC-003**: Existing run behavior and result counts remain unchanged for the same input.

## Implementation Scope

- Add:
  - `nandi_dp_runner/src/mode/common/monitoring.js`
- Update:
  - `nandi_dp_runner/src/mode/cattle-low-activity-24-hs/logger.js`
  - `nandi_dp_runner/src/mode/cattle-without-events-48-hs-all-accounts/logger.js`
  - `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/logger.js`
  - `nandi_dp_runner/src/mode/cattle-low-activity-24-hs/index.js`
  - `nandi_dp_runner/src/mode/cattle-without-events-48-hs-all-accounts/index.js`
  - `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/index.js`
  - `nandi_dp_runner/src/mode/cattle-low-activity-24-hs/handler.js`
  - `nandi_dp_runner/src/mode/cattle-without-events-48-hs-all-accounts/handler.js`
  - `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/handler.js`
  - `nandi_dp_runner/src/mode/gps-alert/logger.js`
  - `nandi_dp_runner/src/mode/gps-alert/index.js`

## Assumptions

1. Structured logs are sufficient first step before wiring to external telemetry sinks.
2. This change does not include Cronitor/Sentry SDK wiring yet; it prepares consistent timing data at source.
3. Mode execution remains sequential as currently implemented.
