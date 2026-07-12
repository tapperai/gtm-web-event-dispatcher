# gtm-web-event-dispatcher — CLAUDE.md

## Push / remote-write guard
Never `git push`, open a PR, or trigger any publish (GTM Community Template
Gallery submission) without EXPLICIT, per-branch approval. Local commits are
fine. This repo has no CI deploy — publishing is a manual GTM Gallery action.

## What this repo is
A single **Google Tag Manager custom tag template** ("Tapper - Event
Dispatcher"), distributed via the GTM Community Template Gallery. At runtime on
a customer's website it reads the `tclid` identifier from `localStorage` and
fires a tracking pixel to the Tapper ingestion endpoint with a user-supplied
event name. It is client-side vendor code — no build, no server, no package
manager. Depends on nothing in the monorepo; the Tapper side of the contract is
the `api.tapper.ai/gtm/track` endpoint (owned by back-end / tracker).

## Stack
- **GTM custom template** — everything lives in ONE file: `template.tpl`.
- Logic is **GTM Sandboxed JavaScript** (a restricted JS dialect, NOT Node/
  browser JS). Only APIs `require`-d from the GTM sandbox are available
  (`sendPixel`, `encodeUriComponent`, `localStorage`, `data`, `gtmOnSuccess`,
  `gtmOnFailure`). No `fetch`, no DOM, no npm.
- No `package.json`, no lockfile, no `node_modules`.

## The gate command
There is no local compiler. Correctness is proven by **loading `template.tpl`
into the GTM Template Editor and running the built-in test/preview** (the
`___TESTS___` block + "Run code"). Any new sandbox API you `require` MUST also
be granted in `___WEB_PERMISSIONS___` or the template throws at runtime.

## File layout — `template.tpl` sections (order is fixed, `___NAME___` delimited)
- `___TERMS_OF_SERVICE___` — boilerplate; do not edit.
- `___INFO___` — display name, brand thumbnail (base64), description, `containerContexts: ["WEB"]`.
- `___TEMPLATE_PARAMETERS___` — user-facing fields. Currently one `TEXT` field `"Event Name"`.
- `___SANDBOXED_JS_FOR_WEB_TEMPLATE___` — the actual dispatch logic.
- `___WEB_PERMISSIONS___` — sandbox grants: allowed `sendPixel` URL pattern + `localStorage` key `tclid` (read=true, write=false).
- `___TESTS___` / `___NOTES___`.
- `metadata.yaml` — version SHA log (`changeNotes` per release). Add a new entry per published change.

**To add a config field:** edit `___TEMPLATE_PARAMETERS___`, then read it in the
sandbox JS via `data["Field Name"]`.
**To call a new sandbox API:** `require(...)` it in the JS block AND add the
matching grant in `___WEB_PERMISSIONS___`.
**To change the endpoint / add a query param:** edit the `url` in the JS block
AND widen the `sendPixel` `allowedUrls` pattern in `___WEB_PERMISSIONS___`
(current: `https://api.tapper.ai/gtm/track?tclid=*&event=*`).

## Conventions to match
- Current dispatch shape (keep it):
  ```js
  const tclid = localStorage.getItem("tclid");
  const event = data["Event Name"];
  if (localStorage.getItem("tclid") && event) {
    const url = 'https://api.tapper.ai/gtm/track?' + "tclid=" + encodeUriComponent(tclid) + "&event=" + encodeUriComponent(event);
    sendPixel(url, data.gtmOnSuccess, data.gtmOnFailure);
  } else {
    data.gtmOnSuccess();
  }
  ```
- **Always call `data.gtmOnSuccess()` / `data.gtmOnFailure()`** on every path —
  GTM hangs the tag otherwise. The no-op branch (missing tclid/event) still
  calls `gtmOnSuccess()`.
- Always `encodeUriComponent(...)` any value placed into the URL.
- The identifier key is exactly `tclid` (this is the tracker click id; NOT
  `tcid`, NOT `clid`). Read-only from localStorage.

## Footguns
- Sandbox JS is NOT real JS: no arrow-function niceties beyond what GTM supports,
  no `String.prototype` helpers that aren't sandbox-provided, no template
  literals for the URL in older container contexts — the existing code uses
  string concatenation deliberately; match it.
- Editing the JS without updating `___WEB_PERMISSIONS___` = runtime permission
  error in preview, not a compile error. The two blocks must stay in sync.
- The base64 brand thumbnail in `___INFO___` is load-bearing template data —
  don't reformat or drop it.

## What not to do
- Don't add a build system, `package.json`, or npm deps — GTM templates are
  single-file and self-contained.
- Don't rename `template.tpl` or reorder the `___SECTION___` blocks.
- Don't change the endpoint host or the `tclid` key without confirming the
  server contract (back-end / tracker own `/gtm/track`).
- Don't publish to the GTM Gallery without approval; bump `metadata.yaml` with a
  new SHA + `changeNotes` when you do.

## Deploy
Published manually to the **GTM Community Template Gallery** (no ArgoCD/Firebase/
pm2). A new version = a new commit SHA recorded in `metadata.yaml`.
