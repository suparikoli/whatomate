# apps: push, bridge and release contract (polygin-saas)
<!-- Detail for `G1.md`. Requirement IDs APP-01 to APP-05, SEC-03, SEC-12, SEC-13. Status: proposed. Moved out of G1 in the merge pass, 2026-11-19. -->

## A Code facts (read 2026-11-19)
- `/ws` upgrades first, then needs `{"type":"auth","payload":{"token":<jwt>}}` within 5 s. The token comes from `GET /api/auth/ws-token` (30 s, org-resolved).
- Login sets cookies only: `whm_access` (httpOnly, path `/api`), `whm_refresh` (httpOnly, single-use JTI in Redis, rotated), `whm_csrf` (double submit). SameSite=Lax. The body holds no tokens.
- Middleware also accepts `Authorization: Bearer` and `X-API-Key`; both skip CSRF.
- Events exist: `new_message` (to whole org), `agent_transfer`, `agent_transfer_assign`, `transfer_escalation`, `call_incoming`, `call_transfer_waiting`, call end states. `CallStatusMissed` exists.
- The web client alerts only the assigned user on incoming messages, gated by `new_message_alerts`. Calls ring only users online on the hub.
- The hub is in memory, one process. Redis pub/sub carries only campaign stats.
- Finding (SEC-03): `validateWSTokenFn` accepts any valid JWT, including refresh tokens. It does not check `Subject: ws`.

## C Application: push contract (ADR-062, ADR-063, ADR-084)
- Call: `push.Enqueue(type, org, payload)`. Callers: core event sites, saas billing, connect.
- Types live in `internal/push/types.go`. Each names a recipient rule: `assigned_user`, `members_with_permission(<resource>)` or `superadmins`.
- Agent types: `message.new`, `chat.assigned`, `chat.escalated`, `call.missed`.
- Admin types: `connect.event.failed`, `connect.delivery.dead`, `connect.app.revoked`, `connect.webhook.paused`, `billing.access_changed`.
- Delivery rules: skip if the type is off in `User.Settings.push.types`; quiet hours send silent (badge only); if the user is online on the hub, phone push waits 60 s and drops if the contact is read. Collapse per contact.
- Payload: event type, org id, contact id, path, badge, collapse key, and the title and body allowed by `push.preview`. Never phone numbers, media or message ids (ADR-063, SEC-12).
- Token life: register on every start and OS token change (upsert). Delete on sign-out, APNs 410 or `BadDeviceToken`, FCM `UNREGISTERED`. Prune after 60 days unseen.
- `push.RevokeUserDevices(user, org)`: apps adds the calls in core user and auth handlers for deactivation, removal from org and password reset.
- Tenant purge: apps registers a purge for `push_devices` with billing (ADR-081).
- Desktop transport: `/api/push/desktop/stream`, authenticated by the device secret only (ADR-085). It carries rendered notifications and badge counts, never the org event feed.

## C Application: bridge adapter (`frontend/src/native/`)
- Detects the shell from `window.Capacitor` and `window.__TAURI__`. Sets `data-platform` on `<html>` (ADR-026).
- Reports "phone shell" to billing views, which hide buying (ADR-065).
- Passes the push token to the device API. Opens SSO in the system browser (ADR-066).

## D Technology: targets, builds and prerequisites (ADR-067)
- Targets: desktop installer under 15 MB; tray idle under 60 MB with the WebView unloaded; phone download under 20 MB. "Ready for calls" keeps the WebView loaded (ADR-064).
- Linux: WebKitGTK lacks reliable WebRTC, so Linux calls open in the default browser.
- Builds: Mac with Xcode for iOS and macOS (sign, notarise); home cluster pods for Android and Linux; a Windows machine for MSI/NSIS.
- Prerequisites the user supplies: Apple Developer Program (organisation, D-U-N-S), Developer ID and Distribution certs, APNs `.p8` key, App Store Connect API key; Google Play organisation account, upload key, Firebase project and service account; Windows code-signing cert or Azure Trusted Signing; Tauri updater key pair; privacy policy and support URLs; a demo tenant for store review. All secrets live in env vars (E10).
- Release: bump version, build, sign, notarise, TestFlight and Play internal track, review, staged rollout. Desktop gets a signed update manifest.
