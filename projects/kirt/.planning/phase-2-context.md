# Phase 2: Core Search & Document Management — Context

## Phase Goal

Deliver a working search experience with deduplicated results, freshness indicators, and document upload/ingest — the data layer that all generation features depend on. By end of phase, a user can search across all ingested content with hybrid (keyword + semantic) results, upload a document and see it classified and indexed, and track ingestion job progress.

## What to Build

### Hybrid Search
- Hybrid search endpoint: single query across BM25 (keyword) + semantic (vector via Weaviate) — no separate search tabs
- Foundation `GET /v1/query/` integration with hybrid mode
- Structured filter support: `doc_type`, date range, client/account, author, status
- Relationship traversal queries via Weaviate cross-references (e.g., "all MSAs related to this client")
- Search result ranking: account intelligence dossier > recent meeting notes (last 90 days) > SF data > public research > playbooks/templates
- Freshness indicator: flag anything >30 days since last modified as potentially stale

### Deduplication UX
- 1 result per logical document by default (dedup filter applied before results reach KIRT)
- "N copies found" badge on result cards when `source_refs[]` has >1 entry
- Expandable detail: all source locations, `is_canonical` marker, last modified per location
- Exact duplicate detection via content hash (Foundation provides this; KIRT surfaces the UX)
- User can set a source_ref as canonical (`is_canonical` flag written to Foundation annotations)
- User can mark non-canonical copies as archived (Foundation `status: archived`, SP file untouched)
- Admin-configurable location-ranking for default canonical selection

### Document Upload & Ingest
- File upload UI → Foundation `POST /v1/ingest/` routing
- Auto-classification at ingest (LLM-based, V1 categories):
  - Meeting notes → signal extraction route
  - Transcript → diarization + signal route
  - BSE/account profile → entity extraction route
  - SOW/contract → offerings linkage route
  - Deck/presentation → subtype classification route
  - Template → mark as template, never modify
  - KIRT-generated → provenance tracking route
- Misclassification manual override by user
- Job progress tracking via Foundation `GET /v1/jobs/<id>/`
- In-app notification badges: new data available, classification complete

### Admin-Triggered Source Ingestion
- Admin UI section: add/remove SP sites and document libraries as input sources
- Triggers Foundation `POST /v1/connections/` and `GET /v1/connections/`
- Crawl schedule configuration
- Output destination configuration (separate from input sources — where KIRT writes deliverables back)

### SF CSV Import
- Upload CSV → parse → validate → ingest into Foundation via `POST /v1/ingest/`
- Supports SF report exports: accounts, contacts, opportunities, activities, notes
- Column mapping configuration for different SF export formats
- Validation: required fields, data type checks, duplicate detection

### Discovery Taxonomy Note
Discovery question taxonomy is already available in `demo-data/config/discovery-questions.json` (200+ questions, tree structure: business → industry → solution). No formalization work needed during this phase.

## Requirements Covered

- R05 — Hybrid search (BM25 + semantic)
- R06 — Structured filters
- R07 — Relationship traversal via Weaviate cross-references
- R08 — Deduplicated results (1 per logical document)
- R09 — "N copies found" badge
- R10 — Search result ranking
- R11 — Freshness indicator (>30 days flag)
- R12 — Exact duplicate detection via content hash
- R13 — User file upload → Foundation ingest
- R14 — Auto-classification at ingest
- R15 — Job progress tracking
- R16 — Misclassification manual override
- R17 — In-app notification badges (new data)
- R18 — Admin-initiated ingestion of new SP sites
- R47 — Canonical document selection via location-ranking
- R48 — User sets canonical source_ref
- R49 — User marks non-canonical copies as archived
- R52 — Output vs. input separation in admin config
- R04 — MSA `ai_processing_allowed` flag check at classification (opt-out content: keyword index only)

## Key Constraints

- Foundation enforces permissions at pre-query time. KIRT passes JWT on every request and surfaces only what Foundation returns. No post-fetch filtering.
- Dedup is invisible by default — 1 result per document. "N copies" detail is opt-in (accessible, not the default view).
- `ai_processing_allowed` flag must be checked before any LLM call in the classification pipeline. Opted-out content gets BM25 keyword indexing only. This must be tested explicitly.
- Auto-classification is LLM-based — requires the LLM abstraction layer from Phase 3 to be stubbed here or the two phases coordinate. Classification can use a single default provider until the full abstraction layer is complete.
- Location-ranking config is admin-managed, not hardcoded.

## Tech Stack

- **Search:** Foundation `GET /v1/query/` (hybrid BM25 + Weaviate HNSW semantic)
- **Dedup:** Content hash from Foundation canonical records; `source_refs[]` field
- **Classification:** LLM via abstraction layer stub (provider: admin-configurable default)
- **Upload:** Django file handling → `httpx` multipart to Foundation `POST /v1/ingest/`
- **Job polling:** Foundation `GET /v1/jobs/<id>/` — poll or SSE (Server-Sent Events) for progress
- **Annotations:** Foundation `POST /v1/annotations/` for classification writes
- **Frontend:** React search UI, result cards, upload component, badge indicators

## Dependencies

- Phase 1 complete: Foundation API client, auth, health endpoint
- Foundation v3 live with searchable content (requires some demo data ingested before integration testing)
- LLM abstraction layer stub (single provider sufficient for Phase 2 classification; full multi-provider in Phase 3)
- Admin configuration surface (minimal: location-ranking and input sources; full admin UI in Phase 5)

## Research Context

**Hybrid search pattern:** Foundation's Weaviate integration uses HNSW for approximate nearest neighbor vector search combined with BM25 for keyword relevance. Blending in a single query is the standard Weaviate hybrid search API — no need to build a custom blending layer. The blend happens in Foundation; KIRT just sends the query.

**Knowledge hierarchy for ranking:** The search result ranking (account dossier > meeting notes > SF > public > playbooks) is domain knowledge from the spec. Implement this as a weighted boost on document type, not a separate re-ranking pass — Foundation's query API supports result boosting via filters.

**Classification at ingest:** LLM-based classification with labeled training examples (spec requirement). For V1, use few-shot classification in the prompt — no model training needed. The label set is fixed: meeting notes, transcript, BSE, SOW, deck, template, KIRT-generated. Store the classification result as a Foundation annotation so it can be read back without re-running inference.
