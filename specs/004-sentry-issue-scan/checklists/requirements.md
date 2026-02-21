# Requirements Checklist

- [x] Sentry auth token available in environment (`SENTRY_AUTH_TOKEN`).
- [x] Org and project slugs confirmed (defaults or explicit values).
- [x] Issue list queried for the past 14 days (or best available range).
- [x] Results summarized with counts and last seen timestamps.
- [x] PII redacted in any output.
- [x] Temporary run findings kept out of spec; tracked in automation memory/inbox.
- [ ] 2026-02-21 run: network access to Sentry API verified.
