# ADR-001: Billing lives in the AGPL core as `internal/billing`

Status: proposed (G1, module saas). Requirements: SAAS-04, SAAS-05, SAAS-09.

## Context
- Nothing in the fork covers plans, limits or payments.
- The core is AGPL-3.0. Code linked into the binary becomes part of the covered work.
- The catalogue Cashfree plugin is Medusa-only. Its flow is reusable; its code is not.

## Decision
- Build one Go package `internal/billing` inside the fork.
- Its models live in `internal/models`, like all fork models.
- It is the only writer of plans, subscriptions, usage, payments and billing events.

## Alternatives
- Separate closed billing service over HTTP. Keeps the code private. Adds a service, a network hop and a second database owner. Needs approval (E23).
- Reuse `@mithtech-medusa/cashfree` through a Medusa instance. Adds a whole Medusa stack for one feature.

## Consequences
- Billing source is public under AGPL. Secrets stay in env vars.
- Self-hosters can turn billing off with a config setting.
- One binary, one database, one transaction for plan and tenant changes.
