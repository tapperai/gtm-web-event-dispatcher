# GTM Event Dispatcher — Testing Framework

> **Parent:** [gtm-event-dispatcher](gtm-event-dispatcher/SPEC.md)

There is no automated test framework, simulator script, or CI in this repo —
verified 2026-08-26: no `package.json`, no `y-scripts/`, no `.github/workflows/`.
The only gate is Google Tag Manager's own Template Editor.

---

## Pipeline Overview

```
Flow: pixel dispatch
  Page load in a GTM preview container
    → tag fires (Tapper - Event Dispatcher)
    → reads tclid from localStorage + Event Name param
    → sendPixel to https://api.tapper.ai/gtm/track
    → GTM preview panel shows tag status (Fired / Not fired) + console
```

---

## Prerequisites

- A Google Tag Manager account with access to create/edit a container.
- The `template.tpl` file opened in **Google Tag Manager → Templates →
  New Custom Template → import `template.tpl`** (or paste its contents).
- A test page (or GTM's built-in preview against any URL) where you can set
  `localStorage.setItem("tclid", "<any-test-value>")` via the browser
  devtools console before the tag fires.

---

## Manual Test Procedure

1. In the GTM Template Editor, open `template.tpl` and click **"Run code"**
   inside the `___TESTS___` panel — this is the fastest structural check
   (correct sandbox API usage, no syntax errors, permissions declared) but it
   does not hit the real network endpoint.
2. To test the real dispatch path: create a Tag in a test/preview GTM
   container using this template, set **Event Name** to a test value (e.g.
   `test_event`), and set the trigger to All Pages.
3. Enter GTM **Preview mode** against a page you control.
4. In the page's devtools console, before the tag fires (or before
   navigating), run:
   ```js
   localStorage.setItem("tclid", "test-tclid-123");
   ```
5. Reload / trigger the tag. In the GTM Preview panel, confirm the tag shows
   **Fired**, and in the browser's Network tab confirm a request to:
   ```
   https://api.tapper.ai/gtm/track?tclid=test-tclid-123&event=test_event
   ```
6. **No-op path**: clear `localStorage.removeItem("tclid")` and reload — the
   tag should still show **Fired** in GTM (it calls `gtmOnSuccess()` even in
   the no-op branch) but no pixel request should appear in the Network tab.

**Verify server-side receipt:** this repo cannot verify that
`api.tapper.ai/gtm/track` recorded the event — that endpoint is owned by
`back-end` / `tracker`; check there for confirmation the pixel was ingested.

---

## Known Gap (verified 2026-08-26)

`master`'s `template.tpl` calls `localStorage.getItem("tclid")[0]` — indexing
`[0]` into the result, which truncates `tclid` to its first character before
it's placed in the pixel URL. An unmerged branch (`flawless-fixes`, commit
`9805e41`) fixes this. Re-test step 4/5 above against that branch's
`template.tpl` if/when it's reviewed — with the bug present, the URL sent in
step 5 would actually be `...tclid=t&event=test_event`, not the full test
value.
