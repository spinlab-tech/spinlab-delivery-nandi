# Spec: Sentry Issue Scan (Automation Run)

## Summary
Run the Sentry issue scan automation for Nandi to check for recent issues (past 14 days when possible) and summarize findings.

## Context
The automation is scheduled to run regularly and should surface unresolved or recent issues for Nandi. This run is a read-only check and does not modify code or configuration.

## Traceability
- Affected Repositories: nandi-be, nandi_backend, nandi_frontend, nandi_runner, nandi-frontend-app, nandi_udp_server
- Change Type: meta
- Linked Work Items: None

## Acceptance
- Attempt to query Sentry issues for the last 14 days.
- If credentials are missing, report the blocker and required setup steps.
- No code or config changes are made.

## Reporting Policy
- Keep this spec stable and focused on scope, traceability, and acceptance criteria.
- Store per-run Sentry findings in automation memory and run inbox summaries, not in this spec.
