# Open questions (polygin-saas)
<!-- Merged from the four G1 packs, plus questions raised in the merge pass. Written 2026-11-19. 28 open, 2 closed. -->

IDs keep the pack's number: S = saas, B = brand-ui, C = connect, A = apps, X = raised in the merge.
Block: High stops a module's build. Medium stops one feature. Low sets a default or a later item.
The recommended option is first in each row.

## Top three blockers
1. B1: the TonalDepth licence. It blocks wave 0, so every redesigned and new screen.
2. S5: the email channel. It blocks signup verification, invites (SEC-01) and billing notices.
3. C1: the guard order. It blocks the intake logic and every plugin's sync rule.

## brand-ui
| ID | Block | Question | Options | Blocks |
|---|---|---|---|---|
| B1 | High | May `tonaldepth-core` and `tonaldepth-vue` ship under a licence the AGPL frontend can use? | MIT (Recommended); AGPL-3.0; vendor built copies into the fork | Wave 0 release, so BRAND-02 and all new screens (ADR-025) |
| B2 | High | How do AGPL users get the TonalDepth source to build? | Publish both to public npm (Recommended); vendor them in the fork | A compliant frontend release |
| B3 | High | Does wave 0 wait for the tonaldepth version-gap recovery? | Wait (Recommended); release the Vue package alone | Wave 0 timing |
| B5 | Low | What do flow builders show below 1024 px? | Read-only notice (Recommended); hide them | Chatbot and IVR view group (ADR-026) |

## saas
| ID | Block | Question | Options | Blocks |
|---|---|---|---|---|
| S5 | High | Which channel sends transactional email? | SMTP through Go stdlib with a provider account (Recommended); WhatsApp notices only | SAAS-06, SAAS-10, SEC-01, SEC-14, billing and Superadmin notices |
| S2 | High | Who issues tax invoices and GST? | Push each paid Payment to Mithtech's ERPNext as a Sales Invoice (Recommended); build invoices in Polygin | First paid launch (SAAS-05) |
| S3 | High | What happens at the monthly message limit, and do event sends count as campaigns? | Block campaigns and event sends, keep 1:1 replies (Recommended); hard block all; paid overage | `messages_out` checks in send, campaign, worker and connect |
| S8 | Medium | What are the plan names, prices and limits? | User supplies the list (Recommended); launch with one plan | Seed data, Cashfree plans, store review demo |
| S7 | Medium | How do Superadmins prove a second factor? | TOTP (Recommended); SSO only | Platform console (SAAS-03, SEC-11) |
| S6 | Medium | What issues TLS for custom domains? Needs approval (E23) | Caddy on-demand TLS (Recommended); Cloudflare for SaaS | Custom domains (SAAS-07, SEC-09) |
| S1 | Medium | How is a mid-period upgrade charged? | Prorated one-off PG order (Recommended); new full period | Plan change flow |
| S9 | Medium | What mode does impersonation start in? | Read-only, write on a second reason (Recommended); full write always | ADR-008 build |
| S4 | Low | Are grace 7 days, read-only 30 days and deletion 90 days later right? | Confirm (Recommended); change the numbers | Config defaults and tenant purge (ADR-006, ADR-081) |

## connect
| ID | Block | Question | Options | Blocks |
|---|---|---|---|---|
| C1 | High | Accept that guard 3 applies only when a route has no sender? | Accept ADR-045 (Recommended); plugins always skip synced records | Intake guards (INT-04, INT-05), plugin sync rules |
| C3 | Medium | Do Tally and ERPNext keep the same invoice number and GSTIN after sync? | Confirm on a sample sync (Recommended); assume not and rely on origin | Guard 2 for Tally and ERPNext |
| C4 | Medium | Where does the Tally connector run? | Own Windows service on the customer LAN (Recommended; the desktop helper calls no business APIs); inside the desktop helper | INT-09 |
| C2 | Low | Is a 30-day duplicate window right? | 30 days, tenant sets 1 to 90 (Recommended); 90 days by default | Default value only |

## apps
| ID | Block | Question | Options | Blocks |
|---|---|---|---|---|
| A1 | High | Approve the new app repositories (E23)? | Two repos, `polygin-desktop` and `polygin-mobile` (Recommended); one monorepo `polygin-apps` | All APP-02 and APP-03 work |
| A2 | High | Approve shells that load the hosted web app, with a lawyer to confirm the AGPL position? | Approve and engage a lawyer (Recommended); bundle the frontend and license the shells AGPL | APP-02, APP-03, branding in shells (ADR-060, ADR-083) |
| A6 | Medium | Do the Apple and Google organisation accounts exist? | Start enrolment now; D-U-N-S takes weeks (Recommended); they exist | Store builds, push keys, TestFlight |
| A3 | Medium | Which Windows machine and code-signing route? | Azure Trusted Signing on a Windows VM (Recommended); OV or EV cert on an existing machine | Windows release of APP-02 |
| A4 | Low | Where do desktop installers and the update manifest live? | Object storage on a Mithtech download domain (Recommended); GitHub Releases | Desktop auto-update |
| A5 | Low | Are per-tenant white-label store apps out of release 1? | Out of release 1 (Recommended); in release 1 | Nothing in release 1 if out |

## Cross-cutting (raised in the merge)
| ID | Block | Question | Options | Blocks |
|---|---|---|---|---|
| X3 | Medium | What do billing meters do when Redis is down? | Fail open, log and reconcile (Recommended); fail closed | `Consume` in the billing service (ADR-081) |
| X5 | Medium | May an SSO first login join a tenant without an invite? | Only with an invite or an Admin-verified email domain (Recommended); any user of the configured provider | SEC-01 completeness. Check SSO provisioning in G2 |
| X1 | Medium | How does the desktop window get its controls? | Native OS title bar on Windows and Linux (Recommended); grant window IPC to the remote origin and build TdWindowControls | Desktop shell, component 41 (ADR-026, ADR-060) |
| X2 | Low | Do outbound webhooks keep delivering while a tenant is suspended? | Keep delivering (Recommended); pause until active | Webhook worker rule (ADR-047) |
| X6 | Low | When do legacy unscoped `whm_` keys stop working? | Keep in release 1, marked legacy; set a sunset later (Recommended); force scopes now | Nothing in release 1 (ADR-085) |

## Closed
| ID | Question | Answer | Source |
|---|---|---|---|
| B4 | May a tenant replace the name "Polygin" in the UI? | No. Tenants set logo, colours and a custom domain only. A name change needs a new requirement | `requirements.md` SAAS-07; ADR-083 |
| C5 | Should new keys use the `pgn_` prefix? | Yes. `whm_` keys stay valid | Merge pass, ADR-087 |
