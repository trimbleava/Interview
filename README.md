# Position Software Developer

Software Developer
Location: Remote (Must be US-Based) Citizenship: US Citizen Required
Role Overview

We are seeking a Backend Developer to join our team and contribute to the delivery of high-quality software features.

In this role, you will collaborate closely with senior developers to implement and improve our core Python-based infrastructure. We are looking for a developer who excels at feature delivery while **adhering to established architectural patterns**. You will have the opportunity to work across our technology stack, ensuring our applications remain performant and reliable while supporting our long-term technical goals.

## Key Responsibilities

Feature Development: Build and maintain efficient, reusable, and reliable backend code using Python frameworks (Django, Flask, or FastAPI) following established patterns.

Asynchronous Processing: Architect and manage robust task queues (e.g., Celery, RabbitMQ, Redis Pub/Sub, Kafka) to support high-throughput operations.

Implementation & Maintenance: Assist in implementing system architecture decisions, ensuring code quality and maintainability within service-based environments.

System Optimization: Design and manage scalable database systems (Postgres) and implement high-performance caching strategies (Redis)

Implementation: Utilize async programming patterns to maximize application performance.

Collaboration: Work closely with the product and development teams to translate requirements into scalable technical solutions.

## Technical Qualifications

Python Proficiency: 3+ years of professional experience with Python and familiarity with web frameworks like Django, Flask, or FastAPI.

Background Processing: Experience with task queues (e.g., Celery, RabbitMQ, or Kafka) to handle distributed processing.

Database & Caching: Proficiency with Postgres or another relational database and experience developing sophisticated caching strategies using Redis.

Concurrency: Strong understanding of async programming and its application in backend systems.

Version Control (Git): Proficiency in using Git for version control and collaborative software development.

## Preferred Skills (Nice-to-Have)

Kotlin: Prior experience with Kotlin or a desire to work with it as we expand our capabilities.

Service-Oriented Architecture: Experience designing and maintaining microservices or service-based architectures.

Drone/AI Integration: Prior exposure to aerial robotics, drone swarm technologies, or AI-integrated web applications is a significant plus.

Python Tooling: Experience with modern Python tools such as UV, Prek, and Pytest.

ArcGIS: Experience working with ArcGIS mapping software and geospatial data integration.

# Backend Developer Interview Prep

## Frame yourself in one line

I'm a full-stack/backend engineer with deep experience building FastAPI and Django services for federal geospatial platforms — including message-queue-driven data pipelines (Kafka), PostgreSQL/PostGIS, [MCP server integrations](#what-is-mcp-server), and AI-integrated features — plus hands-on ArcGIS/ArcPy background


## Backend system with high-throughput or large data volumes.

**ASTRA / Airspace Impact Analysis Platform** 

- Built with FastAPI + Cesium.js, ingesting flight data from FlightRadar24 deliveries, adsb.lol ADS-B archives, and FAA extracts, staged through an Azure blob pipeline (raw → processed → normalized → daily spatial index).

- Before loading any full flight trace, a query service reads a lightweight daily index and narrows candidates down by bounding box and altitude range — everything outside that range is never opened. Same two-phase pattern as a GiST spatial index: cheap filter first, expensive check only on survivors.

- Implemented a 3D spatial intersection engine using line-segment clipping with interpolation (not simple point-in-polygon), so sparse ADS-B sampling doesn't produce false negatives on real airspace crossings — producing precise entry/exit points, crossing duration, and altitude stats.

- The same analysis feeds two outputs: a CZML builder for the Cesium 3D viewer, and a separate narrative/table/figure-library pipeline that assembles an automated NEPA-style PDF compliance report.

- **Talking point on throughput/architecture**: The raw flight archives were huge, so instead of scanning them per query I built an index-first pre-filter — bounding box and altitude range checked against a lightweight daily index before ever touching a full trace file. That's the same principle behind database spatial indexing, just built by hand over flat files, and it's directly the kind of caching/optimization asked with Postgres and Redis.

### ASTRA — airspace impact analysis platform

<img src="astra_pipeline.svg" alt="Airspace Impact Analysis Platform" width="80%" height="auto">

**What it does**

Determines whether historical or proposed flight traffic passes through a designated airspace
volume (a Military Operations Area or other Special Use Airspace), and produces both an
interactive 3D visualization and an automated compliance-style report from the same underlying
analysis — built to support NEPA-style airspace impact assessments.

**Architecture, stage by stage**

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

**Feature Table**

| Requirement | What this project demonstrates |
|---|---|
| Python 3+ | Typed dataclasses, clean service/core/api layering, Shapely-based geometric algorithms |
| System optimization / caching strategies | Index-based bounding-box pre-filtering avoids loading full trace files — the same cheap-filter-then-precise-check pattern as a spatial database index, built by hand over flat files |
| Database & scalable systems | Daily indexed metadata catalog (bounding box, altitude range, time window) functions as a lightweight query layer in front of the raw archive |
| Implementation & maintainability within a service-based environment | Report generation is deliberately split into narrative/table/figure libraries plus an orchestration-only builder — each piece can change independently |
| FastAPI web framework | Routers cleanly separated by domain (airspaces, flights, viewer), each mounted under its own API prefix |
| ArcGIS / geospatial data integration (preferred) | Shapely-based 3D geometric intersection engine handling real airspace volumes (lateral polygon + altitude band), not just 2D point-in-polygon |

### Describe your experience with service-oriented architecture / microservices

**TerraNexus platform** *(see [architecture diagram & README](#terranexus-platform-diagram))*

- Layered SaaS platform: external connectors (data ingestion from third-party sources), ingestion/canonicalization, scenario modeling, and analytics/orchestration, plus domain-specific applications on top.

- Each layer deployed as its own FastAPI microservice, containerized with Docker.

- Used **Kafka** for event/message passing between ingestion and downstream services, **MongoDB** for flexible document storage, **PostgreSQL/PostGIS** for spatial data.

- Built connector services with async transport (SSE) between services — a strong, current example of designing service boundaries.

- **Talking point**: "This platform was essentially a service mesh of purpose-built FastAPI microservices coordinated through Kafka — ingestion services publish events, downstream modeling/analytics services consume them. That maps closely to the task-queue/high-throughput requirements in this JD."

**MCP connector suite** *(see [architecture diagram & README](#mcp-connector-suite-diagram))*

- A supporting piece of the same platform's ingestion layer: nine independent MCP servers, each wrapping a different external platform (social media, news, AI research) behind a standardized tool interface, each its own FastAPI service in its own Docker container.

- The data acquisition adapter in front of them has an explicitly documented architectural boundary — it normalizes and passes data through, and deliberately does *not* do storage, dedup, or scoring, so that responsibility stays cleanly downstream.

- **Talking point**: "This is a good concrete example of service-boundary discipline — I documented what the adapter is and isn't responsible for, in writing, so the system stays legible as more connectors get added. That's the kind of judgment behind 'established architectural patterns,' not just a list of tools."

### Tell me about a Django project you've built end-to-end

**SoloGrow**

- Django-based mentoring platform: mentor/mentee roles, Google Calendar OAuth integration, Stripe payments, and an AI mentor chat feature built on the Anthropic API (with full conversation history maintained server-side).

- Handled real production concerns: OAuth refresh token edge cases, CI/CD via GitHub Actions to GCP, Nginx + Gunicorn + PostgreSQL + Let's Encrypt in production, layered settings (`settings_base.py` → `settings_production.py`).

- **Talking point**: I've taken a Django app from local dev all the way through a real deployment pipeline — Gunicorn behind Nginx, Postgres, SSL — so I'm comfortable with both the framework and what it takes to run it reliably in production.

### Do you have experience with async programming in Python?

- Point to FastAPI work (async by default) across ASTRA and TerraNexus microservices — async endpoints for I/O-bound operations like fetching/parsing large flight archives and spatial queries.

- If pushed on Celery specifically and you haven't used it directly, be honest but bridge: *I haven't used Celery directly in production, but I've built the equivalent pattern with Kafka-based async processing between services in TerraNexus — producer/consumer decoupling, idempotent processing, retry handling. I'd expect the concepts to transfer quickly, and I'm comfortable ramping up on Celery/RabbitMQ specifically.*

### Tell me about your GIS/ArcGIS background

**Sea Level Rise projects (Mayport, Newport), ArcPy work** *(see [pipeline diagram & README](#mayport-sea-level-rise-diagram))*

- Built DoD sea level rise assessment tools using ArcGIS Pro automation, IDW/spatial interpolation, LiDAR data processing, and automated layout generation via ArcPy.

- Also built a Cesium/Three.js 3D digital-twin viewer and a 4D hydrodynamic visualization tool ([What is ManilaBay, MERF](#what-is-manilabay-merf)) — processing hundreds of millions of records from MATLAB/Delft3D models through Python pipelines with checkpoint/resume logic *(see [pipeline diagram & README](#matlab-to-arcgis-pipeline-diagram))*.

- **Talking point**: "Checkpoint/resume was essential there — some of those pipelines ran for hours over massive GDB files, so I built in resumability rather than re-running from scratch on failure. That's the same reliability mindset you'd want in a task-queue-driven system.

### Any experience with AI-integrated applications?

**SoloGrow AI mentor feature**

- Built an AI mentor chat feature parallel to the human-mentor flow, using the Anthropic API with full conversation history, a dedicated chat UI, and Stripe checkout tied to AI mentor access.

- This is a direct, current example matching their "AI-integrated web applications" nice-to-have.

**WebGPU RAG knowledge graph (prototype)** *(see [pipeline diagram & README](#webgpu-rag-prototype-diagram))*

- A second, different flavor of AI integration — a full retrieval-augmented generation pipeline (chunking, embedding index, vector retrieval, in-browser LLM generation) running entirely client-side via WebGPU, no backend or API key required.

- Worth mentioning as a contrast to SoloGrow: this one shows you understand the RAG pattern itself (retrieval → grounding → generation), not just calling a hosted API. Caveat honestly that it's a JavaScript/React prototype, not Python, if the conversation goes there.

### Tell me about a time you had to debug something gnarly across a distributed/service system.

**3D viewer debugging (NAVFAC platform)**

- Debugged text sprite rendering issues in [geocentric (ECEF) space vs. local/visual coordinate systems](#what-is-ecef) — a subtle cross-system coordinate mismatch bug.

- Built a PostMessage API for parent-child iframe communication, scene versioning with save/load, and movement history/undo — shows you can reason about state synchronization across service/component boundaries, which is core to distributed backend work too.


## 2. Technical Concept Refreshers

### Celery basics (if you want a quick mental model going in)

- **Broker** (RabbitMQ or Redis) holds the task queue; **workers** pull tasks and execute them; optional **result backend** stores return values.

- Key concepts to be ready to discuss: task idempotency, retries with backoff (`autoretry_for`, `max_retries`), task routing to specific queues, `chain`/`group`/`chord` for task composition, and monitoring via Flower.

- Analogy to draw on: your Kafka producer/consumer pattern in TerraNexus — a "task" being published and a worker consuming it is conceptually the same shape as a Celery task being queued and picked up by a worker.

### Redis caching strategies (common interview angle)
- Cache-aside (lazy loading): app checks cache first, falls back to DB, writes result to cache.

- Write-through vs. write-behind caching.

- TTL/eviction policies (LRU, LFU) and cache invalidation strategies — the classic "two hard things in computer science" joke is fine to make here.

- Using Redis for more than caching: session storage, rate limiting, pub/sub, and as a lightweight task broker for Celery.

- **Real example, not just theory** — **DFLOW** *(see [pipeline diagram & README](#dflow-diagram))*: a Django/GeoDjango storm surge model verification app using `django-redis` as the cache backend. Cache-aside for UI facet options, composite-key caching for expensive NetCDF-derived graph computations, and Redis also backing Django's session store. This is your go-to answer when caching questions get specific — it has a real, slightly-blunt invalidation tradeoff (`cache.clear()` on every home load) that's worth naming honestly if asked.

- **A second, different Redis pattern** — **multi-tenant leak survey mapping app** *(see [pipeline diagram & README](#leak-survey-mapping-diagram))*: Redis as the full Django session backend (`SESSION_ENGINE = "django.contrib.sessions.backends.cache"`, sessions live entirely in Redis, not the DB) for a real-time, multi-tenant WebSocket workflow via Django Channels. Good for showing Redis breadth beyond cache-aside — use both projects together if the interviewer digs in.

### Async Python patterns

- `async`/`await`, event loop, `asyncio.gather` for concurrent I/O.

- Why async matters for I/O-bound work (DB calls, external API calls) vs. CPU-bound work (where you'd want multiprocessing or offloading to a worker queue instead).

- FastAPI's native async support — you can speak to this directly from your ASTRA and TerraNexus work.

### Postgres at scale

- Indexing strategy (B-tree vs. GIN/GiST — GiST/PostGIS spatial indexes are a great crossover point given your PostGIS experience).

- Connection pooling (pgbouncer), query optimization via `EXPLAIN ANALYZE`.

- Spatial indexing specifically, since it shows deeper Postgres knowledge than most candidates will have.

The real mental model.

**B-tree** is Postgres's default index type. It works by keeping keys in sorted order, so it's great for equality and range queries: `WHERE id = 5`, `WHERE created_at > '2026-01-01'`, `ORDER BY`. It answers "is this value less than, equal to, or greater than that value" — a total ordering. That's fine for scalars (numbers, dates, text) but breaks down for geometry, because there's no single natural ordering for "is this polygon less than that polygon."

**GIN (Generalized Inverted Index)** is built for when a single row contains *multiple values* you want to search into — arrays, JSONB keys, full-text search tokens. It maps each individual value back to the rows containing it, like a book index. Fast lookups, but slower to update since every value needs its own index entry. You'd use GIN for something like `WHERE tags @> ARRAY['flood']` or full-text search on a document.

**GiST (Generalized Search Tree)** It's a balanced tree, but instead of sorting on a single comparable value, each node stores a "bounding" summary of everything below it — for spatial data, that's a bounding box. Querying walks the tree checking "does my search area overlap this node's bounding box?" and prunes whole branches that don't. It generalizes to any data where you can define an overlap/containment relationship, which is why it fits geometry so well — polygons, lines, points don't have a natural sort order, but they do have a natural "does this overlap that" relationship.

That's exactly what PostGIS's `GEOMETRY`/`GEOGRAPHY` columns use by default: `CREATE INDEX idx_name ON table USING GIST (geom_column);`. When you run something like `ST_Intersects(a.geom, b.geom)` or `ST_DWithin`, Postgres uses the GiST index to quickly narrow down candidate rows by bounding-box overlap before doing the expensive exact geometry calculation — this two-phase approach (cheap bounding-box filter, then precise check) is often called the "index-primary, exact-secondary" pattern, and PostGIS handles it transparently for you.

Quick way to say this in the interview if it comes up: *B-tree is for ordered scalar data, GIN is for indexing multiple values per row like arrays or full-text tokens, and GiST is for data with spatial or overlap relationships — which is why PostGIS geometry columns default to GiST indexes. The index does a fast bounding-box filter first, then Postgres runs the precise geometric operation only on the candidates that survive.*

**SP-GiST** (space-partitioned GiST, good for non-balanced structures like quadtrees) and **BRIN** (block range index, good for huge tables that are naturally ordered on disk, like time-series data) — you probably won't need them, but if someone asks "what other index types exist," naming them shows breadth.

## 3. Questions to Ask Them

- What does the current task queue setup look like — Celery with RabbitMQ or Redis as the broker? Are you using it mainly for background jobs, or also for real-time/high-throughput pipelines?

- How is the service-oriented architecture split today — is it a handful of larger services or a broader microservices footprint?

- You mentioned drone swarm/aerial robotics integration — is that an active initiative, or more exploratory at this point?

- What does the caching layer look like in production — mostly cache-aside, or are there write-through patterns for specific data?

- Since this is remote and US-citizen-required, is that tied to a specific federal contract, similar to DoD/NAVFAC-type work? (this lets you naturally mention your federal contracting background)

## 4. One risk area to prep for honestly

You don't have explicit Celery/RabbitMQ production experience in your history — your equivalent experience is Kafka-based async processing in TerraNexus. Don't overstate Celery specifically; instead, confidently bridge from Kafka/async patterns you've actually built, and be upfront that you'd ramp up quickly on Celery's API since the underlying concepts (queues, workers, retries, idempotency) are the same ones you've already designed around.


# TerraNexus platform diagram

![TerraNexus platform architecture](terranexus_pipeline.svg)

Full writeup, JD mapping table, and the strongest talking point for this project: [terranexus_README.md](terranexus_README.md)

# ASTRA pipeline diagram

![ASTRA pipeline architecture](astra_pipeline.svg)

Full writeup, JD mapping table, and the strongest talking point for this project: [astra_README.md](astra_README.md)

# Mayport Sea Level Rise diagram

![Mayport SLR pipeline architecture](mayport_slr_pipeline.svg)

Full writeup, JD mapping table, and the strongest talking point for this project: [mayport_slr_README.md](mayport_slr_README.md)

# WebGPU RAG prototype diagram

![WebGPU RAG pipeline architecture](webgpu_rag_pipeline.svg)

Full writeup and honest framing for this project: [webgpu_rag_README.md](webgpu_rag_README.md)

# MCP connector suite diagram

![MCP connector suite architecture](mcp_servers_pipeline.svg)

Full writeup, JD mapping table, and the strongest talking point for this project: [mcp_servers_README.md](mcp_servers_README.md)

# DFLOW diagram

![DFLOW pipeline architecture](dflow_pipeline.svg)

Full writeup, JD mapping table, honest framing, and the strongest talking point for this project: [dflow_README.md](dflow_README.md)

# Leak survey mapping diagram

![Leak survey mapping architecture](leak_survey_pipeline.svg)

Full writeup, JD mapping table, and the strongest talking point for this project — including how its Redis usage differs from DFLOW's: [leak_survey_README.md](leak_survey_README.md)

# MATLAB-to-ArcGIS pipeline diagram

![MATLAB to ArcGIS pipeline architecture](matlab_arcpy_pipeline.svg)

Full writeup and JD mapping table: [matlab_arcpy_pipeline_README.md](matlab_arcpy_pipeline_README.md)

# What is SAR - satellite change detection diagram

![Change detection pipeline architecture](sar_change_detection_pipeline.svg)

*This one isn't part of the case for the current backend role — no Django/FastAPI/task queues, and
it's intentionally not mapped against this JD. Keeping it here for reference in case a future
geospatial-intelligence or remote-sensing role comes up. Full writeup:
[sar_change_detection_README.md](sar_change_detection_README.md)*


# What is ManilaBay, MERF

The ManilaBay project refers to a cumulative impact assessment of reclamation projects on the
rehabilitation of Manila Bay in the Philippines.

- **Modeling conducted by:** Marine Environment and Resources Foundation, Inc. (MERF) — a private,
  non-profit research organization that developed the flood-risk and water-circulation models used
  in the assessment.
- **Funded and commissioned by:** Department of Environment and Natural Resources (DENR) — the
  Philippine national government agency responsible for environmental regulation, which used the
  study's findings to review reclamation projects around the bay.

# What is ECEF

ECEF stands for Earth-Centered, Earth-Fixed. It's a 3D Cartesian coordinate system where the origin sits at the Earth's center of mass, and everything is expressed as (X, Y, Z) in meters — no latitude, longitude, or altitude involved.

The Z-axis points through the North Pole.
The X-axis points through the equator at the prime meridian (0° longitude).
The Y-axis is perpendicular to both, completing a right-handed system.
The whole frame rotates with the Earth, so a fixed point on the ground (like a building) always has the same ECEF coordinates regardless of Earth's rotation relative to the sun or stars.

This is different from the coordinate systems people usually think in:

Geodetic (lat/lon/height) — human-readable, but not Cartesian, so math like distance or vector operations gets messy.
ENU (East-North-Up) — a local tangent-plane system centered wherever you're standing. Great for "which way is up" at one spot, but it rotates depending on where on the globe you are.
ECEF — the bridge between the two. It's Cartesian (so you can do normal vector math, matrix transforms, etc.) but still globally consistent, which is exactly why 3D geospatial engines like Cesium and iTowns use it internally as their base reference frame.

That's almost certainly what bit you in the 3D viewer bug: iTowns renders the whole scene in ECEF/geocentric space, but text sprites (billboards that should always face the camera and sit "upright" relative to the ground) need to be oriented using the local ENU frame at that point on Earth — not the global ECEF axes. If you compute the sprite's up/right vectors using the wrong frame, the label looks correctly positioned in 3D but tilts or drifts as the camera orbits, because "up" in ECEF terms isn't the same as "up" relative to the ground at that specific point on a curved planet. The fix is almost always: convert the ECEF position to local ENU at that point, orient the sprite in ENU space, then transform back.

# What Is MCP Server

MCP stands for Model Context Protocol — an open standard (originally introduced by Anthropic) that lets AI models connect to external tools, data sources, and services through a consistent interface, rather than each application needing a custom integration for every model it talks to.

In the context of what we've been building in your interview prep — the "MCP connector suite" project — MCP servers are the pieces that expose an external service (BlueSky, a news API, etc.) as a standardized set of callable tools that any MCP-compatible client (like Claude Desktop, or a service inside your platform) can use, without that client needing to know the specifics of each underlying API.

If it comes up in the interview: it's a good one-liner to have ready — "MCP is a standard protocol for connecting AI applications to external tools and data sources — I used it to build a set of independent servers, each wrapping a different third-party platform behind one consistent tool-calling interface."

# What Is Salary

This is one of the most common tripwires in interviews, so it's worth getting the framing right on both parts.

## "Why do you want to leave your current role?"

Don't lead with "I'm underpaid" — even though that's honestly a big part of it, salary-first framing tends to read as reactive rather than intentional, and it invites the interviewer to wonder if you'll just leave again for the next higher bidder. The move is to frame it as **scope/market alignment**, which is honest without being the whole story:

> "I've built a lot at [current company] — geospatial platforms, service architectures, production systems running at real scale — and I'm proud of that work. But I've reached a point where the scope of what I'm delivering has outgrown where I'm leveled and compensated, and I'm looking for a role where both the work and the pay reflect that. This role specifically caught my attention because it's a natural next step — deeper work with task queues, service-oriented architecture, and the kind of backend systems I've been building toward."

Why this works: it's true, it doesn't badmouth your current employer (never do that, even if justified — it's the single fastest way to make an interviewer wary), and it pairs the compensation point with genuine interest in the *work*, which is what they actually want to hear.

## "What if they offer the same salary?"

A few different things could happen here — worth being ready for each:

**If it's this new company's initial offer, and it lands at your current salary:** Don't accept or decline on the spot. Compensation is more than base salary — ask about the full picture: bonus/equity, PTO, remote flexibility, benefits, and importantly, the *growth ceiling*. A same-salary offer with a clear path to a higher band in 12 months, or work that meaningfully builds your resume toward your next jump, can still be worth it. If it's genuinely a lateral move with no upside, that's useful information — you can say so honestly:

> "I appreciate the offer. Given the scope of what I'd be taking on here, I was hoping for something closer to $X. Is there flexibility on the number, or on other parts of the package?"

That's a normal, expected ask — not confrontational. Companies build in room to negotiate; asking doesn't put the offer at risk.

**If your current employer counters to match once you give notice:** Be cautious here. Counteroffers solve the number but rarely solve the underlying reason you started looking — the ceiling, the scope, the recognition. Data on counteroffers is pretty consistent: a large share of people who accept one still leave within 6–12 months anyway, because the original reasons resurface. If you accept a counter, ask yourself honestly whether the actual issue (not just the number) got fixed.

One practical note: in most US states, employers legally can't ask your current salary directly — if it comes up, you can redirect to a target range instead of your exact number ("I'm looking for something in the $X–Y range based on my experience and the scope of this role") rather than anchoring low by disclosing what you're underpaid at now.

```
git add astra_README.md astra_pipeline.svg dflow_README.md dflow_pipeline.svg interview_prep_full.md leak_survey_README.md matlab_arcpy_pipeline.svg matlab_arcpy_pipeline_README.md mayport_slr_README.md mcp_servers_README.md mcp_servers_pipeline.svg  sar_change_detection_README.md terranexus_README.md terranexus_pipeline.svg webgpu_rag_README.md webgpu_rag_pipeline.svg mayport_slr_pipeline.svg sar_change_detection_pipeline.svg
```
