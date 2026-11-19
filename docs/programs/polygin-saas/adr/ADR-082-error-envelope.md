# ADR-082: One error envelope and one code registry

Status: proposed (merge pass). Modules: all.

## Context
- saas defined codes such as `plan_limit_reached` and `tenant_suspended`.
- connect added `code` and `details[]` to the existing envelope, with its own codes.
- apps must handle codes from both in the shells.

## Decision
- Keep the existing error envelope. Add `code` (snake case) and `details[]` (objects) for every module.
- Keep one code registry in the core response helpers. Each module adds its codes there by pull request.
- Shared codes: `plan_limit_reached` (402), `plan_feature_off` (403), `tenant_suspended` (403), `tenant_blocked` (403), `scope_missing` (403), `impersonation_forbidden` (403), `rate_limited` (429), `invalid_payload` (422).
- Module codes keep their module prefix only when two modules could clash.

## Alternatives
- Per-module envelopes. Rejected: every client parses several shapes.
- RFC 7807 problem details. Rejected for now: breaks existing clients.

## Consequences
- The frontend maps codes to i18n strings once.
- Plugins switch on `code`, never on message text.
