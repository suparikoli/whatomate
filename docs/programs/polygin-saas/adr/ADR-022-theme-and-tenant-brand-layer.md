# ADR-022: Theme modes and the tenant brand layer

Status: Proposed (G1, brand-ui). Date: 2026-11-19.

## Context
- `tonaldepth-core` switches themes with `data-theme="light|dark"` and falls back to `prefers-color-scheme`.
- The fork's `useColorMode` sets `.dark` or `.light` classes.
- SAAS-07 needs a per-tenant logo, colours and custom domain. Claymorphism depends on the ground, surface and shadow tokens staying fixed.
- `internal/frontend/embed.go` already injects `window.__BASE_PATH__` into `index.html`. It caches one copy.
- Capacitor and Tauri load the hosted app on `app.polyg.in` (ADR-060). That host names no tenant, so the server cannot inject a tenant brand there.

## Decision
- `useColorMode` sets `data-theme` on `<html>`. It also sets the class until step 5. System mode removes the attribute.
- A tenant may override only `--td-brand`, `--td-brand-deep` and `--td-brand-text`, each for light and dark.
- `tonaldepth-vue` provides `applyBranding(payload)`. It validates contrast before it writes a `<style id="td-tenant">` block. Brand text needs 4.5:1 on `--td-surface`. The white label needs 4.5:1 on the brand fill. A failing colour is darkened or lightened until it passes.
- The payload holds: brand colours per theme, logo URLs for light and dark, and a display name read from `Organization.Name` (ADR-083).
- Custom domains: `embed.go` injects the style block and `window.__BRAND__`, cached per host. The saas module builds this.
- `app.polyg.in`, in browsers and shells: after login and each org switch, the app calls `GET /api/branding`, applies it and caches it for the next start.
- Default: Polygin assets and core token values.

## Alternatives
- Let tenants set any token. Rejected: it breaks depth and contrast.
- Apply branding only at runtime on the web. Rejected: the page shows the default colours first.

## Consequences
- The saas module owns storage, the endpoint and injection. brand-ui owns the payload shape and `applyBranding`.
- Mithtech surfaces stay the same for every tenant. Only the papaya role changes.
