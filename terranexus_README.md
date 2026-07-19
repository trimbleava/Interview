# TerraNexus — event-driven geospatial intelligence platform

![Platform architecture](terranexus_pipeline.svg)

*(Renamed for interview use — this is the same platform referred to elsewhere as GeoDESX. Same
architecture, generic naming.)*

## What it does
A layered platform that ingests data from multiple external sources, fuses and canonicalizes it,
triggers scientific/scenario models automatically, and surfaces the results through a single
schema-driven API — all coordinated through Kafka rather than direct service-to-service calls.

## Architecture, stage by stage
1. **External connectors** — independent microservices wrapping social media, weather, and
   geospatial APIs behind a common interface, normalizing each source to a shared schema before
   it enters the pipeline.
2. **Kafka event bus** — the backbone of the whole platform. Every stage is a Kafka consumer
   group reading from one topic and publishing to the next; no service calls another service
   directly.
3. **Ingestion service** — a Kafka consumer with a topic-to-handler dispatch table (a small
   internal router: topic name → handler class), so adding a new data type means registering a
   new handler, not touching the consumer loop. Fuses and canonicalizes incoming data, then
   publishes a trigger event for the modeling stage.
4. **Simulation service** — an event-driven model executor. Listens for trigger events, runs the
   appropriate scientific/scenario model, and publishes the results as a "processed" event.
5. **Orchestration service** — consumes processed events, applies decision logic, and exposes
   analytics — the layer that turns raw model output into something a user-facing app can query.
6. **Schema-driven API gateway** — a FastAPI service with *zero hardcoded routes*; all routing is
   generated at startup from a YAML schema describing each backend service's endpoints. Adding a
   new backend route means editing the schema file, not the gateway code.
7. **Datastores** — Postgres with the PostGIS extension (GiST-indexed geometry columns) for
   spatial data, MongoDB for flexible/semi-structured documents.
8. **Storage abstraction layer** — a factory-pattern wrapper that toggles between cloud blob
   storage and local filesystem based on environment config, with retry logic and chunked upload
   for large raster files (GeoTIFFs).

## Technology summary

| Technology | How it's used in this project |
|---|---|
| `confluent_kafka.Consumer` | Topic-based consumer groups drive every stage — ingestion, simulation, orchestration each poll their own topic |
| Dispatch-table routing | Topic name → handler class lookup, so a new data/event type is a registration, not a new code path |
| `asyncio` | Non-blocking poll loop — `asyncio.sleep` yields control between Kafka polls instead of blocking |
| FastAPI | Each layer (gateway, ingestion, simulation, orchestration, connectors) is its own FastAPI microservice |
| Schema-driven routing | Gateway generates all routes at startup from a YAML schema — zero hardcoded routes |
| Postgres + PostGIS | GiST-indexed geometry columns for spatial data |
| MongoDB | Flexible/semi-structured document storage alongside the relational store |
| Storage abstraction (factory pattern) | Toggles between cloud blob storage and local filesystem via env config, with retry logic and chunked upload for large rasters (GeoTIFFs) |
| Docker | Every service ships with its own Dockerfile for independent deployment |

## How this maps to the job description

| JD requirement | What this project demonstrates |
|---|---|
| Asynchronous processing / task queues (Kafka explicitly listed) | Real `confluent_kafka.Consumer` usage — topic-based consumer groups, dispatch-table routing, async polling loop (`asyncio.sleep` yielding control between polls) |
| Service-oriented architecture / microservices | Every stage (ingestion, simulation, orchestration, gateway, each external connector) is an independently deployable service with its own Dockerfile |
| Database & caching — Postgres, sophisticated strategies | Postgres/PostGIS with GiST spatial indexing; MongoDB for the document side; a dedicated storage abstraction layer for large binary assets |
| Async programming patterns | Consumer loops built on `asyncio`, non-blocking polling with explicit event-loop yields |
| Implementation & maintainability within a service-based environment | Schema-driven gateway (routes generated from YAML, not hardcoded) and dispatch-table pattern in the ingestion service — both are "add a config entry, not new code paths" designs |
| Collaboration / translating requirements into scalable solutions | Platform was designed in explicit layers (ingestion → modeling → orchestration) so each layer's owner can change internals without breaking the topic contracts between them |
| ArcGIS / geospatial data integration (preferred) | PostGIS spatial data model; scientific models covering sea level rise, flood risk, infrastructure condition assessment |

## Strongest talking point for this JD
Lead with the Kafka piece — it's a direct, literal match to "task queues (e.g. Celery, RabbitMQ,
Redis Pub/Sub, **Kafka**)" in the requirements, not an analogy. Be ready to describe the
consumer-group/dispatch-table pattern concretely: each service polls its topic, decodes the
JSON payload, and routes it to a handler based on a lookup table — and to explain *why* Kafka
over Celery here (durable event log + replay, multiple independent consumer groups reading the
same stream, rather than a single task being claimed and discarded).
