# Backlog

Living list of every TODO the user has called out. When the user mentions
something new, capture it here verbatim before starting work. When something
lands, move it under "Done" with the SHA / date.

Categories:

- **P0 — open**: explicitly asked, not shipped yet.
- **P0 — needs verification**: code says done but never end-to-end checked
  with the user; treat as suspect until verified.
- **P1 — open**: asked-for but lower urgency, or feature-parity items the
  user wants but hasn't blocked on.
- **P2 — open**: nice-to-have, future parity, deferred.
- **Done**: shipped + verified.

---

## P0 — open

(none as of 2026-08-26)

---

## P0 — needs verification

- **2026-09-29: receiving endpoint gone.** `api.tapper.ai/gtm/track` returns
  404. back-end stopped serving it when its Express-to-Fastify rewrite
  (`1df7f45a`) reached prod on 2026-01-27; the last image with the route was
  `fd857e33` (2026-01-26). `dcc475b3` (2026-02-26) only deleted the dead file.
  The `flawless-fixes` merge question is moot. The open decision is retire
  (Gallery removal first, then `gh repo archive`, the roster Repositories row
  and root CLAUDE.md) or restore an endpoint; see the spec's Remaining Work.
- **Merge decision for `flawless-fixes` branch**: this branch (pushed
  2026-07-12, never opened as a PR, never merged) fixes a real `tclid`
  truncation bug in `template.tpl` and rewrites `CLAUDE.md`. Nobody has
  confirmed with the operator whether to merge it, close it, or whether the
  repo is retired and the fix is moot. See `docs/gtm-event-dispatcher/SPEC.md`
  → Remaining Work.
- **"RETIRED-IN-EFFECT" estate-board verdict (2026-08-26)**: not confirmed by
  repo evidence (not archived on GitHub, recent push, no archived marker in
  root `tapper /CLAUDE.md`). Needs an operator decision either way — see the
  same spec's Overview section for the full evidence trail.

---

## P1 — open

(none as of 2026-08-26)

---

## P2 — open

(none as of 2026-08-26)

---

## Meta / hygiene

- Keep this file fresh: each new explicit user ask gets an entry **before** I
  start coding it. After it ships and the user confirms, move it under Done.
- Never silently drop an item — if I push back on scope, note the rationale
  inline.

---

## Done

- **2026-08-26** — wired `document-first-template` submodule + bootstrapped
  `docs/` (this commit, operator sweep 2026-08-26).
