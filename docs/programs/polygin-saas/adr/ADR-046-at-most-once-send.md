# ADR-046: Send at most once per event

Status: proposed (G1 connect). Requirement: INT-04.

## Context
- A message must go out once.
- A worker can crash after Meta accepts the send but before Polygin records it.
- The campaign worker uses Redis Streams, which redeliver pending jobs (`queue/redis.go`).

## Decision
- Store event states: `queued`, `sending`, `sent`, `failed`, `needs_review`.
- The worker claims an event with `FOR UPDATE SKIP LOCKED` and sets `sending` before it calls Meta.
- An event left in `sending` past a timeout goes to `needs_review`. It is never resent by itself.
- Admin sees `needs_review` events and can resend one by hand.
- Redis pub/sub only wakes the worker. A sweeper also picks up `queued` events, so a lost wake-up only delays.
- Send through one internal function extracted from `SendTemplateMessage`.

## Alternatives
- At-least-once with Redis Streams. Rejected: a crash would send twice.
- Store the `wamid` first. Not possible: Meta returns it only after the send.

## Consequences
- In a rare crash, a message may not go out until an Admin acts. This is the chosen trade-off.
- `event.failed` and the push event to `apps` tell the Admin.
