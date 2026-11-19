# ADR-062: Push service as a package in the fork, with its own desktop stream

Status: proposed (G1, module `apps`).

## Context
- Nothing in the fork sends push. The catalogue has `ntfynext`, which is Frappe-only and does not reach phones.
- Event sites already exist: incoming `new_message`, `agent_transfer_assign`, `transfer_escalation`, missed calls.
- The WebSocket hub sends `new_message` to the whole org. It is in memory in one process.
- Redis Streams already carry campaign jobs (`internal/queue`).

## Decision
- Add `internal/push` and `handlers/push.go` to the fork. It owns the Device entity (`push_devices`).
- Event sites call `push.Enqueue(event)`. A worker reads Redis Stream `polygin:push`.
- The worker applies one rule set: recipient (assigned user, as the web client does today; or members with a permission for Admin types), per-type prefs, quiet hours, presence delay, privacy render, badge.
- The type registry and recipient rules follow ADR-084. saas and connect call `push.Enqueue` directly; there is no event bus.
- Transports: FCM HTTP v1 (existing `golang.org/x/oauth2`), APNs HTTP/2 with ES256 (existing `golang-jwt`), and `/api/push/desktop/stream` for the desktop helper.
- The desktop stream authenticates with a per-device secret. It carries rendered notifications and badge counts only.

## Alternatives
- A separate push microservice. Rejected: extra deploy, and it must read users, contacts and settings from the fork's database.
- Desktop joins the existing `/ws` hub. Rejected: the hub would send the full org message feed to a background process, and it would count as online for call routing.
- Web Push (VAPID) for desktop. Rejected: needs a service worker in the frontend and still needs the helper for tray and calls.

## Consequences
- No new Go dependency.
- Push depends on Redis being up. Jobs survive a restart through the stream.
- If the hub moves to many processes, presence checks need a shared store. Note for the core team.
- Removing a device revokes its desktop secret at once.
