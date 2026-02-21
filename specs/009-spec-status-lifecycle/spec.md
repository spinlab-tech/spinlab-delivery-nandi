# Feature Specification: Spec Status Lifecycle Governance

**Feature Branch**: `[009-spec-status-lifecycle]`  
**Created**: 2026-02-21  
**Status**: Implemented  
**Input**: User description: "Audit implementation status of delivery specs and add AGENTS policy to keep spec status aligned with reality."

## Multi-Repo Traceability *(mandatory)*

**Affected Repositories**:

- `nandi-be`
- `nandi-frontend-app`
- `nandi_backend`
- `nandi_dp_runner`
- `nandi_frontend`
- `nandi_runner`
- `nandi_udp_server`
- `spinlab-delivery-nandi`

**Change Type**: meta

**Linked Work Items**:

- `TBD`

## Summary

Define and enforce a spec-status lifecycle so spec metadata reflects implementation reality, and backfill current delivery specs to accurate statuses.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Accurate Spec State For Stakeholders (Priority: P1)

As a maintainer, I need each spec to expose a reliable status so I can understand delivery state without reading all implementation diffs.

**Why this priority**: Incorrect `Draft` states make planning and reporting unreliable.

**Independent Test**: Inspect all existing specs under `specs/` and confirm each has one valid status that matches available evidence.

**Acceptance Scenarios**:

1. **Given** an implemented work item, **When** reading its spec metadata, **Then** `**Status**` is `Implemented`.
2. **Given** partially completed work with pending items, **When** reading its spec metadata, **Then** `**Status**` is `Partially Implemented`.

---

### User Story 2 - Policy-Guided Status Updates (Priority: P1)

As a maintainer, I need AGENTS guidance to require status evaluation for touched specs so metadata drift does not reappear.

**Why this priority**: Without explicit process gates, status lines quickly become stale.

**Independent Test**: Review parent and nested `AGENTS.md` and confirm lifecycle values, update rules, and final-response requirements are explicit and aligned.

**Acceptance Scenarios**:

1. **Given** a task touches one or more specs, **When** final delivery is produced, **Then** status rationale is required for each touched spec.
2. **Given** nested and parent policy files, **When** read together, **Then** nested policy references and preserves parent lifecycle semantics.

---

### User Story 3 - Existing Specs Backfilled (Priority: P2)

As a maintainer, I need current specs backfilled now so the new policy starts from a clean baseline.

**Why this priority**: Enforcing future updates without backfill leaves current dashboards misleading.

**Independent Test**: Verify `001` through `008` statuses match the audited decisions.

**Acceptance Scenarios**:

1. **Given** `specs/001` through `specs/008`, **When** metadata is read, **Then** statuses reflect audited state.
2. **Given** previously non-standard metadata (`specs/004`), **When** inspected, **Then** required metadata fields include explicit status.

### Edge Cases

- A diagnosis-only item can still be `Partially Implemented` when some evidence steps are blocked.
- A future task may touch a spec without changing code and still require status re-evaluation.
- Specs with narrative lines containing the word "status" must still keep exactly one top-level metadata status line.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Parent `AGENTS.md` MUST define a status lifecycle with allowed values `Draft`, `In Progress`, `Partially Implemented`, `Implemented`, and `Blocked`.
- **FR-002**: Parent `AGENTS.md` MUST require touched specs to have status evaluation and updates when evidence changes.
- **FR-003**: Parent `AGENTS.md` MUST require final responses to include status rationale lines for touched specs.
- **FR-004**: Nested `spinlab-delivery-nandi/AGENTS.md` MUST align with and not weaken parent lifecycle semantics.
- **FR-005**: Existing specs `001` through `008` MUST be backfilled to audited statuses.
- **FR-006**: `specs/004-sentry-issue-scan/spec.md` MUST be normalized to include standard metadata with explicit status.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: All specs under `specs/*/spec.md` contain exactly one top-level `**Status**` metadata line using allowed values.
- **SC-002**: Backfilled statuses for `001` through `008` match the audited mapping with no remaining stale `Draft` entries where implementation evidence exists.
- **SC-003**: Parent and nested AGENTS policies include explicit lifecycle and final-response status rationale requirements.

## Execution Outcome

- Backfilled statuses:
  - `specs/001-always-remove-last-sensor/spec.md`: `Implemented`
  - `specs/002-multi-repo-speckit-policy/spec.md`: `Implemented`
  - `specs/003-fields-inventory-nplus1-fix/spec.md`: `Implemented`
  - `specs/004-sentry-issue-scan/spec.md`: `Partially Implemented`
  - `specs/005-dashboard-query-performance/spec.md`: `Implemented`
  - `specs/006-cattle-low-gps-movement-runtime-review/spec.md`: `Partially Implemented`
  - `specs/007-low-movement-connectivity-gating/spec.md`: `Draft`
  - `specs/008-dp-runner-monitoring-wrapper/spec.md`: `Implemented`
