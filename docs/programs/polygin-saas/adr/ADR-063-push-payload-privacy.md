# ADR-063: Push payload privacy is set per tenant, default sender name only

Status: proposed (G1, module `apps`).

## Context
- Push payloads pass through Apple and Google servers.
- Lock screens show the title and body to anyone holding the phone.
- Tenants differ: a clinic needs no text on lock screens; a shop may want full previews.

## Decision
- Add `push_preview` to `Organization.Settings`: `hidden` ("New message"), `sender` (contact name only, the default), `full` (name and first 100 characters).
- Users may lower the level in `User.Settings`. They may not raise it above the tenant level.
- When the contact has no profile name, show "Customer", never the phone number.
- Payloads never carry phone numbers, media, message ids or note text.
- The data part carries only: event type, org id, contact id, path, badge, collapse key.

## Alternatives
- Always full text. Rejected: leaks customer data to lock screens and third parties.
- Always hidden, with an iOS Notification Service Extension that fetches text. Deferred: more native code and an extra auth path.

## Consequences
- Admins get one setting on the tenant settings page (G2 of `apps`).
- Previews under `hidden` give Agents less triage help.
