[← Back to interview prep](interview_prep_full.md#tell-me-about-your-gisarcgis-background)

# Mayport Sea Level Rise — ArcGIS pipeline

![Mayport SLR pipeline architecture](mayport_slr_pipeline.svg)

## What it does
Turns raw LiDAR point cloud data and documented sea level rise projections into scenario-based
flood depth maps for a DoD coastal installation, then packages and publishes those results both
as offline map deliverables and as interactive 3D web scenes.

## Architecture, stage by stage
1. **LiDAR processing** — compressed LAZ point cloud tiles are converted to LAS, cleaned (VLRs
   and extra bytes stripped, points rearranged for spatial indexing), and built into a LAS
   dataset in NAD83 UTM Zone 17N, from which a bare-earth DEM is derived.
2. **Bathtub flood model** — for each SLR scenario, a depth raster is computed as scenario water
   elevation minus DEM elevation, masked with a conditional raster operation so only cells below
   the scenario water level are kept. Every scenario is produced in both meters and feet.
3. **Scenario configuration** — SLR scenario elevations aren't hardcoded numbers; each one is
   documented against the source methodology (global SLR projection + site-specific adjustments +
   MHHW offset + extreme water level), with the underlying storm event data (e.g., Hurricane
   Matthew and Irma observed storm tides) kept alongside the config for traceability.
4. **ArcGIS Pro layout automation** — depth rasters are reprojected to Web Mercator for web
   sharing, AOI bookmarks are copied between maps programmatically, and new layouts are cloned
   per scenario rather than built by hand each time.
5. **Two delivery paths from the same processed data**:
   - **Map tile packages** — per-scenario layouts packaged for offline delivery and print.
   - **ArcGIS Online publishing** — the same reprojected depth rasters are published as a tile
     service and assembled into an interactive 3D web scene for stakeholder review.

## How this maps to the job description

| JD requirement | What this project demonstrates |
|---|---|
| Python proficiency, 3+ years | Modular ArcPy automation split across LiDAR processing, flood modeling, config, and publishing — not one monolithic script |
| System optimization / scalable processing | LAS point-cloud cleanup (VLR/extra-byte removal, point rearrangement) done once as a preprocessing step specifically to make downstream DEM generation faster |
| Implementation & maintainability | Scenario elevations and their derivations live in one documented config module, not scattered magic numbers — changing a projection methodology means editing one file |
| Collaboration / translating requirements into technical solutions | Scenario values are traceable back to the source methodology (SLR + adjustments + MHHW + storm data), so a reviewer can audit *why* a number is what it is |
| ArcGIS (preferred skill) | Full ArcPy pipeline: point cloud processing, raster algebra for flood modeling, layout/bookmark automation, and ArcGIS Online publishing via the `arcgis` Python API |

## Strongest talking point for this JD
This project is a good answer whenever "maintainability" or "established architectural patterns"
comes up — the scenario configuration is deliberately separated from the processing logic, so a
reviewer, a new SLR projection update, or a change in methodology touches one config file instead
of hunting through the code for hardcoded elevation values. That's the same principle the JD is
asking about with "adhering to established architectural patterns," just applied to a geospatial
domain instead of a typical backend service.
