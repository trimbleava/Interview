[← Back to interview prep](interview_prep_full.md#redis-caching-strategies-common-interview-angle)

# Multi-tenant leak survey mapping — real-time workflow

![Leak survey mapping architecture](leak_survey_pipeline.svg)

## What it does
A multi-tenant Django application used by gas utility companies to generate leak-survey maps and
map overlays — each tenant (utility company) has its own configuration, GeoServer SLD styling, and
data model overrides, and survey/overlay generation runs as a real-time, multi-step workflow over
a WebSocket connection rather than a single blocking request.

## Architecture, stage by stage
1. **WebSocket client** — the browser opens a persistent connection for the duration of a survey
   or overlay generation job, rather than polling.
2. **Django Channels (ASGI)** — routes each connection to the right consumer
   (`SurveyMapConsumer`, `OverlayConsumer`, `DeleteOverlayConsumer`, `OverlayLegendConsumer`)
   based on the workflow being run.
3. **Tenant activation** — a slug identifies which utility company (Southwest Gas Nevada, Texas,
   Philippines; Vectren; others) is connecting, and loads that tenant's specific config file, SLD
   styles, and model overrides — each tenant is its own folder with its own settings, not a shared
   config with conditionals.
4. **Redis-backed session** — as the multi-step workflow runs (engine startup → tenant config
   parsing → GeoServer styling → generation), state like region, survey name, and copyright is
   written into `self.scope["session"]`, which is backed entirely by Redis rather than the
   database.
5. **Survey/overlay generation engine** — drives GeoServer SLD styling and the underlying GIS/ETL
   engine, streaming progress messages back to the client over the same socket as each step
   completes.

## A different Redis pattern than DFLOW — worth knowing the distinction
DFLOW uses Redis as an **explicit cache-aside layer**: application code calls `cache.get()` /
`cache.set()` around expensive computations. This project uses Redis differently —
`SESSION_ENGINE = "django.contrib.sessions.backends.cache"` means Django sessions live **entirely**
in Redis, not the database at all. There's no explicit `cache.get`/`cache.set` call anywhere in
the view code; the caching is structural, baked into how Django's session framework is configured,
which matters specifically because a WebSocket-driven, multi-step, per-tenant workflow needs
durable, fast per-connection state between steps.

**Also worth being precise about in the interview**: `channels-redis` is a real dependency in
`requirements.txt`, and a Redis-backed `CHANNEL_LAYERS` config exists in the settings file — but
it's currently commented out. That configuration would enable Redis-backed WebSocket group
messaging for horizontal scaling across multiple workers. As it stands in this codebase snapshot,
it's scaffolded but not active — be ready to say exactly that if asked, rather than overclaiming a
distributed pub/sub layer that isn't currently wired in.

## How this maps to the job description

| JD requirement | What this project demonstrates |
|---|---|
| Database & caching — sophisticated caching strategies using Redis | A second, structurally different Redis pattern from DFLOW: Redis as the full session backend for a stateful, multi-step real-time workflow, not just a cache-aside wrapper |
| Async programming patterns | Django Channels / ASGI — WebSocket handling is inherently async-first, distinct from the synchronous request/response pattern in DFLOW |
| Service-oriented / multi-tenant architecture | Each tenant is a self-contained module (own config, own SLD styles, own model overrides) activated dynamically by slug at connection time |
| Implementation & maintainability | Tenant-specific customization lives in per-tenant folders and config files rather than branching logic scattered through shared views |

## Strongest talking point for this JD
Use this project specifically to show Redis breadth, not just repetition of the DFLOW story: "I've
used Redis two different ways — as an explicit cache-aside layer in one project, and as the actual
backing store for session state in a real-time, multi-tenant WebSocket workflow in another. The
second one mattered because the workflow is stateful across multiple steps, and needed something
faster and more ephemeral than the database for that." That's a stronger answer than describing
either project alone.
