# Sunrise — Pictured Rocks

Shared packing list and itinerary for Sep 4–7, 2026. Installable on Android, works offline, no login.

```
index.html              the whole app
manifest.webmanifest    makes it installable
sw.js                   offline cache + notifications
icon-192.png  icon-512.png  icon-maskable-512.png
apple-touch-icon.png  favicon-32.png
```

## Deploy — everything sits flat in the repo root

**No `icons/` folder.** Every file above goes side by side at the root, and the manifest, the
service worker and `index.html` all reference them that way (`icon-192.png`, not `icons/icon-192.png`).
That mismatch is what broke the install prompt and the sidebar logo: Chrome silently refuses to offer
installation if a single manifest icon 404s.

Push, then **Settings → Pages → Deploy from a branch → main / (root)**.
Live at `https://<user>.github.io/<repo>/` in about a minute.

> Every path is relative, so it works from a repo subpath. Don't change `./` to `/` in
> `manifest.webmanifest` or `sw.js` — that breaks GitHub Pages.

After deploying, open the site and **hard-refresh once**. The old service worker is still holding
`sunrise-v3`; `CACHE` is now `sunrise-v4`, so one clean load is enough to pick everything up.

## Install on Android

Chrome ⋮ → **Install app**, or the prompt that appears. If it doesn't appear, go to
**Share → Why won't it install?** — it checks the four preconditions in turn (HTTPS, service worker
registered, manifest readable, every icon loads) and names the one that failed, including which icon
is missing. The Install row also tells you when you're already running the installed copy.

On iPhone it's **Share → Add to Home Screen**. iOS never fires an install prompt, by design.

## Turn on live sync

A Firebase config is already pasted in near the top of `index.html`'s `<script>`. The header pill
reads **Live · everyone** when it connects and **On this device** when it doesn't.

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

## Itinerary

Four days, two views, switched by the segmented control, plus an **Edit** toggle beside it.

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
**Share → Reset the itinerary**, which throws away every edit for everyone.

`CAT` above it is the item catalogue that drives autocomplete and category guessing.

## Layout

- **Phone** — bottom tab bar, single column, sheets slide up from the bottom
- **iPad** — two-column list, sheets become centred dialogs
- **Desktop** — the tab bar becomes a left rail; itinerary map pins beside the timeline

Dark mode follows the system, including the map — warm white becomes deep violet, water goes to
midnight blue. Reduced motion is respected. Safe-area insets handled.

## After editing

Bump `CACHE` in `sw.js` (`sunrise-v4` → `v5`). Otherwise installed phones keep serving the old copy.
