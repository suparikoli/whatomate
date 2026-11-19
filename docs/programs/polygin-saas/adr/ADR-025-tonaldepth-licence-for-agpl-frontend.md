# ADR-025: TonalDepth licence and access for the AGPL frontend

Status: Proposed. Needs the user's decision (open questions B1 and B2). Date: 2026-11-19.

## Context
- The Polygin frontend is part of the AGPL-3.0 core. It is built into the binary and served to users.
- `tonaldepth-core` and `tonaldepth-vue` are `UNLICENSED` in their `package.json` files. They live on the private registry npm.po5.in.
- AGPL users have a right to the corresponding source and to build it. A private, unlicensed dependency blocks this.
- The fonts are SIL OFL 1.1. They are compatible.

## Decision (recommended)
- License `tonaldepth-core` and `tonaldepth-vue` under MIT.
- Publish both to the public npm registry, or another public source the fork can build from.
- Keep `tonaldepth-react`, `-ui` and the other packages as they are.

## Alternatives
- License the two packages under AGPL-3.0. This works, but limits reuse in closed Mithtech apps.
- Vendor the built packages into the fork under a compatible licence. This works, but copies the design system into the fork.
- Keep them `UNLICENSED`. Rejected: it conflicts with the AGPL obligations.

## Consequences
- Wave 0 cannot release until this is decided.
- Competitors may reuse the TonalDepth Vue look. The Mithtech brand and logos stay protected by trademark, not by copyright.
