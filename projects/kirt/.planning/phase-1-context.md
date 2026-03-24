# Phase 1: Foundation Integration & Project Skeleton — Context

## Phase Goal

Stand up a working Django backend that is authenticated, connected to Foundation v3.0, observable, and locally testable with mock Foundation API responses. No user-facing features yet — this phase proves the integration scaffolding is solid before anything is built on top of it.

## What to Build

- Django project initialized with MCP-native architecture (mirror Foundation reference POC: 9-layer structure, monolith design)
- Foundation API client — typed wrapper for all 13 Foundation v3.0 REST endpoints:
  - `POST /v1/ingest/`, `GET /v1/query/`, `GET /v1/retrieve/<id>/`, `GET /v1/list/`
  - `DELETE /v1/delete/<id>/`, `POST /v1/annotations/`, `GET /v1/annotations/`
  - `GET /v1/jobs/<id>/`, `GET /v1/connections/`, `POST /v1/connections/`
  - `GET /v1/auth/`, `GET /v1/health/`, `GET /v1/health/quality/`
- AD/SSO authentication via M365/Entra Graph API (JWT middleware)
- JWT claims passthrough to Foundation on every API request
- Health endpoint `/health/`: KIRT app status + Foundation connectivity
- Error tracking: Sentry integration (non-negotiable; set up first)
- Basic usage logging: who called what, when (structured log output)
- Redis Streams subscription: `document.ingested` event listener (stub/handler registered, processing logic deferred to Phase 2)
- Mock Foundation API server for local development (enables full unit test coverage without live Foundation dependency)
- Environment-based configuration (no hardcoded values): Foundation base URL, Sentry DSN, AD tenant ID, Redis connection
- MSA opt-out flag (`ai_processing_allowed`) check built into Foundation API client from day one — every query/retrieve response checked before passing to any LLM pipeline
- Celery + Redis task queue setup (worker configuration, task routing, result backend)
- Docker deployment setup (Dockerfile, docker-compose.yml for local dev and production)
- CI/CD pipeline skeleton (GitHub Actions workflow targeting Contabo CVPS3)
- SSE endpoint skeleton for real-time push notifications to frontend

## Requirements Covered

- R01 — SSO login via AD/Entra identity
- R02 — Single AD group, full platform access
- R03 — JWT claims to Foundation on every request
- R04 — MSA opt-out enforcement (partial — `ai_processing_allowed` flag check in API client)
- R55 — Health endpoint
- R56 — Error tracking (Sentry)
- R57 — Basic usage logging
- R61 — Error state when Foundation unavailable (health check triggers this)

## Key Constraints

- MCP-native architecture from the start. Don't retrofit it later.
- Foundation v3.0 only. Do not design API client around v4 capabilities.
- All Foundation calls must pass JWT claims from the authenticated user. No service-account bypass.
- Auth enforcement happens before any route is served.
- Secrets via environment variables only. No hardcoded Foundation URLs, Sentry DSNs, or AD credentials.
- Django error logging + Sentry: both active. Not one or the other.
- Mock server must cover all 13 endpoints with realistic response shapes — not stubs that return empty objects.

## Tech Stack

- **Backend:** Python, Django (MCP-native), Django REST Framework
- **Auth:** Microsoft Graph API / MSAL (AD/Entra SSO), JWT middleware
- **Foundation client:** `httpx` (async) or `requests` (sync), typed Python client
- **Redis:** `redis-py` for Streams subscription
- **Error tracking:** Sentry SDK for Python
- **Testing:** `pytest`, `pytest-django`, `responses` library (for mocking Foundation HTTP calls)
- **Config:** `python-decouple` or `django-environ` for environment-based config

## Dependencies

- Foundation v3.0 running and accessible (Contabo endpoint)
- Foundation API endpoint documentation and credentials
- AD/Entra tenant ID and app registration (KIRT needs a registered app in Entra for SSO)
- Sentry project created, DSN available
- Redis instance accessible (Foundation's Redis for Streams)

## Research Context

**Architecture pattern:** Foundation reference POC uses MCP-native Django monolith with 9 layers and 179 passing tests. Mirror this structure exactly — same module organization, same testing philosophy, same configuration approach. This is prior art, not a guess.

**Auth pattern:** M365/Entra SSO → JWT → Foundation permission enforcement. KIRT is the middle layer. Every request carries the user's Entra identity through the chain. No separate user database in KIRT for V1.

**Mock-first development:** Build the mock Foundation server before writing any feature code. This is the pattern that lets unit tests run without live infrastructure. The mock must return realistic data shapes — empty responses cause false-positive test passes.
