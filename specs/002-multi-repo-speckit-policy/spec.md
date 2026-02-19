# Feature Specification: Multi-Repo Spec-Kit Enforcement Policy

**Feature Branch**: `[002-multi-repo-speckit-policy]`  
**Created**: 2026-02-19  
**Status**: Draft  
**Input**: User description: "Implement a parent-level multi-repo Spec Kit policy for the Nandi workspace with advisory enforcement, centralized records, and repo-relative path hygiene."

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

- [ ] Policy rollout task for workspace-level AGENTS alignment

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Enforce Shared Recording Workflow (Priority: P1)

As a maintainer operating across multiple repositories, I need one parent policy so Spec Kit records are created consistently for all change types.

**Why this priority**: Without a single policy, traceability is inconsistent across repositories.

**Independent Test**: Review parent policy and verify it defines mandatory Spec Kit recording, advisory warning behavior, and final summary requirements.

**Acceptance Scenarios**:

1. **Given** a Codex session starts in workspace root, **When** a change is requested in any child repo, **Then** the policy requires a central Spec Kit record in `spinlab-delivery-nandi/specs/`.
2. **Given** a task has no existing spec record, **When** implementation begins, **Then** policy requires an advisory warning before proceeding.

---

### User Story 2 - Keep Nested Delivery Rules Aligned (Priority: P2)

As a maintainer, I need the nested delivery repo policy to reference the parent policy and avoid conflicts.

**Why this priority**: Nested conflicts create ambiguous behavior for cross-repo sessions.

**Independent Test**: Verify nested `AGENTS.md` references parent policy and only contains delivery-repo responsibilities.

**Acceptance Scenarios**:

1. **Given** a session runs inside `spinlab-delivery-nandi`, **When** policy files are read, **Then** nested policy points to parent policy as authoritative.

---

### User Story 3 - Preserve Repo-Relative Documentation Hygiene (Priority: P3)

As a maintainer, I need Spec Kit prompts and templates to explicitly prevent absolute filesystem paths in generated docs.

**Why this priority**: Absolute paths break portability and leak local filesystem details.

**Independent Test**: Verify prompt/template guidance explicitly requires repo-relative references.

**Acceptance Scenarios**:

1. **Given** a new spec or checklist is generated, **When** references are written, **Then** they use repo-relative paths.
2. **Given** path hygiene checks run on generated specs/checklists, **When** searching for absolute path patterns, **Then** no absolute path references are found.

### Edge Cases

- What happens when only docs/config/meta files change? The workflow still requires a central spec record.
- What happens if work touches multiple repos in one task? One central spec must list all affected repos.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The workspace MUST include a parent `AGENTS.md` that governs all repositories under the workspace root.
- **FR-002**: The parent policy MUST require a Spec Kit record for every change type, including docs/config/meta changes.
- **FR-003**: The parent policy MUST define advisory enforcement behavior that requires warning before implementation when no record exists.
- **FR-004**: The parent policy MUST require final responses to include the updated or created spec path.
- **FR-005**: The parent policy MUST require each spec to include `Affected Repositories`, `Change Type`, and `Linked Work Items`.
- **FR-006**: The nested delivery repo policy MUST reference the parent policy as authoritative and avoid conflicting scope or enforcement statements.
- **FR-007**: Spec Kit prompt/template guidance MUST require repo-relative paths for generated markdown references.

### Key Entities *(include if feature involves data)*

- **Policy Artifact**: Workspace instruction file that defines required recording behavior and enforcement semantics.
- **Spec Record**: Central feature folder under `spinlab-delivery-nandi/specs/<id>-<slug>/` containing `spec.md` and checklists.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Parent and nested policy files are present and aligned with no conflicting enforcement statements.
- **SC-002**: New spec records include traceability fields for affected repositories, change type, and linked work items.
- **SC-003**: Generated spec/checklist documentation examples remain repo-relative and contain no absolute filesystem references.
