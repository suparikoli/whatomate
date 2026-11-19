# ADR-023: Rename scope for Polygin

Status: Proposed (G1, brand-ui). Date: 2026-11-19.

## Context
- "Whatomate" appears 817 times in 239 files. Most are Go import paths under `github.com/shridarpatil/whatomate`.
- User-visible names: `index.html` title and meta, `AppLayout.vue` (2), 6 locale files (1 to 3 keys each), the config default `App.Name`, the health `service` field, the favicon, and about 40 docs pages and 2 docs logos.
- Build names: `Makefile` `BINARY_NAME`, `docker/Dockerfile`, `docker-compose.yml`, `.goreleaser.yml`, the env prefix `WHATOMATE_`, and `frontend/package.json`.
- No email templates exist in the fork today.
- AGPL-3.0 requires the licence text and the upstream copyright notices to stay.

## Decision
- Rename to Polygin: page title, meta, logo, favicon, layout strings, docs site text and logos, and the `App.Name` default.
- Replace the name in locale strings with an `{appName}` parameter. It reads `App.Name`, and later the tenant display name.
- Name the binary `polygin` with `-o polygin`. Keep the `cmd/whatomate` folder.
- Name the Docker image `polygin`. Rename the compose services.
- Read `POLYGIN_` env variables first and fall back to `WHATOMATE_`.
- Rename the frontend package to `polygin-frontend`.
- Future emails (billing, signup) must use `{appName}` and the tenant brand.
- Keep: the Go module path, `cmd/whatomate`, internal Redis and queue names, test fixtures, the LICENSE text, upstream copyright notices, and a "based on Whatomate" credit in the README, docs and About.
- Keep the Meta app name as registered (D3).

## Alternatives
- Rename the Go module path. Rejected: every Go file changes; every upstream merge conflicts.
- Search and replace every string. Rejected: it breaks merges and the AGPL attribution.

## Consequences
- The rename is a small, reviewable diff.
- Developers still see `whatomate` in import paths. This is expected.
