[← Back to interview prep](interview_prep_full.md#redis-caching-strategies-common-interview-angle)

# DFLOW — storm surge model verification app

![DFLOW pipeline architecture](dflow_pipeline.svg)

## What it does
A Django/GeoDjango web application that lets a user pick a hydrodynamic model, scenario, storm
event, and observation station on a Leaflet map, then generates a time-series comparison graph of
modeled storm surge/water level against real station observations — used to verify how well a
DFlow/Delft3D model run matches what actually happened during a given storm.

## Architecture, stage by stage
1. **Facet selection UI** — a Leaflet-based map lets the user pick model, scenario, event category,
   storm event, and station; the available options for each dropdown come from a shared "facets"
   structure rather than being queried fresh on every request.
2. **Facet cache (Redis)** — `get_globals()` is a textbook cache-aside implementation: check
   `cache.get("facets")` first, and only fall back to querying the database to rebuild the facet
   options if the cache misses, then write the result back with `cache.set()`.
3. **Verification graph builder** — reads model output directly from NetCDF files (`netCDF4`) and
   pulls the matching station observation records, aligns them on a common time axis, and produces
   a structured comparison series.
4. **Result cache (Redis)** — the built graph data is cached under a composite key
   (`station_id + model + scenario + prediction_type + event`), so re-requesting the same
   station/scenario/event combination skips the NetCDF read and recomputation entirely.
5. **Output** — the cached or freshly-built graph payload is returned as JSON and rendered
   client-side as a D3 time-series chart.

Two things run alongside the main flow:
- **PostGIS / GeoDjango** — station locations, coastline, and bay/boundary geometries are modeled
  as real PostGIS geometry fields (`gismodels.PolygonField`, `MultiPolygonField`), feeding both the
  Leaflet map layer and the station lookups the graph builder needs.
- **Redis session store** — the same `django-redis` cache backend also backs Django's
  `cached_db` session engine, so sessions and application-level caching share one Redis instance.

## How this maps to the job description

| JD requirement | What this project demonstrates |
|---|---|
| Database & caching — Postgres, sophisticated caching strategies using Redis | Real `django_redis.cache.RedisCache` backend, used for cache-aside facet loading, composite-key result caching, and serialized-queryset caching — not just configured, actually load-bearing in the request path |
| Django web framework | Full Django project: GeoDjango models, class-based views, management commands, custom serializers |
| Python proficiency, 3+ years | NetCDF scientific data handling (`netCDF4`, `scipy.interpolate`, `pandas`) combined with a standard Django app structure |
| Database & scalable systems (Postgres) | PostGIS-backed geometry models for spatial station/boundary data, queried through GeoDjango's ORM |
| Implementation & maintainability | Facet and result caching are isolated in small, focused functions (`get_globals`, per-station cache keys) rather than scattered inline through views |

## Honest note for the interview
The cache invalidation strategy here is blunt in places — `HomeView` calls `cache.clear()` on every
load, which wipes the *entire* cache, not just the facets key. That's a real, defensible tradeoff
in a low-traffic internal tool (simplicity over precision), but be ready to name it as exactly that
if asked — a good interviewer may probe on why a targeted `cache.delete()` wasn't used instead, and
"I'd tighten that to a targeted invalidation in a higher-traffic system" is a strong, honest answer.

## Strongest talking point for this JD
This is your most direct, load-bearing Redis example — lead with the composite-key result caching
in the graph builder when caching questions come up. It's not just "I configured Redis"; it's
"I cached an expensive derived computation — a NetCDF read plus alignment against station
observations — behind a cache key built from the specific parameters that make that result unique,"
which is the actual skill the JD is asking about.
