# ADR-060: App shells load the hosted web app; they do not bundle the AGPL frontend

Status: proposed (G1, module `apps`). A lawyer must confirm the licence position before release.

## Context
- The Vue frontend is AGPL-3.0. It is embedded in the Go binary.
- A shell that bundles the built frontend is very likely a combined work, so the shell becomes AGPL.
- We are not the sole copyright holder of Whatomate, so we cannot relicense it. App Store terms add use limits. The FSF considers those limits in conflict with the GPL family.
- Login sets SameSite=Lax httpOnly cookies and returns no tokens in the body. A bundled page runs on `capacitor://localhost` or `tauri://localhost`. That origin is cross-site, so cookies fail and every call needs CORS.

## Decision
- Each shell loads `https://app.polyg.in` in the system WebView.
- The shell ships only our own code: an offline and error page, the native bridge, push, deep links, tray and updater.
- The AGPL frontend talks to the shell through injected globals (`window.Capacitor`, `window.__TAURI__`). The frontend adds no shell npm package.
- Navigation outside `app.polyg.in` opens the system browser.

## Alternatives
- Bundle the frontend in the shell. Gains: offline start, faster cold start. Costs: likely AGPL shell, App Store licence risk, Bearer-token auth rework, CORS for every origin, UI lags the server until store review passes (1 to 3 days), API drift between app and server versions.

## Consequences
- UI updates ship with each server deploy. No store review for UI changes.
- No offline start. The shell shows its own offline page with retry.
- Apple guideline 4.2 rejects thin web wrappers. We counter with native push, badges, deep links, camera and microphone, and the tray on desktop.
- Capacitor documents `server.url` mainly for live reload. A G3 spike confirms push and bridge access on the remote origin on iOS and Android.
- Tauri grants IPC to the remote origin through a `remote.urls` capability. Grant only notification, badge and deep-link commands.
- The shell licence is our choice. The server keeps its AGPL source link (SAAS-08).
- Lawyer to confirm: the remote shell is a separate program; the bridge calls do not make a combined work; our shell licence choice.
