# Feature Specification: Always Remove Previous Sensor On Re-Pair

**Feature Branch**: `001-always-remove-last-sensor`  
**Created**: 2026-02-19  
**Status**: Draft  
**Input**: User description: "Always remove previous sensor references and sensor records when pairing a new sensor, without requiring removeLastSensor input."

## Traceability

- **Affected Repositories**: `nandi-be`, `spinlab-delivery-nandi`
- **Change Type**: `bugfix`
- **Linked Work Items**: `TBD`

## User Scenarios & Testing _(mandatory)_

### User Story 1 - Re-Pair Works Without FK Failures (Priority: P1)

As an operations user, I can pair a sensor to a member even when that sensor name is already used elsewhere, and the operation succeeds without database constraint failures.

**Why this priority**: This is the blocking production issue affecting daily sensor pairing.

**Independent Test**: Trigger one sensor pairing where the same sensor name is already linked to members. The request succeeds and the target member is paired.

**Acceptance Scenarios**:

1. **Given** one or more members are linked to sensors named `X`, **When** a user pairs sensor name `X` to another member, **Then** the old references are removed, old `X` sensors are removed, and the target member is paired successfully.
2. **Given** the target member had a previous sensor, **When** a new sensor is paired, **Then** previous sensor references are removed and the previous sensor is removed.

---

### User Story 2 - Cross-Account Reassignment By Name (Priority: P2)

As an operations user, I can reassign a sensor by name even if it was previously attached in another account context.

**Why this priority**: Field operations reattach physical sensors across account boundaries.

**Independent Test**: Seed sensor name `X` in another account, pair `X` in the current account, and confirm the operation succeeds with global cleanup by name.

**Acceptance Scenarios**:

1. **Given** sensor name `X` exists across multiple accounts, **When** `X` is paired to a member, **Then** all previous `X` sensor records are removed regardless of account and the target member receives a newly created sensor record.

---

### User Story 3 - API Input Is Simplified (Priority: P3)

As a client integrator, I no longer need to provide or manage a `removeLastSensor` flag.

**Why this priority**: Reduces client complexity and removes ambiguous behavior.

**Independent Test**: Send pairing request with only required pairing fields and verify success; verify cleanup behavior remains the same regardless of optional extra fields.

**Acceptance Scenarios**:

1. **Given** a valid pair request, **When** the request does not include `removeLastSensor`, **Then** pairing succeeds with full cleanup behavior.

---

### User Story 4 - Bulk Pairing Uses The Same Cleanup Rules (Priority: P1)

As an operations user, I can pair many members in one request and each pairing applies the same cleanup-and-recreate behavior as single-member pairing.

**Why this priority**: Batch pairing must not bypass cleanup logic, or old references and conflicting sensor rows reappear.

**Independent Test**: Send one `PATCH /members/pair-sensors` request with non-duplicate rows and verify each successful row applies full cleanup behavior before creating and assigning a new sensor.

**Acceptance Scenarios**:

1. **Given** a bulk pair request row with sensor name `X`, **When** the row is processed, **Then** all previous references to any sensor named `X` are removed, old `X` sensors are deleted, a fresh sensor row is created, and it is assigned to the target member.
2. **Given** duplicated `mac_address` values in the same bulk payload, **When** the request is processed, **Then** each duplicated row is marked failed with a deterministic duplicate error and no pairing transaction is executed for those rows.
3. **Given** duplicated `caravana` values in the same bulk payload, **When** the request is processed, **Then** each duplicated row is marked failed with a deterministic duplicate error and no pairing transaction is executed for those rows.
4. **Given** one row fails in a bulk payload, **When** other rows are valid and non-duplicate, **Then** valid rows still execute and return their own success results.

### Edge Cases

- Sensor name exists on many members and many accounts at once.
- Target member has no previously assigned sensor.
- Target member is already linked to a sensor with the same name.
- The cleanup set includes the target member itself before final re-pairing.
- Duplicate `mac_address` values appear in one bulk payload.
- Duplicate `caravana` values appear in one bulk payload.
- A bulk payload has a mix of duplicate rows, successful rows, and rows that fail due to data lookup errors.

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST locate all sensor records matching the requested sensor name before pairing.
- **FR-002**: System MUST remove member-to-sensor references for all members linked to those matched sensors before deleting those sensors.
- **FR-003**: System MUST delete all matched sensors by name regardless of account ownership.
- **FR-004**: System MUST create a new sensor record for the requested name and assign it to the target member.
- **FR-005**: System MUST always remove references to the target member’s previously assigned sensor and remove that previous sensor when it differs from the newly assigned sensor.
- **FR-006**: System MUST not require `removeLastSensor` as an input parameter for this operation.
- **FR-007**: System MUST record a sensor pairing event for the target member after successful reassignment.
- **FR-008**: System MUST apply cleanup and pairing as one atomic operation so partial cleanup is not persisted.
- **FR-009**: System MUST apply FR-001 through FR-008 for each non-duplicate row in `PATCH /members/pair-sensors`.
- **FR-010**: System MUST execute each bulk row in an independent atomic transaction and return per-row success or failure.
- **FR-011**: System MUST reject every bulk row whose `mac_address` value is duplicated in the same payload and return `Duplicate mac_address in payload: <value>`.
- **FR-012**: System MUST reject every bulk row whose `caravana` value is duplicated in the same payload and return `Duplicate caravana in payload: <value>`.
- **FR-013**: System MUST continue processing non-duplicate bulk rows even when another row fails.

### Key Entities _(include if feature involves data)_

- **Sensor Pair Request**: Input that identifies target member, sensor name, and account context for creation.
- **Sensor**: Persisted sensor identity that can be reassigned and recreated during pairing.
- **Member**: Entity that holds current sensor linkage and is re-paired by this workflow.
- **Member Sensor Pairing Event**: Audit event recording successful pairing changes.

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: `PATCH /members/:id/new-sensor` completes successfully for 100% of cases where sensor-name collisions previously caused FK failures.
- **SC-002**: Zero new production errors for foreign key violations tied to sensor deletion in this endpoint for 14 consecutive days after rollout.
- **SC-003**: Client integrations can pair sensors without sending `removeLastSensor` and achieve the same final cleanup behavior.
- **SC-004**: For each successful request, exactly one final sensor link remains on the target member and no deleted sensor remains referenced by any member.
- **SC-005**: `PATCH /members/pair-sensors` applies the same cleanup-and-recreate behavior for each successful row as single-member pairing.
- **SC-006**: For payloads with duplicate `mac_address` or duplicate `caravana`, 100% of duplicate rows return deterministic duplicate errors and do not mutate pairing state.
