# gtm-web-event-dispatcher — Documentation

> **Approach:** [DOCUMENT_FIRST.md](DOCUMENT_FIRST.md) — Write the spec, then write the code.
>
> **Template:** [SPEC.md](../document-first-template/_templates/SPEC.md) — Copy this to start a new spec.

This is a **single-purpose repo**: one Google Tag Manager custom tag template
(`template.tpl`), distributed via the GTM Community Template Gallery. It has
one domain.

---

## Domains

| Domain | Status | Description |
|--------|--------|-------------|
| [gtm-event-dispatcher](gtm-event-dispatcher/SPEC.md) | `SHIPPED` | GTM custom tag that reads `tclid` from `localStorage` and fires a tracking pixel to `api.tapper.ai/gtm/track`. |

## Other docs

- [BACKLOG.md](BACKLOG.md) — living TODO list for this repo.
- [TESTING.md](TESTING.md) — how to test `template.tpl` (GTM Template Editor, no automated framework).
- [ENVIRONMENT_SPINUP.md](ENVIRONMENT_SPINUP.md) — this repo has no infrastructure to spin up; see that file for what actually applies.
