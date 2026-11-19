# ADR-086: Every module writes audit rows through `internal/audit`

Status: proposed (merge pass). Modules: all. Extends ADR-008.

## Context
- `AuditLog` exists in core (`internal/audit`). saas adds `impersonation_session_id`.
- Only the saas pack said what it audits. connect and apps did not.

## Decision
- The audit writer reads the impersonation session from request context. Callers never pass it.
- saas audits: plan, price and override changes; manual subscriptions; access state changes; impersonation start and end; domain and branding changes.
- connect audits: app connect, pause, revoke; key issue and rotate; scope changes; route changes; delivery replay; manual resend of a `needs_review` event.
- apps audits: tenant `push.preview` changes. Device register and delete are not audited (high volume); `push_devices` keeps `last_seen_at`.
- brand-ui audits nothing. It owns no server record.
- Audit rows never hold secrets, tokens or message text.

## Alternatives
- Module-specific audit tables. Rejected: a second audit owner.

## Consequences
- Tenant Admins see platform and connected-app changes in one audit view.
