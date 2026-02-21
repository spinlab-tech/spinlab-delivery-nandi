# AGENTS.md

## Delivery Repo Policy

This repository is the canonical Spec Kit record store for the multi-repo workspace rooted at `../../Nandi`.

Parent policy reference:

- `../AGENTS.md` (authoritative for scope and enforcement across all repositories)

If this file and the parent policy differ, follow `../AGENTS.md`.

### Repository Responsibilities

1. Keep all workspace change records under `specs/<id>-<slug>/`.
2. Keep `spec.md` and checklists current for every tracked work item.
3. Keep `.codex/prompts/*` and `.specify/templates/*` aligned with parent path hygiene and traceability requirements.
4. Use repo-relative paths only in generated spec/checklist markdown output.
5. Apply parent `Spec Status Lifecycle` rules to all specs in this repository, including allowed values and final-response status rationale requirements.

### Status Lifecycle Alignment

1. Do not redefine or expand status values beyond the parent policy.
2. When any spec in `specs/` is touched, evaluate and update its `**Status**` line if evidence changed.
3. Ensure each touched spec has exactly one top-level `**Status**` metadata line.
4. Keep checklist execution notes aligned with the chosen status when practical.

### Local Workflow Notes

1. Create new features with `.specify/scripts/bash/create-new-feature.sh`.
2. Keep requirement checklists in `specs/<id>-<slug>/checklists/`.
3. Ensure each spec contains:
   - `Affected Repositories`
   - `Change Type`
   - `Linked Work Items`
