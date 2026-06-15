---
name: longdo-map-rest
description: >-
  Call the Longdo location REST APIs from any language or backend — search and
  suggest, smart search, address extraction, geocoding, reverse geocoding
  (single and batch), nearby POI, routing (route guide, path, GeoJSON, matrix,
  snap-to-road, reachable span, TSP), and traffic speed. Use when calling Longdo
  location services over HTTP from a server, CLI, mobile, or non-JS client. For
  the in-browser map itself, see the longdo-map-js skill.
---

# Longdo Location REST APIs

Official reference: https://map.longdo.com/docs/rest (legacy: https://api.longdo.com/map/doc/rest.php)
Get a free API key: https://map.longdo.com/console/

These endpoints are language-agnostic HTTP services. Call them from your backend,
a CLI, mobile, or any non-JS client. The in-browser map is a separate concern —
see **longdo-map-js**.

## Conventions that apply to every endpoint

- **Auth:** every call needs `key=YOUR_API_KEY`.
- **Methods:** `GET` or `POST` with `application/x-www-form-urlencoded`.
- **Format:** JSON by default. Pass `callback=fnName` for JSONP.
- **Locale:** `locale=th` (default) or `locale=en`; some endpoints accept `-`
  for multilingual output.
- **Coordinates:** `lon` (longitude) and `lat` (latitude), in WGS84 degrees.
- **Hosts vary by service:** search/suggest live on `search.longdo.com`,
  address/POI on `api.longdo.com`, routing on `api.longdo.com/RouteService`.
  Use the exact URLs below — don't assume one host.

> **Keep the key server-side.** Unlike the JS map key (which is domain-locked in
> the console and meant for the browser), treat REST keys as backend secrets.
> Call these from your server, not from client JS where the key would leak. If
> you must call from the browser, use a domain-restricted key and proxy the
> sensitive ones.

---

## Search & geocoding

### Suggest (autocomplete)
`GET https://search.longdo.com/mapsearch/json/suggest`

| Param | Notes |
|-------|-------|
| `keyword` | search term |
| `area` | geocode filter (CSV) |
| `offset` / `limit` | paging; `limit` default 10 |
| `dataset` | data sources (CSV — see Datasets) |
| `key`, `callback` | standard |

Response: `suggestions[]` with `w` (suggestion text), `d` (highlighted), `s` (dataset).

### Search (POI + address)
`GET https://search.longdo.com/mapsearch/json/search`

| Param | Notes |
|-------|-------|
| `keyword` | search term |
| `lon`, `lat` | bias/center coordinates |
| `span` | distance filter (`deg`/`m`/`km`) |
| `tag` | category filter (CSV) |
| `limit` | default 20 |
| `locale`, `key`, `callback` | standard |

Response: metadata (`hasmore`, `start`, `end`, `keyword`) plus a results array —
each with type, id, name, coordinates, tags, contact, verification, distance.

A ready-made search-box demo: https://mapdemo.longdo.com/searchbox/

### Smart Search
`GET https://search.longdo.com/smartsearch/json/search` — extends Search with:
`forcesmartsearch` (0 conditional / 1 always), `extendedsearch` (0–3 external
fallback), `extendedlimit` (min results before extending), `cache` (1/0),
`extractaddress` (return a parsed address object). Adds a `source` field
(`longdo`/`OSM`) per result.

### Extract Address (parse free-text address)
`GET https://search.longdo.com/smartsearch/json/extract_address/v2`
Params: `text` (address string), `correction` (default 1), `censor` (strip
contact details, default 0). Returns parsed components — `house_no`, `floor`,
`room`, `moo`, `place`, `building`, `village`, `road`, `subdistrict`,
`district`, `province`, `postcode` — each with confidence + similarity scores.

### Geocoding (address text → coordinates)
`GET https://search.longdo.com/addresslookup/api/addr/geocoding`
Params: `text` (address), `dataset`. Returns a confidence score, location
components (`houseno`, `road`, `village`, `moo`, `subdistrict`, `district`,
`province`, `zipcode`) and candidate coordinate points.

### Reverse geocoding (coordinates → address)
`GET https://api.longdo.com/map/services/address`
Params: `lon`, `lat` (or `id` for a Longdo Map id), `locale`, exclusion flags
`noaddress` / `noelevation` / `noroad`, `key`, `callback`. Returns admin
divisions (`geocode`, `province`, `district`, `subdistrict`, `postcode`),
elevation, and nearest road with its position.

### Batch reverse geocoding (≤100 points)
`POST https://api.longdo.com/map/services/addresses`
Params: `lon[]`, `lat[]` (up to 100) or a `wkt` geometry, plus the same
exclusion flags. Returns an array in reverse-geocode format.

### Geocode / postcode lookup
`GET https://api.longdo.com/POIService/json/address`
Params: `geocode` **or** `postcode`, `locale`, `key`, `callback`. By geocode →
single area; by postcode → array of areas (code, province, district,
subdistrict, coordinates).

### Nearby POI
`GET https://api.longdo.com/POIService/json/search`
Params: `lon`, `lat`, `span` (`deg`/`m`/`km`), `tag`, `zoom` (importance
filter), `dataset`, `locale`, `key`, `callback`. Same result shape as Search.

---

## Routing

### Calculate route (turn-by-turn)
`GET https://api.longdo.com/RouteService/json/route/guide`

| Param | Notes |
|-------|-------|
| `flon`, `flat` | origin |
| `tlon`, `tlat` | destination |
| `mode` | `t` traffic-aware, `c` fastest, `d` shortest, `e` shortest+traffic |
| `type` | travel methods, combinable: `1` road, `8` ferry, `16` tollway |
| `restrict` | `0` none, `1` no motorcycle |
| `time` | Unix timestamp for departure |
| `maxresult` | route variations 1–8 (default 1) |
| `locale`, `key`, `callback` | standard |

Returns from/to metadata, distance/time summary, and turn-by-turn guidance
(road names, segment details). Turn codes: 0 left, 1 right, 2 slight left,
3 slight right, 5 straight, 6 tollway, 9 arrive, 11 ferry, 4 unknown.

### Route path geometry
`GET https://api.longdo.com/RouteService/json/route/path` — param `id` (from a
guide response). Returns the coordinate array for drawing the line.

### Route as GeoJSON
`GET https://api.longdo.com/RouteService/geojson/route` — same params as guide.
Returns a GeoJSON `FeatureCollection` (metadata + road features with turn
instructions). Easiest to render directly on a map.

### Route matrix (many-to-many)
`GET https://api.longdo.com/RouteService/json/route/matrix`
Params: `flon[]`, `flat[]` (≤25 origins), `tlon[]`, `tlat[]` (≤25
destinations), plus `mode`/`type`/`restrict`/`time`/`locale`. Returns a 2D
matrix of travel time (seconds) and distance (meters) per O-D pair.

### Snap to road
`GET https://api.longdo.com/RouteService/json/route/snap` — param `wkt`
(LineString). Returns coordinates snapped to the road network.

### Reachable span (isochrone-ish)
`GET https://api.longdo.com/RouteService/json/span/get`
Params: `flon`, `flat`, `span` (distance in m, max 30000; or time in s, max
1800), `flag` (`1` roads, `2` boundary; bitwise-combinable). Returns reachable
road lines and/or a boundary polygon.

### TSP route planner (optimal visiting order)
`POST https://api.longdo.com/route-tsp-v2`
Params: `lon[]`, `lat[]` (stops), `cost` (include per-step detail, true/false),
`otsp` (open loop — `false` returns to start, `true` does not), `mode`, `key`.
Returns the optimal index sequence, ordered locations, total distance/time, and
(with `cost=true`) `time_step` / `distance_step` arrays.

### Closed roads (read/write)
- Get: `GET https://api.longdo.com/RouteService/json/close/get` → polygon/
  LineString WKT + weekly time ranges (hour values 0–167).
- Set: `POST https://api.longdo.com/RouteService/json/close/set` → params
  `polygon[]` (WKT), `time[]` (CSV hour ranges). Requires authentication.

---

## Traffic & area

### Traffic speed
`GET https://api.longdo.com/RouteService/json/traffic/speed`
Params: `lon`, `lat`, `range` (degrees; default & max `0.001`), `time` (Unix
ts for prediction), `locale` (or `-` for multilingual), `key`, `callback`.
Returns road name, direction (forward/reverse), speed in **m/s**, source
(real-time vs predicted), and the nearest position on the road.

### Area info
`GET https://api.longdo.com/GeoServiceMS/json/areainfo` — param `polygon`
(GeoJSON). Returns the building count inside the polygon.

---

## Datasets (for Suggest / Search / POI)

`poi_p` POI · `poi_r` roads · `poi_a` admin · `poi_b` house numbers ·
`s_osmpo` OSM points · `s_osmline` OSM lines · `s_osmpol` OSM polygons ·
`overture_p` Overture Map. Pass as a CSV in `dataset`.

---

## Examples

Reverse geocode (Node, key kept server-side):

```javascript
const params = new URLSearchParams({
  lon: '100.5018', lat: '13.7563', locale: 'en', key: process.env.LONGDO_KEY,
});
const res = await fetch(`https://api.longdo.com/map/services/address?${params}`);
const addr = await res.json();
// → { province, district, subdistrict, postcode, road, ... }
```

Route as GeoJSON, ready to drop on a map:

```javascript
const p = new URLSearchParams({
  flon: '100.5018', flat: '13.7563',   // origin
  tlon: '100.5340', tlat: '13.7460',   // destination
  mode: 't', type: '1', locale: 'en', key: process.env.LONGDO_KEY,
});
const geojson = await (await fetch(
  `https://api.longdo.com/RouteService/geojson/route?${p}`)).json();
```

Search with location bias (Python):

```python
import os, requests
r = requests.get("https://search.longdo.com/mapsearch/json/search", params={
    "keyword": "coffee", "lon": 100.5018, "lat": 13.7563,
    "span": "2km", "limit": 20, "locale": "en", "key": os.environ["LONGDO_KEY"],
})
results = r.json()["data"]
```

---

## Reference

- REST API docs: https://map.longdo.com/docs/rest
- Legacy reference (all endpoints): https://api.longdo.com/map/doc/rest.php
- API console (get a key): https://map.longdo.com/console/
- In-browser map → **longdo-map-js** skill
