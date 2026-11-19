# Cross-module contracts (polygin-saas)
<!-- One page. Every Provides / Needs pair between saas, brand-ui, connect and apps, checked both ways. Written 2026-11-19. Status: proposed. -->

## C Application: contract pairs
| # | Provider to consumer | Contract | Both sides agree now | ADR |
|---|---|---|---|---|
| K1 | saas to connect, core send paths | Billing service: `Limit`, `Feature`, `Headroom`, `Consume`. Billing owns the limit cache | yes, after fixes M1 to M4 | 003, 081 |
| K2 | connect to saas | `connected_apps` counter function. `Consume(events_in, 1)` on each `accepted` event | yes, after M3 | 081 |
| K3 | saas to all | `RequireAccess` middleware. Codes `tenant_suspended`, `tenant_blocked` | yes, after M5 | 002, 082 |
| K4 | core to all | One error envelope: `code`, `message`, `details[]`. One code registry | yes, after M2 | 082 |
| K5 | saas to apps | `GET /api/billing/summary`. WebSocket `billing.access_changed` (web only) | yes | 065 |
| K6 | apps to saas | Bridge adapter `frontend/src/native/` reports the shell. Billing views hide buying in a phone shell | yes, after G2 | 065 |
| K7 | brand-ui to saas | Branding payload shape, `applyBranding()`, contrast guard, TdColorPicker | yes, after M6, M7 | 022, 083 |
| K8 | saas to brand-ui, apps | Storage in `Settings.branding`, host lookup, `GET /api/public/branding` (by host), `GET /api/branding` (session org), injection in `embed.go` | yes, after G1 | 010, 083 |
| K9 | brand-ui to apps | Responsive shell at 360 px, safe areas, 44 px targets, brand assets for icons | yes | 026 |
| K10 | apps to brand-ui | Bridge adapter sets `data-platform`. No user-agent sniffing | yes, after G7 | 026, 060 |
| K11 | apps to saas, connect, core | `push.Enqueue(type, org, payload)`. Type registry with recipient rules (ADR-084) | yes, after M8 | 062, 084 |
| K12 | saas to all | Request context: impersonator ID and session ID. Audit writer tags rows by itself | yes | 008, 086 |
| K13 | saas to connect, apps | Impersonation guard: no changes to billing, passwords, API keys, SSO, webhook secrets | yes, after M10 | 008, 085 |
| K14 | connect to plugins | Intake v1, outbound webhooks v1, plugin rules. See `modules/connect/contract.md` | yes | 041 to 049 |
| K15 | brand-ui to connect | Names: `pgn_` key prefix, `X-Polygin-*` headers, kept `whm_` cookies | yes | 087 |
| K16 | each module to saas | Tenant purge function for its own tables, called when a tenant moves to `closed` | yes, after G6 | 081 |

## Needs that no pack provided (gaps), and the fix
- G1. apps and brand-ui need branding for the session org on `app.polyg.in`. saas gave branding by host only. The `whm_access` cookie has path `/api`, so `embed.go` cannot see the session. Fix: saas adds `GET /api/branding`.
- G2. saas needs the AGPL footer from brand-ui; brand-ui left it to saas. Fix: brand-ui owns the footer, link and build commit.
- G3. apps asked brand-ui for a "Your devices" view. brand-ui builds no new screens (ADR-024). Fix: apps owns that view, built with TonalDepth.
- G4. apps asked connect for message, assignment, escalation and missed-call events. Those are core events. Fix: apps adds the `push.Enqueue` calls at the core sites (already in its placement table).
- G5. connect needs a per-minute event limit. The saas registry had none. Fix: add key `events_per_min`, kind `rate`.
- G6. Nobody deletes tenant data on `closed`. Fix: each owner registers a purge function (K16).
- G7. brand-ui needs `data-platform` set. Nobody said who sets it. Fix: the apps bridge adapter.
- G8. Email is needed by saas (verify, invite, billing notices, Superadmin notices), connect (webhook paused) and brand-ui (`{appName}`). No pack provides a sender. Fix: saas owns it; choice is open question S5.
- G9. connect Admin alerts need a recipient rule. apps had "assigned user" only. Fix: ADR-084 adds "members with a permission".
- G10. Device removal on member removal, deactivation and password reset needs calls in core user and auth handlers. Fix: apps adds them through `push.RevokeUserDevices`.
- G11. The WebSocket refresh-token finding had no owner. Fix: SEC-03, apps pack.
- G12. Desktop window controls need window IPC. ADR-060 grants only notification, badge and deep link. Fix: open question X1.

## Mismatches found and resolved
- M1. Limit key names. Resolved: `events_in`, `connected_apps`, `events_per_min` (ADR-081).
- M2. Error codes. connect used 429 `plan_quota_exceeded` and 403 `plan_limit_connected_apps`. saas used 402 `plan_limit_reached`. Resolved: 402 `plan_limit_reached` with `details` key, limit, used. 429 `rate_limited` stays for the minute limit only.
- M3. Usage. connect "reports counts" after the fact; saas wanted atomic consume. Resolved: connect calls `Consume` at the accept step. Dropped events consume nothing.
- M4. Limit cache. connect had its own 60 s cache. Resolved: connect calls the billing service, which owns the cache.
- M5. Tenant status. connect checked status itself. Resolved: `/api/v1` runs behind `RequireAccess`. It also returns `tenant_blocked`.
- M6. Contrast. saas rejected a failing accent; brand-ui darkened it. Resolved: the guard in `tonaldepth-vue` is the one rule. The screen shows the adjusted colour before save. The server checks format only.
- M7. Payload. saas stored one accent and a product name. brand-ui wanted three brand tokens per theme and a display name. Resolved: store the brand-ui payload; name comes from `Organization.Name`.
- M8. Push events. connect named `connect.event.failed` and others; apps listed only Agent events. Resolved: one registry. connect and billing types go to members with a permission.
- M9. Shells. brand-ui assumed a bundled `index.html`; apps loads the hosted app (ADR-060). Resolved: hosted app; branding after login through `GET /api/branding`.
- M10. Keys versus roles. Keys act as their creator's role, but ADR-040 says a removed creator must not break the key. Resolved: connected-app keys hold scopes only; legacy keys keep the creator's role (ADR-085).
- M11. Module name. saas and connect said `brand`. Resolved: `brand-ui`.
- M12. Read-only banner. ADR-006 put it in apps. The banner is a web view in the shells. Resolved: saas builds it.
- M13. Message meter and events. Each event send also consumes `messages_out` through the shared send function. Event sends under the monthly cap wait on open question S3.

## Behaviour both sides must keep
- Inbound Meta messages are stored while `suspended`. Outbound sends, campaigns and event intake stop.
- A 402 or 403 from intake is final for the plugin. A 429 means retry after `Retry-After`.
- Billing meters fail open when Redis is down, as the rate limiter does. Guard 2 never uses Redis (ADR-081, open question X3).
