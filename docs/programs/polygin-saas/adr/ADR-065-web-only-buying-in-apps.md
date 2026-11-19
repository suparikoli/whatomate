# ADR-065: Apps show plan status; buying happens on the web only

Status: proposed (G1, module `apps`). Follows decision D4.

## Context
- Plans are sold by Cashfree on the web (SAAS-05) or assigned by hand for Mithtech contracts (SAAS-09).
- Apple and Google rules limit buttons, links and calls to action for outside purchases in store apps.
- The phone shells load the same web views as browsers (ADR-060).

## Decision
- The bridge adapter reports "phone shell" to the frontend.
- In a phone shell, billing views show plan name, status, renewal date and limits. They hide prices, buy, upgrade and checkout links.
- The text says to manage the plan on the web, with no link.
- The desktop helper is not sold through a store. It shows the normal web billing views.

## Alternatives
- In-app purchase. Rejected by D4: store fees and a second billing owner.
- Hide billing entirely in apps. Rejected: APP-04 needs plan status.

## Consequences
- `saas` must expose plan status and honour the phone-shell flag in billing views.
- A suspended or expired plan shows a clear status screen in the app.
