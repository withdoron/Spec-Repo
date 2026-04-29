# MAP-VIEW.md — Network Page Map View

> Interactive map showing businesses within a network, plotted by address.
> "Find what's near you."

**Status:** 📋 SPEC COMPLETE — Ready for build
**Created:** 2026-02-20
**Priority:** HIGH — Harvest Network launch depends on this
**Estimated effort:** 2 Cursor sessions (geocoding + map component, then integration + polish)

---

## Problem

The Harvest Network page lists businesses in a grid. Users can see names and categories but can't tell which egg seller or farm stand is near them. For a network built around local food — where proximity is the #1 factor — a list isn't enough.

## Solution

Add a map view toggle to network pages showing all businesses in that network plotted on an interactive map. Users can switch between the existing grid view and the map view.

---

## Architecture

### Geocoding Strategy: Geocode on Save, Store Coordinates

**Why not geocode at render time:**
- API calls on every page load = slow, rate-limited, fragile
- Same addresses re-geocoded repeatedly = wasteful
- Map can't render until all API calls return = bad UX

**Approach:**
1. When a business saves/updates address fields (city, state, zip_code, address), geocode the address and store lat/lng on the Business entity
2. Map reads stored coordinates — zero API calls at render time
3. If geocoding fails (bad address, API down), business still saves but lat/lng stays null — it simply doesn't appear on the map

### Geocoding Service: Nominatim (OpenStreetMap)

- Free, no API key required
- Rate limit: 1 request per second (fine for save-time geocoding)
- Endpoint: `https://nominatim.openstreetmap.org/search?q={address}&format=json&limit=1`
- Returns: `[{ lat, lon, display_name }]`
- Must include a User-Agent header per Nominatim usage policy (e.g., `LocalLane/1.0`)

### Map Library: Leaflet + react-leaflet

- Already installed in the project
- Free, open source
- Uses OpenStreetMap tiles (no API key)
- Tile URL: `https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png`

---

## Data Model Changes

### Business Entity — New Fields

| Field | Type | Notes |
|-------|------|-------|
| latitude | number | Geocoded from address. Null if not geocoded. |
| longitude | number | Geocoded from address. Null if not geocoded. |
| geocoded_at | string (ISO datetime) | When last geocoded. Used to know if re-geocoding needed after address change. |

These fields need to be added to the Business entity in Base44 dashboard.

### Server Function Update

In `functions/updateBusiness.ts`, action `update_profile`:

1. Add `latitude`, `longitude`, `geocoded_at` to the field allowlist
2. After filtering fields, check if any address fields changed (address, city, state, zip_code)
3. If address changed, call Nominatim to geocode:
   - Build query string: `${address}, ${city}, ${state} ${zip_code}`
   - Fetch from Nominatim with User-Agent header
   - If result found: set latitude, longitude, geocoded_at on the update payload
   - If no result or error: leave lat/lng unchanged (don't overwrite existing good coordinates with null)
4. Geocoding is best-effort — never block the save on geocoding failure

---

## UI Design

### Network Page — View Toggle

Add a view toggle (grid/map) to the network page header, same pattern as Events.jsx:

- Grid view: existing business card grid (default)
- Map view: Leaflet map with OpenStreetMap tiles
- Toggle: same amber/slate styling as Events.jsx grid/list toggle

### Map Behavior

- **Center:** Eugene, OR (44.0521, -123.0868)
- **Zoom:** 11 (greater Eugene area)
- **Pins:** Custom amber markers for each business with coordinates
- **Pin popup:** Business name (linked to profile), category, address, thumbnail
- **No coordinates:** Business doesn't appear on map. Subtle count: "X businesses not shown (no address on file)"
- **Mobile:** Full width, min height 300px, touch zoom/pan
- **Empty state:** Show grid view with message if no businesses have coordinates

### Privacy

If `display_full_address` is false, show pin at city-level precision (not exact address). Respect the toggle the business owner set.

---

## Component Structure

```
src/components/maps/
  NetworkMap.jsx          — Leaflet map component
  BusinessMapPin.jsx      — Custom pin with popup
  MapViewToggle.jsx       — Grid/Map toggle (reusable)
```

---

## Surfaces

| Surface | Map Available | Notes |
|---------|--------------|-------|
| Network Pages | V1 | Primary use case |
| Directory | Future | All businesses |
| MyLane | No | Personal dashboard |
| Business Profile | No | Single business |

---

## Build Phases

### Phase 1: Geocoding Infrastructure
1. Add latitude, longitude, geocoded_at to Business entity in Base44
2. Update updateBusiness.ts to geocode on address change
3. One-time migration to geocode existing businesses
4. Test: update address, verify coordinates stored

### Phase 2: Map Component
1. Create NetworkMap.jsx with react-leaflet
2. Create BusinessMapPin.jsx with popup
3. Import Leaflet CSS (required)
4. Test with hardcoded data

### Phase 3: Network Page Integration
1. Add MapViewToggle to NetworkPage.jsx
2. Filter businesses to those with coordinates
3. Wire NetworkMap with real data
4. Handle empty states
5. Mobile testing

### Phase 4: Polish
1. Custom amber pin markers
2. "X businesses not shown" count
3. Marker clustering (react-leaflet-cluster) if needed
4. Loading state

---

## Constraints

- No Google Maps API — Leaflet + OpenStreetMap only
- No paid geocoding — Nominatim only
- Geocoding server-side in updateBusiness.ts
- Map is read-only
- Don't block saves on geocoding failure
- Leaflet CSS must be imported

---

## Dependencies

| Dependency | Status |
|------------|--------|
| react-leaflet | Already installed |
| leaflet | Already installed |
| Nominatim API | Free, no setup |
| Business entity fields | Need to add |
| updateBusiness.ts geocoding | Need to build |

---

## Open Questions

| Question | Current Thinking |
|----------|-----------------|
| Directory map view? | V2 after network pages |
| Map as default view? | No — grid default, map toggle |
| Pin clustering? | V2, not needed at pilot scale |
| display_full_address = false? | Show city-level pin, not exact |

---

*This spec is part of the LocalLane Spec-Repo. Reference before building.*
