# ADR-024: Keep upstream merges workable after the redesign

Status: Proposed (G1, brand-ui). Date: 2026-11-19.

## Context
- The fork tracks upstream Whatomate. A redesign of 50 views changes most template files.
- Upstream adds features in `<script>` blocks and templates.

## Decision
- Redesign views in place. Keep file names, paths, component names, props and emitted events.
- Change `<template>` markup and classes. Leave `<script>` logic alone unless a requirement needs it.
- Keep `@/components/ui/*` imports through the shims (ADR-020).
- Put Polygin-only screens in new folders. Other modules own them.
- Keep an `upstream` remote and an `upstream-main` branch. Merge it on a fixed schedule, for example monthly.
- Before each merge, list the upstream views changed since the last merge. Re-check those views in both themes.
- Keep the Go module path and `cmd/whatomate` (ADR-023).

## Alternatives
- Fork the frontend into a separate repo. Rejected: it loses upstream fixes and the embedded build.
- Stop merging upstream. Rejected: it loses security and Meta API fixes.

## Consequences
- Template conflicts still happen. They are resolved by hand, view by view.
- Script-only upstream changes usually merge cleanly.
