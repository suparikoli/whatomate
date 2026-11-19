# ADR-040: A connected app owns a scoped API key and one webhook

Status: proposed (G1 connect). Requirements: INT-01, INT-02, INT-03.

## Context
- API keys exist (`api_keys`). A key acts with its creator's role and has no scopes.
- Webhooks exist (`webhooks`). Nothing links a key and a webhook to one business app.
- Admins need to connect, scope and revoke each app in one place.

## Decision
- Add a new `connected_apps` table, owned by connect: tenant, app type, name, status, last seen.
- Extend `api_keys` with `connected_app_id` and `scopes`. Do not create a second credential store.
- Extend `webhooks` with `connected_app_id`. One webhook per connected app.
- Scopes: `events:write`, `messages:send`, `templates:read`, `contacts:read`, `contacts:write`, `routes:read`.
- Add a `RequireScope` middleware. A key with empty scopes keeps legacy full access.
- A connected-app key gets only its scopes, not its creator's role (ADR-085).
- Revoke in one transaction: app `revoked`, its keys inactive, its webhook inactive.
- Rotate by issuing a second key. Allow at most two active keys per app.
- Issue new keys with the `pgn_` prefix. Keep `whm_` valid.
- Generate the webhook secret on the platform. Encrypt it at rest with `internal/crypto`.

## Alternatives
- A separate `app_credentials` table. Rejected: it duplicates `api_keys` (rule E02).
- OAuth client credentials. Rejected for now: plugins run server-side and API keys already work.

## Consequences
- `validateAPIKey` must set scopes in the request context.
- Deactivating the key's creator does not break the key, because the key holds scopes only (ADR-085). G3 tests this.
- Key create, rotate and revoke are refused during impersonation (ADR-008, ADR-085).
- The legacy API keys and webhooks pages keep working for keys without an app.
