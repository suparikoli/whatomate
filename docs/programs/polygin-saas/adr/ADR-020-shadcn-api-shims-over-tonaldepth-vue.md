# ADR-020: shadcn-API shims over tonaldepth-vue

Status: Proposed (G1, brand-ui). Date: 2026-11-19.

## Context
- The fork's views import 30 shadcn-vue primitives from `@/components/ui/*`. Button alone has 80 imports.
- D1 decided to build the Vue components in `tonaldepth-vue`, wrapping Reka UI.
- `tonaldepth-vue` exports `Td*` names with TonalDepth variants (`primary`, `filled`, ...). shadcn-vue uses `default`, `destructive`, `outline`, `ghost`, `link` and size `icon`.
- Rewriting imports and props in 50 views makes every upstream merge conflict on those lines.

## Decision
- Build each component in `tonaldepth-vue`, with the React package as the reference for API and look.
- Make `reka-ui` ^2 a peer dependency, so the fork keeps one copy (2.7.0 today).
- Keep the fork's `components/ui/<name>/index.ts` paths and export names.
- Replace each shim body with a thin re-export or wrapper of the `Td*` component. Map shadcn variant and size names in the shim.
- Keep lucide icons. TonalDepth components take icons through slots.

## Alternatives
- Rewrite every view to import `Td*` directly. Rejected: large template and script diffs against upstream.
- Restyle the shadcn-vue files in the fork only. Rejected: not reusable by other Mithtech Vue apps; copies the design system.
- Swap lucide for TonalDepth icons now. Rejected: 119 files change; the TonalDepth icon source is still open.

## Consequences
- Views keep their imports. Most redesign diffs are class and markup changes.
- The shim layer is a fork-owned contract. Its tests must cover each variant mapping.
- New views may import `Td*` directly.
