# ADR-042: The platform computes the business key from a record type registry

Status: proposed (G1 connect). Requirement: INT-04.

## Context
- The same invoice can arrive from Tally and from ERPNext.
- If each plugin built its own key, two plugins could build different keys for one invoice.

## Decision
- Keep a record type registry in code (`internal/events/recordtypes.go`). The platform owns it.
- Each record type lists its key fields. Example: `sales_invoice` uses `gstin`, `fiscal_year`, `number`.
- Include the fiscal year. Indian invoice numbers can restart each fiscal year.
- Plugins send the key fields in `key`. They never send a hash.
- Normalise: trim, uppercase, collapse inner spaces. Do not strip leading zeros.
- Hash with SHA-256 over `record_type | event | fields in registry order`. Scope it to the tenant.
- Different events on one record are different keys. `invoice.created` and `invoice.paid` both go out.
- Seed the full record type list in G3, one entry per plugin record.

## Alternatives
- Plugin-built `business_key` string. Rejected: no single owner for the rule.
- Hash the whole payload. Rejected: two apps format the same invoice differently.

## Consequences
- A new record type needs a core release. This keeps the rule in one place.
- Guard 2 fails if a sync changes the invoice number or GSTIN. Guards 1 and 3 then remain.
