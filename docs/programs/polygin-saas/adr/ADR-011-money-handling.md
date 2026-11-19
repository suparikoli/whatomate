# ADR-011: Money as integer paise in INR, with a price snapshot per subscription

Status: proposed (G1, module saas, money floor). Requirements: SAAS-04, SAAS-05.

## Context
- Floats lose precision on money.
- A plan price can change. Existing customers keep the price they agreed to until renewal terms change.
- Tax treatment and invoices are not decided (open question 2).

## Decision
- Store every amount as a 64-bit integer in paise, with a `currency` column fixed to INR for now.
- Copy the price into `Subscription` at purchase. Later `PlanPrice` edits do not change it.
- Never edit a price row in use. Retire it and add a new one.
- Compare the amount in each Cashfree event with the expected amount. Flag a mismatch for review; do not apply it.
- `Payment` records the gateway fact only. Accounting entries and tax invoices belong to the accounting owner.

## Alternatives
- Decimal type in Postgres. Correct, but the Go side then needs a decimal library (N01).
- Prices read live from `PlanPrice`. Simple, but silent price changes for existing customers.

## Consequences
- All display code formats paise to rupees at the edge.
- Adding another currency later means new price rows, not new columns.
