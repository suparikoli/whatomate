# ADR-008: Audited impersonation sessions replace silent Superadmin org access

Status: proposed (G1, module saas). Requirements: SAAS-03.

## Context
- Today a Superadmin reaches any tenant through the `X-Organization-ID` header (`handlers/app.go`) or `SwitchOrg` (`handlers/auth.go`).
- Neither path records a reason. Audit rows show the Superadmin's user ID only.
- Permission checks short-circuit for Superadmins (`handlers/cache.go`). They get full rights in every tenant.
- A SaaS customer needs to know when the platform owner looked at their data.

## Decision
- Add `ImpersonationSession`: Superadmin, tenant, optional acting user, mode (read or write), reason, start, expiry (max 60 min), end, IP.
- Starting a session issues a short-lived token with the session ID and the impersonator ID.
- Cross-tenant access by a Superadmin needs an open session. The header and `SwitchOrg` paths refuse otherwise.
- Every audit row written in a session carries `impersonation_session_id` (extends `AuditLog`).
- Read mode blocks all writes. Write mode needs a second reason.
- Never allow changes to billing, passwords, API keys or SSO settings while impersonating. Never show decrypted secrets.
- A Superadmin cannot impersonate another Superadmin.
- The UI shows a fixed banner with an "End session" action.
- Tenant Admins can see impersonation sessions in their audit view.

## Alternatives
- Keep the header switch and log it. Cheap, but no reason, no expiry and no read mode.
- Support access only through the tenant sharing a login. Insecure and untraceable.

## Consequences
- Platform support works without breaking tenant trust.
- This changes fork behaviour for Superadmins. Tests in `auth` and `app` must cover it.
