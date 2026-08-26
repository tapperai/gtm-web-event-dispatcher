# GTM Event Dispatcher — Environment Spinup

> **Status:** `SHIPPED`
>
> **Created:** 2026-08-26
> **Last updated:** 2026-08-26
>
> **Implemented in:** gtm-web-event-dispatcher

## Overview

This repo has **no infrastructure to spin up**. It is a single-file Google
Tag Manager Community Gallery template (`template.tpl`) — no server, no
database, no queue, no container, no cloud project of its own. Verified
2026-08-26: no `package.json`, no Dockerfile, no CI workflow, no
infra-as-code in this repo.

The only "environment" is:

1. A Google Tag Manager account with permission to create/edit custom
   templates and containers (for testing — see `docs/TESTING.md`).
2. The GTM Community Template Gallery submission flow (for publishing — a
   manual, human-driven UI action, not automated by anything in this repo).

The one real dependency is the receiving endpoint,
`https://api.tapper.ai/gtm/track`, which is owned and deployed by the
`back-end` / `tracker` repos — see their own `ENVIRONMENT_SPINUP.md` /
`docs/` for that side.

Every other section of the standard Environment Spinup template
(Cloud Services, Databases, Event Stores/Message Queues, Container
Orchestration, CI/CD, Cross-Repo Sync Scripts, Secrets, Spinup Procedure,
Teardown/DR) is **not applicable** to this repo and has been omitted rather
than filled with placeholder text.

## Verification

- Load `template.tpl` in the GTM Template Editor and confirm it parses with
  no errors (Templates → New Custom Template → paste/import).
- See `docs/TESTING.md` for the manual dispatch test.
