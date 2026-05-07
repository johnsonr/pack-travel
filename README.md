# pack-travel

Trip planning — Google Maps via OpenAPI, Airbnb via MCP, optional Brave web
research. Bundles a `plan-trip` goal and the chained actions
(`extract-travel-brief` → `find-points-of-interest` →
`research-points-of-interest` → `propose-travel-plan` → `find-accommodation`)
that together produce a day-by-day itinerary with stays.

## Required: `GOOGLE_MAPS_API_KEY`

Maps is exposed via the official Google Maps Platform OpenAPI 3 spec
(vendored at `apis/google-maps-platform-openapi3.json`). Authentication
is a Google Cloud API key passed in the `key` query parameter — the same
key you'd use from any other Maps client.

Set the env var before starting the assistant:

```bash
export GOOGLE_MAPS_API_KEY=AIza...
```

Get a key from the [Google Cloud console](https://console.cloud.google.com/google/maps-apis/credentials). Enable at minimum: **Geocoding API**, **Directions API**, **Distance Matrix API**, **Places API**, **Time Zone API**. Maps is billed per request against the Cloud project; restrict the key by API and (where possible) by IP/referrer.

The key is not per-user — it bills a single Cloud project, so it lives in
the deployment's environment, not the per-user credential store.

## Namespace

Methods land under `gateway.googleMaps`. Curated to ~8 ops covering
travel-planning needs:

- `gateway.googleMaps.geocode({address})`
- `gateway.googleMaps.directions({origin, destination, mode})`
- `gateway.googleMaps.distanceMatrix({origins, destinations, mode})`
- `gateway.googleMaps.placeDetails({place_id})`
- `gateway.googleMaps.findPlaceFromText({input, inputtype})`
- `gateway.googleMaps.nearbySearch({location, radius, type})`
- `gateway.googleMaps.textSearch({query})`
- `gateway.googleMaps.timezone({location, timestamp})`

Skipped from the full spec: Roads (snapToRoads / nearestRoads — driving
data collection, not trip planning), Geolocation, Street View, place
photos, elevation, autocomplete.

## Optional: research pack for web search

Several actions use the `brave` web search tool to fill gaps the typed
Maps surface doesn't cover (e.g. "what's worth seeing near X", recent
reviews, opening hours). Install [pack-research](https://github.com/johnsonr/pack-research) to enable it. Without it the actions still run but lose
web-search capability.

## Airbnb (still MCP)

Accommodation search uses the `mcp/openbnb-mcp` Docker MCP server (no
official OpenAPI spec exists). Requires Docker.

## Migration history

- **0.3.0** — Maps migrated from MCP (`mcp/google-maps` Docker) to
  OpenAPI. Drops the Docker dep for Maps, gives the LLM real Google
  response types instead of MCP's text-wrapped output. Requires the
  framework's `OpenApiLearner` to honor operation-level `servers:`
  overrides per OpenAPI 3 (added 2026-05-08) — the Maps spec uses
  per-operation hosts because it spans `maps.googleapis.com`,
  `roads.googleapis.com`, and `www.googleapis.com`.
