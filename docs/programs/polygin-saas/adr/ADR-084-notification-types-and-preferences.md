# ADR-084: Notification types, recipients and preferences have one owner

Status: proposed (merge pass). Modules: apps (owner), saas, connect. Updates ADR-062.

## Context
- apps owns push and user push preferences. Its recipient rule is "the assigned user".
- connect wants to alert Admins of failed events, dead deliveries and revoked apps.
- saas sends billing notices on day 0, 3 and 6, and Superadmin notices. It has no channel yet.
- apps asked connect for an event bus. None exists, and none is needed.

## Decision
- `internal/push/types.go` is the one registry of notification types. Each type names its recipient rule and default.
- Recipient rules: `assigned_user`; `members_with_permission(<resource>)`; `superadmins`.
- Agent types: `message.new`, `chat.assigned`, `chat.escalated`, `call.missed`.
- Admin types: `connect.event.failed`, `connect.delivery.dead`, `connect.app.revoked`, `connect.webhook.paused` (permission `connected_apps`); `billing.access_changed` (permission `billing`).
- Event sites call `push.Enqueue(type, org, payload)` directly. There is no event bus.
- `User.Settings.push.types` turns a type on or off. Quiet hours and preview follow ADR-063.
- Channels: WebSocket (core hub, web only), push (apps), email (saas sender, open question S5).
- Billing and security emails are mandatory. Push preferences never turn them off.

## Alternatives
- Each module sends its own alerts. Rejected: three rule sets and three preference stores.
- An internal event bus. Rejected: a new moving part for four callers.

## Consequences
- Adding a type is a pull request to the registry.
- A missing email sender delays billing emails, not push.
