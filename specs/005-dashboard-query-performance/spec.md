# Feature Specification: Dashboard Query Performance

**Feature Branch**: `[005-dashboard-query-performance]`  
**Created**: 2026-02-20  
**Status**: Draft  
**Input**: User description: "Admin dashboard `count-by-day` and `events-by-last-day` are taking ~18-19 seconds and must be optimized."

## Multi-Repo Traceability *(mandatory)*

**Affected Repositories**:

- `nandi-be`
- `nandi_frontend`
- `spinlab-delivery-nandi`

**Change Type**: bugfix

**Linked Work Items**:

- `TBD`

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Fast dashboard metrics loading (Priority: P1)

As an admin user, I need dashboard metrics endpoints to return quickly so the dashboard is usable without long waits.

**Why this priority**: This is a core admin screen and current load times are high enough to block normal operations.

**Independent Test**: Call `/member-events/count-by-day` and `/members/events-by-last-day` with realistic account data and verify improved latency while preserving response shapes.

**Acceptance Scenarios**:

1. **Given** a valid date range for `count-by-day`, **When** the endpoint is called, **Then** the response keeps the same fields and reflects correct daily totals.
2. **Given** dashboard loads member activity table, **When** `events-by-last-day` is called, **Then** the response keeps the same fields and counts semantics expected by current frontend rendering.
3. **Given** dashboard frontend loads count data, **When** it requests recent range, **Then** requests stay bounded to recent days.

### Edge Cases

- Date ranges with no events should still return zero-filled day entries for requested days.
- Members with no events in the last-day window should still be returned with zero count if current behavior expects full member list.
- Large historical tables must avoid full scans for range-based counts.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: `GET /member-events/count-by-day` MUST keep response shape `{date, member_events, member_messages}` while reducing query overhead.
- **FR-002**: `GET /members/events-by-last-day` MUST keep response shape compatible with dashboard consumers and include member identity/context fields (`id`, `account_id`, `tag_id`, `member_field_name`, `member_type`, `_count`, `accounts`).
- **FR-003**: Dashboard frontend MUST continue to request a bounded recent date range for `count-by-day`.
- **FR-004**: Database access path for both endpoints MUST support efficient range filtering on event timestamps.
- **FR-005**: `GET /members/events-by-last-day` MUST aggregate the last 24-hour counts from `member_events` first, then complete missing members with zero counts via join.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Dashboard endpoint latency for `count-by-day` decreases materially from current baseline (~18s under reported load).
- **SC-002**: Dashboard endpoint latency for `events-by-last-day` decreases materially from current baseline (~19s under reported load).
- **SC-003**: Existing frontend dashboard renders successfully with unchanged response contract fields.

## Notes

- Schema ownership currently lives outside `nandi-be`; DB index changes are applied directly in SQL, then reflected into Prisma schema via introspection (`prisma db pull`).
