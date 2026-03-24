# Integration Map — System Connections

> How systems connect in the KIRT + Foundation architecture. Aligned to `kirt-spec-seed.md`.

## System Topology

```
┌──────────────────────────────────────────────────────────────────┐
│                        SOURCE SYSTEMS                            │
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────┐  ┌───────────┐ │
│  │ SharePoint   │  │ Salesforce   │  │ OpenAir  │  │ Public    │ │
│  │ (proposals,  │  │ (2 instances │  │ (time,   │  │ Data APIs │ │
│  │  SOWs, docs) │  │  Converge +  │  │  project │  │ (SEC, news│ │
│  │              │  │  Mainline)   │  │  data)   │  │  feeds)   │ │
│  └──────┬───────┘  └──────┬───────┘  └────┬─────┘  └─────┬─────┘ │
│         │  Scoped site     │  CSV export   │  Export      │ API   │
└─────────┼──────────────────┼───────────────┼─────────────┼───────┘
          │                  │               │             │
          ▼                  ▼               ▼             ▼
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│                    FOUNDATION (v3.0)                              │
│               Shared Data Platform — Admin Only                  │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │ REST API (13 endpoints)                                     │ │
│  │ POST /v1/ingest/  POST /v1/query/  GET /v1/retrieve/<id>/  │ │
│  │ GET /v1/list/  DELETE /v1/delete/<id>/  POST/GET annotations│ │
│  │ GET /v1/jobs/<id>/  GET /v1/connections/  GET /v1/health/*  │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────┐ ┌────────────┐ ┌────────┐ ┌────────┐ ┌───────┐ │
│  │ Weaviate   │ │ Iceberg/   │ │ Postgres│ │ Redis  │ │Dagster│ │
│  │ HNSW+hybrid│ │ Parquet    │ │ 16     │ │ 7      │ │       │ │
│  │ vector+BM25│ │ versioned  │ │ 3 DBs  │ │ broker │ │ 6-asset│ │
│  │            │ │ lakehouse  │ │        │ │+streams│ │pipeline│ │
│  └────────────┘ └────────────┘ └────────┘ └────────┘ └───────┘ │
│                                                                  │
└──────────────────────────┬───────────────────────────────────────┘
                           │
                    REST API + Redis Streams
                    (document.ingested events)
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│                         KIRT                                     │
│               Intelligence Layer — User-Facing                   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │ React + Vite + Tailwind + Shadcn (frontend)                 │ │
│  │ Django + MCP-native (backend)                               │ │
│  │ AD/SSO via Graph API (auth)                                 │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌───────────────┐  │
│  │ Search Engine     │  │ Generation Engine │  │ Classification│  │
│  │ hybrid query      │  │ LLM-powered       │  │ LLM-based     │  │
│  │ knowledge ranking │  │ template routing   │  │ auto-classify │  │
│  │ dedup display     │  │ section regen      │  │ on ingest     │  │
│  └──────────────────┘  └──────────────────┘  └───────────────┘  │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌───────────────┐  │
│  │ Scoring Engine    │  │ Upload Pipeline   │  │ Admin/Config  │  │
│  │ growth scoring    │  │ user files →      │  │ SP paths      │  │
│  │ opportunity score │  │ Foundation ingest  │  │ scoring weights│ │
│  │ maturity gate     │  │ auto-classify     │  │ catalog mgmt  │  │
│  └──────────────────┘  └──────────────────┘  └───────────────┘  │
│                                                                  │
└──────────────────────────┬───────────────────────────────────────┘
                           │
                    Write-back (user-initiated)
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│                        OUTPUTS                                   │
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │ SharePoint       │  │ KIRT UI          │  │ Foundation      │ │
│  │ output folders   │  │ (user views)     │  │ annotations     │ │
│  │                  │  │                  │  │                  │ │
│  │ Account Briefs   │  │ Search results   │  │ Classifications │ │
│  │ Engagement Preps │  │ Account intel    │  │ Scores          │ │
│  │ Client Docs      │  │ Cross-sell recs  │  │ Enrichments     │ │
│  │ Reframe Decks    │  │ Stakeholder maps │  │ Tags            │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## Data Flow Directions

| From | To | Direction | What Flows | Method |
|------|----|-----------|-----------|--------|
| SharePoint | Foundation | Read | Proposals, SOWs, docs, templates | Scoped SP site connector, delta sync |
| Salesforce | Foundation | Read | Account, pipeline, activity (CSV) | Report exports uploaded to SP site |
| OpenAir | Foundation | Read | Time entry, project data | Data export |
| KIRT (user upload) | Foundation | Write | Meeting notes, transcripts, ad-hoc docs | `POST /v1/ingest/` |
| Foundation | KIRT | Read | Search results, doc metadata, enrichments | `POST /v1/query/`, `GET /v1/retrieve/`, annotations APIs |
| Foundation | KIRT | Event | New document notifications | Redis Streams (`document.ingested`) |
| KIRT | Foundation | Write | Classifications, scores, tags | `POST /v1/annotations/` |
| KIRT | SharePoint | Write | Generated deliverables | SP API, user-initiated with confirmation |
| M365/Entra | KIRT → Foundation | Auth | JWT claims, group memberships | Graph API, passed on every request |

## Authentication Flow

```
User → M365/Entra (AD/SSO via Graph API) → KIRT → Foundation
```

- KIRT authenticates the user via M365/Entra identity
- JWT claims passed to Foundation query API on every request
- Foundation enforces `sp_permissions[]` intersection with user's M365 group memberships
- RBAC inherited from AD groups — KIRT adds no additional permissions
- MSA opt-out clients: KIRT queries via Graph API passthrough (read-only, no Foundation ingest)

## Source-of-Truth Model

```
SharePoint ────────── Document source-of-truth (files live here)
       │
       ▼
Foundation ────────── Raw data source-of-truth (ingested, normalized, searchable)
       │
       ▼
KIRT ─────────────── Intelligence source-of-truth (enrichments, scores, classifications)
       │
       ▼
SharePoint ────────── Deliverable output (generated docs written back, user-initiated)
```

## Phase 2+ Integrations (Future)

| System | Direction | Purpose | Version |
|--------|-----------|---------|---------|
| Salesforce API | Read | Real-time account data (replaces CSV) | V4+ |
| Email/Calendar | Read | Meeting scheduling, correspondence signals | V4+ |
| News/Industry feeds | Read | M&A, regulatory, competitive intelligence | V2 (SEC EDGAR) |
| Red Hat FlightPath | Read | Assessment frameworks, maturity models | Reference only |

## What KIRT Does NOT Access Directly

KIRT never touches Foundation's internal infrastructure:
- No direct Weaviate queries — goes through Foundation's `/v1/query/`
- No direct Iceberg reads — goes through Foundation's `/v1/retrieve/`
- No direct PostgreSQL access — uses Foundation's APIs
- No direct Redis access — subscribes to Foundation's event stream

This separation means Foundation can migrate infrastructure (Contabo → Azure) without KIRT code changes.
