# Backpacker — Pictured Rocks

Shared packing list and itinerary for Sep 4–7, 2026. Installable on Android, works offline, no login.

```
index.html              the whole app
manifest.webmanifest    makes it installable
sw.js                   offline cache + notifications
icon-192.png  icon-512.png  icon-maskable-512.png
apple-touch-icon.png  favicon-32.png
```

> Renamed from Sunrise. Live data is unaffected — Firebase rooms are keyed by room code, not by
> app name. Anything saved locally is migrated from the old `sunrise:` keys on first load, so nobody
> loses a list, and the old keys are left in place in case you roll back.

## Deploy — everything sits flat in the repo root

**No `icons/` folder.** Every file above goes side by side at the root, and the manifest, the
service worker and `index.html` all reference them that way (`icon-192.png`, not `icons/icon-192.png`).
That mismatch is what broke the install prompt and the sidebar logo: Chrome silently refuses to offer
installation if a single manifest icon 404s.

Push, then **Settings → Pages → Deploy from a branch → main / (root)**.
Live at `https://<user>.github.io/<repo>/` in about a minute.

> Every path is relative, so it works from a repo subpath. Don't change `./` to `/` in
> `manifest.webmanifest` or `sw.js` — that breaks GitHub Pages.

After deploying, open the site and **hard-refresh once**. `CACHE` is now `backpacker-v6`, so one
clean load picks up the rename and everything below.

## Install on Android

Chrome ⋮ → **Install app**, or the prompt that appears. If it doesn't appear, go to
**Share → Why won't it install?** — it checks the four preconditions in turn (HTTPS, service worker
registered, manifest readable, every icon loads) and names the one that failed, including which icon
is missing. The Install row also tells you when you're already running the installed copy.

On iPhone it's **Share → Add to Home Screen**. iOS never fires an install prompt, by design.

## Day and night

**Share → Look & feel** has three settings: **Automatic** follows the phone, **Day** and **Night**
pin it. There's also a sun/moon button in the top-right corner of every screen that flips straight
between the two.

Night isn't an inversion. It's the same sky after sunset — indigo, plum, dusty rose — and the map
goes with it: warm-white land becomes deep violet, water goes to midnight blue, and the tiles switch
to CARTO's dark basemap. The choice is remembered and applied before the first pixel is drawn, so
there's no flash on open, and the switch cross-fades rather than snapping.

## The sky

The gradient belongs to the page, not to the header. It starts at night behind the status bar, runs
down through the countdown, the sun-arc tracker and the packed percentage, and dissolves into the
canvas across the frosted search bar — so the search row *is* the join, and there's no line anywhere
saying "header ends here". Each screen measures its own header at runtime and hands the number to the
gradient, so the fade lands in the right place whatever the content does.

Scrollbars are hidden throughout. Scrolling, momentum and overscroll containment are untouched.

## Turn on live sync

A Firebase config is already pasted in near the top of `index.html`'s `<script>`. The header pill
reads **Live · everyone** when it connects, **On this device** when it doesn't, and **Offline ·
saved here** when the phone has no signal — edits still land, and sync catches up.

Test-mode rules expire after 30 days and allow anyone to read/write. Before the trip, set rules to
require a long room code and switch to one:

```json
{ "rules": { "rooms": { "$room": {
  ".read":  "$room.length > 12",
  ".write": "$room.length > 12"
} } } }
```

Then **Share → Join a different list** → something like `khan-picrocks-9f3k2p`. The link becomes the key.

## Notifications

Off until you turn them on in **Share → Turn on change alerts**. Then: at most **one summary every
5 hours**, and only when the app isn't the screen you're looking at.

**What this can't do:** fire when the app is fully closed. That needs Firebase Cloud Messaging plus a
Cloud Function watching the database — a server, which GitHub Pages can't host.

## Packing

Search, category chips, hide-packed and a per-category progress bar, as before. New since:

- **Who's bringing it** — an optional name on any item. It shows as a pill on the row, autocompletes
  from names already used on the list, and is searchable like anything else.
- **Undo** — every destructive action offers a way back for a few seconds: an item, a stop, a note, a
  whole day, wiping the list, resetting the itinerary. Nothing is final until the toast fades.
- **Copy the list as text** — under Share. Plain text with ☐ / ☑ boxes, grouped by category, for
  whoever isn't on the app. On a phone it opens the share sheet instead of the clipboard.
- **A buzz on the tick**, with a different pattern when a whole category closes out. Turn it off
  under Look & feel; the row hides itself on hardware that can't vibrate.
- **`/`** focuses the search box on desktop.

## Itinerary

Four days, two views, switched by the segmented control, plus an **Edit** toggle beside it. During
the trip it opens on the day you're actually living, and that tab reads **TODAY**.

**List** — a timeline with times in the left gutter, Fixed and Optional badges, and the day's caveats
as cards underneath. The trip-wide warnings sit under Friday.

**Map** — a real base map. Warm-white land, Lake Superior and Lake Michigan with Green Bay and the
Door Peninsula, Grand Island, Lake Winnebago, and the roads this trip actually uses (I-43, US-41,
US-2, M-94, M-28, H-58) with town names. Everything is drawn from coordinates through a single
isotropic projection, so the proportions are honest and **the scale bar is real**.

North is always up. Rather than rotating the map, the camera sits behind the SUV and the ground
slides underneath as you scroll the stops — the route fills in behind it, pins light up as they're
passed, and the current stop is named in the corner. The SUV is drawn from behind and above with the
roof rack loaded. Tap **Following the car** in the top-left corner to pull back to the whole day, and
tap again to drop back in.

Still: **drive with offline Google Maps.** This is an at-a-glance picture of the day, not navigation.

### Editing

Hit **Edit** in the itinerary bar and everything on the screen becomes editable, synced to everyone:

- **Stops** — time, timezone, icon, place, notes, a Fixed/Optional badge, and coordinates. Reorder
  with ↑ ↓, delete, or **+ Add a stop** at the bottom of the day.
- **Days** — weekday, date, title, summary and the fact chips. **+ DAY** at the end of the day tabs
  adds one; a day can be deleted from its own sheet.
- **Notes** — the caveat cards under each day, added and edited in place.

Paste coordinates straight from Google Maps (right-click a spot → the numbers at the top of the menu).
A stop without coordinates still shows in the timeline, is flagged **No pin**, and inherits the
previous stop's position on the map so the route never breaks.

The original plan is still the `TRIP` object in `index.html`, and it's what comes back from
**Share → Reset the itinerary**, which throws away every edit for everyone — with one undo.

`CAT` above it is the item catalogue that drives autocomplete and category guessing.

## Layout and feel

- **Phone** — bottom tab bar, single column, sheets slide up from the bottom
- **iPad** — two-column list, sheets become centred dialogs
- **Desktop** — the tab bar becomes a left rail; itinerary map pins beside the timeline

Sheets on a phone track the finger from the grip or the title bar, resist when dragged upward, and
can be thrown shut — the landing point is projected from the release velocity, so a flick goes where
it looks like it's going.

Reduced motion, reduced transparency and increased contrast are all respected; under reduced motion
the sheet drag steps aside and the theme switch stops cross-fading. Safe-area insets handled.

## After editing

Bump `CACHE` in `sw.js` (`backpacker-v6` → `v7`). Otherwise installed phones keep serving the old copy.
