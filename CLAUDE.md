# CLAUDE.md — my-journey

Standing context for this project. For shared patterns, see `~/Projects/kepando-dev/docs/`.

## Project overview

A personal life timeline. Events are entered as they come to mind and catalogued onto a
timeline that runs from birth to today. Two views of the same data: a vertical scroll
(birth at the top, today at the bottom) and a horizontal "life line" with one lane per
category, so overlapping periods are visible at a glance.

Key files:
- `index.html` — entire app (HTML + CSS + JS)
- `manifest.json` — PWA manifest
- `service-worker.js` — offline support + caching

## Data model

Events are **moments** — each has a single date. There are no start/end spans; durations
are answered by the Compare view, which measures the time between any two events (or
between an event and today).

`/MyJourney/events.json` in OneDrive:

```json
{
  "version": 1,
  "birthDate": "1988-03-18",
  "events": [
    {
      "id": "u1724567890123",
      "date": "2024-07-24",
      "precision": "day",
      "category": "home",
      "title": "Bought our current home",
      "location": "Portland, OR",
      "photo": "u1724567890123.jpg",
      "note": "Five days of living out of boxes in between."
    }
  ]
}
```

- `precision` is `day` | `month` | `year`. `date` is always a full ISO date; precision
  controls how it is displayed and how much of it is meaningful. A `year` entry stores
  `YYYY-01-01` and renders as just the year.
- Day of the week is shown only for `day` precision.
- `location` is a free-text string, optional, and absent entirely on events created
  before it existed — read it as `ev.location || ''` and render it only when non-empty.
  It is plain text with no geocoding. `dateLine()` composes it onto the date.
- `photo` is a filename in `/MyJourney/photos/`, named `{event id}.jpg`, and optional in
  the same way `location` is. Images are downscaled to a 1600px long edge and
  re-encoded as JPEG in the browser before upload, so phone-sized originals are not
  stored. The file is deleted when the photo is cleared or the event is deleted.
  Thumbnails and full views come from Graph's pre-authenticated URLs (`/thumbnails/0/small`
  and `@microsoft.graph.downloadUrl`) so they can be used as a plain `<img src>` with no
  Authorization header; they expire after about an hour, which is why `photoUrls` caches
  per session rather than persisting.
- `category` is one of: `home`, `career`, `family`, `milestone`, `health`, `travel`.
  Category colors are CSS custom properties named `--c-{category}`, defined for both
  light and dark themes.

`BIRTH` is a constant in `index.html`. It drives every age calculation and the start of
the horizontal axis. Changing it means changing that constant.

## Permissions

These actions are pre-approved:
- Read any file in this project
- Edit `index.html`, `manifest.json`, `service-worker.js`
- Run `git add`, `git commit`, `git push` for normal commits

## Preferences

- Keep responses short and direct — no trailing summaries
- Don't add docstrings, comments, or type annotations to code that wasn't changed
- Don't introduce abstractions or helpers for one-off things
- Prefer editing existing files over creating new ones

## Environment

- Single-file PWA deployed to GitHub Pages at `https://kepando.github.io/my-journey/`
- MSAL for Microsoft auth (personal accounts, authority: consumers)
- OneDrive for data storage (`/MyJourney/` subfolder), debounced 1500ms, last-write-wins
- No Claude API usage — this app has no AI features
- Responsive: mobile-first, breakpoints at 600px (tablet) and 900px (desktop)
- Fonts load from Google Fonts (Archivo, Public Sans, IBM Plex Mono) rather than the
  system stack

## Gotchas

- `[hidden] { display: none !important; }` is load-bearing. Several elements that toggle
  with the `hidden` attribute also carry `display: flex`, which would otherwise win.
- Bump `CACHE` in `service-worker.js` on each deploy to bust stale caches.
- The status bar style is `default`, deliberately, **not** `black-translucent`. With
  `black-translucent` the web view extends under the status bar and the page has to
  reserve that space itself via `env(safe-area-inset-top)` — which iPad reports as zero
  while still painting the status bar over the header. Detecting standalone mode to
  compensate does not work either: iPadOS matches neither `(display-mode: standalone)`
  nor `navigator.standalone` reliably. With `default`, iOS insets the web view and the
  problem cannot occur. Don't switch it back for the sake of an edge-to-edge look
  without testing on an iPad.
- Top spacing still goes through `--safe-top` rather than `env(safe-area-inset-top)`
  directly, so there is one knob if this ever needs adjusting.
- **The document does not scroll — `.main` does.** `html` and `body` are
  `overflow: hidden`, and `.main` is a fixed, full-bleed `overflow-y: auto` container
  between the header and the bottom nav. This is deliberate: iPadOS 26 draws a "scroll
  edge effect" (a blur-and-dim band) over the top of the *main scroll view* in installed
  web apps, which faded out the header. Confirmed by loading the same URL side by side —
  crisp in a Safari tab, faded in the home-screen app — after ruling the page out
  (opaque header, no backdrop-filter, no mask, identical render in a desktop browser).
  Anything reading or setting scroll position must use `#main`, not `window`; the
  pull-to-refresh gesture reads `scroller.scrollTop`. Note `#app` is consequently a
  zero-height wrapper, since all three of its children are fixed — assert on `.header`
  or `#main`, never on `#app`'s box.
- The header and bottom nav are **opaque**. They were translucent with a backdrop blur,
  which let content scrolling underneath bleed through and read as the top of the screen
  fading out. Don't reintroduce transparency there without checking it on a device.

## Updating an installed copy

iOS standalone has no browser chrome, so there is no built-in pull-to-refresh and no
address bar to reload from. Two paths get new code into an installed app:

- The page implements its own pull-to-refresh (see the last `<script>` in `index.html`),
  which calls `registration.update()` and then reloads.
- On foreground (`visibilitychange`), the page re-checks for a service worker update.
  When one activates it posts `SW_UPDATED` and the page reloads itself.

Both depend on `CACHE` in `service-worker.js` changing — the browser only notices an
update when the worker's own bytes differ. **Bump it on every deploy or installed copies
will never update.** The `SW_UPDATED` reload is skipped on first install, when there is
no previous worker to replace.

## Shared framework reference

For implementation patterns, read the relevant doc from `~/Projects/kepando-dev/docs/`:
- Auth: `msal-auth-patterns.md`
- Storage: `onedrive-storage.md`
- Deployment: `pwa-deployment.md`
- Responsive: `responsive-design.md`
