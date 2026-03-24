# KIRT — Roadmap Analysis

> Analysis of all documentation that produced the spec seed. Documents gaps found, conflicts resolved, Foundation v3/v4 dependencies, and outlier ideas absorbed. This is the "show your work" companion to `kirt-spec-seed.md`.

---

## 1. Gaps Identified in Original Spec Seed

The original spec seed was architecture-first, focused on Foundation's data model and KIRT's relationship to it. These gaps were found and reconciled into the updated spec seed.

### 1.1 SAF Framework Phases (from MVP Gap Analysis)

The original spec seed had no concept of the SAF's 6-phase structure — the entire business logic model. Without this, the build would be architecturally correct but functionally aimless.

| Phase | What It Does | Original Coverage |
|-------|-------------|-------------------|
| 1. Account Intelligence | Growth scoring, white space grid, public data | Not mentioned |
| 2. Prepared Engagement | Meeting brief, discovery questions, stakeholder map | Partial — doc generation covered briefs but not question taxonomy or stakeholder mapping |
| 3. Discovery & Scoring | Signal capture from transcripts, opportunity scoring, maturity gate | Not mentioned |
| 4. Client Deliverables | Reframe deck, assessment kickoff, transformation roadmap, EBR/QBR | Partial — doc generation was generic, not mapped to specific output types |
| 5. Enablement & Progression | Insight drops, win/loss patterns, advancement triggers | Not mentioned |
| 6. Continuous Threads | Stakeholder tracking, outcome register, white space grid, governance | Not mentioned |

**Resolution:** Full SAF mapping added to spec seed with manual effort baselines and what KIRT replaces per phase.

### 1.2 Scoring Models (from MVP Gap Analysis)

Three scoring models were referenced across docs but absent from spec seed:
- **Growth scoring** — account-level, drives tier assignment (Phase 1)
- **Opportunity scoring** — signal-based from transcripts (Phase 3)
- **Offering maturity scoring** — gates deliverable generation (Phase 3)

**Resolution:** Scoring models section added to spec seed with inputs (to define), outputs, and version targeting.

### 1.3 Offerings Catalog (from MVP Gap Analysis)

Called out as the **single biggest blocker**. Foundational to white space grid, maturity scoring, cross-sell, and discovery question routing. Does not exist yet.

**Resolution:** Dedicated section in spec seed. Flagged as content dependency, not a KIRT feature. Added to solutioning session for team decision.

### 1.4 Discovery Question Taxonomy (from MVP Gap Analysis + Data Sources)

200+ questions across 20+ variants. Three modules: business, industry, solution. Mapped to offerings. "Next-best-question" AI prompting concept. Source material may be informal notes rather than a structured taxonomy — formalization needed during build.

**Resolution:** Integrated into Engagement Prep output definition in spec seed.

### 1.5 Data Normalization / Entity Resolution (from POC Data Architecture)

Customer names in 3+ variants across 50+ systems. 36 merged companies, two SF instances. Critical blocker for cross-account intelligence. Entity resolution prototype is a pre-build workstream.

**Resolution:** Acknowledged as a Foundation-layer concern. KIRT spec assumes Foundation handles normalization before data reaches KIRT. Entity resolution quality affects KIRT output quality but is not a KIRT build task.

### 1.6 Document Classification & Taxonomy (from POC Data Architecture)

Original spec treated all docs as generic "canonical records." POC doc defined 7 classification types with distinct processing pipelines.

**Resolution:** Document classification table added to spec seed under Document Upload & Ingestion flow.

### 1.7 User Personas (from Pre-Build to Demo-Ready)

Original spec described a generic "user." Roadmap references 6 personas (EA, AE, Sales Leadership, Practice Lead, others). Role-based views are a demo "should-have."

**Resolution:** Persona-based views deferred to V2. Added to solutioning session for design.

### 1.8 Confirmed Tech Stack (from Overview + Architecture)

Original spec didn't mention tech stack. Confirmed across multiple docs: React/Vite/Tailwind/Shadcn + Django + Weaviate + Iceberg + AD/SSO + Azure.

**Resolution:** Tech stack table added to spec seed.

### 1.9 Day-1 Outputs (from Pre-Build to Demo-Ready)

Original spec's "Document Generation Lifecycle" was too abstract. Roadmap defined 4 specific outputs: Account Brief, Engagement Prep, Client Deliverable, Cross-Sell.

**Resolution:** Day-1 Outputs section added to spec seed with detailed section structures (Account Brief template influenced by Project Nighthawk's Client Intelligence Dossier).

### 1.10 Effort Baselines & Value Metrics (from Overview + Roadmap)

Manual effort per phase: 2-4 hrs (Phase 1), 1-2 hrs (Phase 2), 30-60 min (Phase 3), 2-8 hrs (Phase 4). Target: 80%+ reduction in prep time, 50-70% quality improvement.

**Resolution:** Added to SAF Framework Mapping table in spec seed.

### 1.11 Salesforce Relationship (from Salesforce Positioning)

The "why not just Salesforce?" question was unaddressed. Positioning doc makes clear: SF = system of record for structured data, KIRT = unstructured analysis + semantic search + AI generation + pattern intelligence.

**Resolution:** Salesforce Relationship section added to spec seed.

---

## 2. Conflicts Resolved

### 2.1 Source-of-Truth Model
**Conflict:** Spec seed said Foundation owns everything. POC doc said KIRT owns intelligence. Overview said KIRT has its own ingestion.
**Resolution:** Three-tier model adopted. SP = documents, Foundation = raw data, KIRT = intelligence. User uploads go through Foundation's ingest pipeline (path of least resistance). KIRT owns classification, scoring, and enrichment metadata.

### 2.2 Graph / Relationship Layer
**Conflict:** Spec assumed Weaviate handles all relationships. POC doc questioned if Neo4j was needed.
**Resolution:** Weaviate is the v1 solution until it doesn't work. If traversal queries prove insufficient at scale, evaluate alternatives then. Don't design for Neo4j now, don't preclude it.

### 2.3 Agentic RAG Scope
**Conflict:** Spec treated agentic RAG as core. Build docs recommended deferring.
**Resolution:** Agentic RAG is V3. MVP uses pre-computed enrichments and standard hybrid search.

### 2.4 SP Output Folder Convention
**Conflict:** Everyone agreed deliverables go to SP. No folder convention decided.
**Resolution:** Configurable per deployment via admin settings. Default convention uses `[Client]-[DocType]-[Date].docx` naming, shallow folder structures.

### 2.5 Infrastructure: Contabo vs. Azure
**Conflict:** Build roadmap assumed Azure. Foundation runs on Contabo.
**Resolution:** Build KIRT against Foundation's REST API (infra-agnostic). Demo can run on Contabo. Azure migration doesn't gate MVP.

---

## 3. Foundation v3 vs. v4 Dependency Map

### What Foundation v3.0 Provides (shipped)

| API / Capability | What KIRT Uses It For |
|-----------------|----------------------|
| `POST /v1/ingest/` | Async file ingestion via Dagster pipeline |
| `POST /v1/query/` | Hybrid semantic + keyword search with freshness scoring |
| `GET /v1/retrieve/<id>/` | Full document metadata and content |
| `GET /v1/list/` | Paginated listing with filters |
| `DELETE /v1/delete/<id>/` | Soft delete or GDPR purge |
| `POST /v1/annotations/` | Write classifications, tags, summaries |
| `GET /v1/annotations/` | Read annotations |
| `GET /v1/jobs/<id>/` | Dagster job tracking |
| `GET /v1/connections/` | Connector health |
| `GET /v1/health/quality/` | Data quality metrics, drift detection |
| Redis Streams events | `document.ingested` — near-real-time reaction |
| Weaviate HNSW + hybrid | Vector + keyword search |
| Iceberg versioned storage | Document version chains |
| Content-addressable document_id | SHA-256 dedup |
| `sp_permissions[]` enforcement | Permission gate on every query |

### What Foundation v4.0 Would Add (planned, not built)

| v4 Capability | KIRT Feature It Would Enable | Blocked Without It? |
|--------------|-----------------------------|--------------------|
| OpenMetadata governance | Automated data catalog, lineage visualization | **No** — manual catalog fine for MVP |
| Automated reconciliation | Auto-detect stale copies, re-canonicalization | **No for MVP, yes for V2+** |
| Spark query engine | Complex aggregations for scoring at scale | **Potentially blocks V2 scoring** at 200+ accounts |
| Advanced lineage | Full provenance chain | **No for MVP, yes for V3** |

### Bottom Line

**V1 MVP is fully buildable on Foundation v3.0.** No blockers. V2 gets harder without v4 (scoring at scale, automated reconciliation). V3 benefits from v4's lineage. Build V1 now, plan v4 in parallel with V2.

---

## 4. Outlier Analysis

### 4.1 Pre-Build to Demo-Ready

**Not absorbed (business, not development):**
- Pre-build gates (executive alignment, IT governance, legal, template ground truth) — preconditions for the build, not spec content
- Owner assignments and week-by-week timeline — project management
- Risk register — execution concern

**Absorbed:**
- Demo definition (must-haves/should-haves) → V1 exit criteria in spec seed
- Parallel workstreams that affect KIRT → admin-configurable settings (directory structures, SP paths, scoring weights tunable per deployment)

### 4.2 Project Nighthawk — Ideas Absorbed

Nighthawk is personal/OneDrive/Copilot-powered. KIRT is organizational/custom-built. Nighthawk is the "duct tape" version; KIRT is the enterprise product.

| Nighthawk Concept | Where It Landed in Spec Seed | Version |
|-------------------|------------------------------|---------|
| Client Intelligence Dossier section structure | Account Brief template | V1 |
| Knowledge hierarchy for retrieval | Search result ranking | V1 |
| Flat-over-deep folder principle | SP output conventions | V1 |
| File naming convention `[Client]-[DocType]-[Date].docx` | SP output conventions | V1 |
| Prose over tables for generated content | SP output conventions | V1 |
| Stale data flagging (>30 days) | V2 scope | V2 |
| Automated SF export → narrative summaries | V2 scope | V2 |
| SEC EDGAR public company monitoring | V2 scope | V2 |
| Structured win/loss capture | V3 scope | V3 |
| Cross-client pattern analysis | V3 scope | V3 |
| Multi-agent architecture per SAF phase | V4+ scope | V4+ |

**Key Nighthawk warnings absorbed:** Document quality drives retrieval quality (KIRT enforces structure). Favor prose over tables in generated output. Freshness/staleness indicators needed from Day 1.

### 4.3 Salesforce BI/AI Architecture Brief

Validates KIRT's architecture: lakehouse-centric pattern aligns with enterprise recommendation. KIRT consumes SF through Foundation (no point-to-point sprawl). If KIRT ever writes back to SF (v4+), only high-value insights — scores, recommendations, summaries.
