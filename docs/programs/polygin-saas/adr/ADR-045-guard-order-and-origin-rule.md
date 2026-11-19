# ADR-045: Guard order, and the platform applies the origin rule

Status: proposed (G1 connect). Requirements: INT-04, INT-05.

## Context
- Three guards are agreed. (1) One sender app per record type. (2) Business key and window. (3) Synced records carry their origin, and plugins skip them.
- Guards 1 and 3 conflict. Say Tally creates an invoice and syncs it to ERPNext, and the sender is ERPNext. ERPNext skips it by guard 3. Polygin drops Tally's copy by guard 1. Nobody sends.
- A dropped event must not claim the key. Otherwise the right sender would be dropped as a duplicate.

## Decision
- Plugins always send the event with `origin` filled. The platform decides.
- Order at intake: auth and limits, then validation, then route lookup, then guard 1, then guard 3, then guard 2.
- Guard 1: if the route has a sender and the caller is another app, return `ignored_not_sender`.
- Guard 3: apply it only when the route has no sender. Drop when `origin.synced_from` names another active connected app. Return `ignored_synced`.
- Guard 2: claim the key (ADR-043). Only events that pass guards 1 and 3 claim it.
- Plugins may skip locally by the same rule. They read their routes from `GET /api/v1/connected-apps/me/routes`.

## Alternatives
- Plugins always skip synced records. Rejected: it breaks the sender rule, as shown above.
- Every event claims the key. Rejected: a non-sender would block the sender.

## Consequences
- Polygin receives more calls than with plugin-side skipping. Rate limits count them.
- This changes the agreed wording of guard 3. It is an open question for the user.
