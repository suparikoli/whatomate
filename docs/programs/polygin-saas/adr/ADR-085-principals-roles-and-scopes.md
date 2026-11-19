# ADR-085: Roles for people, scopes for apps, secrets for devices

Status: proposed (merge pass). Modules: core, saas, connect, apps. Updates ADR-040.

## Context
- API keys act with their creator's role today (`middleware.go`). connect adds scopes.
- ADR-040 says deactivating the creator must not break a connected app's key. That clashes with role inheritance.
- saas adds role resources `billing` and `branding`. connect needs one for connected apps.
- apps adds a per-device secret for the desktop stream.
- ADR-008 blocks API key changes while a Superadmin impersonates.

## Decision
- People: authorisation comes from `CustomRole` and `Permission` (core). New resources: `billing`, `branding`, `connected_apps`.
- Connected-app keys are service principals. They get only their scopes. They do not read the creator's role.
- An Admin may grant a scope only if their own role allows the matching actions.
- Legacy keys (`whm_`, no scopes, no app) keep the creator's role. The keys page marks them "legacy".
- Plan gate: `feature.api_keys` gates legacy keys. Connected apps are gated by the `connected_apps` limit.
- Device secrets authenticate `/api/push/desktop/stream` only. They are not API keys. Deleting the device revokes the secret.
- Impersonation guard (saas): no create, rotate or revoke of keys, and no view of webhook secrets, in any session.
- Every route keeps `RequireAccess` before the scope or role check.

## Alternatives
- Scope intersected with the creator's role. Rejected: a removed creator breaks the plugin.
- Device as an API key with a push scope. Rejected: one leaked desktop file would open the API.

## Consequences
- `validateAPIKey` sets either scopes or a role in the request context, never both.
- G3 tests cover a key whose creator is deactivated.
