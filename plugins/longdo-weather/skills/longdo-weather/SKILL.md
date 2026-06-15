---
name: longdo-weather
description: >-
  Use the Longdo Weather API — rain radar, current conditions, wind, and
  forecasts for Thailand. Covers authentication with the shared Longdo API key
  and the common request conventions. Use when integrating Longdo weather data.
  NOTE: the exact endpoint paths and response fields still need to be confirmed
  against the official docs (see the "Needs verification" section) — this skill
  documents the verified conventions, not invented endpoints.
---

# Longdo Weather API

Official docs: https://weather.longdo.com/docs
Live service: https://weather.longdo.com/
Get a free API key: https://api.longdo.com/console/

Longdo Weather (by Metamedia Technology) provides **rain radar, current
conditions, wind, and forecasts** for Thailand, rendered on top of the Longdo
Map. It is part of the same Longdo API ecosystem as the map and location REST
services.

> **Status of this skill:** the verified, stable facts below (auth, request
> conventions, the service surface) are safe to rely on. The precise endpoint
> URLs, query parameters, and JSON response schemas live behind the JavaScript
> "API Docs" app at https://weather.longdo.com/docs, which can't be extracted
> automatically. Confirm those against the official page before shipping — see
> **Needs verification**.

## Authentication

The Longdo Weather API uses the **same unified Longdo API key** as the map and
REST services. Create / manage keys in the console:
https://api.longdo.com/console/

- Pass your key as `key=YOUR_API_KEY` on each request (the convention shared by
  all Longdo REST services).
- **Keep the key server-side.** Like the location REST APIs, call weather
  endpoints from your backend and proxy results to clients rather than exposing
  the key in browser/mobile code. See the **longdo-map-rest** skill for the
  same key-handling guidance.

## Shared request conventions

Longdo REST services (which the weather API follows) use:

- `GET`/`POST` with `application/x-www-form-urlencoded`.
- JSON responses by default; `callback=fnName` for JSONP.
- Location via `lon` / `lat` in WGS84 degrees.
- `locale=th` (default) or `locale=en`.

So a weather request will almost certainly take, at minimum, a location
(`lon`/`lat` or a place/area identifier), your `key`, and an optional `locale`.

## Rain radar on a Longdo map

Longdo Weather overlays its rain radar on a Longdo map, which means the radar is
exposed as a map tile layer. In the **JavaScript API** (see **longdo-map-js**)
custom raster layers are added with `new longdo.Layer(name, { ... })` and
`map.Layers.add(...)`. To display the rain radar layer, get the exact layer
name / tile template from the official weather docs and add it the same way you
add any Longdo raster layer.

## Needs verification (fill in from https://weather.longdo.com/docs)

When you have the official docs (or can paste them), this skill should be
completed with the concrete details:

- [ ] Base URL(s) for the weather endpoints.
- [ ] Endpoint paths for: **current conditions**, **forecast**, **wind**, and
      **rain radar** (tile layer name / template).
- [ ] Required vs. optional parameters per endpoint.
- [ ] Units (temperature °C, wind speed, rainfall mm, timestamps/timezone).
- [ ] Response JSON shape per endpoint, with a worked example.

Until those are confirmed, do **not** guess endpoint paths in generated code —
point the user at the official docs instead.

## Reference

- Weather API docs: https://weather.longdo.com/docs
- Weather service: https://weather.longdo.com/
- API console (get a key): https://api.longdo.com/console/
- Shared key/REST conventions → **longdo-map-rest** skill
- Adding the radar tile layer to a map → **longdo-map-js** skill
