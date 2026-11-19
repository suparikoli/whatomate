# ADR-049: Plugins use only public API v1

Status: proposed (G1 connect). Requirements: INT-06 to INT-09.

## Context
- The core is AGPL-3.0. Plugins are closed programs in their own repos.
- The ERPNext and Medusa plugins call the old `polyg.in/api`. Decision D2 starts clean.

## Decision
- Plugins call only documented `/api/v1` routes over HTTPS. They link no core code.
- Each plugin uses one connected app and its key. It sends `X-Polygin-Plugin: <app>/<version>`.
- Each plugin sends registry key fields and `origin`. It writes `origin` on every record it creates by sync.
- Each plugin treats 200 outcomes as final, honours `Retry-After`, verifies signatures and dedupes by delivery id.
- The core keeps no plugin-specific code. Tenant differences live in event routes.
- The OpenAPI file for v1 is the contract. Plugin packs build against it.

## Alternatives
- Go plugins or embedded scripts in the core. Rejected: they would mix closed code into the AGPL process.
- One shared plugin SDK first. Deferred: write it once two plugins exist.

## Consequences
- Every plugin need becomes a public API change. It goes through ADR-041 versioning.
- The Tally connector placement is still open. It is open question C4 in `../open-questions.md`.
