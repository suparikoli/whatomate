# ADR-004: Cashfree Subscriptions for renewals, PG orders for one-off payments, plain REST

Status: proposed (G1, module saas). Requirements: SAAS-05; decision D4.

## Context
- Billing is web only (D4). Mobile apps show status and open the web to buy.
- Recurring charges in India need a mandate and a pre-debit notice. Cashfree Subscriptions handles both.
- Some payments are one-off: "pay now" after a failure, and a prorated upgrade.
- Rule N01: no new dependency without approval. Cashfree has a Go SDK.

## Decision
- Create one Cashfree plan per `PlanPrice`. Store its ID on the row.
- Create a Cashfree subscription at checkout. The customer authorises a mandate (UPI AutoPay, card or eNACH) on Cashfree.
- Use Cashfree PG orders for one-off payments.
- Call the Cashfree REST API with Go `net/http`. Add no SDK.
- Read app ID, secret and webhook secret from env vars. Keep sandbox and production apart by config.
- Confirm exact endpoint and event names against the Cashfree docs in G3.

## Alternatives
- PG orders only, with a reminder each period. No mandate. Higher churn from missed renewals.
- Cashfree Go SDK. Less code, but a new dependency and version churn.

## Consequences
- We own a small client and its tests against the sandbox.
- Mandate limits and extra authentication rules sit with Cashfree and the bank.
