# Sunrise — Pictured Rocks

Shared packing list and itinerary for Sep 4–7, 2026. Installable on Android, works offline, no login.

```
index.html              the whole app
manifest.webmanifest    makes it installable
sw.js                   offline cache + notifications
icons/                  app icons (192, 512, maskable, apple-touch, favicon)
```

## Deploy

Push all four to the repo root, then **Settings → Pages → Deploy from a branch → main / (root)**.
Live at `https://<user>.github.io/<repo>/` in about a minute.

> **Every path is relative**, so it works from a repo subpath. Don't change `./` to `/` in `manifest.webmanifest` or `sw.js` — that breaks GitHub Pages.

## Turn on live sync

Everything works immediately without this, saved per-device. For cross-phone sync:

1. [console.firebase.google.com](https://console.firebase.google.com) → your project
2. **https://console.firebase.google.com/project/_/database** → Realtime Database → Create → **test mode**
3. **https://console.firebase.google.com/project/_/settings/general** → Your apps → `</>` → Register
4. Copy the `firebaseConfig` object → paste over `const FIREBASE_CONFIG = null;` near the top of `index.html`'s `<script>`
5. Check `databaseURL` is one of the keys. Firebase sometimes omits it; it's on the database page.

The header pill flips from **On this device** to **Live · everyone**.

Test-mode rules expire after 30 days and allow anyone to read/write. Before the trip, set rules to require a long room code and switch to one:

```json
{ "rules": { "rooms": { "$room": {
  ".read":  "$room.length > 12",
  ".write": "$room.length > 12"
} } } }
```

Then **Share → Join a different list** → something like `khan-picrocks-9f3k2p`. The link becomes the key.

## Install on Android

Open the Pages URL in Chrome → **⋮ → Install app** (or the prompt that appears). It also shows up under **Share → Add to home screen** once Chrome offers it. Runs full screen, own icon, works offline.

On iPhone it's **Share → Add to Home Screen** — iOS gives no install prompt.

## Notifications

Off until you turn them on in **Share → Turn on change alerts**. Then: at most **one summary every 5 hours**, and only when the app isn't the screen you're looking at.

**What this can't do:** fire when the app is fully closed. That needs Firebase Cloud Messaging plus a Cloud Function watching the database — a server, which GitHub Pages can't host. What you get is background-tab and background-PWA alerts, which covers the realistic case of the app sitting open on someone's phone.

## Itinerary

Four days, two views, switched by the segmented control.

**List** — a timeline with times in the left gutter, Fixed and Optional badges, and the day's caveats as cards underneath. The trip-wide warnings (Munising Falls closed, the lost hour, Labor Day parking, no halal butcher, dead cell service) sit under Friday.

**Map** — the day's route with an SUV that drives along it as you scroll the stops. The route fills in behind the car and the current stop is named in the corner.

The map is a real equirectangular projection stretched to fill the frame. North is up and the order is true, but proportions are not — these routes are almost entirely north–south and an honest fit would be a thin ribbon down the middle. That's why there's no scale bar and the card says so. **Drive with offline Google Maps.**

The **before we leave** checklist under Share is synced like everything else, so one person confirming the marine forecast ticks it for everyone.

## What's editable

Trip content is hardcoded in the `TRIP` object in `index.html` — days, stops, coordinates, notes, alerts and the checklist. Add a stop by adding to a day's `stops` array with `ll:[lat,lng]`; the map and timeline both pick it up with no other changes.

`CAT` above it is the item catalogue that drives autocomplete and category guessing.

## Layout

- **Phone** — bottom tab bar, single column, sheets slide up from the bottom
- **iPad** — two-column list, sheets become centred dialogs
- **Desktop** — the tab bar becomes a left rail; itinerary map pins beside the timeline instead of above it; three columns past 1440px

Dark mode follows the system. Reduced motion is respected. Safe-area insets handled.

## After editing

Bump `CACHE` in `sw.js` (`sunrise-v3` → `v4`). Otherwise installed phones keep serving the old cached copy.
