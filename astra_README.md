[← Back to interview prep](interview_prep_full.md#tell-me-about-a-backend-system-you-built-that-had-to-handle-high-throughput-or-large-data-volumes)

# ASTRA — airspace impact analysis platform

![ASTRA pipeline architecture](astra_pipeline.svg)

## What it does
Determines whether historical or proposed flight traffic passes through a designated airspace
volume (a Military Operations Area or other Special Use Airspace), and produces both an
interactive 3D visualization and an automated compliance-style report from the same underlying
analysis — built to support NEPA-style airspace impact assessments.

## Architecture, stage by stage
1. **Data acquisition** — scripts pull historical flight data from multiple external sources:
   FlightRadar24 (FR24) delivery files, adsb.lol ADS-B archives, and FAA data extracts.
2. **Azure ingestion pipeline** — FR24 deliveries move through a staged pipeline: raw files land
   in blob storage, get extracted/copied into a processed tier, are normalized into a common
   `FlightTrack` JSON schema, and are summarized into a daily spatial index (bounding box,
   altitude range, time window per flight).
3. **Index query service** — before loading any full flight trace, the service reads the
   lightweight daily index and narrows the candidate set down to only the traces whose bounding
   box and altitude range could plausibly intersect the requested airspace volume. Everything
   else is never even opened. *(This is the same two-phase pattern PostGIS uses with GiST
   indexes — cheap bounding-box filter first, precise check only on survivors — applied here to
   flat-file flight archives instead of a database.)*
4. **3D spatial intersection engine** — for each candidate track, flight segments are tested with
   a line-segment clipping approach rather than simple point-in-polygon checks: each segment is
   checked for lateral overlap with the airspace polygon, positions are interpolated along any
   overlapping segment (since sparse ADS-B sampling can otherwise miss a real crossing), and each
   interpolated point is checked against the airspace's altitude band. Only segments passing both
   the lateral and vertical test are kept, producing precise entry/exit points, crossing duration,
   and altitude statistics.
5. **Two outputs from one analysis**:
   - **CZML builder** — converts qualified crossings into CZML, color-coded by traffic category
     (military, air taxi, air carrier, general aviation), served to a Cesium-based 3D viewer.
   - **Report builder** — a separate, composable pipeline (narrative library, table library,
     figure library, all coordinated by an orchestration-only report builder) assembles the same
     underlying analysis into a formatted PDF compliance report.

## How this maps to the job description

| JD requirement | What this project demonstrates |
|---|---|
| Python proficiency, 3+ years | Typed dataclasses, clean service/core/api layering, Shapely-based geometric algorithms |
| System optimization / caching strategies | Index-based bounding-box pre-filtering avoids loading full trace files — the same cheap-filter-then-precise-check pattern as a spatial database index, built by hand over flat files |
| Database & scalable systems | Daily indexed metadata catalog (bounding box, altitude range, time window) functions as a lightweight query layer in front of the raw archive |
| Implementation & maintainability within a service-based environment | Report generation is deliberately split into narrative/table/figure libraries plus an orchestration-only builder — each piece can change independently |
| FastAPI web framework | Routers cleanly separated by domain (airspaces, flights, viewer), each mounted under its own API prefix |
| ArcGIS / geospatial data integration (preferred) | Shapely-based 3D geometric intersection engine handling real airspace volumes (lateral polygon + altitude band), not just 2D point-in-polygon |

## Strongest talking point for this JD
Lead with the index-query optimization when caching/database-strategy questions come up — it's a
concrete, hand-built example of the exact bounding-box-pre-filter-then-precise-check pattern that
GiST spatial indexes automate in Postgres. Being able to explain *why* that two-phase approach
matters (avoid the expensive exact computation until you've already ruled out the vast majority of
candidates) shows you understand the underlying principle, not just the tool that implements it.
