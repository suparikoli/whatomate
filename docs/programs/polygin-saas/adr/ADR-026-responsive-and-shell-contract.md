# ADR-026: Responsive and shell contract for web, Capacitor and Tauri

Status: Proposed (G1, brand-ui). Date: 2026-11-19.

## Context
- APP-03 (Capacitor, iOS and Android) and APP-02 (Tauri desktop) reuse this frontend.
- Today 39 view files use responsive prefixes 95 times. The layout assumes a desktop sidebar.
- The Arabic locale exists, so layouts must work right to left.

## Decision
- Support a minimum width of 360 px.
- Breakpoints: below 768 px, bottom nav and stacked list and detail; 768 to 1023 px, collapsed sidebar; 1024 px and up, full sidebar.
- Chat, contacts, notifications and profile must work fully on phones.
- Settings must work on tablets and phones.
- Flow builders (chatbot, IVR) need 1024 px. Smaller screens get a notice (open question 5).
- Set `viewport-fit=cover`. Add safe-area tokens from `env(safe-area-inset-*)`. Apply them to the header, bottom nav, composer, toasts and dialogs.
- Dialogs become bottom sheets below 768 px.
- Make touch targets at least 44 by 44 px. Give every hover-only action a tap path.
- Keep the composer above the on-screen keyboard.
- Tauri: add a title-bar inset token, a `data-tauri-drag-region` header, and TdWindowControls on Windows and Linux.
- Set `data-platform="web|ios|android|desktop"` on `<html>`. Do not sniff the user agent.
- Use logical CSS properties in every TonalDepth Vue component.
- Use no absolute asset paths. The API base URL comes from configuration.

## Alternatives
- Build separate mobile screens. Rejected: two UIs to keep in step with upstream.

## Consequences
- Playwright screenshots run at 360, 768 and 1280 px, in light and dark, and in RTL for one view per group.
- The mobile and desktop modules own the native plugins and the build.
