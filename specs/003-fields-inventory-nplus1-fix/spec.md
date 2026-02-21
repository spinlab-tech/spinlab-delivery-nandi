# Feature Specification: Fields Inventory N+1 Fix

**Feature Branch**: `[003-fields-inventory-nplus1-fix]`  
**Created**: 2026-02-20  
**Status**: Implemented  
**Input**: User description: "Fix `GET /fields` members inventory N+1 query while preserving behavior."

## Multi-Repo Traceability *(mandatory)*

**Affected Repositories**:

- `nandi-be`

**Change Type**: bugfix

**Linked Work Items**:

- Sentry issue `NANDI-BE-7`

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Faster fields list with same data shape (Priority: P1)

As an admin user, I need `GET /fields` to return the same grouping payload without per-field inventory query overhead.

**Why this priority**: The endpoint is used frequently and currently triggers Sentry N+1 performance findings.

**Independent Test**: Call `findAll` with `includeMembersInventory=true` and verify inventory aggregation values remain correct while inventory query is batched.

**Acceptance Scenarios**:

1. **Given** multiple fields are returned, **When** inventory is requested, **Then** inventory lookup is done in one batched query and grouped output matches previous behavior.
2. **Given** inventory is not requested, **When** `findAll` runs, **Then** no inventory query is executed.

### Edge Cases

- No fields returned from base query should skip inventory lookup.
- Fields with no inventory rows should still expose inventory grouping entries with zero counts.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: `findAll` MUST avoid per-field `fields_inventory` queries when `includeMembersInventory` is enabled.
- **FR-002**: Inventory rows MUST be fetched in one batched query keyed by returned field IDs.
- **FR-003**: `membersByGroup` aggregation output MUST remain behavior-compatible with current API consumers.
- **FR-004**: Unit tests MUST validate batching behavior and compatibility of grouped inventory output.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Unit test confirms `fields_inventory.findMany` is called once for a multi-field response.
- **SC-002**: Unit test confirms inventory is not queried when `includeMembersInventory` is false.
- **SC-003**: Existing payload semantics for grouped counts remain unchanged in test assertions.
