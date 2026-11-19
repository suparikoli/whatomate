# ADR-087: Names, prefixes and headers after the rename

Status: proposed (merge pass). Modules: brand-ui (owner of the rename), connect, apps. Extends ADR-023. Closes open question C5.

## Context
- connect asked brand-ui to sign off the `pgn_` key prefix and header names.
- ADR-023 did not cover key prefixes, cookies or HTTP headers.
- Cookies `whm_access`, `whm_refresh`, `whm_csrf` exist. Users never see them.

## Decision
- New API keys use `pgn_`. `whm_` keys stay valid.
- New HTTP headers use `X-Polygin-*`. `X-Webhook-Signature` stays.
- Cookie names stay `whm_*`. Renaming them logs every user out and conflicts with upstream.
- New Redis keys and streams use `polygin:`. Existing ones keep their names.
- The plugin header is `X-Polygin-Plugin: <app>/<version>`.

## Alternatives
- Rename cookies too. Rejected: forced logout and upstream conflicts, for an invisible name.
- Keep `whm_` for new keys. Rejected: keys are shown to Admins and plugin authors.

## Consequences
- Developers see both prefixes in code. This is expected, as with the Go module path.
