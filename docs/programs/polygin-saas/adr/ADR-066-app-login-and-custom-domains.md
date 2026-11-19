# ADR-066: Apps sign in on app.polyg.in; SSO uses the system browser

Status: proposed (G1, module `apps`).

## Context
- One platform serves every tenant. Users switch orgs with the existing `SwitchOrg`.
- Custom domains (SAAS-07) are per tenant. iOS associated domains and Tauri remote capabilities are fixed at build time.
- Login sets first-party cookies. The WebView keeps them like a browser.
- Google and others block OAuth inside embedded WebViews.

## Decision
- Every app connects to `https://app.polyg.in`. The login page is the existing web login in the WebView.
- After login and each org switch, the app applies tenant branding from `GET /api/branding` (ADR-083).
- SSO: the page asks the shell to open the provider in the system browser (ASWebAuthenticationSession, Custom Tabs, default browser on desktop). The callback issues a one-time code (Redis, 60 s, single use). The deep link returns it, and the WebView exchanges it for cookies.
- Sign-out calls device delete first, then the existing logout.

## Alternatives
- Ask for a workspace address and load the tenant's custom domain. Rejected for release 1: dynamic origins break universal links and static capabilities. Keep for self-hosted builds later.
- Native login screen with Bearer tokens. Rejected: duplicates the web login and needs tokens in the response body.

## Consequences
- `saas` must keep `app.polyg.in` login working for tenants with custom domains.
- Email links on custom domains open the browser, not the app. Push deep links still open the app.
- The fork gains a small login-code endpoint in `handlers/auth.go`.
