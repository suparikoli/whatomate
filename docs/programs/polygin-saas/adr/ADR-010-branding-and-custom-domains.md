# ADR-010: Branding keys in `Organization.Settings`; custom domains in `TenantDomain`

Status: proposed (G1, module saas). Requirements: SAAS-07, SAAS-08.

## Context
- Each tenant sets a logo, colours and a custom domain.
- `Organization.Settings` (JSONB) already holds tenant preferences such as timezone.
- Hostname lookups happen before login. They need a unique index.
- Rule E25 bans raw colours in the UI. All colour comes from TonalDepth tokens.

## Decision
- Store logo (light and dark), favicon, product name and accent under `Settings.branding`.
- Store logo files with the existing `[storage]` backend (local or S3).
- Map the accent to TonalDepth token overrides. Reject an accent that fails 4.5:1 contrast in light or dark mode.
- Add `TenantDomain` with a unique hostname, a DNS TXT token and a status (pending, verified, active, failed).
- Activate a domain only after the TXT check passes.
- The TLS proxy asks Polygin before it issues a certificate. Polygin says yes only for active domains.
- On a custom domain, the session's org must match the domain's org. Otherwise the request fails.
- Serve `GET /api/public/branding` by host for the login page. It returns only public brand fields.
- The AGPL source link stays in the footer on every tenant. Branding cannot remove it.
- Gate branding and custom domains with plan features `feature.branding` and `feature.custom_domain`.

## Alternatives
- Branding columns on `Organization`. Typed, but adds columns for every new brand field.
- Custom domains as a JSONB key. No unique index, slow host lookup.

## Consequences
- Needs a TLS proxy with on-demand certificates (open question 6).
- The `brand` module owns the token override mechanism and the footer component.
