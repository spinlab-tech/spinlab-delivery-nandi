# Feature Specification: Cattle Low GPS Movement Runtime Review

**Feature Branch**: `[006-cattle-low-gps-movement-runtime-review]`  
**Created**: 2026-02-21  
**Status**: Partially Implemented  
**Input**: User description: "pull latest changes in all repositories in main branches, then review issues in the new cattle alert for movement under 25 meters/day; process runtime increased from ~10s to ~50s."

## Multi-Repo Traceability *(mandatory)*

**Affected Repositories**:

- `nandi_dp_runner`
- `nandi_runner`
- `spinlab-delivery-nandi`

**Change Type**: bugfix

**Linked Work Items**:

- `TBD`

## Summary

This work item is diagnosis-only. It validates the runtime regression impact introduced by the new `CATTLE_LOW_GPS_MOVEMENT_24_HS_ALL_ACCOUNTS` mode and produces prioritized findings plus low-risk remediation options without implementing code changes.

Follow-up behavior spec for connectivity-based creation suppression:

- `specs/007-low-movement-connectivity-gating/spec.md`

## Scope

### In Scope

- Validate repository sync status against `main` for all workspace repositories.
- Inspect low-movement alert code path and orchestration path.
- Collect available runtime evidence from this environment.
- Produce prioritized findings with concrete file-level evidence.

### Out of Scope

- Any production behavior change.
- Any schema/API/type modification.
- Any code implementation of fixes.

## Public APIs / Interfaces / Types

- No public API, schema, or type changes in this diagnosis pass.

## Execution Results

### Repository Sync Results

- Executed `git checkout main` and `git pull --ff-only` for all repositories:
  - `nandi-be`
  - `nandi-frontend-app`
  - `nandi_backend`
  - `nandi_dp_runner`
  - `nandi_frontend`
  - `nandi_runner`
  - `nandi_udp_server`
  - `spinlab-delivery-nandi`
- Fast-forward updates occurred in:
  - `nandi-frontend-app` (`d06a1c5` -> `88eec68`)
  - `nandi_runner` (`ab47587` -> `5f3258e`)
- All repositories ended on clean `main` tracking `origin/main`.

### Runtime Evidence Collected (2026-02-21)

1. **Per-mode elapsed time from run logs**
   - **Status**: Blocked
   - **Reason**: Run logs for production/VPS execution were not present in this workspace checkout.
2. **Redis cardinality evidence (`SCARD members`, `SCARD alerts`, `SCARD cattleLocation`)**
   - **Status**: Blocked
   - **Reason**: Redis is hosted in VPS context; localhost Redis probing from this environment was not available.
3. **DB timing evidence (safe/read-only and explain probes)**
   - `SELECT NOW()` round-trip: **983 ms**
   - `SELECT COUNT(*) FROM alerts`: **8114 rows**, **1903 ms**
   - `SELECT COUNT(*) FROM alerts WHERE alert_type='CATTLE_LOW_GPS_MOVEMENT_24_HS'`: **989 rows**, **997 ms**
   - `SELECT COUNT(*) FROM alerts WHERE alert_type='CATTLE_LOW_GPS_MOVEMENT_24_HS' AND current_status IN ('CREATED','RENEWED')`: **989 rows**, **279 ms**
   - `SELECT id ... WHERE entity_id IN (200 ids) AND alert_type='CATTLE_LOW_GPS_MOVEMENT_24_HS' AND current_status='CREATED' AND last_updated_date=<latest>`: **199 rows**, **961 ms**
   - `EXPLAIN` for the created-ID lookup above: access type **ALL**, scanned rows **8167**, **no key used** (full table scan behavior)
   - `EXPLAIN` for renew/resolve updates by `id IN (...)`: **PRIMARY range** lookup
   - `EXPLAIN` for member status updates by `id IN (...)`: **PRIMARY range** lookup
4. **alerts_run_result telemetry inspection**
   - Table has no duration column (schema fields: `account_id`, `type`, `processDate`, `created`, `renewed`, `resolved`).
   - Latest run snapshot includes `CATTLE_LOW_GPS_MOVEMENT_24_HS_ALL_ACCOUNTS` with `created=988`, `renewed=1`, `resolved=0`, indicating high per-run affected volume.

### Created-Alerts Validation Against DB Logic

Validation target: latest low-movement run:

- `processDateTs=1771646400` (`2026-02-21T04:00:00Z`)
- Derived alert decision window:
  - `startTs=1771545600` (`2026-02-20T00:00:00Z`)
  - `endTs=1771632000` (`2026-02-21T00:00:00Z`)
  - `processDateARTTs=1771635600` (`2026-02-21T01:00:00Z`)

Mimicked logic used for validation:

1. Parse `member_events` with `type='LAST_POSITION'` and `value='lat:lon'` in `[startTs,endTs]`.
2. Compute per-member total Haversine distance using consecutive points ordered by `ts`.
3. Reconstruct pre-run open low-movement alerts using latest `alerts_events` status before `processDateARTTs`.
4. Expected create rule:
   - member `is_alertable in (true,1)`
   - computed distance `< 25`
   - pre-run open low-movement alert count `= 0`
5. Compare expected members vs actual created events at `UNIX_TIMESTAMP(alerts_events.date)=1771635600`.

Validation outcome:

- Reported created count (`alerts_run_result`): **988**
- Actual created events at run timestamp: **988** (unique members: 988)
- Expected created by DB mimic: **972**
- Match:
  - intersection: **970**
  - actual not expected: **18**
  - expected not actual: **2**
- Additional checks on the 988 created:
  - created with computed distance `>=25m`: **3**
  - created with pre-run open alert count `>0`: **0**
  - created with zero position points in `member_events` window: **926**

Interpretation:

- Core behavior mostly aligns (970/988) when reconstructed from DB tables.
- Mismatch bucket (18) is concentrated in data-quality/state-consistency cases:
  - **10** are currently `is_alertable=0`
  - **5** refer to member IDs not present in current `members` table
  - **3** have computed movement clearly over threshold (`104m`, `526m`, `737m`)
- There are **2** expected-but-not-created members, both account `52` test-like devices (`member_id` 3129, 3130), consistent with cache/snapshot drift risk.

### Movement Bucket Detail for the 970 Matched Members

From the matched set (`intersection=970`), movement distance distribution:

- `0-5m`: **967**
- `5-10m`: **2**
- `10-15m`: **0**
- `15-20m`: **1**
- `20-25m`: **0**

Cumulative movement thresholds in the matched set:

- `>=5m`: **3**
- `>=10m`: **1**
- `>=15m`: **1**
- `>=20m`: **0**

### Connectivity Alert Overlap for the 970 Matched Members

Connectivity definition used:

- Alert type: `CATTLE_WITHOUT_EVENTS_48_HS`
- Open state at low-movement run context: latest status `<= processDateART` is `CREATED` or `RENEWED`

Results:

- Matched members with open connectivity alert: **873 / 970** (**90.0%**)
- Matched members with connectivity `CREATED/RENEWED` exactly at low-movement run timestamp: **873 / 970** (**90.0%**)

Interpretation:

- Most matched low-movement creates are happening together with connectivity/no-events alert conditions, which supports the concern that data availability (not true low movement) is dominating many low-movement outcomes.

## Targeted Code Paths

- `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/get-cattle-location-data.js`
- `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/query.js`
- `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/utils.js`
- `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/handler.js`
- `nandi_dp_runner/src/index.js`
- `nandi_runner/src/controllers/events/index.ts`
- `nandi_runner/src/db/redis/save-data.ts`

## Findings (Prioritized)

### Finding 1 (High): Full Redis scans with sequential hash reads dominate low-movement mode

- **Symptom**: Runtime grows sharply when low-movement mode is enabled.
- **Root-Cause Evidence**:
  - `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/query.js` reads all member IDs and all alert IDs, then sequentially performs `HGETALL` per key.
  - `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/get-cattle-location-data.js` reads all `cattleLocation` IDs and sequentially performs `HGETALL` per key before time filtering.
  - These patterns are O(N) over total Redis set sizes for every run.
- **Estimated Impact**: High.
- **Recommended Fix Option**: Batch hash reads via Redis pipelining and avoid full-keyspace scans by narrowing candidate sets to the active date window and alert type.

### Finding 2 (High): `cattleLocation` ingestion has no retention control, so scan cost can grow unbounded

- **Symptom**: Daily read workload for low-movement mode can inflate over time regardless of current-day activity.
- **Root-Cause Evidence**:
  - `nandi_runner/src/controllers/events/index.ts` continuously writes new `cattleLocation` keys.
  - `nandi_runner/src/db/redis/save-data.ts` performs `HSET` + `SADD` only, with no `EXPIRE`, `SREM`, or cleanup policy.
  - Low-movement collector scans from the global `cattleLocation` set each run.
- **Estimated Impact**: High (increasing over time).
- **Recommended Fix Option**: Introduce retention (TTL and/or periodic compaction) and remove processed/expired keys from the tracking set.

### Finding 3 (High): Redis snapshot drift can create alerts for non-alertable/missing members

- **Symptom**: Created alerts include members that are non-alertable or not present in current members table.
- **Root-Cause Evidence**:
  - Validation mismatch set (`actualNotExpected`) includes 18 created members.
  - In that mismatch set: 10 currently `is_alertable=0`, 5 are absent from current `members`.
  - Low-movement mode sources members from Redis cache (`smembers members` + `hgetall members:*`) instead of authoritative DB state.
- **Estimated Impact**: High (correctness risk).
- **Recommended Fix Option**: Build candidate member list from DB (`members.is_alertable=1`) and use Redis only for location/event stream payloads.

### Finding 4 (Medium): Post-insert created-alert ID lookup is a measurable SQL hotspot

- **Symptom**: Created-alert flow incurs additional DB latency after insert.
- **Root-Cause Evidence**:
  - `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/handler.js` runs a second query to fetch created IDs after insert.
  - `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/query.js` uses:
    `SELECT id FROM alerts WHERE entity_id IN (?) AND alert_type='CATTLE_LOW_GPS_MOVEMENT_24_HS' AND current_status='CREATED' AND last_updated_date=?`
  - Probe timing: **961 ms** for 200 entities; `EXPLAIN` shows **full table scan** (`type=ALL`, `key=NULL`).
- **Estimated Impact**: Medium (consistent additive latency per run).
- **Recommended Fix Option**: Remove the post-insert lookup (derive IDs from insert batch semantics) or add an index supporting `(alert_type, current_status, last_updated_date, entity_id)`.

### Finding 5 (Medium): Main runner orchestration is strictly serial, so new mode latency is fully additive

- **Symptom**: End-to-end runtime increases roughly by the new mode’s duration.
- **Root-Cause Evidence**:
  - `nandi_dp_runner/src/index.js` awaits each mode one after another (`48hs all accounts`, then `low gps movement`, then each `24hs` account mode).
  - `nandi_dp_runner/src/mode/cattle-low-gps-movement-24-hs-all-accounts/index.js` internally also executes resolve/create/renew handlers sequentially.
- **Estimated Impact**: Medium.
- **Recommended Fix Option**: Split independent jobs by schedule or parallelize safe-independent mode execution with concurrency guards.

### Finding 6 (Low): Missing duration telemetry prevents precise regression attribution in historical runs

- **Symptom**: Historical run records cannot answer where time is spent.
- **Root-Cause Evidence**:
  - `alerts_run_result` schema does not store elapsed time, only counts.
- **Estimated Impact**: Low for runtime itself, high for observability.
- **Recommended Fix Option**: Add per-phase timing logs and persist duration metrics for each mode and key sub-step.

## Recommendation Shortlist (Low Risk)

- Use Redis pipelining/batching for hash reads in low-movement collection paths.
- Avoid full-scan of `alerts` for active low-movement status checks.
- Add retention/cleanup policy for `cattleLocation` keys.
- Remove/replace post-insert created-alert `SELECT` pattern.
- Add durable per-phase timing instrumentation in the runner.

## Safe Fix Proposal (No Schema Changes)

### Goal

Reduce low-movement runtime and avoid incorrect creates while preserving current alert semantics.

### Proposal A (Recommended, low risk)

1. **Authoritative member source from DB**
   - Replace Redis member scan for this mode with:
     - `SELECT id, account_id, member_field_id, member_type, mac_address, is_alertable FROM members WHERE is_alertable = 1`
   - Benefit: prevents creates for non-alertable/missing members due cache drift.

2. **Keep Redis for location stream, but batch reads**
   - For `cattleLocation`, keep existing source but use Redis pipeline/mget-style batching to reduce round-trips.
   - Avoid loading unnecessary keys by date prefilter strategy where possible.

3. **Preserve open-alert lookup in DB (not Redis)**
   - Build open low-movement alert map from `alerts` table:
     - `alert_type='CATTLE_LOW_GPS_MOVEMENT_24_HS'`
     - `current_status IN ('CREATED','RENEWED')`
   - Benefit: stable consistency with persisted state.

4. **Fix created-ID retrieval hotspot**
   - Replace post-insert `SELECT ... entity_id IN (...) AND last_updated_date=?` with:
     - deterministic ID derivation strategy from insert metadata, or
     - targeted query path with supporting composite index.

5. **Add minimal timing instrumentation**
   - Log duration for each phase:
     - member load
     - location load
     - distance calculation
     - resolve/create/renew persistence
   - Persist duration summary to run results or structured logs.

### Proposal B (Very small patch, immediate guardrail)

1. Keep current architecture.
2. Add one guard in low-movement classification:
   - require at least 2 valid points before calculating low-movement eligibility.
3. This reduces noisy creates from missing data windows, but does not solve scan-cost or cache-drift root causes.

### Rollout Safety

1. Deploy with dry-run flag that logs candidate counts without writes for 1 day.
2. Compare dry-run counts vs current production counts per account.
3. Enable writes after count deltas are within expected bounds.

## Test Cases and Scenarios

1. Baseline scenario: run full daily pipeline and measure per-mode elapsed time contribution.
2. Cardinality stress scenario: correlate `SCARD cattleLocation` with low-movement mode runtime.
3. SQL overhead scenario: isolate created-alert lookup overhead versus other write-path queries.
4. Consistency scenario: verify created/renewed/resolved counts remain coherent during profiling.
5. Re-run scenario: run same date twice and check repeat-scan scaling behavior.

### Execution Status in This Pass

- Scenario 1: **Blocked** (VPS run logs unavailable in workspace).
- Scenario 2: **Blocked** (Redis cardinality probe unavailable from this environment).
- Scenario 3: **Partially executed** (DB timing and explain evidence collected).
- Scenario 4: **Partially executed** (count coherence observed in `alerts_run_result`, no full pipeline replay).
- Scenario 5: **Not executed** (requires runnable VPS Redis-backed environment).

## Assumptions and Defaults

1. Local branches are clean and currently on `main`.
2. Redis and execution logs are hosted in VPS context and are not directly available in this workspace session.
3. Deliverable is diagnosis-only with no code changes.
4. No hard runtime SLO is imposed in this pass; objective is root-cause clarity and prioritized fix options.
