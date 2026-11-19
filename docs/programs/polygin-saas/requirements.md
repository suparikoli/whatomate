# Polygin SaaS: requirements and G0 reuse check

Updated 2026-11-19. Gate: G0. Base: fork `suparikoli/whatomate` (Go, Vue 3, PostgreSQL, Redis). Licence: AGPL-3.0 open core.

Mark: `exists` (no work), `extend` (owner exists, add to it), `new` (nothing to reuse).

## Confirmed scope (ADM phase A)

- Goal: Polygin, a white-labelled SaaS messaging platform on our Whatomate fork.
- Roles: Superadmin (platform), Admin (customer company), Agent.
- Release 1: SaaS layer, rebrand, TonalDepth redesign, connected apps, desktop helper, mobile apps.
- Release 2: Telegram, Instagram, email, web chat widget, Signal (optional sidecar).
- Plugins: ERPNext, Medusa, FieldOps, merqPOS, TradeTerminal, Tally. Closed source, own repos.

## Release 1

| ID | Requirement | Mark | Owner and where |
|---|---|---|---|
| CORE-01 | Keep all WhatsApp API messaging, templates, campaigns, catalog | exists | fork: `handlers/messages.go`, `templates.go`, `campaigns.go` |
| CORE-02 | Keep message and flow builder, keywords, conditional replies | exists | fork: `handlers/chatbot*.go`, `flows.go`, views `chatbot/` |
| CORE-03 | Keep AI caller, IVR, call transfers | exists | fork: `internal/calling`, `handlers/ivr_flows.go`, views `calling/` |
| CORE-04 | Keep Embedded Signup with QR coexistence and history sync | exists | fork: `handlers/coexistence.go`, `accounts.go` |
| CORE-05 | Keep teams, custom roles, SLA, canned replies, tags, audit, SSO, analytics | exists | fork: matching handlers and settings views |
| SAAS-01 | One tenant per customer company | exists | fork: `Organization` model, org scoping in middleware |
| SAAS-02 | Admin and Agent roles per tenant | exists | fork: `models/roles.go`, custom roles |
| SAAS-03 | Superadmin console: tenants, usage, suspend, impersonate | extend | fork: `User.IsSuperAdmin` flag exists; no console |
| SAAS-04 | Plans, limits and usage metering | new | fork: new `billing` package |
| SAAS-05 | Subscription billing with Cashfree | new | fork: `billing`. Catalogue `@mithtech-medusa/cashfree` is Medusa-only: reuse the flow, not the code |
| SAAS-06 | Self-service signup and tenant onboarding | extend | fork: `auth.go`, `RegisterView.vue` |
| SAAS-07 | Per-tenant branding: logo, colours, custom domain | new | fork: `Organization` settings |
| SAAS-09 | Superadmin assigns a plan by hand for bundled Mithtech contracts (for example ERPNext implementation and maintenance); no online charge | new | fork: `billing` |
| SAAS-08 | AGPL source link in the app footer | new | fork: frontend layout |
| BRAND-01 | Rename every Whatomate name, logo and string to Polygin | extend | fork: frontend, i18n, docs, binary name |
| BRAND-02 | Redesign all 50 views with TonalDepth, light and dark | extend | fork: `frontend/`. Blocker: `tonaldepth-vue` has 5 components; React has 103 |
| INT-01 | Public API with API keys | exists | fork: `handlers/apikeys.go` |
| INT-02 | Outgoing webhooks | exists | fork: `handlers/webhooks.go`, `webhook_dispatch.go` |
| INT-03 | Connected apps page per tenant | new | fork: new settings view |
| INT-04 | Event intake API with a business key; drop duplicates | new | fork: new `events` package. No idempotency exists today |
| INT-05 | One sender app per record type, set per tenant | new | fork: `events` settings |
| INT-06 | ERPNext plugin | extend | Frappe app `polygin`. It calls the old `polyg.in/api` today; repoint |
| INT-07 | Medusa plugin | extend | `@mithtech-medusa/communication`. It calls the old `polyg.in/api` today; repoint |
| INT-08 | FieldOps, merqPOS, TradeTerminal plugins | new | one module in each app. `INTEGRATION.md` is the contract start |
| INT-09 | Tally connector | new | new small connector; placement in G1 |
| APP-01 | Push notification service (FCM, APNs, desktop) | new | fork: new `push` package. `ntfynext` does not fit phones |
| APP-02 | Desktop helper for macOS, Windows, Linux (Tauri) | new | new repo; needs approval (E23) |
| APP-03 | iOS and Android apps: inbox, messaging, notifications (Capacitor) | new | new repo; needs approval (E23) |
| APP-04 | Mobile apps show the plan and its status; buying happens on the web | new | mobile repo; no in-app purchase |

## Release 1: added in the G1 merge (2026-11-19)

The G1 packs imply these rows. Existing IDs are unchanged. "Pack" names the G1 pack that covers each row.

| ID | Requirement | Mark | Owner and where | Pack |
|---|---|---|---|---|
| SAAS-10 | Send transactional email: verification, invites, billing and Superadmin notices | new | saas; sender choice is open question S5 | saas |
| BRAND-03 | One responsive shell from 360 px, with safe areas, for web, phone and desktop | extend | fork `frontend/` layout, `tonaldepth-vue` patterns (ADR-026) | brand-ui |
| INT-10 | Report delivery status to the sending app (`message.status` webhook) | new | fork `webhook_dispatch.go`, `updateMessageStatus` (ADR-047) | connect |
| INT-11 | Admin reviews `needs_review` events and replays dead webhook deliveries | new | connect `internal/events`, connected apps page (ADR-046, ADR-047) | connect |
| APP-05 | Per-user push preferences and quiet hours; per-tenant preview privacy | new | fork `internal/push` (ADR-063, ADR-084) | apps |
| SEC-01 | Joining an existing tenant needs a signed, single-use invite. Close join-by-organization-ID | extend | fork `handlers/auth.go` `Register` (ADR-009) | saas |
| SEC-02 | A Superadmin enters a tenant only through an audited session with a reason and expiry | extend | fork `handlers/app.go`, `auth.go` `SwitchOrg`, new `ImpersonationSession` (ADR-008) | saas |
| SEC-03 | The WebSocket accepts only ws-subject tokens. It rejects refresh and access tokens | extend | fork `validateWSTokenFn` | apps |
| SEC-04 | Store webhook secrets encrypted at rest. Never show them during impersonation | extend | fork `webhooks` with `internal/crypto` (ADR-040) | connect |
| SEC-05 | Deliver webhooks durably: stored outbox, 8 retries, dead letters, replay | extend | fork `webhook_dispatch.go`, new `webhook_deliveries` (ADR-047) | connect |
| SEC-06 | API keys carry scopes. Connected-app keys act only within their scopes | extend | fork `api_keys`, `middleware.go` (ADR-040, ADR-085) | connect |
| SEC-07 | Sign webhooks with a timestamp, so receivers can reject replays | extend | fork `webhook_dispatch.go` (ADR-047) | connect |
| SEC-08 | Test tenant isolation with two tenants over every endpoint | new | fork test suite | saas |
| SEC-09 | A custom domain serves only its own tenant. Certificates issue only for verified domains | new | fork `handlers/branding.go`, `TenantDomain` (ADR-010) | saas |
| SEC-10 | Accept Cashfree webhooks only when signed. Store each once. Map the tenant from our stored ID | new | fork `internal/billing` (ADR-005) | saas |
| SEC-11 | Superadmins sign in with a second factor | new | fork `handlers/auth.go`; method is open question S7 | saas |
| SEC-12 | Push payloads carry no phone numbers, media or message ids, and text only up to the tenant preview level | new | fork `internal/push` (ADR-063) | apps |
| SEC-13 | Encrypt push tokens. Hash desktop device secrets. Revoke devices on sign-out, removal and password reset | new | fork `internal/push` (ADR-062, ADR-085) | apps |
| SEC-14 | Verify the signup email before the tenant's first outbound send | new | fork `handlers/auth.go` (ADR-009) | saas |

## Release 2

| ID | Requirement | Mark | Owner and where |
|---|---|---|---|
| CH-01 | Channel layer: accounts, contacts, messages no longer assume WhatsApp | extend | fork: `models.go`, `messages.go`, `webhook.go`. Largest refactor |
| CH-02 | Telegram | new | fork: channel adapter |
| CH-03 | Instagram DMs and comments | new | fork: channel adapter (Meta Graph API; needs Meta App Review) |
| CH-04 | Email (IMAP and SMTP) | new | fork: channel adapter |
| CH-05 | Web chat widget for customer sites | new | fork: adapter plus embeddable script. `handlers/widgets.go` is dashboard widgets, not this |
| CH-06 | Signal through an optional sidecar | new | sidecar service. Risk: unofficial, can be banned |

## Findings that change the plan

1. `tonaldepth-vue` is at 0.1.0-alpha.0 with 5 components. The redesign needs about 40 more first.
2. A product named Polygin already runs at polyg.in (`polygin-main`: Express, MySQL, dormant). Both existing plugins call its API.
3. Nothing in the fork covers billing, push, branding or duplicate protection.

## Decisions

- D1 (decided 2026-11-19): build out `tonaldepth-vue` to cover the redesign. Each component wraps Reka UI primitives. Work lands in the `tonaldepth` repo as wave 0 of BRAND-02.
- D2 (decided 2026-11-19): start clean. No data or code comes from the old polyg.in server. No migration requirement.
- D3 (decided 2026-11-19): keep `app.polyg.in`. The Meta app stays as registered.
- D4 (decided 2026-11-19): web only. The apps show the plan and its status. Plans can also come bundled with a Mithtech contract (SAAS-09).
