# Foundation — Data Layer

> Shared data platform for the Forge consulting ecosystem. Ingests, normalizes, and serves — consumer apps own all workflow.
>
> Authoritative visual reference: `architecture.html` (this directory)

## Status

**v3.0 — Shipped.** Deployed on Contabo (Forge ecosystem). Target: Azure enterprise deployment.

## Purpose

Foundation is the data infrastructure layer. It ingests documents and structured data from source systems, normalizes entities, stores versioned content, generates embeddings, and serves everything through a REST API. No intelligence happens here — that's KIRT's job.

Foundation is admin-only. No end-user surface. Admins manage data source connections, ingestion schedules, normalization rules, and health monitoring.

## Stack

| Component | Technology | Notes |
|-----------|-----------|-------|
| Backend | Django (gunicorn, 3 workers) | SSO integration, data ingestion APIs |
| Storage | Apache Iceberg / Parquet | Versioned lakehouse on Azure Blob (target), local filesystem (current) |
| Vector DB | Weaviate | HNSW + hybrid search, 12 GB limit, nomic-embed-text (768-dim) |
| Database | PostgreSQL 16 | 3 DBs: foundation, polaris, dagster |
| Task Queue | Celery + Redis 7 | Prefork, 4 procs, 4 queues. Redis also handles Streams events. |
| Orchestration | Dagster | Webserver + Daemon. 6-asset ingest pipeline. |
| Iceberg Catalog | Polaris | REST catalog on port 8181 |
| Auth | AD/SSO via Graph API (target), token-based (current) |
| Cloud | Azure (target), Contabo VPS (current) |

## API Surface (13 Endpoints)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/v1/ingest/` | Async file ingestion via Dagster pipeline |
| POST | `/v1/query/` | Hybrid semantic + keyword search with freshness scoring |
| GET | `/v1/retrieve/<id>/` | Full document metadata and content |
| GET | `/v1/list/` | Paginated listing with lifecycle/metadata filters |
| DELETE | `/v1/delete/<id>/` | Soft delete or GDPR hard purge |
| POST | `/v1/annotations/` | Write classifications, tags, summaries |
| GET | `/v1/annotations/` | Read annotations for a document |
| GET | `/v1/jobs/<id>/` | Async job status (Dagster run tracking) |
| GET | `/v1/connections/` | Connector instance health |
| GET | `/v1/auth/permissions/` | API key permissions and role |
| GET | `/v1/health/<svc>/` | Per-service health check |
| GET | `/v1/health/quality/` | Data quality metrics and drift detection |
| GET | `/health/` | Quick reachability |

## Data Flow

```
Sources (SharePoint, Salesforce CSV, User Uploads via KIRT)
  ↓
Connectors (MSAL + httpx, delta sync)
  ↓
REST API → POST /v1/ingest/
  ↓
Dagster Pipeline (6-asset ingest chain)
  ↓
Parse (Docling / LlamaIndex) → Store Raw (Iceberg/Parquet) → Chunk (semantic strategies) → Embed (Ollama nomic-embed) → Index (Weaviate HNSW)
  ↓
Shadow Record (PostgreSQL metadata) → Redis Streams Event (document.ingested)
```

## Data Sources

### Phase 1 (Current)

| Source | Method | Content |
|--------|--------|---------|
| SharePoint | Scoped SP site, opt-in | Proposals, SOWs, architecture docs |
| Salesforce | Report exports (CSV) to SP site | Account data, pipeline, activity history |
| OpenAir | Data export | Time entry, project data |
| User uploads | Via KIRT → Foundation `POST /v1/ingest/` | Meeting notes, transcripts, ad-hoc documents |

### Phase 2+ (Future)

| Source | Method | Status |
|--------|--------|--------|
| Salesforce API | Direct integration | Deferred — waiting for new SF implementation |
| Email / Calendar | TBD | Needs privacy/policy review |
| Industry / News feeds | TBD | Needs scoping |

## Key Concepts

| Concept | Detail |
|---------|--------|
| `document_id` | SHA-256 of (tenant + source_path + content). Content-addressable. |
| `logical_document_id` | SHA-256 of (tenant + source_path). Groups version chains. |
| `lifecycle_state` | active → deleted_at_source → purged \| superseded |
| `embedding_status` | pending → embedded \| failed \| stale |
| Collections | SemanticChunks (text) + SlideChunks (PPTX with slide metadata) |
| Tenant isolation | Every query scoped by tenant_id. API keys bound to tenants. |
| Version tracking | parser_version, chunker_version, embedding_model on every chunk |

## Design Principles

- Foundation is a **data platform only** — no end-user UI, no business logic, no LLM classification
- Consumer apps (KIRT, Intel, Recon) own all workflow and intelligence
- All consumer apps interact through the REST API — never access Weaviate, Iceberg, or PostgreSQL directly
- SP files are never modified by Foundation — enrichments live in Foundation only

## What KIRT Consumes

KIRT is Foundation's primary consumer app. It reads via the query/retrieve/list/annotations APIs and writes back via the annotations API. KIRT triggers ingestion for user uploads via `POST /v1/ingest/`. Foundation emits `document.ingested` events via Redis Streams that KIRT subscribes to for near-real-time updates.

See `kirt-spec-seed.md` → "Data KIRT Consumes from Foundation" for the complete mapping.

## Roadmap

| Version | Status | What It Delivered |
|---------|--------|------------------|
| v1.0 — MVP | Shipped | Core platform: auth, lakehouse, parsing, embedding, vector search, API, connectors |
| v2.0 — Hardening | Shipped | Tech debt, multi-user RBAC, SharePoint connector hardening, Polaris catalog |
| v3.0 — Consumer App Readiness | Shipped | Dagster orchestration, Redis Streams, annotations, job tracking, data quality, version chains, GDPR purge |
| v4.0 — Platform Maturity | Planned | OpenMetadata governance, automated reconciliation, Spark query engine, advanced lineage |

KIRT V1 builds entirely on Foundation v3. See `kirt-spec-seed.md` → "Foundation v3 Alignment" for the dependency map.
