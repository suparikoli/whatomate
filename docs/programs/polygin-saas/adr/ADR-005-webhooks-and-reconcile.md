# ADR-005: Signed, idempotent Cashfree webhooks with a reconcile job

Status: proposed (G1, module saas, money floor). Requirements: SAAS-05.

## Context
- Webhooks can arrive twice, out of order or never.
- The browser return URL can be forged. It proves nothing about payment.
- A wrong tenant mapping would credit or suspend the wrong customer.

## Decision
- Add the public route `POST /api/billing/webhooks/cashfree` to the auth skip list in `cmd/whatomate/main.go`.
- Verify the HMAC-SHA256 signature over timestamp and raw body. Reject stale timestamps.
- Insert each event into `BillingEvent` with a unique provider event ID. A duplicate returns 200 and does nothing.
- Map the event to a tenant only through our stored Cashfree subscription or order ID. Ignore tenant hints in the payload.
- Apply state changes in one database transaction with the event row.
- Never move a subscription backwards on an older event. Compare event time with the last applied time.
- Change status only from a verified webhook or a server-side fetch. The return page only shows "processing".
- A reconcile ticker fetches pending and past_due subscriptions from Cashfree every 15 minutes. A Redis lock keeps it to one instance.

## Alternatives
- Trust the return URL. Fast, but forgeable.
- Webhooks only. Simpler, but a lost webhook leaves a paid tenant suspended.

## Consequences
- Every money change has a stored, signed source event.
- Failed processing keeps the event with its error for retry and review.
