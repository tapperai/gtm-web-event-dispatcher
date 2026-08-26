# GTM Event Dispatcher

> **Status:** `SHIPPED`
>
> **Created:** 2026-08-26
> **Last updated:** 2026-08-26
>
> **Implemented in:** gtm-web-event-dispatcher

## Overview

A single Google Tag Manager (GTM) Community Template Gallery **custom tag**
called "Tapper - Event Dispatcher". Once added to a customer's GTM container
and configured with an "Event Name" parameter, it fires on the page,
reads the `tclid` (tracker click id) out of the browser's `localStorage`, and
sends a tracking pixel to `https://api.tapper.ai/gtm/track` carrying the
`tclid` and the event name. The endpoint is owned by the `back-end` /
`tracker` repos, not this one. The whole thing is one file, `template.tpl`,
written in GTM Sandboxed JavaScript (a restricted JS dialect executed inside
Google's sandbox — not Node, not browser JS). There is no build step, no
server, no package manager, and no dependency on the rest of the tapperai
monorepo-of-repos.

**Repo status (verified 2026-08-26):** the estate-board verdict for this repo
was "RETIRED-IN-EFFECT". That is **not** confirmed by the repo itself:
- The GitHub repo `tapperai/gtm-web-event-dispatcher` is **not archived**
  (`isArchived: false`).
- The repo's true most-recent activity is on the `flawless-fixes` branch, not
  `master`: `9805e41` (**2026-06-28**) fixes a real bug (tclid truncation, see
  below), and `274b1dc` (**2026-07-12** — six weeks before this sweep) does a
  `CLAUDE.md` rewrite. `master`'s own HEAD, `ffe6c6f`, is a stale
  `metadata.yaml` bump from **2024-03-12** and touches neither the tclid fix
  nor `CLAUDE.md`.
- That most-recent work lives on a branch, `gtm-web-event-dispatcher/flawless-fixes`,
  which was **pushed directly and never opened as a PR** and is **not merged
  into `master`** (GitHub's default branch, per `git remote show origin`).
  `master` is still on the older, buggier code (`ffe6c6f`'s parent chain).
- There are no open or closed PRs on the repo (`gh pr list --state all` is
  empty) — so "never reviewed / never merged" is the accurate description,
  not "retired".
- The root `tapper /CLAUDE.md` repo directory does **not** mark this repo
  archived (unlike `front-end-new`, `ml-modelling`, `ai-suggestions`, etc.,
  which carry an explicit ⛔ ARCHIVED note).

So the honest state as of 2026-08-26 is: **abandoned-in-place, not formally
retired** — a small, still-functional GTM template with unmerged fixes sitting
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
        api.tapper.ai/gtm/track  (owned by back-end / tracker, not this repo)
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

Not a server — this is a client emitting a GET-style tracking pixel. The
receiving contract is owned by `back-end` / `tracker`:

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

1. **gtm-web-event-dispatcher: merge or formally close `flawless-fixes`** —
   the branch has a real bug fix (tclid truncation) sitting unmerged and
   unreviewed since 2026-07-12. Either open the PR tapper PR law requires and
   merge it into `master`, or explicitly decide (and record, e.g. in this
   spec or the root `tapper /CLAUDE.md` table) that the repo is retired and
   the fix isn't worth shipping.
2. **gtm-web-event-dispatcher: confirm live Gallery usage** — no signal in
   this repo says whether any customer's GTM container currently references
   the published template. If nobody uses it, the repo should be marked
   ⛔ ARCHIVED in the root index per the "archive both never delete" rule
   instead of left in this ambiguous state.
