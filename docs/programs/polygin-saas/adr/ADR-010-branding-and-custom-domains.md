# ADR-010: Branding keys in `Organization.Settings`; custom domains in `TenantDomain`

Status: proposed (G1, module saas). Requirements: SAAS-07, SAAS-08.

## Context
- Each tenant sets a logo, colours and a custom domain.
- `Organization.Settings` (JSONB) already holds tenant preferences such as timezone.
- Hostname lookups happen before login. They need a unique index.
- Rule E25 bans raw colours in the UI. All colour comes from TonalDepth tokens.

## Decision
- Store the brand-ui branding payload under `Settings.branding`: brand colours per theme, logo and favicon files (ADR-083). The name comes from `Organization.Name`; no product name is stored.
- Store logo files with the existing `[storage]` backend (local or S3).
- The colours are TonalDepth token overrides. The `tonaldepth-vue` contrast guard (4.5:1, light and dark) is the one rule. The screen shows the adjusted colour before save; the server checks format only (ADR-083).
- Add `TenantDomain` with a unique hostname, a DNS TXT token and a status (pending, verified, active, failed).
- Activate a domain only after the TXT check passes.
- The TLS proxy asks Polygin before it issues a certificate. Polygin says yes only for active domains.
- On a custom domain, the session's org must match the domain's org. Otherwise the request fails.
- Serve `GET /api/public/branding` by host for the login page. It returns only public brand fields.
- Serve `GET /api/branding` for the session org. The web app and the shells call it after login and after an org switch (ADR-083).
- The AGPL source link stays in the footer on every tenant. Branding cannot remove it.
- Gate branding and custom domains with plan features `feature.branding` and `feature.custom_domain`.

## Alternatives
- Branding columns on `Organization`. Typed, but adds columns for every new brand field.
- Custom domains as a JSONB key. No unique index, slow host lookup.

## Consequences
- Needs a TLS proxy with on-demand certificates (open question S6).
- The `brand-ui` module owns the payload shape, the token override mechanism and the footer component.
