# ADR-041: Event intake API v1 contract and versioning

Status: proposed (G1 connect). Requirement: INT-04.

## Context
- Plugins today call send endpoints with a template name. Each plugin then owns template choice.
- No endpoint accepts a business event. No public route is versioned.

## Decision
- Add `POST /api/v1/events`. It needs scope `events:write`. The body is at most 64 KB.
- Fields: `schema`, `event`, `record_type`, `key`, `occurred_at`, `recipient`, `data`, `origin`, `attachments`.
- Event names use `<record>.<verb>`, for example `invoice.created`.
- Return an outcome, never an error, for a valid event: `accepted` (202); `duplicate`, `ignored_not_sender`, `ignored_synced`, `no_route` (200).
- Keep the existing envelope. Add `code` and `details[]` to errors.
- Error codes: `invalid_payload`, `unknown_record_type` (422); `scope_missing`, `tenant_suspended` (403); `rate_limited`, `plan_quota_exceeded` (429).
- Version by path. Allow only additive changes inside v1. A breaking change gets `/api/v2` with a 12-month overlap.
- Alias the existing routes that plugins use under `/api/v1`.
- Publish the contract as an OpenAPI file in G3.

## Alternatives
- Version by header. Rejected: harder to see in logs and proxies.
- Return 409 for duplicates. Rejected: plugins treat 4xx as failures and alert or retry.
- Batch intake in v1. Deferred: add `POST /api/v1/events:batch` later as an additive change.

## Consequences
- Plugins send business data, not template names.
- Plugins can retry any network failure safely. The business key makes the call idempotent.
