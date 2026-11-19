# ADR-047: Durable, signed outbound webhooks with dead letters

Status: proposed (G1 connect). Requirement: INT-02.

## Context
- `sendWebhook` retries 3 times (1 s, 2 s, 4 s) inside a goroutine with a 2-minute limit. A restart loses the delivery.
- The signature covers the body only. A captured request can be replayed.
- No webhook reports delivery status. `updateMessageStatus` dispatches nothing.

## Decision
- Add a new `webhook_deliveries` table: webhook, event, payload, attempt, next attempt, status, last code, last error.
- Insert a delivery row instead of starting a goroutine. The worker sends rows with `FOR UPDATE SKIP LOCKED`.
- Retry on network errors, 408, 429 and 5xx. Use 8 attempts: 30 s, 2 min, 10 min, 30 min, 1 h, 3 h, 6 h, 12 h.
- Mark other 4xx answers and the last failure `dead`. Admin can replay a dead delivery. Connect tells `apps` for push.
- Pause a webhook after 24 hours of failures. Tell the Admin.
- Keep `X-Webhook-Signature: sha256=<hmac of body>`.
- Add `X-Polygin-Delivery-Id`, `X-Polygin-Timestamp`, and `X-Polygin-Signature: t=<ts>,v1=<hmac of ts.body>`. Receivers reject timestamps older than 5 minutes.
- Add events `message.status`, `event.processed`, `event.failed`. Add `related_event_id` to `message.incoming` when the reply quotes an event's message.
- Send `message.status` and `event.*` only to the app whose event produced the message, plus legacy webhooks that subscribe.
- Keep payloads for 30 days.

## Alternatives
- A Redis Stream per delivery. Rejected: streams have no delayed retry, and Redis loss drops deliveries.
- A hosted webhook service. Rejected: new dependency (rule N01).

## Consequences
- Existing receivers see more retries. They must stay idempotent; INTEGRATION.md already asks for this.
- The table grows with traffic. The 30-day sweep bounds it.
