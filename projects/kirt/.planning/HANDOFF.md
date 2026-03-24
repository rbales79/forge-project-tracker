# PreFlight Handoff — KIRT

> **Intake Synthesis Mode.** All decisions extracted from `kirt-spec-seed.md` and `demo-data-requirements.md` (2026-03-24). Decisions marked `[FROM SOURCE]` are locked — treat as if confirmed in Q&A. No decisions were deferred or overridden during synthesis.

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
- **SharePoint** = document source-of-truth (files live here, never modified by Foundation/KIRT)
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
- SP files are **never** modified by Foundation or KIRT. KIRT writes new files to SP (deliverables); it does not edit source files unless the user explicitly initiates a write-back to a specific canonical source_ref.
- KIRT does not maintain its own document store. All document storage routes through Foundation via `POST /v1/ingest/`.
- Permission inheritance is strict: KIRT never surfaces a document the user couldn't access in its source system. Enforced pre-query in Foundation, not post-fetch in KIRT.

### Scope Constraints `[FROM SOURCE]`
- **Single-tenant V1.** Internal Pellera deployment only. Foundation's `tenant_id` scoping supports future multi-tenancy without rewrite.
- **Single AD group, full access V1.** Bronze/silver/gold tier enforcement is V2+.
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
- Offerings catalog v0.1 does not exist yet — highest content dependency, must be created from PPT decks and practice collateral before demo.
- Discovery question taxonomy: 200+ questions across 20+ variants. May exist as informal notes — structured taxonomy file must be created during build.

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
- **R28** Upload meeting notes → generate: polished recap, structured action items (with owners/dates), draft follow-up email, account intelligence update
- **R29** Works from notes alone, enriched by existing account context
- **R30** Lowest-friction entry point: single upload → something sendable

#### Client Deliverable Generation
- **R31** Input: company name + keywords → client-ready documentation with account history
- **R32** Template pulled from SP (via Foundation's template canonical)
- **R33** Research run → LLM generation → SP write-back → provenance summary shown to user

#### Cross-Sell Recommendations
- **R34** Pattern-based suggestions from engagement history: "accounts like this also bought..."
- **R35** Triggered after data ingestion or meeting note upload
- **R36** Requires offerings catalog as reference frame (less precise without it, still generates)

#### Document Generation Infrastructure
- **R37** Generation progress: real-time indicators via Foundation job API; "Still working — pulling data from N sources" if >60 seconds
- **R38** Human-in-the-loop editing: inline edit, section regeneration
- **R39** Prompt-level feedback learning: store edited outputs, use as few-shot context for future generation for same account/type
- **R40** Concurrent generation handling: both complete, both write to SP, KIRT surfaces "2 versions generated" indicator, user picks winner
- **R41** Content versioning badge: "Generated v3, last updated 2026-03-20"
- **R42** SP write-back conventions: configurable output folders, `[Client]-[DocType]-[Date].docx` naming
- **R43** Write-back is explicit and user-initiated with confirmation step

#### SAF Features
- **R44** SAF progress tracking as optional overlay for EA users (not default view)
- **R45** Growth scoring: 5 dimensions (revenue, strategic value, relationship depth, engagement frequency, white space size), equal weights (20% each) in V1, tunable via admin
- **R46** Manual tier override for growth scoring

#### Duplicate/Variant Management
- **R47** Canonical document selection via admin-configurable location-ranking
- **R48** User can set a source_ref as canonical (writes `is_canonical` flag to Foundation)
- **R49** User can mark non-canonical copies as archived (Foundation `status: archived`, SP file untouched)

#### Admin Configuration
- **R50** Admin-configurable settings (without code changes): SP input sources, SP output folders, crawl schedules, location-ranking, scoring weights, LLM provider/model, offerings catalog
- **R51** Admin-triggered GDPR purge with annotation cleanup
- **R52** Output vs. input separation: source documents and KIRT-generated deliverables go to separate SP locations

#### Observability
- **R53** Health endpoint `/health/`: KIRT app health + Foundation connectivity
- **R54** Error tracking: Django error logging + Sentry (non-negotiable)
- **R55** Basic usage logging: who searched, who generated, when

#### Responsive UI
- **R56** Mobile read access: search, view Account Brief, view Engagement Prep (responsive via Tailwind)
- **R57** Task-oriented UI: "What do you need?" — not phase-oriented
- **R58** Generation and admin: desktop-only in V1

#### Error Handling
- **R59** Error state shown when Foundation is unavailable
- **R60** Graceful degradation on every feature — works with partial data, communicates what's missing

### Deferred (V2+)

- **D01** Near-duplicate detection (fuzzy matching) — V2
- **D02** Cross-format dedup (docx ↔ pdf) — V2
- **D03** Bronze/silver/gold access tiers (AD-group-to-role mapping) — V2
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
- KIRT does not edit source files in SharePoint. It writes new deliverable files only.
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
| LLM strategy | Multi-provider abstraction. No provider-specific code. | Gemini, Azure OpenAI, AWS Bedrock + OpenAI-compatible. Admin-configurable. |
| LLM approved providers | Gemini, Azure OpenAI, AWS Bedrock | Approved by Pellera. Any OpenAI-compatible endpoint also supported. |
| Form factor | Web app, responsive | Teams tab is future. No native app. Generation/admin: desktop-only V1. |
| Tenancy | Single-tenant V1 | Internal Pellera deployment only. Foundation `tenant_id` ready for future. |
| Access control V1 | Single AD group, full access | Bronze/silver/gold is V2+. |
| Document ownership | Foundation owns storage | KIRT routes uploads via `POST /v1/ingest/`. No KIRT-owned document store. |
| SP write policy | Write-back is explicit, user-initiated | No automatic SP edits. Confirmation required. |
| Permission model | SP permission inheritance. Strict. | Foundation enforces at pre-query time. KIRT never post-filters. |
| Duplicate default | 1 result per document, dedup invisible | "N copies" badge accessible. V1: content hash exact dedup. V2: fuzzy. |
| Canonical selection | Admin-configurable location-ranking | Not a hardcoded SP hierarchy. User can override. |
| Scoring V1 | Equal weights (20% each across 5 dimensions). Manual override. | V2+: ML-informed. V1 precision is less critical because of manual override. |
| MSA opt-out | `ai_processing_allowed` flag. Keyword search only. | No LLM processing on opted-out content. No exceptions. |
| GDPR purge | KIRT owns purge logic. Must explicitly delete own annotations. | Foundation cascade doesn't cover annotations. KIRT is responsible. |
| Feedback learning V1 | Prompt-level. Store edited outputs as few-shot context. | RAG loop is V2. Fine-tuning is V3+. |
| Concurrent generation | Both complete. Both write SP. User picks winner. | Content merging is V3+. |
| Error state | Show error state when Foundation unavailable | No cached/degraded mode until V2. |
| Offerings catalog | V0.1 extracted from PPT/collateral. Best-effort. | Does not exist yet — highest content dependency. Must be created pre-demo. |
| Discovery questions | Full 200+ question set. Tree structure (business→industry→solution). | May exist as notes — formalize during build. Required for Engagement Prep. |
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
| RISK-01 | Offerings catalog doesn't exist | CRITICAL | Dependency | Mitigated: v0.1 extraction from PPT/collateral before build |
| RISK-02 | Foundation availability for testing | HIGH | Dependency | Mitigated: mock Foundation API for unit tests; integration tests against live v3 |
| RISK-03 | Discovery question taxonomy as informal notes | HIGH | Dependency | Mitigated: formalize taxonomy during Phase 2; unblocks Engagement Prep |
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

## Complexity Assessment

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
| Deployment model | Moderate | Single-tenant V1; Azure target well-defined |
| Real-time requirements | Moderate | Redis Streams events, job progress tracking, in-app notification badges |

**Overall: Complex. Recommended path: Full GSD**

Rationale: Multi-service architecture, 4+ external integrations, 3 user modes with different permission levels, 20K+ line codebase, parallel frontend/backend workstreams, and compliance requirements (GDPR, MSA opt-out) all require structured phase management.

---

## Execution Guide

> For lightweight path consumers. GSD ignores this section and derives its own phase structure.

### Build Order

**Layer 1 — Foundation Integration & Project Skeleton**
- Django project setup (MCP-native, mirroring Foundation POC structure)
- Foundation API client (all 13 endpoints, typed, tested)
- AD/SSO auth via Graph API; JWT middleware
- Health endpoint `/health/`
- Error tracking (Sentry)
- Basic usage logging
- Local development with mock Foundation API responses

**Layer 2 — Core Search & Document Management**
- Hybrid search (BM25 + semantic) via Foundation query API
- Structured filter support (doc_type, date, client, author, status)
- Deduplication (content hash, 1-result default, "N copies" badge)
- Freshness indicator (>30 days flag)
- Search result ranking (knowledge hierarchy)
- Document upload → Foundation ingest → auto-classification
- Job progress tracking

**Layer 3 — Content Generation Pipeline**
- LLM provider abstraction layer (multi-provider, admin-configurable)
- Account Brief generation (all 9 sections, <60 second target)
- Data completeness indicators on every generated output
- Graceful degradation rules (partial data → generates + flags gaps)
- SP write-back (deliverable → configurable output folder + annotation)
- Human-in-the-loop editing (inline edit, section regeneration)
- Prompt-level feedback learning (store edits as few-shot context)
- Concurrent generation handling
- Content versioning badge

**Layer 4 — Engagement Prep, Meeting Notes & Cross-Sell**
- Discovery question taxonomy (200+ questions, tree structure)
- Engagement Prep generation (all sections)
- Stakeholder map (pre-populated, user-editable)
- Meeting Summary & Follow-Up (upload → recap + action items + follow-up email)
- Cross-sell recommendations (pattern-based, offerings catalog)
- SAF progress overlay (optional, EA users)
- Growth scoring (5-dimension formula, admin-tunable, manual override)
- White space grid

**Layer 5 — Admin, Config & Polish**
- Admin configuration UI (SP sources, output paths, location-ranking, scoring weights, LLM provider, offerings catalog)
- MSA opt-out enforcement (audit `ai_processing_allowed` at every LLM entry point)
- Admin-triggered GDPR purge with annotation cleanup
- In-app notification badges and dashboard indicators
- Responsive frontend polish (mobile read flows)
- Demo data ingestion (4-account matrix: public existing, public prospect, private existing, private prospect)
- Integration testing against live Foundation v3

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

### Scope Enforcement

If a task is not in the V1 Requirements Signals Must-Have list, it is deferred. The Deferred list is not v2 scope — it's explicitly not V1 scope. No exceptions without explicit user approval.

### Definition of Done

- All Must-Have requirements (R01–R60) implemented and tested
- Demo runs 3 consecutive times without intervention
- 4 demo accounts ingested into Foundation with realistic data coverage
- Admin settings documented in a config reference
- Sentry error tracking active
- No hardcoded LLM providers, SP paths, or scoring weights

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
