# ADR-083: One branding contract across saas, brand-ui and apps

Status: proposed (merge pass). Updates ADR-010 and ADR-022.

## Context
- ADR-010 stored one accent and a product name, and rejected a failing accent.
- ADR-022 wanted three brand tokens per theme and a display name, and adjusted a failing colour.
- ADR-022 assumed the shells bundle `index.html`. ADR-060 loads the hosted app instead.
- On `app.polyg.in` the host names no tenant. The `whm_access` cookie has path `/api`, so `embed.go` cannot see the session.
- SAAS-07 lists logo, colours and custom domain. It does not list a product name.

## Decision
- brand-ui owns the payload shape: `colors.light` and `colors.dark`, each with `brand`, `brand_deep`, `brand_text`; logo and favicon file IDs for light and dark.
- saas stores that shape in `Organization.Settings.branding`. Logo files use `internal/storage`.
- The display name is `Organization.Name`. It is read, never copied. The product name stays "Polygin".
- The contrast guard in `tonaldepth-vue` is the one rule. The branding screen runs it and shows the adjusted colour before save.
- The server validates format only. `applyBranding()` runs the guard again at load.
- `GET /api/public/branding` resolves by host, for login pages on custom domains.
- `GET /api/branding` resolves by the session org. The web app and the shells call it after login and after each org switch.
- `embed.go` injects the brand only on a verified custom domain. `app.polyg.in` serves the Polygin default first.
- brand-ui owns the shell footer with the AGPL source link and the build commit (SAAS-08). Branding cannot remove it.

## Alternatives
- Server-side contrast check in Go as well. Rejected: two copies of one rule.
- Inject by session cookie in `embed.go`. Rejected: needs a cookie path change and breaks the index cache.

## Consequences
- A tenant on `app.polyg.in` sees default colours until login. This is accepted.
- An API client that skips the screen can store a weak colour. The guard still fixes it at load.
