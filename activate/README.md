# Activate booking widget — HTTPS integration rig

A throwaway host page for integration-testing the Smeetz **Activate** booking
widget as a third-party embed over HTTPS.

Live: <https://amardebza.github.io/activate/>

The page is a deliberate *stress rig* modelled on activate-games.fr: a fixed
header at `z-index:99999`, a fixed bottom CTA bar at `99998`, and aggressive
host CSS — so the widget's shadow-DOM isolation and drawer overlay are tested
against a hostile host, not a blank page.

## What it exercises

| | |
|---|---|
| Bundle | `https://sdk.activate-test.smeetz.com/activate-booking.js` (the deployed test build) |
| Origin | `https://amardebza.github.io` — must be in the tenant's `allowedOrigins`, else `SmeetzSdk.init` returns `Unlicensed` |
| Language | `lang="fr"` on the element (falls back to `<html lang>`); after boot the venue config's locale wins |
| Tracking | `window.dataLayer` is created before the widget loads, so the funnel events (`smtz_*`) have a sink |
| Deep link | `?smtz-offer=<1h\|1h30\|uuid>` and `?smtz-date=YYYY-MM-DD` on this page's URL auto-open the drawer |

Try: <https://amardebza.github.io/activate/?smtz-offer=1h30&smtz-date=2026-08-25>

## Embed contract

```html
<script>window.dataLayer = window.dataLayer || [];</script>
<script defer src="https://sdk.activate-test.smeetz.com/activate-booking.js"></script>
<activate-booking publishable-key="pub_…" environment="test" lang="fr"></activate-booking>
```

Two things that silently break it:

- **`type="module"`** — the bundle is an IIFE, and `document.currentScript` is
  always `null` inside a module script, so asset-base resolution falls back to
  the host origin and the fonts 404.
- **A `dataLayer` created after the widget boots** — the early funnel events
  are pushed at mount and are simply lost.

## Known issue on the currently deployed bundle

On a first visit with no deep link and `lang="fr"`, the entry card falls back to
hardcoded prices (`data-live="false"`) and advertises **29,90 €** for 1h30 while
checkout charges **20,90 €**. Fixed on `activate-app` branch
`fix/teaser-price-locale-pin`; this page will show correct prices once that is
merged and deployed.
