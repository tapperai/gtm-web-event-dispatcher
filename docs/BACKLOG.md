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

(none as of 2026-10-01)

---

## P0 — needs verification

- **2026-09-29: receiving endpoint gone.** `api.tapper.ai/gtm/track` returns
  404. back-end stopped serving it when its Express-to-Fastify rewrite
  (`1df7f45a`) reached prod on 2026-01-27; the last image with the route was
  `fd857e33` (2026-01-26). `dcc475b3` (2026-02-26) only deleted the dead file.
  The open decision is retire or restore; see the spec's Remaining Work for
  every step on both paths, including the back-end and estate work.
- **2026-10-01: is anyone still firing the template?** A measurement is
  running: `/gtm/track` is carved out of the 1% load-balancer log sampling
  since 2026-10-01 04:00 UTC, with a calibration probe sent at 04:02:19 UTC.
  Read on or after 2026-10-15 and drop the carve-out after; the command and
  the yes/no reading are in the spec's Remaining Work item 1.

---

## P1 — open

(none as of 2026-10-01)

---

## P2 — open

(none as of 2026-10-01)

---

## Meta / hygiene

- Keep this file fresh: each new explicit user ask gets an entry **before** I
  start coding it. After it ships and the user confirms, move it under Done.
- Never silently drop an item — if I push back on scope, note the rationale
  inline.

---

## Done

- **2026-10-01: moot, `flawless-fixes` merge decision.** Its tclid fix would
  still post to the 404, so merging it changes nothing while the endpoint is
  gone. It comes back only on the restore path (spec, Remaining Work item 3).
- **2026-10-01: settled, the "RETIRED-IN-EFFECT" estate-board verdict
  (2026-08-26).** The 404 proves it: the template cannot deliver an event.
  Formal retirement (archive, Gallery removal) is still open above.

- **2026-08-26** — wired `document-first-template` submodule + bootstrapped
  `docs/` (this commit, operator sweep 2026-08-26).
