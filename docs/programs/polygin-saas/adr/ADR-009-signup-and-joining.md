# ADR-009: Self-signup creates a new tenant; joining needs an invite

Status: proposed (G1, module saas). Requirements: SAAS-06, SAAS-01.

## Context
- `Register` (`handlers/auth.go`) takes any `organization_id` and adds the caller with the default role.
- Anyone who learns a tenant UUID can join that tenant. In a SaaS this breaks isolation.
- `CreateOrganization` already seeds system roles in one transaction.

## Decision
- Add a signup flow that creates an `Organization`, its system roles, the Admin membership and a trial `Subscription` in one transaction.
- Extract the seeding part of `CreateOrganization` into one shared function. Do not copy it.
- Require email verification before the first outbound send.
- Keep the existing register rate limit. Add a check against disposable email domains later if abuse appears.
- Close join-by-organization-ID. Joining an existing tenant needs a signed, single-use invite token from an Admin.
- Put the old behaviour behind a config setting, off by default, for single-tenant self-hosters.

## Alternatives
- Superadmin creates every tenant by hand. Safe, but no self-service.
- Keep join-by-ID and hide the UUIDs. Security by obscurity.

## Consequences
- Signup needs a mail sender for verification and invites (open question 5).
- `RegisterView.vue` changes to a company signup form.
