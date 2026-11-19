# ADR-021: Tailwind for layout only; colours from TonalDepth tokens

Status: Proposed (G1, brand-ui). Date: 2026-11-19.

## Context
- The fork uses Tailwind 3.4 with shadcn HSL variables, a dark-first `.light` variant, and violet, WhatsApp, Facebook and glass palettes.
- Views hold 1,072 raw colour classes and 585 `light:` variants in 101 files. 75 classes use opacity modifiers such as `bg-primary/10`.
- TonalDepth forbids raw colours and outline-only depth.
- Upstream keeps using Tailwind. Removing it from views would conflict with every upstream template change.

## Decision
- Step 1: point Tailwind colour names (`background`, `primary`, `muted`, ...) at `--td-*` tokens. Use a colour function with `color-mix` so opacity modifiers keep working.
- Step 1 also loads `tonaldepth-core` tokens and fonts, replacing Inter.
- Steps 2 and 3: as each view group migrates, replace raw palette classes and `light:` variants with tokens and TonalDepth components.
- Add a no-raw-colour check. It runs on migrated folders first, then on the whole tree.
- Step 5: remove the custom palettes, the `light` variant plugin, the `.dark/.light` CSS, the shadcn HSL variables and the bridge.
- Keep Tailwind for layout, spacing and responsive utilities. Do not upgrade to Tailwind 4 in this program.

## Alternatives
- Remove Tailwind entirely. Rejected: large upstream conflict surface, and no gain for layout.
- Big-bang restyle without a bridge. Rejected: half-migrated views would mix two looks for weeks.

## Consequences
- Untouched views show TonalDepth colours from step 1, though not yet claymorphic depth.
- Chat bubble CSS in `assets/index.css` moves to tokens during the chat group.
- After step 5, a raw colour in a view fails the check.
