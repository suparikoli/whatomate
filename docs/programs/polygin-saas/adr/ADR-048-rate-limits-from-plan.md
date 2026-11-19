# ADR-048: Rate limits and quotas come from the tenant's plan

Status: proposed (G1 connect). Requirements: INT-04, SAAS-04.

## Context
- The `saas` module owns plans, limits and usage (`billing`).
- A Redis fixed-window limiter exists (`middleware/ratelimit.go`). It is keyed by IP or user and fails open.

## Decision
- Reuse the limiter. Key it by connected app. Read the limit `events_per_minute` from the plan.
- Return 429 `rate_limited` with `Retry-After` when the minute limit is hit.
- Count accepted events per tenant per period. Report the count to `saas`.
- Return 429 `plan_quota_exceeded` when `events_per_month` is used up.
- Check `max_connected_apps` when an Admin creates an app. Return 403 `plan_limit_connected_apps`.
- Return 403 `tenant_suspended` for a suspended tenant.
- Cache plan limits in process for 60 seconds.
- Count dropped events against the minute limit. Do not count them against the monthly quota.

## Alternatives
- Limits as connect settings. Rejected: `saas` owns plan data (rule E02).
- A fail-closed limiter. Rejected: a Redis outage would stop all intake. Guard 2 does not depend on Redis.

## Consequences
- Connect needs the `saas` plan read API before G3 build starts.
- A plan change takes up to 60 seconds to apply.
