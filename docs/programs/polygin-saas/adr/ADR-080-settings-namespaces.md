# ADR-080: Settings JSONB namespaces, one owner each

Status: proposed (merge pass). Modules: saas, connect, apps, brand-ui.

## Context
- Three packs add keys to `Organization.Settings` and `User.Settings` (JSONB).
- apps used `push_preview` as a top-level key at both levels. A flat key space invites collisions.
- Rule E02 needs one owner per setting.

## Decision
- Each module writes only its own namespace, through its own handler.
- `Organization.Settings.branding.*`: saas (`handlers/branding.go`). Shape from brand-ui (ADR-083).
- `Organization.Settings.connect.*`: connect (`internal/events`). Today: `dedupe_window_days`.
- `Organization.Settings.push.*`: apps (`internal/push`). Today: `preview`.
- `User.Settings.push.*`: apps. Keys: `types`, `quiet_hours`, `preview`.
- Existing keys (timezone, `calling_enabled`, `new_message_alerts`, `call_ringtone`, and others) stay with core.
- A plan feature may gate a core key, for example `feature.calling` gates `calling_enabled`. The key keeps its owner.

## Alternatives
- Typed columns per setting. Rejected: many migrations, and more upstream conflict.
- One flat key space. Rejected: no visible owner.

## Consequences
- ADR-063 key names change to `push.preview`.
- A new setting needs a namespace owner in its pack.
