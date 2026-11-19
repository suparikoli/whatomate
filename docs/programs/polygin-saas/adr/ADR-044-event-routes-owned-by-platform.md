# ADR-044: Event routes hold the sender app and the template mapping

Status: proposed (G1 connect). Requirements: INT-04, INT-05.

## Context
- Each tenant chooses one sender app per record type (guard 1).
- Today each plugin picks the template. Six plugins would hold six copies of that choice.

## Decision
- Add a new `event_routes` table, owned by connect. One row per tenant, record type and event.
- Columns: sender connected app (nullable), WhatsApp account, template, parameter map, optional flow, enabled.
- The parameter map links template parameters to JSON paths in the event, for example `"1": "$.data.customer_name"`.
- A route with a flow sends the template first. The contact's reply then starts the flow.
- Admin edits routes on the connected apps page. Plugins can read them through `routes:read`.
- An event without an enabled route is stored as `no_route`. It sends nothing.

## Alternatives
- Mapping inside each plugin. Rejected: no single owner (rule E02).
- Sender setting in `Organization.Settings`. Rejected: it belongs with the route it controls.

## Consequences
- ChatbotFlow needs a new entry point: start from a template reply. Today flows start only from keywords.
- WhatsApp needs a template outside the 24-hour window. So every route starts with a template.
