# ADR-043: The duplicate check uses one atomic PostgreSQL insert

Status: proposed (G1 connect). Requirement: INT-04. Guard 2, the guarantee.

## Context
- Several Polygin instances serve the API. Two copies of one event can arrive at the same moment.
- The Redis rate limiter fails open when Redis is down (`middleware/ratelimit.go`). A guarantee cannot fail open.

## Decision
- Store keys in a new `event_keys` table: tenant, key hash, first event id, expires at.
- Make tenant plus key hash the primary key.
- Claim a key in one statement: `INSERT ... ON CONFLICT DO UPDATE ... WHERE expires_at < now() RETURNING id`.
- A returned row means first. No row means duplicate.
- Insert the key and the event in one transaction.
- Window: 30 days by default. Admin sets 1 to 90 days in `Organization.Settings` (`connect.dedupe_window_days`).
- A sweeper deletes expired keys daily.

## Alternatives
- Redis `SET NX EX`. Rejected: data loss on Redis restart and fail-open behaviour.
- A unique index on the `events` log. Rejected: it cannot express a time window.

## Consequences
- The check costs one indexed insert per event.
- A repeat after the window sends again. Origin (guard 3) covers late imports.
