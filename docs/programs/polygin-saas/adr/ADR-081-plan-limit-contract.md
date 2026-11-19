# ADR-081: One plan-limit contract for every module

Status: proposed (merge pass). Modules: saas (owner), connect, core send paths. Updates ADR-003 and ADR-048.

## Context
- saas and connect named the same limits differently (`events_in` and `events_per_month`, `connected_apps` and `max_connected_apps`).
- connect needed a per-minute limit. The saas registry had none.
- connect planned to report counts and cache limits itself. saas planned an atomic consume and its own cache.
- Count keys would make billing query tables it does not own.

## Decision
- Billing owns the key registry (`internal/billing/keys.go`) and the limit cache.
- Kinds: `count`, `meter` (monthly), `rate` (per minute), `flag`.
- Keys: `seats`, `wa_accounts`, `connected_apps` (count); `messages_out`, `ai_minutes`, `events_in` (meter); `events_per_min` (rate); `feature.*` (flag).
- The module that owns the counted records registers a counter function. Example: connect counts non-revoked `connected_apps` rows.
- connect calls `Consume(org, events_in, 1)` once per `accepted` event. Dropped events consume nothing.
- Every send through the shared send function consumes `messages_out`. This includes event sends.
- The connect limiter reads `events_per_min` through `Limit`. It keeps the existing Redis fixed-window limiter.
- Errors: 402 `plan_limit_reached` with `details` key, limit, used. 403 `plan_feature_off`. 429 `rate_limited` with `Retry-After` for `rate` keys only.
- Meters fail open when Redis is down. The miss is logged. The flush ticker reconciles from send logs. Confirm: open question X3.
- Each module registers a tenant purge function for its own tables. Billing calls them when a tenant moves to `closed`.

## Alternatives
- Let each module name and cache its own limits. Rejected: two owners for plan data.
- 429 for the monthly quota. Rejected: a plugin would retry for days. A 402 tells it to stop and alert.
- Fail closed. Rejected: a Redis outage would stop all sending.

## Consequences
- ADR-041 and ADR-048 drop `plan_quota_exceeded` and `plan_limit_connected_apps`.
- A plan change applies within the billing cache TTL (60 s).
