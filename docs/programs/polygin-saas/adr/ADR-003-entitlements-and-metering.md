# ADR-003: One entitlements service; counts read source tables; meters use Redis

Status: proposed (G1, module saas). Requirements: SAAS-04; contract for `connect` and `apps`.

## Context
- Limits apply to seats, WhatsApp accounts, monthly messages, AI minutes, events and connected apps.
- Campaign workers send in parallel. A read-then-write check would overshoot the limit.
- A copied seat counter would drift from `user_organizations`.

## Decision
- Store limits and feature flags as `PlanLimit` rows (key, value; -1 means unlimited).
- A Superadmin `LimitOverride` row wins over the plan value for that subscription.
- Keep the key registry in `internal/billing/keys.go`. Billing owns keys. Each module owns the counted records.
- Count keys (`seats`, `wa_accounts`, `connected_apps`) count rows in their source table at check time.
- Meter keys (`messages_out`, `ai_minutes`, `events_in`) use an atomic Redis script: check and increment in one step.
- A ticker flushes meters to `UsageCounter` (org, key, month) each minute. Redis rebuilds from it after a restart.
- Expose four calls: read a limit, check a feature, check headroom, consume.
- Return HTTP 402 `plan_limit_reached` with key, limit and used.

## Alternatives
- Limits as JSONB on `Plan`. Simpler, but no per-key override and no index.
- Postgres row lock per increment. Correct, but slow on campaign bursts.

## Consequences
- Up to one minute of meter data can be lost if Redis dies. The send logs allow a rebuild.
- New modules add a key by pull request to the registry, not a new table.
