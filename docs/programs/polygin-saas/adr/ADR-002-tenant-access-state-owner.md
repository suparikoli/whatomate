# ADR-002: `Organization.access_state` is the single owner of tenant access

Status: proposed (G1, module saas). Requirements: SAAS-01, SAAS-03, SAAS-05.

## Context
- Billing can stop a tenant (unpaid). The Superadmin can stop a tenant (abuse, contract end).
- Two flags in two places would let them disagree.
- Today the request chain checks only that the user is active (`middleware.go` `OrganizationContext`).

## Decision
- Extend `Organization` with `access_state` (active, suspended, blocked, closed), `access_reason` (billing, platform) and `access_changed_at`.
- `Subscription.status` owns billing state. It never gates requests directly.
- One billing function maps subscription changes to `access_state`. It never overrides a `platform` reason.
- A new `RequireAccess` middleware reads `access_state` from a Redis cache with a 60 s TTL. Changes clear the cache.
- `RequireAccess` uses the same org resolution as `getOrgID` (header switch included). The logic moves into one shared function.

## Alternatives
- Derive access from subscription status on every request. Mixes billing and platform holds. Costs a join per request.
- Use `User.IsActive` for every member. Loses the tenant view and is slow to reverse.

## Consequences
- Every module reads one field to know if a tenant may act.
- Inbound webhook handlers also read it (store inbound, skip outbound).
