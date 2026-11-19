# ADR-061: Two new repositories, polygin-desktop and polygin-mobile

Status: proposed. Needs the user's approval (rule E23).

## Context
- The desktop helper (Tauri, Rust) and the phone apps (Capacitor, Swift and Kotlin projects) have different toolchains, signing and release cycles.
- Neither shell bundles the frontend (ADR-060). They share only a small offline page and the bridge contract.
- The fork stays the single owner of all business code.

## Decision
- Create `polygin-desktop`: Tauri 2 shell for macOS, Windows and Linux. Official plugins only: notification, autostart, deep-link, single-instance, updater.
- Create `polygin-mobile`: Capacitor shell for iOS and Android. Official plugins only: push-notifications, app, status-bar, splash-screen.
- Keep the bridge contract (`frontend/src/native/` in the fork) as the source of truth. Each repo has a contract test against it.

## Alternatives
- One monorepo `polygin-apps` with `desktop/` and `mobile/`. Gains: one place for the shared offline page and the contract. Costs: mixed Rust, Xcode and Gradle builds in one tree; a phone release tags the desktop code too.
- Put the shells in the fork. Rejected: it mixes our shell licence with AGPL code and bloats the core repo.

## Consequences
- Two repos to keep in step with the bridge contract.
- The offline page is duplicated. It is one small page.
- Each repo has its own gate and release notes.
