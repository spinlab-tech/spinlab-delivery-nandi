# AGENTS.md

## Project Policy: Mandatory Spec Kit Records

This policy is required for all project changes made by Codex agents.

### Scope

Applies to:

- New features
- Initiatives
- Bugfixes
- Refactors that change behavior

### Required Workflow

For every in-scope change:

1. Create or reuse a Spec Kit feature folder under `specs/<id>-<short-name>/`.
2. Update the feature `spec.md` to record what changed and why.
3. Update or create checklist artifacts when requirements/acceptance criteria change.
4. Include the spec path in the final delivery summary.

### Tooling

Use Spec Kit scripts from this repository:

- `.specify/scripts/bash/create-new-feature.sh`
- Other `.specify` workflows as needed (`specify`, `plan`, `tasks`, etc.)

### Output Convention

- Use repo-relative paths in generated documentation (no absolute filesystem paths).

### Exceptions

Allowed only for:

- Pure documentation edits with no product behavior impact
- Repository meta changes that do not affect runtime behavior

If an exception is used, explicitly state it in the final summary.
