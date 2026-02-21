# Feature Specification: Low-Movement Creation Gating on Connectivity Alerts

**Feature Branch**: `[007-low-movement-connectivity-gating]`  
**Created**: 2026-02-21  
**Status**: Draft  
**Input**: User description: "ignore low-movement alert creation when there is a current connectivity alert; we should not trigger low-movement alerts if there are connectivity issues."

## Multi-Repo Traceability *(mandatory)*

**Affected Repositories**:

- `nandi_dp_runner`
- `spinlab-delivery-nandi`

**Change Type**: bugfix

**Linked Work Items**:

- `TBD`

## Summary

Prevent false positive low-movement alerts caused by missing telemetry by adding a creation gate:

- Do not create `CATTLE_LOW_GPS_MOVEMENT_24_HS` alerts for members that currently have an open connectivity alert.

Connectivity alert for this rule is `CATTLE_WITHOUT_EVENTS_48_HS` in open state.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Avoid low-movement false positives under connectivity issues (Priority: P1)

As an operator, I need low-movement alerts to be suppressed when connectivity is degraded so alerts reflect real animal behavior, not missing data.

**Why this priority**: Current runs show strong overlap between low-movement and connectivity alerts, which causes noisy and misleading alerts.

**Independent Test**: For members with open connectivity alerts at low-movement decision time, run low-movement process and confirm no new low-movement alerts are created.

**Acceptance Scenarios**:

1. **Given** a member has an open `CATTLE_WITHOUT_EVENTS_48_HS` alert, **When** low-movement processing runs, **Then** no `CATTLE_LOW_GPS_MOVEMENT_24_HS` alert is newly created for that member.
2. **Given** a member has no open connectivity alert and movement is below threshold, **When** low-movement processing runs, **Then** creation behavior remains unchanged.
3. **Given** connectivity alert is solved/closed before run, **When** low-movement processing runs, **Then** creation is allowed if low-movement criteria are met.

### Edge Cases

- Members with missing or stale Redis member snapshots must still respect connectivity gating using authoritative alert state.
- Members with multiple connectivity alerts must be treated as gated if any open connectivity alert exists.
- Existing low-movement alerts should not be duplicated; creation gate applies only to new creation candidates.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Low-movement creation step MUST exclude members with open `CATTLE_WITHOUT_EVENTS_48_HS` alerts.
- **FR-002**: Open connectivity state for gating MUST be defined as `CREATED` or `RENEWED` at decision time.
- **FR-003**: The gate MUST apply before low-movement `INSERT_INTO_ALERTS` is executed.
- **FR-004**: Low-movement renew/resolve behavior MUST remain unchanged unless explicitly extended in a later spec.
- **FR-005**: Runner output/logging MUST include gated-member count for observability.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In test runs where connectivity alerts are open, low-movement created count excludes those members.
- **SC-002**: False-positive low-movement creates during connectivity issues decrease materially versus baseline behavior.
- **SC-003**: Non-gated members continue to create low-movement alerts with unchanged threshold logic.

## Proposed Implementation Scope (for follow-up execution)

- Add connectivity gating in low-movement creation candidate pipeline:
  - `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/query.js`
  - `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/utils.js`
  - `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/index.js`
  - `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/handler.js`
- Include run-time metric/log for gated members in low-movement mode logs.

## Assumptions

1. This specification changes only low-movement alert **creation** gating.
2. Connectivity alert source of truth is existing alert state (`alerts` and/or equivalent cached representation), with status semantics aligned to `CREATED`/`RENEWED`.
3. No schema changes are required for this gating behavior.
