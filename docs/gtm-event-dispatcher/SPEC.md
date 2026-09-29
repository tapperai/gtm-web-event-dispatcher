# GTM Event Dispatcher

> **Status:** `DEPRECATED`
>
> **Created:** 2026-08-26
> **Last updated:** 2026-09-29
>
> **Implemented in:** gtm-web-event-dispatcher

## Overview

A single Google Tag Manager (GTM) Community Template Gallery **custom tag**
called "Tapper - Event Dispatcher". Once added to a customer's GTM container
and configured with an "Event Name" parameter, it fires on the page,
reads the `tclid` (tracker click id) out of the browser's `localStorage`, and
sends a tracking pixel to `https://api.tapper.ai/gtm/track` carrying the
`tclid` and the event name. The whole thing is one file, `template.tpl`,
written in GTM Sandboxed JavaScript (a restricted JS dialect executed inside
Google's sandbox — not Node, not browser JS). There is no build step, no
server, no package manager, and no dependency on the rest of the tapperai
monorepo-of-repos.

**Receiving endpoint is gone (verified 2026-09-29):** `https://api.tapper.ai/gtm/track`
returns 404 with back-end's own `Route GET:/gtm/track not found` reply. It was
an Express route in back-end. back-end's Express-to-Fastify rewrite (`1df7f45a`,
2026-01-24) stopped mounting it, and prod has not served it since back-end image
`597ee213` deployed on 2026-01-27. The last prod image with the route was
`fd857e33` (2026-01-26), and no later image restored it. `dcc475b3` (2026-02-26,
on back-end master 2026-03-10) only deleted the file, which was already
unmounted. No route for it exists on back-end or tracker `master`/`dev`, and
tracker never had one. Every version of this template, including both published
versions (`26093be`, `75029f9`) and every branch, may only send to this URL: its
`send_pixel` permission allows nothing else. So every pixel lands on a 404 and
the template does nothing useful today.

**Repo status (verified 2026-08-26, re-checked 2026-09-29):** the estate-board verdict for this repo
was "RETIRED-IN-EFFECT". That is **not** confirmed by the repo itself:
- The GitHub repo `tapperai/gtm-web-event-dispatcher` is **not archived**
  (`isArchived: false`).
- The most recent change to the template code is on the unmerged
  `flawless-fixes` branch, not `master`: `9805e41` (**2026-06-28**) fixes a real bug (tclid truncation, see
  below), and `274b1dc` (**2026-07-12** — six weeks before this sweep) does a
  `CLAUDE.md` rewrite. Before PR #2, `master` HEAD was `7323684`, which is docs and
  submodule only: PR #1 (`febf7bc`) wired the docs on 2026-08-27, then `7323684`
  switched the submodule URL to https the same day. The last `template.tpl`
  change on `master` is `7a7be66` and the last `metadata.yaml` change before
  PR #2 is `ffe6c6f`, both 2024-03-12. PR #2 (2026-09-29, the deprecation
  docs) edits only `metadata.yaml`'s `documentation:` line; `versions:` is
  untouched.
- That most-recent work lives on a branch, `gtm-web-event-dispatcher/flawless-fixes`,
  which was **pushed directly and never opened as a PR** and is **not merged
  into `master`** (GitHub's default branch, per `git remote show origin`).
  `master` is still on the older, buggier code (`ffe6c6f`'s parent chain).
- Before PR #2 (this deprecation docs change, opened 2026-09-29) there was
  one PR: #1 (the docs wiring, merged 2026-08-27). `flawless-fixes` was
  never opened as a PR, so its fix was never reviewed or merged.
- A remote branch `main` also exists at `9805e41`, the tclid fix commit and
  parent of `flawless-fixes`. `main` was the original default branch. `master`
  was created at `ffe6c6f` and `main` deleted on 2026-06-15, then `main` was
  re-created at `9805e41` on 2026-06-28. The default branch is `master`.
- The root `tapper /CLAUDE.md` repo directory does **not** mark this repo
  archived (unlike `front-end-new`, `ml-modelling`, `ai-suggestions`, etc.,
  which carry an explicit ⛔ ARCHIVED note).

So the honest state as of 2026-08-26 is: **abandoned-in-place, not formally
retired** — a small GTM template whose receiving endpoint no longer exists, with unmerged fixes sitting
on a stale feature branch, and no repo-level signal (archival, PR, changelog)
that anyone decided to stop maintaining it. Whether any live GTM container
still uses this template's published gallery version could not be verified
from this repo (GTM Gallery usage isn't visible from git).

---

## Architecture

```
Customer's website (GTM container)
    |
    |  page loads, tag fires (Tapper - Event Dispatcher)
    v
GTM Sandboxed JS runtime
    |
    |-- require('localStorage').getItem("tclid")   [read-only grant]
    |-- data["Event Name"]                          [user-configured tag param]
    |
    |  both present?
    ├── NO  ──> data.gtmOnSuccess()  (no-op, tag still reports success)
    └── YES ──> build URL:
                https://api.tapper.ai/gtm/track?tclid={encoded}&event={encoded}
                |
                v
                require('sendPixel')(url, gtmOnSuccess, gtmOnFailure)
                |
                v
        api.tapper.ai/gtm/track  (back-end stopped serving it 2026-01-27; returns 404)
```

---

## Schema

No schema owned by this repo. The only "data model" is the GTM template's own
parameter list (`___TEMPLATE_PARAMETERS___` in `template.tpl`):

| Field (GTM name) | Type | Required | Read in JS as |
|---|---|---|---|
| `Event Name` | `TEXT` | de facto required (pixel only fires when truthy) | `data["Event Name"]` |

---

## Contracts

Not a server — this is a client emitting a GET-style tracking pixel. There is no
receiving contract any more. back-end stopped serving this path on 2026-01-27
(Fastify rewrite `1df7f45a`); the dead Express route was deleted later in
`dcc475b3`. What the template sends:

| Method | Path | Query params sent by this template |
|--------|------|-------|
| `GET` (pixel) | `https://api.tapper.ai/gtm/track` | `tclid={localStorage tclid}`, `event={configured Event Name}` |

---

## Routes

N/A — no server code in this repo.

---

## Operational Procedures

### Publishing a new version to the GTM Community Template Gallery

1. Edit `template.tpl` (the only source file).
2. Load it into the GTM Template Editor and run the built-in preview/test
   (the `___TESTS___` block + "Run code") — there is no local compiler or CI
   gate.
3. Bump `metadata.yaml` with the new commit SHA and a `changeNotes` line.
4. Submit through the GTM Community Template Gallery UI manually (no CI/CD —
   `git push` alone does not publish anything to customers).

**Verified drift as of 2026-08-26:** `metadata.yaml` on `master` records only
two versions (`26093be` Initial Release, `75029f9` "Update identifier
retrieval method"). The `flawless-fixes` branch's tclid-truncation fix
(`9805e41`) has **no corresponding `metadata.yaml` entry** — i.e. even if that
branch were merged, the changelog step of the publish procedure was never
completed for it.

---

## Edge Cases

- **No `tclid` in localStorage, or empty `Event Name`**: the tag no-ops and
  still calls `data.gtmOnSuccess()` — this is required by GTM (an un-called
  success/failure callback hangs the tag), and is why the `else` branch exists.
- **`tclid` truncation bug (fixed only on the unmerged branch):** `master`'s
  `template.tpl` calls `localStorage.getItem("tclid")[0]`, indexing `[0]`
  into the result and so truncating `tclid` to its first character before it
  is placed in the pixel URL. The `flawless-fixes` branch's commit `9805e41`
  ("Fix tclid truncation, dead doc link, changelog SHA") fixes this — see
  `git show 9805e41` in this repo for the exact diff — but that fix is **not
  live on `master`** and therefore not in whatever was last published to the
  Gallery from `master`.

---

## Testing

See `docs/TESTING.md` — there is no automated test framework or simulator
script for this repo; testing is the GTM Template Editor's built-in
`___TESTS___` block + manual preview mode.

---

## Files

- `template.tpl` — the entire template: `___INFO___` (display metadata, brand
  thumbnail), `___TEMPLATE_PARAMETERS___` (the `Event Name` field),
  `___SANDBOXED_JS_FOR_WEB_TEMPLATE___` (dispatch logic), `___WEB_PERMISSIONS___`
  (sandbox API grants), `___TESTS___` / `___NOTES___`.
- `metadata.yaml` — published-version SHA + changelog log.
- `README.md` — customer-facing Gallery description.
- `CLAUDE.md` — repo pointer (this sweep adds the Document First block; a
  fuller AI-codegen-oriented rewrite exists only on the unmerged
  `flawless-fixes` branch as of 2026-08-26).

---

## Remaining Work

1. **Merging `flawless-fixes` is moot while the endpoint is gone:** the tclid
   fix would still post to a 404.
2. **Decide: retire or restore.** back-end stopped serving
   `api.tapper.ai/gtm/track` on 2026-01-27, so the template is inert. To
   retire, do these steps in order. First, if the template is listed in the GTM
   Community Template Gallery, take it out: Google's documented removal is
   deleting `metadata.yaml` or `LICENSE` from the repo. An archived repo is
   read-only, so this must happen before archiving. Then, per the
   archive-both-never-delete rule, run `gh repo archive`, set the roster
   Repositories row to archived with a dated note, mark the repo ⛔ ARCHIVED in
   the root `tapper /CLAUDE.md` table, and correct its `(LIVE)` row in roster
   `REPOS.md`. To restore instead, ship a receiving endpoint and record which
   one in this spec.
