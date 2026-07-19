[← Back to interview prep](interview_prep_full.md#describe-your-experience-with-service-oriented-architecture--microservices)

# MCP connector suite — external signal acquisition

![MCP connector suite architecture](mcp_servers_pipeline.svg)

## What it does
A collection of independent MCP (Model Context Protocol) servers, each wrapping a different
external platform (social media, news, AI-powered research) behind a standardized tool interface,
so any MCP-speaking client can pull external signals through one consistent protocol instead of
integrating against nine different third-party APIs directly. Built as the external "sensing"
layer for the TerraNexus platform's ingestion pipeline.

## Architecture, stage by stage
1. **MCP client** — any MCP-speaking application (Claude Desktop, or a service inside TerraNexus)
   connects to the suite as a set of remote tools.
2. **Nginx reverse proxy** — sits in front of all nine services, routing by port, with structured
   access logging that captures upstream connect/response timing per request.
3. **Nine platform MCP servers** — each is an independent FastAPI service (BlueSky, X/Twitter,
   LinkedIn, YouTube, TikTok, Instagram, Facebook, a news aggregation service, and an AI-powered
   deep research service), each in its own Docker container, each wrapping a third-party scraping
   or search API behind a common MCP tool-call interface.
4. **Data acquisition adapter** — a deliberately thin, stateless layer with a documented
   architectural boundary: it calls the upstream MCP servers, normalizes whatever comes back into
   a small, consistent "Observation" schema with source provenance attached, and explicitly does
   **not** do durable storage, deduplication, confidence scoring, or cross-source analysis — that
   responsibility is pushed downstream on purpose, so this layer stays swappable and easy to reason
   about.
5. **Normalized observations** are handed off to downstream ingestion, where storage, indexing,
   and analysis actually happen.

Two supporting systems run alongside the main flow:
- **Config manager** — a single source of truth for every service's configuration, with a layered
  precedence (`local` overrides `environment` overrides `example` defaults), so the same codebase
  runs correctly across local dev, staging, and production without code changes.
- **Jenkins CI/CD** — a parameterized pipeline that can build, test, and deploy any subset of the
  nine services independently, followed by a health-check pass across all service ports before a
  deployment is considered complete.

## How this maps to the job description

| JD requirement | What this project demonstrates |
|---|---|
| Service-oriented architecture / microservices (preferred) | Nine independently deployable FastAPI services, each with its own Dockerfile, each swappable without touching the others |
| Implementation & maintainability within a service-based environment | The acquisition adapter's boundary is explicitly documented — what it does and, just as importantly, what it deliberately does *not* do — so the architecture stays legible as it grows |
| System architecture decisions | Layered environment configuration (local → environment → example) as a single source of truth, rather than config scattered per service |
| Version control / collaborative development (Git) | Parameterized Jenkins pipeline supports per-service builds, forced rebuilds, and selective deployment, backed by a health-check gate before release |
| FastAPI web framework | Every one of the nine services is built on FastAPI |
| Async programming patterns | Services use `aiohttp` for non-blocking upstream API calls and expose WebSocket endpoints alongside REST |

## Strongest talking point for this JD
Lead with the *architectural boundary* documentation in the acquisition adapter when "adhering to
established architectural patterns" comes up — it's a concrete example of deciding, in writing,
what a service is and isn't responsible for (acquire and normalize, don't store or score), which
is exactly the kind of discipline that keeps a service-oriented system maintainable as more
services get added. It's a stronger answer than just listing the tech stack, because it shows the
judgment behind the architecture, not just familiarity with the tools.
