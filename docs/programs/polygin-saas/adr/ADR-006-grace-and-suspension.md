# ADR-006: Grace period, then read-only suspension; inbound messages are kept

Status: proposed (G1, module saas). Requirements: SAAS-03, SAAS-05. Numbers wait on open question S4 (`../open-questions.md`).

## Context
- A failed renewal should not cut off a tenant at once.
- The tenant's own customers keep writing in. Losing those messages hurts them, not only the tenant.
- Cashfree retries a failed mandate charge on its own schedule.

## Decision
- On a failed renewal, set the subscription to `past_due`. Set `grace_until` from `Plan.grace_days` (default 7).
- During grace the tenant works normally. Admins see a banner and get notices on day 0, 3 and 6.
- Admins can pay at once with a one-off PG order, or update the mandate.
- When grace ends unpaid, set `access_state` to `suspended`.
- Suspended is read-only. Login, viewing, export and billing work. Outbound sends, campaigns, chatbot replies and calls stop.
- Inbound WhatsApp messages are still stored while suspended.
- A payment returns the tenant to `active` at once.
- After the retention period, the tenant moves to `closed`. Data deletion follows a final notice.
- A trial that ends without a mandate goes straight to `suspended`. Trials have no grace.

## Alternatives
- Hard stop on the first failure. Simple, but harsh and costly in support.
- No suspension, only reminders. Lets unpaid tenants run forever.

## Consequences
- The grace and retention numbers live on `Plan` and in config, not in code.
- saas builds the banner and the read-only state in the web app. The app shells show them, because they load the hosted web app (ADR-060).
