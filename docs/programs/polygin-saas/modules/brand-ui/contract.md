# brand-ui: theme contract, component inventory and rename surface (polygin-saas)
<!-- Detail for `G1.md`. Requirement IDs BRAND-01 to BRAND-03, SAAS-07 (theme part), SAAS-08. Status: proposed. Moved out of G1 in the merge pass, 2026-11-19. -->

## C Application: branding payload and theme runtime (ADR-022, ADR-083)
- Payload shape (brand-ui owns it; saas stores it in `Organization.Settings.branding`):
  - `colors.light` and `colors.dark`, each with `brand`, `brand_deep`, `brand_text`.
  - `logo_light`, `logo_dark`, `favicon`: file IDs in `internal/storage`.
  - `display_name`: filled by the server from `Organization.Name`. Never stored in branding.
- `applyBranding(payload)` in `tonaldepth-vue` writes `<style id="td-tenant">`. It overrides only `--td-brand`, `--td-brand-deep`, `--td-brand-text`.
- Contrast guard: brand text 4.5:1 on `--td-surface`; white label 4.5:1 on the brand fill. A failing colour is darkened or lightened until it passes.
- The guard is the one contrast rule. The saas branding screen runs it before save and shows the result.
- Where the payload comes from: `embed.go` injection on verified custom domains; `GET /api/branding` after login and each org switch on `app.polyg.in`.
- Footer (SAAS-08): brand-ui shell footer with the AGPL source link and the build commit from the binary. Branding cannot hide it.

## C Application: shell contract (ADR-026)
- Minimum width 360 px. Bottom nav below 768 px. Dialogs become bottom sheets below 768 px.
- Safe-area tokens from `env(safe-area-inset-*)`. Touch targets 44 by 44 px.
- `data-platform="web|ios|android|desktop"` on `<html>`, set by the apps bridge adapter.
- Tauri title-bar inset token and drag region. TdWindowControls waits on open question X1.
- RTL through logical CSS properties, for the Arabic locale.

## C Application: component inventory
30 primitives in use. 7 map to existing TonalDepth Vue parts. 23 primitives and 17 patterns are to build: 40 in total.

| Group | Fork primitives in use | Exists in tonaldepth-vue | To build (React reference) |
|---|---|---|---|
| Actions | button (80 imports) | TdButton | icon-button |
| Form | input, textarea, label, select, switch, checkbox, radio-group, command, range-calendar | TdInput, TdTextarea, TdFormField | select, switch, checkbox, radio-group, combobox, date-range-picker |
| Display | badge, tag-badge, card, avatar, separator, skeleton, spinner, progress, alert | TdBadge (tag variant), TdCard | avatar, divider, skeleton, spinner, progress, alert |
| Overlay | dialog, alert-dialog, popover, tooltip, dropdown-menu | none | dialog, alert-dialog, popover, tooltip, dropdown-menu, toast |
| Structure | tabs, accordion, collapsible, breadcrumb, scroll-area, table | none | tabs, accordion, collapsible, breadcrumbs, scroll-area, table, pagination |
| Patterns | `shared/`, `layout/`, flow and chart parts | none | application-shell, bottom-nav, theme-toggle, page-state, search-bar, kpi-card, chart-container, canvas, canvas-node, canvas-panel, upload, file-preview, timeline, color-picker |

## C Application: rename surface (ADR-023, ADR-087)
There are 817 hits in 239 files; most are Go import paths.
- Rename: `index.html` title and meta, layout strings, 6 locale files (as an `{appName}` parameter), the `App.Name` default, the health `service` field, the favicon and logo, about 40 docs pages and 2 docs logos, the binary (`-o polygin`), the Docker image, compose services, `POLYGIN_` env variables (with `WHATOMATE_` as fallback) and `frontend/package.json`.
- New names: `pgn_` API keys, `X-Polygin-*` headers, `polygin:` Redis keys (ADR-087).
- Emails: the fork has none today. New emails must use `{appName}` and the tenant brand.
- Keep: the Go module path, the `cmd/whatomate` folder, internal Redis and queue names, `whm_*` cookies, `X-Webhook-Signature`, test fixtures, the LICENSE text, upstream copyright notices, a "based on Whatomate" credit, and the Meta app name (D3).
