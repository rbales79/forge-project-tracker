# PreFlight Handoff — KIRT

> **Intake Synthesis Mode.** All decisions extracted from `kirt-spec-seed.md` and `demo-data-requirements.md` (2026-03-24). Decisions marked `[FROM SOURCE]` are locked — treat as if confirmed in Q&A. No decisions were deferred or overridden during synthesis. All solutioning session items resolved and merged into spec seed.

---

## What This Is

KIRT (Knowledge In Real Time) is the AI-powered intelligence layer for strategic account management. It sits on top of Forge Foundation v3.0 (the shared data platform) and serves anyone who needs account intelligence — from Enterprise Architects running the full Strategic Account Framework to AEs who need to prep for a meeting in 10 minutes.

KIRT is **not** the SAF. The SAF is one workflow that runs on KIRT. KIRT's capabilities are organized around the SAF's 6 phases, but every feature is independently accessible. Users don't follow a linear pipeline — they use what they need, when they need it.

**Two-product architecture:**
- **Foundation** — shared data platform. Ingests, normalizes, serves raw data. Admin-only, no user surface. Running on Contabo (v3.0), target Azure.
- **KIRT** — user-facing intelligence layer. Greenfield build. All business logic, classification, scoring, generation, and recommendation lives here.

---

## Core Value

**For Enterprise Architects:** 80%+ reduction in document preparation time across the full SAF lifecycle (account briefs, engagement prep, discovery scoring, deliverable generation).

**For AEs and individual contributors:** Meeting prep in 10 minutes. Upload notes, get something sendable. Account intelligence on demand without following any framework.

**For Sales Leadership:** A single source of truth for account intelligence — searchable, current, and generated from the organization's own data (SP documents, SF records, meeting notes).

**The killer differentiator:** KIRT works with whatever data exists. Graceful degradation with data completeness indicators means every feature delivers value even on thin data — it tells you what's missing instead of failing silently.

---

## Context

### Ecosystem
KIRT is a member of the Forge ecosystem, built on Foundation v3.0. Foundation is operational (Contabo). KIRT is a greenfield build — no existing application code.

### Foundation Dependency
Foundation v3.0 provides 13 REST API endpoints that KIRT depends on:
- `POST /v1/ingest/` — document upload routing
- `GET /v1/query/` — hybrid search (BM25 + semantic via Weaviate)
- `GET /v1/retrieve/<id>/` — document retrieval
- `GET /v1/list/` — document listing with filters
- `DELETE /v1/delete/<id>/` — GDPR purge (hard delete option)
- `POST /v1/annotations/` + `GET /v1/annotations/` — KIRT writes classifications, scores, tags
- `GET /v1/jobs/<id>/` — ingestion and generation progress tracking
- `GET /v1/connections/` — source connection management
- `POST /v1/connections/` — configure new data sources
- `GET /v1/auth/` — authentication/identity passthrough
- `GET /v1/health/` — Foundation health check
- `GET /v1/health/quality/` — data freshness and quality metrics
- Redis Streams: `document.ingested` events for near-real-time reactions

Foundation reference POC: 9 layers, 179 tests passing, MCP-native architecture, monolith design.

### Source of Truth Model
- **SharePoint** = document source-of-truth (source files live here, never modified by Foundation/KIRT. KIRT-generated output files in designated folders are updated in place.)
- **Foundation** = raw data source-of-truth (ingested, normalized, searchable)
- **KIRT** = intelligence source-of-truth (enrichments, classifications, scores, generated insights, write-back deliverables)

### Prior Art
Foundation POC architecture (MCP-native, Django monolith, 179 passing tests) is the reference implementation for KIRT's backend architecture. Same pattern, extended for intelligence features.

---

## Constraints

### Technical Constraints `[FROM SOURCE]`
- KIRT builds on Foundation v3.0 only. No v4 features required or assumed.
- Django backend, MCP-native architecture — consistent with Foundation reference POC.
- Multi-provider LLM abstraction from day one: Gemini, Azure OpenAI, AWS Bedrock + any OpenAI-compatible endpoint. Zero provider-specific code in the application. Provider selection is admin-configurable.
- Weaviate is the confirmed vector DB for V1. Evaluate alternatives only if Weaviate proves insufficient — no preemptive replacement.
- **Source** SP files are never modified by Foundation or KIRT. KIRT writes deliverables to designated output folders. Generated output files are updated in place on regeneration — SP versioning preserves previous versions automatically. KIRT does not edit source files in their original locations.
- **Input/output path separation is enforced at admin config time.** Output folder must not equal, be a parent of, or be a child of any configured input source path. Same SP site with different folders is allowed (e.g., `/Acme/Documents/` input, `/Acme/KIRT-Output/` output). This prevents KIRT from re-ingesting its own output and ensures source vs. generated content is always distinguishable by folder structure.
- KIRT does not maintain its own document store. All document storage routes through Foundation via `POST /v1/ingest/`.
- Permission inheritance is strict: KIRT never surfaces a document the user couldn't access in its source system. Enforced pre-query in Foundation, not post-fetch in KIRT.

### Scope Constraints `[FROM SOURCE]`
- **Single-tenant V1.** Internal Pellera deployment only. Foundation's `tenant_id` scoping supports future multi-tenancy without rewrite.
- **Single AD group, full access V1.** Tier enforcement is V2+.
- **Web app only.** Teams tab is future (iframe wrapper). No native app.
- **Generation and admin are desktop-only in V1.** Mobile: read flows (search, view briefs) via responsive Tailwind design.
- **No Foundation v4 features.** Don't design around automated reconciliation, Spark query engine, advanced lineage, or OpenMetadata.
- **Cached/degraded mode is V2+.** V1 shows an error state when Foundation is unavailable.

### Operational Constraints `[FROM SOURCE]`
- All admin-configurable settings must be tunable per deployment without code changes: SP output paths, directory structures, scoring model weights, offerings catalog, LLM provider selection, data source configurations.
- GDPR purge is KIRT's responsibility. Foundation's purge cascade does not include annotations. When a source document is purged, KIRT must explicitly delete its own annotations via the Foundation annotations API.
- MSA opt-out: `ai_processing_allowed` flag must be checked before any LLM pipeline. Opted-out content: keyword search only, no summarization, no AI classification, no generation context.

### Data Constraints `[FROM SOURCE]`
- No seed data in Foundation today. Demo data (4+ accounts) must be ingested before end-to-end testing.
- Offerings catalog v0.1: will be available (real or synthetic) for V1. Features degrade gracefully without it.
- Discovery question taxonomy: 200+ questions structured as business→industry→solution tree, available in `demo-data/config/discovery-questions.json`.

---

## Requirements Signals

### Must-Have (V1 MVP)

#### Authentication & Access
- **R01** SSO login via AD/Entra identity (M365/Entra Graph API)
- **R02** Single AD group, full platform access (no tier enforcement in V1)
- **R03** JWT claims passed to Foundation on every request; Foundation enforces SP permissions
- **R04** MSA opt-out enforcement: `ai_processing_allowed` flag check before any LLM pipeline

#### Search & Discovery
- **R05** Hybrid search: keyword (BM25) + semantic (vector via Weaviate) in a single query — no separate tabs
- **R06** Structured filters: doc_type, date range, client, author, status
- **R07** Relationship traversal via Weaviate cross-references
- **R08** Deduplicated results: 1 result per logical document by default
- **R09** "N copies found" badge on result cards (expandable to see all source locations)
- **R10** Search result ranking: account dossier > recent meeting notes > SF data > public research > playbooks
- **R11** Data freshness indicator: flag anything >30 days as potentially stale
- **R12** Exact duplicate detection via content hash (V1)

#### Document Management & Ingestion
- **R13** User file upload → Foundation ingest via `POST /v1/ingest/`
- **R14** Auto-classification at ingest: meeting notes, transcripts, BSE, SOW, deck, template, KIRT-generated
- **R15** Job progress tracking via Foundation `GET /v1/jobs/<id>/`
- **R16** Misclassification manual override by user (V1)
- **R17** In-app notification badges: new data, stale briefs, generation complete
- **R18** Admin-initiated ingestion of new SP sites/document libraries via KIRT config UI

#### Account Brief Generation
- **R19** Account Brief generation in <60 seconds (soft target)
- **R20** Sections: company overview, key contacts, business priorities, relationship history, open opportunities, competitive landscape, strategic notes, white space analysis, action items
- **R21** Works with partial data — generates from whatever exists, flags gaps
- **R22** Data completeness indicators: sources used, what's missing, quality score, improvement suggestions
- **R23** Deliverable write-back to SP output folder with generation metadata annotation

#### Engagement Prep Generation
- **R24** Engagement Prep package: prior work, pain points, stakeholder map, talking points, discovery questions, next-best-question suggestions
- **R25** Stakeholder map pre-populated (user-editable)
- **R26** Discovery questions from 200+ question taxonomy (tree: business → industry → solution), filtered by account context
- **R27** Works without prior Phase 1 completion — generates from available context

#### Meeting Summary & Follow-Up
- **R28** Upload meeting notes → generate polished recap (prose, not bullet lists)
- **R29** Structured action items extracted from notes (with owners and dates)
- **R30** Draft follow-up email suitable for sending to client
- **R31** Account intelligence update: pain points, competitive mentions, stakeholder signals extracted and fed back into account context via Foundation annotations
- **R32** Works from notes alone, enriched by existing account context. Lowest-friction entry point: single upload → something sendable

#### Client Deliverable Generation
- **R33** Input: company name + keywords → client-ready documentation with account history. All templates use Pellera format (template-driven generation from `demo-data/templates/`).
- **R34** Template pulled from SP (via Foundation's template canonical)
- **R35** Research run → LLM generation → SP write-back → provenance summary shown to user

#### Cross-Sell Recommendations
- **R36** Pattern-based suggestions from engagement history: "accounts like this also bought..."
- **R37** Triggered after data ingestion or meeting note upload
- **R38** Requires offerings catalog as reference frame (less precise without it, still generates)

#### Document Generation Infrastructure
- **R39** Generation progress: real-time indicators via Foundation job API; "Still working — pulling data from N sources" if >60 seconds
- **R40** Human-in-the-loop editing: inline edit, section regeneration
- **R41** Prompt-level feedback learning: store edited outputs, use as few-shot context for future generation for same account/type
- **R42** Concurrent generation handling: both complete, both write to SP, KIRT surfaces "2 versions generated" indicator, user picks winner
- **R43** Content versioning badge: "Generated v3, last updated 2026-03-20"
- **R44** SP write-back conventions: configurable output folders, `[Client]-[DocType]-[Date].docx` naming
- **R45** Write-back is explicit and user-initiated with confirmation step

#### SAF Features
- **R46** SAF progress tracking as optional overlay for EA users (not default view)
- **R47** Growth scoring: 5 dimensions (revenue, strategic value, relationship depth, engagement frequency, white space size), equal weights (20% each) in V1, tunable via admin
- **R48** Manual tier override for growth scoring

#### Duplicate/Variant Management
- **R49** Canonical document selection via admin-configurable location-ranking
- **R50** User can set a source_ref as canonical (writes `is_canonical` flag to Foundation)
- **R51** User can mark non-canonical copies as archived (Foundation `status: archived`, SP file untouched)

#### Admin Configuration
- **R52** Admin-configurable settings (without code changes): SP input sources, SP output folders, crawl schedules, location-ranking, scoring weights, LLM provider/model, offerings catalog
- **R53** Admin-triggered GDPR purge with annotation cleanup
- **R54** Output vs. input separation: source documents and KIRT-generated deliverables go to separate SP locations

#### Observability
- **R55** Health endpoint `/health/`: KIRT app health + Foundation connectivity
- **R56** Error tracking: Django error logging + Sentry (non-negotiable)
- **R57** Basic usage logging: who searched, who generated, when

#### Responsive UI
- **R58** Mobile read access: search, view Account Brief, view Engagement Prep (responsive via Tailwind)
- **R59** Task-oriented UI: "What do you need?" — not phase-oriented
- **R60** Generation and admin: desktop-only in V1

#### Error Handling
- **R61** Error state shown when Foundation is unavailable
- **R62** Graceful degradation on every feature — works with partial data, communicates what's missing

### Deferred (V2+)

- **D01** Near-duplicate detection (fuzzy matching) — V2
- **D02** Cross-format dedup (docx ↔ pdf) — V2
- **D03** Role-based access tiers (AD-group-to-role mapping) — V2
- **D04** Per-account filtering (AE sees only their accounts, requires SF ownership) — V2
- **D05** Opportunity scoring (ML-informed, transcript-based) — V2
- **D06** Offering maturity gate — V2
- **D07** Email/push notifications (configurable per user) — V2
- **D08** Cached/degraded mode when Foundation unavailable — V2
- **D09** RAG feedback loop (edits flow back as KIRT-refined documents) — V2
- **D10** Automated SF export → narrative pipeline — V2
- **D11** Public company monitoring (SEC EDGAR integration) — V2
- **D12** APM (request latency percentiles) — V2
- **D13** Generation quality scoring (satisfaction signals from edit rates) — V2
- **D14** Automated GDPR retention policies — V2
- **D15** Transcript analysis (speaker diarization, signal extraction) — V2
- **D16** Stale data alert when archived SP file gets edited — V2
- **D17** Reframe deck, assessment kickoff, transformation roadmap, EBR/QBR generation — V3
- **D18** Agentic RAG (multi-hop retrieval) — V3
- **D19** Template management with version control — V3
- **D20** Full provenance tracking in generated deliverables — V3
- **D21** Multi-tenancy — V4+
- **D22** SF direct API (replace CSV) — V4+
- **D23** Email/calendar integration — V4+
- **D24** Multi-agent architecture per SAF phase — V4+
- **D25** Teams tab wrapper — future (post V1)
- **D26** WCAG compliance — post-V1
- **D27** Production-scale Azure migration — post-V1

### Non-Goals (explicitly out of scope, permanent)
- KIRT is not a CRM replacement. SF stays as system of record for structured CRM data.
- KIRT does not edit source files in SharePoint. It writes deliverables to output folders and updates them in place on regeneration.
- KIRT does not maintain a document store. Foundation is the single storage layer.
- KIRT does not surface intermediate research/generation process to users. Users see output only.
- KIRT does not decide which document copy is authoritative. Users decide, KIRT surfaces the data.

---

## Key Decisions (Locked)

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Frontend stack | React + Vite + Tailwind + Shadcn | Consistent with Forge ecosystem. Rapid UI dev, accessible component library. |
| Backend stack | Django, MCP-native | Consistent with Foundation reference POC (9 layers, 179 tests). MCP-native from the start. |
| Vector DB | Weaviate (HNSW + hybrid search) | Confirmed V1 solution. Evaluate alternatives only if insufficient. |
| Storage | Apache Iceberg / Parquet on Azure Blob | Research/scoring intermediates only. SP is the document source of truth. |
| Auth | AD/SSO via Graph API | M365/Entra identity. JWT claims to Foundation on every request. |
| Cloud target | Azure (target), Contabo (current Foundation) | Foundation on Contabo; KIRT targets Azure from design. Migration is post-V1. |
| Deployment | Docker on Contabo CVPS3, CI/CD via GitHub Actions | V1 deployment target. Azure migration is future. |
| Task queue | Celery + Redis | Async LLM generation, classification, cross-sell. |
| Real-time | Redis Streams (backend events) + SSE (frontend push) | Backend events from Foundation; SSE pushes updates to browser. |
| SF data | Synthetic CSV for demo, real CSV reports future, API V4+ | CSV import mechanism in Phase 2. Direct API access is V4+. |
| SharePoint test site | forgecf.sharepoint.com | Test/demo site for SP integration. Entra app registration created when needed. |
| LLM strategy | Multi-provider abstraction. No provider-specific code. | Gemini, Azure OpenAI, AWS Bedrock + OpenAI-compatible. Admin-configurable. |
| LLM approved providers | Gemini, Azure OpenAI, AWS Bedrock | Approved by Pellera. Any OpenAI-compatible endpoint also supported. |
| Form factor | Web app, responsive | Teams tab is future. No native app. Generation/admin: desktop-only V1. |
| Tenancy | Single-tenant V1 | Internal Pellera deployment only. Foundation `tenant_id` ready for future. |
| Access control V1 | Single AD group, full access | Tier enforcement is V2+. |
| Document ownership | Foundation owns storage | KIRT routes uploads via `POST /v1/ingest/`. No KIRT-owned document store. |
| SP write policy | Write-back is explicit, user-initiated | No automatic SP writes. Confirmation required. Output files updated in place on regeneration (SP versioning preserves history). |
| Permission model | SP permission inheritance. Strict. | Foundation enforces at pre-query time. KIRT never post-filters. |
| Duplicate default | 1 result per document, dedup invisible | "N copies" badge accessible. V1: content hash exact dedup. V2: fuzzy. |
| Canonical selection | Admin-configurable location-ranking | Not a hardcoded SP hierarchy. User can override. |
| Scoring V1 | Equal weights (20% each across 5 dimensions). Manual override. | V2+: ML-informed. V1 precision is less critical because of manual override. |
| MSA opt-out | `ai_processing_allowed` flag. Keyword search only. | No LLM processing on opted-out content. No exceptions. |
| GDPR purge | KIRT owns purge logic. Must explicitly delete own annotations. | Foundation cascade doesn't cover annotations. KIRT is responsible. |
| Feedback learning V1 | Prompt-level. Store edited outputs as few-shot context. | RAG loop is V2. Fine-tuning is V3+. |
| Concurrent generation | Both complete. Both write SP. User picks winner. | Content merging is V3+. |
| Error state | Show error state when Foundation unavailable | No cached/degraded mode until V2. |
| Offerings catalog | V0.1 real or synthetic. Best-effort. | Will be available for V1 demo. Features degrade gracefully without it. |
| Discovery questions | Full 200+ question set. Tree structure (business→industry→solution). | Available in `demo-data/config/discovery-questions.json`. |
| Demo data | 4 real accounts, anonymized. Public data gathered at runtime. | Matrix: public existing, public prospect, private existing, private prospect. |
| Notifications V1 | In-app badges and dashboard indicators only | Email/push is V2. |
| Foundation version | V3.0 only. No V4 features. | V4 targets land before KIRT V2. |

---

## User Personas

### Persona 1: Maya Chen — Enterprise Architect (Full SAF)

**Technical comfort:** High
**Goals:** Build cumulative account intelligence, manage strategic accounts through SAF phases, generate all deliverable types, track SAF progress per account.

**Core tasks:**
- Run Account Intelligence for target accounts — covers: Account Brief, growth scoring, white space analysis
- Prepare for executive meetings — covers: Engagement Prep with full discovery question set and stakeholder map
- Upload meeting notes post-engagement — covers: Meeting note upload, signal extraction, account enrichment
- Track SAF progress per account — covers: SAF progress overlay (optional for EA users)
- Configure sources and review admin settings — covers: SP site connections, output destination config

**Data lifecycle:** Connects SP sites via admin config (input), generates deliverables to SP output folders (output), reviews and refines generated content, tracks cumulative intelligence across account lifecycle.

**First-run experience:** Connects first SP source, uploads initial account data, generates first Account Brief. Sees completeness indicators showing what data sources are connected and what's missing.

### Persona 2: James Torres — Account Executive (On-Demand)

**Technical comfort:** Medium
**Goals:** Prep for meetings fast, get account context without digging through files, upload notes and forget about the follow-up mechanics.

**Core tasks:**
- "Prepare me for my meeting with Acme" — covers: Engagement Prep (talking points, stakeholder map, discovery questions)
- Upload meeting notes → get follow-up email draft — covers: Meeting Summary & Follow-Up
- "What do we know about this company?" — covers: Account Brief generation from whatever data exists
- Search for a specific document — covers: Hybrid search with dedup and freshness indicators
- "Generate a client deliverable for tomorrow's presentation" — covers: Client Deliverable from name + keywords

**Data lifecycle:** Uploads meeting notes (input), receives generated briefs and follow-ups (output). Doesn't configure sources or manage admin settings.

**First-run experience:** Searches for an account, sees whatever data exists with completeness indicators showing data coverage. Generates first brief immediately — no setup required.

### Persona 3: Sarah Kim — Sales Leadership (Consumer)

**Technical comfort:** Low-Medium
**Goals:** Pipeline visibility, team patterns, account health at a glance. Review prepared materials before client meetings.

**Core tasks:**
- Browse account briefs generated by EAs — covers: Search, Account Brief viewing
- Check cross-sell signals across accounts — covers: Cross-sell recommendations viewing
- Review engagement prep before a client meeting — covers: Engagement Prep viewing
- Search for specific account intelligence — covers: Hybrid search

**Data lifecycle:** Read-only — consumes intelligence generated by others. Does not upload, generate, or configure.

**First-run experience:** Browses accounts, reads existing briefs, searches for specific accounts. Sees the same completeness indicators but doesn't action them.

---

## Risk Summary

| Risk ID | Risk | Severity | Category | Status |
|---------|------|----------|----------|--------|
| RISK-01 | Offerings catalog quality | MEDIUM | Dependency | Mitigated: will be available (real or synthetic) for V1 |
| RISK-02 | Foundation availability for testing | HIGH | Dependency | Mitigated: mock Foundation API for unit tests; integration tests against live v3 |
| RISK-03 | Discovery question taxonomy | RESOLVED | Dependency | Resolved: structured taxonomy generated in `demo-data/config/discovery-questions.json` |
| RISK-04 | LLM generation quality below 80-90% threshold | HIGH | Technical | Mitigated: few-shot examples from prior edits; graceful degradation messaging |
| RISK-05 | SP permission enforcement gaps | HIGH | Security | Mitigated: pre-query enforcement in Foundation; integration test required |
| RISK-06 | MSA opt-out enforcement gap | HIGH | Compliance | Mitigated: `ai_processing_allowed` flag checked at every LLM pipeline entry point |
| RISK-07 | GDPR purge annotation cascade missed | HIGH | Compliance | Mitigated: explicit KIRT annotation cleanup in purge workflow; verified in Phase 5 |
| RISK-08 | Demo data extraction quality | MEDIUM | Dependency | Mitigated: 4-account matrix; clean-first approach |
| RISK-09 | Concurrent generation conflicts | MEDIUM | Technical | Mitigated: both complete, user picks winner; content merge is V3+ |
| RISK-10 | Weaviate traversal complexity | MEDIUM | Technical | Mitigated: V1 limited to 1-hop cross-references |
| RISK-11 | LLM rate limits during demo | MEDIUM | Operational | Mitigated: quota increase + generation result caching for demo window |
| RISK-12 | Admin config surface coverage | MEDIUM | Scope | Mitigated: all configurable parameters audited in Phase 5 |

See `risks-and-considerations.md` for full risk detail (RISK-01 through RISK-15).

---

## Execution Guide

### Complexity Rating

- **Level:** Complex
- **Recommended Path:** Full GSD
- **Rationale:** Multi-service architecture, 4+ external integrations, 3 user modes, compliance requirements (GDPR, MSA opt-out), and 20K+ line codebase require structured phase management.

**Complexity Signals:**

| Signal | Level | Detail |
|--------|-------|--------|
| Services/components | Complex | Frontend (React), Django backend, Foundation API client, Weaviate, Redis Streams, Iceberg, LLM abstraction layer |
| External integrations | Complex | Foundation (13 endpoints), AD/SSO (Graph API), SharePoint (read + write-back), LLM providers (3+) |
| Delivery phases | Complex | 5-phase build order; parallel frontend/backend workstreams |
| Team size | Moderate | Small team execution expected |
| Estimated codebase | Complex | 20K+ lines (Django backend + React frontend + LLM abstraction + admin config) |
| Technology novelty | Moderate | MCP-native Django is new pattern; Weaviate integration is well-documented |
| Data complexity | Complex | Multiple stores (Weaviate, Iceberg, Redis, SP, SF CSV), complex schema, permission enforcement |
| User types | Complex | 3 user modes (Full SAF, On-Demand, Consumer) + Admin |
| Deployment model | Moderate | Single-tenant V1; Docker on Contabo CVPS3, CI/CD via GitHub Actions |
| Real-time requirements | Moderate | Redis Streams events, job progress tracking, in-app notification badges |

### Build Order

Build these layers in sequence. Each layer depends on the one above it.

1. **Foundation Integration & Project Skeleton**
   - Django project setup (MCP-native, mirroring Foundation POC structure)
   - Foundation API client (all 13 endpoints, typed, tested)
   - AD/SSO auth via Graph API; JWT middleware
   - Health endpoint `/health/`, error tracking (Sentry), basic usage logging
   - Local development with mock Foundation API responses
   - Docker deployment setup (Dockerfile, docker-compose.yml)
   - CI/CD pipeline skeleton (GitHub Actions → Contabo CVPS3)
   - Celery + Redis task queue setup
   - SSE endpoint skeleton for real-time push
   - MSA opt-out flag (`ai_processing_allowed`) check in Foundation API client
   - Depends on: nothing (foundation layer)
   - Produces: authenticated, Foundation-connected Django app in Docker with mock API for local testing

2. **Core Search & Document Management**
   - Hybrid search (BM25 + semantic) via Foundation query API
   - Structured filter support (doc_type, date, client, author, status)
   - Deduplication (content hash, 1-result default, "N copies" badge)
   - Freshness indicator (>30 days flag), search result ranking (knowledge hierarchy)
   - Document upload → Foundation ingest → auto-classification
   - SF CSV import mechanism (upload CSV → parse → ingest into Foundation)
   - Job progress tracking
   - Depends on: Layer 1 (Foundation API client, auth, mock server)
   - Produces: working search UI, document upload, classification — users can find and add content

3. **Content Generation Pipeline**
   - LLM provider abstraction layer (multi-provider, admin-configurable)
   - Account Brief generation (all 9 sections, <60 second target)
   - Engagement Prep generation (prior work, stakeholder map, discovery questions, talking points)
   - Meeting Summary & Follow-Up (upload → recap + action items + follow-up email + intelligence update)
   - Stakeholder map (pre-populated from SF + prior meetings, user-editable)
   - Data completeness indicators on every generated output
   - Graceful degradation rules (partial data → generates + flags gaps)
   - SP write-back (deliverable → configurable output folder + annotation)
   - Human-in-the-loop editing (inline edit, section regeneration)
   - Prompt-level feedback learning (store edits as few-shot context)
   - Concurrent generation handling, content versioning badge
   - Public data research pipeline (SEC EDGAR, news, company profiles)
   - Prompt engineering allocation (prompt design, testing, iteration per generation type)
   - Error handling for LLM failures (timeout, rate limit, provider unavailable)
   - Depends on: Layer 2 (search, document access, classification)
   - Produces: 3 of 5 MVP generation types working (Account Brief, Engagement Prep, Meeting Summary)

4. **Deliverables, Intelligence & Scoring**
   - Client Deliverable generation (all templates use Pellera format)
   - Cross-sell recommendations (pattern-based, offerings catalog)
   - White space grid generation
   - SAF progress overlay (optional, EA users)
   - Growth scoring (5-dimension formula, admin-tunable, manual override)
   - Discovery question taxonomy integration
   - Depends on: Layer 3 (LLM abstraction, generation pipeline, SP write-back)
   - Produces: all 5 MVP generation types working, scoring and SAF features active

5. **Admin, Config, Deployment & Polish**
   - Admin configuration UI (SP sources, output paths, location-ranking, scoring weights, LLM provider, offerings catalog)
   - MSA opt-out enforcement audit (`ai_processing_allowed` at every LLM entry point)
   - Admin-triggered GDPR purge with annotation cleanup
   - In-app notification badges and dashboard indicators
   - Responsive frontend polish (mobile read flows)
   - Demo data ingestion (4-account matrix)
   - Integration testing against live Foundation v3
   - CI/CD pipeline completion (GitHub Actions → Contabo CVPS3)
   - Docker production deployment config
   - Playwright E2E test infrastructure setup
   - SF CSV import validation (synthetic CSVs for demo)
   - Depends on: Layer 4 (all features built, all generation types working)
   - Produces: production-ready, demo-reliable, compliance-audited application

> **Rules:**
> - Don't start a layer until the layer it depends on compiles/runs.
> - After completing each layer, verify it works before moving on.
> - If you need to reorder layers, do it — log the change in BUILD_LOG.md and why.

### Build Log

Maintain `.planning/BUILD_LOG.md` throughout the build. Update it as you complete each layer.

Format per layer:
```
## Layer N: <Name> — <COMPLETE/IN PROGRESS/BLOCKED>
- What was built: <brief summary>
- Deviations: <None, or what changed from plan and why>
- Checkpoint: <how you verified this layer works>
```

> **Rules:**
> - Update BUILD_LOG.md after completing every layer, not at the end.
> - If you change something from what HANDOFF specifies — different library,
>   different data model, reordered layers, dropped a feature — don't stop.
>   Make the call, log what you changed and why, keep building.
> - If you add something not in scope, move it to `TODO.md` unless it's
>   required for a success criterion to pass.

### Success Criteria (V1 MVP)

- Non-technical stakeholder watches demo and understands value without explanation
- Demo runs reliably 3 consecutive times
- Account Brief generates in <60 seconds with data completeness indicator
- Engagement Prep package includes filtered discovery questions mapped to account context
- Meeting notes upload → polished summary + draft follow-up email in <2 minutes
- Hybrid search returns deduplicated results with freshness indicators in <3 seconds
- All features work with partial data (thin-data account demo works)
- No Foundation-permissioned document surfaced to unauthorized user
- Admin settings tunable without code changes
- MSA opt-out content never hits LLM pipeline
- GDPR purge deletes KIRT annotations alongside source document

### Test Expectations

- **Unit tests:** Mock Foundation API responses. All generation functions, scoring formulas, classification logic, permission enforcement. Target: 80%+ coverage.
- **Integration tests:** Against live Foundation v3. Test actual search behavior, real ingestion pipeline, real Weaviate cross-reference traversal.
- **E2E tests:** 3 critical flows — Account Brief generation, Meeting Notes upload → Follow-up, Hybrid search. Run against demo data loaded in Foundation.
- **Security tests:** MSA opt-out enforcement, permission inheritance (no unauthorized document surfacing), GDPR purge cascade.
- Bugs in Foundation API behavior → file as Foundation repo issues, not KIRT workarounds.

### Scope Boundaries

**In scope for this build:** All Must-Have requirements R01–R62 (see Requirements Signals above).

**Explicitly out of scope (do not build, do not refactor toward):** All Deferred items D01–D27, all Non-Goals. If it's not in the Must-Have list, it is deferred. No exceptions without explicit user approval.

> **Rule:** If you discover something mid-build that "should" be added, capture it in `TODO.md` and move on. Ship what's scoped.

### Known Risks to Watch

| Risk | What to Do |
|------|-----------|
| Foundation API unavailable | Use mock Foundation server for local dev. Show error state in UI. Don't engineer cached mode. |
| LLM provider rate limits or timeout | Build retry with exponential backoff. Show "Still working..." progress. Fail gracefully with partial output. |
| SP write-back permissions fail | Integration test early (Phase 1). Require Graph API `Sites.ReadWrite.All` permission in Entra app. |
| MSA opt-out content reaches LLM | Audit every LLM entry point in Phase 5. `ai_processing_allowed` check is in Foundation API client from Layer 1. |
| Offerings catalog not ready | Features degrade gracefully without it. Cross-sell uses engagement patterns. White space shows "catalog not configured." |
| Demo data quality insufficient | Synthetic data is acceptable. 4-account matrix covers all scenarios. Don't block on real data. |
| Generation quality below expectations | Allocate prompt engineering time in Layer 3. Store edits as few-shot context. Iterate prompts per generation type. |

> Address mitigations during build, don't defer them.

### Definition of Done

The build is complete when:
1. All Success Criteria checkboxes are checked
2. All layers in BUILD_LOG.md show COMPLETE
3. All Must-Have requirements (R01–R62) implemented and tested
4. Demo runs 3 consecutive times without intervention
5. Unit tests pass (`pytest` + `npm test`)
6. Smoke test passes (Account Brief generation + Meeting Notes upload + Hybrid search end-to-end)
7. BUILD_LOG.md documents any deviations from this guide
8. Sentry error tracking active
9. No hardcoded LLM providers, SP paths, or scoring weights

When all are true, stop. Don't polish, don't refactor, don't add features. If something should be better, add it to `TODO.md`.

---

## Market Positioning

*Full competitive analysis in `research/market-research.md`. Synthesis of key positioning signals below.*

### Category
Enterprise account intelligence and sales enablement platform. Not a standalone CRM, not a meeting recorder, not a generic AI assistant — KIRT is a domain-specific intelligence layer for structured account management.

### Primary Differentiators

1. **Foundation-native:** Operates on a proprietary data platform that ingests and normalizes the organization's own SharePoint, SF, and document data — not a public data aggregator. Intelligence is built from internal context first.

2. **Graceful degradation with transparency:** Every feature works with partial data and tells the user exactly what's missing and why. Competitors generally fail silently or require complete data setup before value is delivered.

3. **SAF-aligned without being SAF-only:** The UI is task-oriented, not framework-oriented. EAs get SAF workflow support; AEs get on-demand intelligence. Same platform, different entry points.

4. **Write-back to SharePoint:** Generated deliverables return to the organization's document source of truth. Not a dashboard that produces output visible only in the tool — deliverables integrate into existing workflows.

5. **Multi-provider LLM from day one:** No vendor lock-in. Admin-configurable provider switching means the organization's approved providers (Gemini, Azure OpenAI, AWS Bedrock) are all supported without code changes.

### Competitive Context

**Revenue intelligence (Gong, Clari/Salesloft, People.ai):** CRM-signal-first — call recordings, emails, pipeline data. Gong is category leader ($300M+ ARR, 27% market share) but cannot read SP documents, cannot generate EBR decks, doesn't write back to SharePoint. People.ai added MCP integration (March 2026) but remains activity-signal-only. Strong complements, not replacements.

**Sales enablement (Seismic + Highspot, now merging):** Seismic and Highspot announced merger February 2026 — the combined entity will create 12–18 months of customer uncertainty. Both platforms are content management (organize + distribute), not intelligence generation. No SP ingestion. No account-specific synthesis. KIRT generates where they organize.

**Account intelligence data (ZoomInfo, 6sense, Demandbase):** Public data aggregators. Zero internal document analysis. Built for net-new outbound, not existing strategic accounts. KIRT complements them — public data enriches KIRT's output.

**KAM platforms (DemandFarm, Altify):** Closest in intent — whitespace grids, account planning, relationship mapping. But CRM-bounded (SFDC-native), no SP ingestion, closed architectures. DemandFarm's "Kampanion" AI agent is adding account research, but still limited to CRM + public signals.

**Microsoft Copilot for Sales:** Most dangerous adjacency. Does read/write SharePoint (via M365). But is a general assistant — no SAF methodology, no Foundation data layer, no multi-source synthesis. The SharePoint Agents framework (GA March 2026) is worth quarterly monitoring: if Microsoft adds Salesforce connectors + account brief templates, the structural overlap grows. Current position: general-purpose vs. KIRT's purpose-built. **Monitor quarterly.**

**Market timing:** Gartner published its first-ever Magic Quadrant for Revenue Action Orchestration (December 2025). Budget and vocabulary now exist for this investment class. AI sales market: $8.8B (2025) → $63.5B (2032), 32%+ CAGR.

### Three Structural Moats (from ECC market research)

1. **Internal document intelligence** — every competitor is blind to SP proposals, delivery assessments, prior EBRs. KIRT ingests these as first-class data.
2. **Deliverable generation, not summaries** — the market produces summaries and CRM updates. KIRT produces finished, consumption-ready artifacts.
3. **SharePoint as source and destination** — reads internal SP archives, writes structured intelligence back. No competitor does both.

### Positioning Statement
> KIRT is the account intelligence layer that knows what your team knows — not just what your CRM records show. It reads your internal documents, synthesizes your account history, and produces deliverables your team actually uses. It runs on your infrastructure, writes to your SharePoint, and encodes your methodology. In 10 minutes, not 4 hours.
