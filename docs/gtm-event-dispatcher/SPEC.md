# GTM Event Dispatcher

> **Status:** `DEPRECATED`
>
> **Created:** 2026-08-26
> **Last updated:** 2026-10-01
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

**Receiving endpoint is gone (verified 2026-09-29, re-checked 2026-10-01):**
`https://api.tapper.ai/gtm/track` returns HTTP 404 with back-end's own
not-found body, which echoes the full request URL, query string included:
`{"error":"Not Found","message":"Route GET:/gtm/track?tclid=x&event=y not found","status_code":404}`.
It was an Express route in back-end (`gtmTrack`, which published a
`LinkVisitEventsQueueMessage` to RabbitMQ `LINK_VISIT_EVENTS_QUEUE`).
back-end's Express-to-Fastify rewrite (`1df7f45a`, 2026-01-24) stopped
mounting it, and prod has not served it since back-end image `597ee213`
deployed on 2026-01-27. The last prod image with the route was `fd857e33`
(2026-01-26), and no later image restored it. `dcc475b3` (2026-02-26, on
back-end master 2026-03-10) only deleted the file, which was already
unmounted. No route for it exists on back-end or tracker `master`/`dev`, and
tracker never had one. Every version of this template, including both
published versions (`26093be`, `75029f9`) and every branch, may only send to
this URL: its `send_pixel` permission allows nothing else. So every pixel
lands on a 404 and the template does nothing useful today.

**Repo status (verified 2026-08-26, re-checked 2026-10-01):** the estate-board
verdict of 2026-08-26 called this repo "RETIRED-IN-EFFECT". The 404 above
confirms that verdict: the template cannot deliver an event anywhere, so it is
retired in effect. It is **not formally retired**:
- The GitHub repo `tapperai/gtm-web-event-dispatcher` is public and **not
  archived** (`isArchived: false`); `metadata.yaml` is still on `master`, so a
  Gallery listing (if one exists) is still active.
- The most recent change to the template code is on the unmerged
  `gtm-web-event-dispatcher/flawless-fixes` branch, not `master`: `9805e41`
  (**2026-06-28**) fixes a real bug (tclid truncation, see Edge Cases), and
  `274b1dc` (**2026-07-12**) rewrites `CLAUDE.md`. That branch was pushed
  directly and never opened as a PR. Its fix would still post to the 404, so
  merging it is moot while the endpoint is gone.
- `master` (GitHub's default branch) carries docs-only changes since 2024:
  PR #1 (`febf7bc`, merged 2026-08-26 20:10 UTC) wired the docs, `7323684`
  switched the submodule URL to https, PR #3 (`e2f5828`, 2026-09-30) dropped a
  copied `DOCUMENT_FIRST.md`, and PR #2 (2026-10-01, the deprecation docs)
  changes only `metadata.yaml`'s `documentation:` line, leaving `versions:`
  untouched. The last `template.tpl` change on `master` is `7a7be66` and the
  last `versions:` change is `ffe6c6f`, both 2024-03-12.
- Stray branches: `main` sits at `9805e41` (the tclid fix, parent of
  `flawless-fixes`) and still carries its own `metadata.yaml` (second
  version listed as `7a7be66`, the commit that last changed `template.tpl`,
  instead of `master`'s `75029f9`, and no `documentation:` line). `main` was the
  original default branch: `master` was created at `ffe6c6f` and `main`
  deleted on 2026-06-15, then `main` was re-created at `9805e41` on
  2026-06-28. `docs/document-first` is the merged PR #1 head.
- The root `tapper /CLAUDE.md` repo table describes the template as inert with
  "Retire or restore: decision pending"; it does not mark the repo archived.

Whether any customer's GTM container still fires this template is not visible
from git or from the GTM Gallery. A measurement is running; see Remaining
Work item 1.

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

### How the GTM Community Template Gallery publishes

Only the first listing is a manual step (a one-time submission to Google).
After that the Gallery follows this repo's `metadata.yaml`: Google re-reads
it periodically, a new `sha` added under `versions:` becomes the new published
version, and other fields (such as `documentation:`) update the listing the
same way. There is no CI here; `git push` of `template.tpl` alone publishes
nothing, but a pushed `metadata.yaml` change does.

To publish a new version:

1. Edit `template.tpl` (the only source file).
2. Load it into the GTM Template Editor and run the built-in preview/test
   (the `___TESTS___` block + "Run code"); there is no local compiler or CI
   gate.
3. Commit, then add that commit's SHA with a `changeNotes` line at the top of
   `metadata.yaml`'s `versions:` and push. The Gallery picks it up from there.

To remove the listing, Google's documented route is deleting `metadata.yaml`
(or `LICENSE`) from the repo. Do it on every branch that carries one, which
today means the stray `main` too (or delete `main`), because `main` was the
default branch when the template was first submitted.

**Verified drift as of 2026-10-01:** `metadata.yaml` on `master` records only
two versions (`26093be` Initial Release, `75029f9` "Update identifier
retrieval method"). The `flawless-fixes` branch's tclid-truncation fix
(`9805e41`) has **no corresponding `versions:` entry**, so even merged it
would not publish.

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
  `flawless-fixes` branch as of 2026-10-01).

---

## Remaining Work

Retiring or restoring the template is an owner decision this spec does not
make. What each path needs, and the measurement that informs it:

1. **Is anyone still firing it? (measurement running since 2026-10-01 04:13 UTC).**
   The 2026-08-26 reading ("0 requests in 14 days" in load-balancer logs) is
   past the 30-day log retention, so it cannot be re-checked, and a zero
   taken since then proves little: since 2026-08-22 the `_Default` log sink
   keeps only a sample of external load-balancer request lines (5%, then 1%
   from 2026-09-22; exclusion `exclude-lb-sampled`), istio-proxy access lines are
   excluded from Cloud Logging and dropped from OpenObserve for 404s, and
   back-end's not-found handler logs nothing. Over the 30-day Cloud Logging
   retention the 1% sample held exactly one `/gtm/track` line, a
   `curl/8.7.1` GET answered 400 (`body_not_allowed`) on 2026-09-07, which is
   a manual probe, not a customer. On 2026-10-01 the exclusion's filter gained
   `AND NOT httpRequest.requestUrl:"/gtm/track"`, so every request to this
   path is now kept. The change was applied at about 04:01 UTC and took
   effect within about 12 minutes: probes at 04:02:19 and 04:03:25 UTC were
   not kept, the calibration probe at 04:13:36 UTC (user agent
   `tapper-gtm-dispatcher-probe/2026-10-01`, answered 404) was.
   Read it on or after 2026-10-15:
   `gcloud logging read 'resource.type="http_load_balancer" AND httpRequest.requestUrl:"/gtm/track"' --project itlinks-to --freshness=15d`.
   **Yes, still in use:** any request whose user agent is a browser (not curl,
   not the probe, not a known crawler). **No:** the probe is present and
   nothing else is. If the probe is missing, the measurement is broken; fix
   the filter before reading anything into a zero. After reading, drop the
   carve-out (restore the filter to
   `resource.type="http_load_balancer" AND sample(insertId, 0.99)`).
2. **To retire**, in order:
   - Take the Gallery listing down: delete `metadata.yaml` on `master` and on
     every other branch that has one (today `main`), or delete `main`. An
     archived repo is read-only, so this comes before archiving.
   - back-end: remove the orphaned `linkVisitEvents` consumer
     (`src/consumers/rabbitmq/links/linkVisitEvents.ts`, imported from
     `links/index.ts`). Its only producer was the deleted `gtmTrack` route;
     the only publish left on `LINK_VISIT_EVENTS_QUEUE` is the consumer's own
     delay-queue requeue. Also its queue rows in `docs/SURFACE-MAP.md` and
     the tests that list the file (`hotQueryIndexes`, `userAgentDetails`,
     `consumerLogFieldsNoRawPayload`).
   - back-end docs that still describe the flow as live: `docs/links/tracking/SPEC.md`
     (the "Link Visit Event (Conversion)" diagram and `LINK_VISIT_EVENTS_QUEUE`
     section, status `SHIPPED`), `docs/links/tracking/TESTING.md`,
     `docs/links/SPEC.md` and `docs/architecture/SPEC.md`.
   - estate: `src/data/flows.ts` and `src/data/graph.ts` (the `n-gtm` edge
     and verdict text, including the "LB logs show 0 requests in 14 days"
     claim, which can no longer be re-checked; replace it with item 1's
     result),
     `src/assets/sections/org-repos.html` ("RETIRING"), and the generated
     topology (`svc:gtm-web-event-dispatcher` still has a live-looking `run`
     edge to back-end from `template.tpl:83`).
   - Then, per the archive-both-never-delete rule: `gh repo archive`, the
     roster Repositories row set to archived with a dated note, the repo
     marked ⛔ ARCHIVED in the root `tapper /CLAUDE.md` table, and its row in
     roster `REPOS.md` corrected.
3. **To restore**, ship a receiving endpoint at `api.tapper.ai/gtm/track`
   (the consumer in item 2 still exists and could be fed again), record it in
   this spec, then decide `flawless-fixes`: its tclid fix needs a PR, a merge
   and a `versions:` entry to publish.
