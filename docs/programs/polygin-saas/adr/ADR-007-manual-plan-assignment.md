# ADR-007: Bundled contracts are a `manual` subscription source

Status: proposed (G1, module saas). Requirements: SAAS-09; decision D4.

## Context
- Some tenants get Polygin inside a Mithtech contract, for example ERPNext implementation and maintenance.
- They pay through that contract. No online charge applies.
- The contract record lives outside Polygin. Polygin must not copy it.

## Decision
- Add `source = manual` to `Subscription`. Only a Superadmin can create or change it.
- Store `contract_ref` as a plain reference (link only), `assigned_by`, and an optional end date.
- A manual subscription has no Cashfree IDs and creates no `Payment` rows.
- The Superadmin can add `LimitOverride` rows for bundle terms.
- Admins see "Managed by Mithtech". They cannot cancel or change it.
- Notify the Superadmin 30 days before the end date.
- On the end date, the subscription goes to `past_due` with the normal grace. The notice goes to the Superadmin first.
- The Admin can move to a paid plan at the end date. That ends the manual row and starts a Cashfree one.

## Alternatives
- A 100 percent discount coupon on a Cashfree plan. Still needs a mandate and a card.
- A flag on `Organization`. Splits plan state across two owners.

## Consequences
- Usage and limits work the same for every source.
- Every manual change is in the audit log with the Superadmin's ID.
