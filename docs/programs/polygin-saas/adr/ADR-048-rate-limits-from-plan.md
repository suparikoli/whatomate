# ADR-048: Rate limits and quotas come from the tenant's plan

Status: proposed (G1 connect). Requirements: INT-04, SAAS-04.

## Context
- The `saas` module owns plans, limits and usage (`billing`).
- A Redis fixed-window limiter exists (`middleware/ratelimit.go`). It is keyed by IP or user and fails open.

## Decision
- Key names, errors and the cache follow ADR-081.
- Reuse the limiter. Key it by connected app. Read the rate key `events_per_min` through the billing service.
- Return 429 `rate_limited` with `Retry-After` when the minute limit is hit.
- Call billing `Consume(org, events_in, 1)` for each accepted event. Billing owns the usage count.
- Return 402 `plan_limit_reached` (key `events_in`) when the monthly meter is used up.
- Check `connected_apps` when an Admin creates an app. Return 402 `plan_limit_reached` (key `connected_apps`). Connect registers the counter function.
- Run `/api/v1` behind `RequireAccess`. It returns 403 `tenant_suspended` or `tenant_blocked`.
- Keep no limit cache in connect. The billing service caches limits for 60 seconds.
- Count dropped events against the minute limit. Do not consume `events_in` for them.

## Alternatives
- Limits as connect settings. Rejected: `saas` owns plan data (rule E02).
- A fail-closed limiter. Rejected: a Redis outage would stop all intake. Guard 2 does not depend on Redis.

## Consequences
- Connect needs the `saas` plan read API before G3 build starts.
- A plan change takes up to 60 seconds to apply.
