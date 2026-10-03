# Slowlane

A small PWA that shows which roads a German 45 km/h moped (Klasse AM, Versicherungskennzeichen) is not allowed to use, so the rider can avoid them. Built for personal use in Berlin first.

## The problem
Google Maps, Waze and most nav apps only offer "avoid highways". That does not cover Kraftfahrstraßen (blue sign, white car), which 45 km/h vehicles also cannot use. Slowlane draws those roads on a map and warns when the rider is close to one. It is an overlay and warning tool, not a router.

## Current state (v0.1, untested against live data)
Everything is in `index.html`, with no build step. Leaflet 1.9.4 from cdnjs, CARTO light raster tiles, Atkinson Hyperlegible from Google Fonts.

- On map move (debounced, zoom 12 or higher) it POSTs an Overpass query for the padded viewport:
  `highway=motorway|motorway_link|trunk|trunk_link` plus `motorroad=yes`, `out geom tags`.
- `classify()` marks ways as `hard` (motorway or `motorroad=yes`, drawn solid red) or `soft` (trunk without a motorroad tag, drawn dashed amber, meaning "check the signs"). `motorroad=no` on a non-motorway is dropped.
- Ways are stored by OSM id in a `Map` and persisted to `localStorage` (cap 4000) so the overlay works offline. Two Overpass endpoints are tried in order.
- "Follow my location" uses `watchPosition`, computes distance to the nearest stored segment, and shows a banner under 80 m (red for hard, amber for soft).
- "Keep screen on" uses the Screen Wake Lock API.
- `sw.js` caches the app shell, Leaflet and fonts. It does not cache tiles or Overpass responses.
- `manifest.webmanifest` and PNG icons are in place, so it is installable from Chrome via Add to Home Screen.

Only syntax checks and sample-data tests of `classify` and `nearest` have been run. It has never run in a real browser or against real Overpass data.

## Constraints
- It must be served over HTTPS from its own origin. It will not work inside Claude's artifact preview, because that sandbox blocks the external requests.
- Overpass is a shared public service: keep queries small, debounced, and cached. Consider a self-hosted or alternative mirror if it gets heavy use.
- Web location tracking only works in the foreground.
- A disclaimer in the UI says the app is a guide only. Keep it.

## Known issues and risks
- OSM tagging is incomplete. Some Kraftfahrstraßen may lack `motorroad=yes`, and some trunk roads are fine for mopeds. The amber class is deliberately a "verify" signal.
- The proximity warning ignores direction and road level. A motorway running next to or over a side street can trigger a false red banner.
- Hard-coded start view is central Berlin.
- No tests, no linting, no tile caching.

## Suggested next steps
1. Run it on a phone against live Overpass data in Berlin. Check how many known Kraftfahrstraßen show up red, and what the amber layer looks like.
2. Reduce false alerts: use heading and speed from `coords`, and ignore roads with `bridge`/`tunnel`/`layer` tags when the rider is on a different level.
3. Add an "on the road?" style sanity check: warn only when the rider is closer to a flagged way than to any non-flagged road.
4. Optional offline tile caching with a size cap.
5. Evaluate adding real routing (Valhalla's motor scooter costing, or GraphHopper custom models) and confirm whether either respects `motorroad`.
6. Longer term: a native Android app (foreground service for background location, voice prompts). The owner is a professional Android engineer, so a Kotlin port is realistic once the web version proves useful. A TWA wrapper (Bubblewrap/PWABuilder) is a quicker Play Store route.

## Conventions
Keep it dependency-light and buildless unless there is a clear reason. Mobile first, large tap targets, high contrast (riders glance at it). Sentence-case UI text, no emoji.
