# whatomate: handoff

Updated 2026-11-19.

## What it is

- Open-source WhatsApp Business Platform. Single Go binary with an embedded frontend.
- The git repo is `whatomate/src/`. The folder `whatomate/` above it is not a repo.
- Docs site: `docs/` (Astro).

## State

- Default branch: `main`. Work on `wip`.
- Gate: none file. Use `make test` (gotestsum when installed). Build: `make build`. Run: `make run`.
- Config: `config.example.toml` is the template. Do not copy secrets into the repo.
- Deploy: `docker/` and a community Helm chart (linked from the quickstart). No deploy target is recorded.
- Last notable work (`main`): `89214ae` merges PR #1 (mithtech branch). `3961850` adds coexistence history sync.

## Open work

- No handoff or status notes existed in this repo. No open-work list is recorded.

## Read next

- `README.md`, `CONTRIBUTING.md`.
- `docs/`: product and install docs.
