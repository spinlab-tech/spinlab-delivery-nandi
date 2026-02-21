# Feature Specification: Sentry Issue Scan (Automation Run)

**Feature Branch**: `[004-sentry-issue-scan]`  
**Created**: 2026-02-20  
**Status**: Partially Implemented  
**Input**: User description: "Run the Sentry issue scan automation for Nandi and summarize recent issues."

## Multi-Repo Traceability *(mandatory)*

**Affected Repositories**:

- `nandi-be`
- `nandi_backend`
- `nandi_frontend`
- `nandi_runner`
- `nandi-frontend-app`
- `nandi_udp_server`
- `spinlab-delivery-nandi`

**Change Type**: meta

**Linked Work Items**:

- `TBD`

## Summary

Run the Sentry issue scan automation for Nandi to check for recent issues (past 14 days when possible) and summarize findings.

## Context

The automation is scheduled to run regularly and should surface unresolved or recent issues for Nandi. This run is a read-only check and does not modify code or configuration.

## Acceptance

- Attempt to query Sentry issues for the last 14 days.
- If credentials are missing, report the blocker and required setup steps.
- No code or config changes are made.

## Reporting Policy

- Keep this spec stable and focused on scope, traceability, and acceptance criteria.
- Store per-run Sentry findings in automation memory and run inbox summaries, not in this spec.

## Coverage Notes

- Ensure all Nandi Sentry projects in org `spinlabtech` are included when running scans.

## Execution Status

- Automation run requirements are partially complete.
- Remaining open item is tracked in `specs/004-sentry-issue-scan/checklists/requirements.md`: network access verification for the dated run.
