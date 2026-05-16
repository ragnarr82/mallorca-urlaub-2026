# CLAUDE.md – Mallorca Urlaub 2026 Planner

## Project Overview

A single-page vacation planner for a family trip to Mallorca, May 18–27, 2026.
Family: 2 adults + 2 children (ages 3 & 6), staying at **Hotel Blau Punta Reina, Cala Mandia**.
Car: Opel Mokka.

**Live URL:** Deployed on GitHub Pages – `index.html` redirects immediately to `mallorca-planer.html`.

## File Structure

```
mallorca-planer.html   # Main app – all CSS, HTML, and JS in one file (1262 lines)
index.html             # GitHub Pages redirect to mallorca-planer.html
```

No build system, no dependencies, no package.json. Open any file directly in a browser.

## Architecture

Everything lives in `mallorca-planer.html`:

| Section | Lines | Purpose |
|---|---|---|
| `<style>` | 7–248 | All CSS with custom properties |
| `<body>` HTML | 250–337 | Static layout skeletons |
| `const weatherData` | 340–351 | Historical May weather data (10 days) |
| `const activities` | 355–811 | All 31 activity objects |
| `const suggestedPlan` | 814–819 | Pre-built 10-day itinerary |
| State variables | 822–826 | Runtime state |
| Functions | 829–1259 | All JS logic |

## Activity Data Schema

Each entry in the `activities` array:

```js
{
  id: 'l1',                  // Unique ID – prefix codes below
  cat: 'eidechse',           // Category – one of the 6 cats below
  emoji: '🦎',
  name: 'Barranc de Biniaraix – Eidechsenparadies',
  tags: ['natur','hike','eidechse','top'],  // Tag codes below
  shortDesc: '…',            // One-liner shown on card
  dist: '1h 15 Min.',        // Drive time from hotel
  lizardInfo: '…',           // Optional – only for eidechse cat
  detail: {
    desc: '…',               // Full description for modal
    route: '…',              // Turn-by-turn hiking directions
    parking: '…',            // Parking spot description
    duration: '2–2,5 Stunden',
    difficulty: 'Leicht ★☆☆',
    distance: '4 km',
    start: '…',              // Starting point name
    tips: ['…', '…'],        // Bullet-point tips array
    images: ['url1', 'url2'],// Image URLs for carousel
    komoot: 'https://…',     // Optional Komoot link
    maps: 'https://maps.apple.com/…',       // Apple Maps driving/walking URL
    walkRoute: 'https://maps.apple.com/…'   // Optional separate walking route
  }
}
```

### ID Prefix Convention

| Prefix | Category |
|---|---|
| `l` | Eidechsen (lizard spots / special hikes) |
| `b` | Natur (nature, caves, viewpoints) |
| `n` | Natur overflow |
| `c` | Kultur (culture, towns, train) |
| `a` | Beach (beaches, snorkeling) |
| `d` | Spaß (fun, water parks) |
| `e` | Food (markets, restaurants) |

### Category Names (`cat` field)

`eidechse` · `natur` · `kultur` · `beach` · `spass` · `food`

Category render order is fixed by `catOrder` array (line 900).

### Tag Codes

`beach` · `natur` · `kultur` · `spass` · `food` · `hike` · `drive` · `half` · `full` · `kids3` · `top` · `eidechse`

- `half` / `full` – half-day or full-day activity (shown in day column)
- `kids3` – suitable from age 3
- `top` – highlighted recommendation
- `hike` – included in the Wanderrouten-Übersicht modal
- `eidechse` – shows lizard info box in modal

## CSS Custom Properties

Defined in `:root` (line 8):

```css
--sun: #f5a623    --sea: #0077b6    --green: #2d6a4f    --green-light: #d8f3dc
--pink: #e07a5f   --pink-light: #fde8e4
--purple: #7c5cbf --purple-light: #ede5f8
--yellow-light: #fff9e6   --gray: #6b7280
--lizard: #4d7c0f --lizard-light: #ecfccb
```

## Key JavaScript Functions

| Function | Description |
|---|---|
| `renderWeather()` | Renders the horizontal weather strip from `weatherData` |
| `loadLiveWeather()` | Fetches live forecast from Open-Meteo API; falls back silently |
| `renderPool(filter)` | Renders left-panel activity cards, optionally filtered by text |
| `renderDays()` | Renders the 10 day columns in the right panel |
| `openModal(id)` | Opens detail modal with carousel, info grid, map links |
| `openHikesOverview()` | Opens the hike summary modal with all `hike`-tagged activities |
| `loadSuggestedPlan()` | Populates `dayPlans` from `suggestedPlan` and re-renders |
| `clearAll()` | Empties all day columns |
| `showToast(msg)` | Shows a 2.5 s toast notification |
| `buildCarousel(images, emoji, cat)` | Builds the image carousel inside the modal |
| `bindPoolDrag()` | Attaches drag events to activity cards in the pool |
| `getPlacedIds()` | Returns a `Set` of IDs already assigned to any day |
| `filterCards(v)` | Calls `renderPool(v)` – wired to search input |
| `toggleCat(cat)` | Collapses/expands a category section |

## State Variables

```js
let dayPlans = {};          // { 'DD.MM': ['id1', 'id2', …] } – the editable plan
let draggedId = null;       // ID of card being dragged
let dragSource = null;      // 'pool' | 'day'
let dragSourceDate = null;  // Source date string when dragging from a day
let currentModalId = null;  // ID of currently open modal
let carouselImages = [];    // Image URLs for open carousel
let carouselCurrent = 0;    // Active carousel slide index
```

`dayPlans` is initialized from `weatherData` (all dates map to `[]`).

## Weather Integration

- **Historical fallback:** `weatherData` array (lines 340–351) – typical May East Mallorca values
- **Live forecast:** Open-Meteo API (`https://api.open-meteo.com/v1/forecast`), free, no key required
- Hotel coordinates used for the API call: `lat=39.51785, lon=3.31448` (Cala Mandia)
- API covers max 16 days ahead; days outside the window keep historical values
- Weather label updates to show live vs. historical coverage

## Apple Maps URL Convention

All `maps` and `walkRoute` values use Apple Maps deep links:

```
https://maps.apple.com/?saddr=LAT,LON&daddr=LAT,LON&dirflg=d   # driving
https://maps.apple.com/?saddr=LAT,LON&daddr=LAT,LON&dirflg=w   # walking
```

Hotel origin is always `saddr=39.51785,3.31448`. The app auto-detects walking vs. driving via `dirflg=w` in the URL to choose the correct button label and style.

## Suggested Plan

Pre-built itinerary at lines 814–819, keyed by date string `'DD.MM'`:

```js
const suggestedPlan = {
  '18.05': ['a6','e3'], '19.05': ['b7','a4'], '20.05': ['e1','b5'],
  '21.05': ['l3','a1'], '22.05': ['c1','e4'], '23.05': ['b6','c5'],
  '24.05': ['b2'],      '25.05': ['c3'],      '26.05': ['l1','c2'],
  '27.05': ['a2','e3'],
};
```

## Deployment

GitHub Pages — no CI, no build step. Push to `main` and GitHub Pages serves the files directly. The `index.html` redirect ensures the canonical URL works.

## Development Guidelines

- **No build step.** Edit `mallorca-planer.html` and open it in a browser to test.
- **Language is German.** All user-facing text, tips, descriptions, and UI labels are in German.
- **Activity IDs must be unique** across the entire `activities` array.
- **Adding an activity:** append to the `activities` array following the schema above. Assign the next sequential ID for the relevant prefix. Add `'hike'` tag if it should appear in the Wanderrouten-Übersicht.
- **Adding a hiking route:** must include `detail.parking`, `detail.route`, `detail.maps` (or `detail.walkRoute`), `detail.duration`, `detail.difficulty`, `detail.distance`. Tag with `'hike'`.
- **Images:** use stable Wikimedia Commons URLs. Avoid hotlinking arbitrary CDNs.
- **Apple Maps links:** always use `saddr=39.51785,3.31448` as the hotel origin. Use `dirflg=w` for walking routes, `dirflg=d` for driving.
- **Eidechse category:** any activity in `cat:'eidechse'` or tagged `'eidechse'` should have a `lizardInfo` string describing lizard-watching conditions.
- **No persistence.** State lives only in-memory; refreshing the page resets the plan. Do not add localStorage without discussing it first.
- **CSS:** use existing custom properties for all colors. Do not hardcode colors that map to an existing variable.
