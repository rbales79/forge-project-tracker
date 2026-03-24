# KIRT — Intelligence Layer

> The user-facing product where all business intelligence happens. Classification, scoring, template routing, deliverable generation, and recommendation engine.
>
> Authoritative spec: `kirt-spec-seed.md` (parent directory)

## Status

**Greenfield build — no application code exists yet.** Foundation reference POC has 9 layers, 179 tests passing, MCP-native architecture. KIRT will be built on top of Foundation v3's data layer.

## Purpose

KIRT is the intelligence layer that transforms Foundation's raw, normalized data into actionable account intelligence, generated deliverables, and strategic recommendations. It automates the Strategic Account Framework's 6 phases.

Users interact with KIRT. They never touch Foundation directly. KIRT owns all business logic — Foundation owns all data.

## Stack

| Component | Technology | Status |
|-----------|-----------|--------|
| Frontend | React + Vite + Tailwind + Shadcn | Confirmed |
| Backend | Django, MCP-native | Confirmed |
| Vector DB | Weaviate (via Foundation) | Confirmed — v1 solution until it proves insufficient |
| Auth | AD/SSO via Graph API (inherited from Foundation) | Confirmed |
| Data Access | Foundation REST API (13 endpoints) | Confirmed |

## Core Capabilities by Version

### V1 — MVP (SAF Phases 1-2)

| Capability | Detail |
|-----------|--------|
| Hybrid search | Keyword (BM25) + semantic (vector) + relationship traversal, blended in single query. Results deduplicated, ranked by knowledge hierarchy, freshness-flagged. |
| Account Brief generation | 1-2 page summary in <60 seconds. Sections: overview, contacts, priorities, history, opportunities, competitive, strategic notes, white space, action items. |
| Engagement Prep generation | Pre-meeting package: prior work, pain points, stakeholder map, talking points, discovery questions (from 200+ taxonomy), next-best-question suggestions. |
| Client Deliverable generation | Minimal input (company name + keywords) → client-ready documentation with account history. |
| Cross-sell recommendations | Pattern-based "accounts like this also bought..." using engagement patterns + unstructured signals. |
| Document upload & classification | Users upload through KIRT → Foundation ingest pipeline. Auto-classification: meeting notes, transcripts, BSE, SOW, decks, templates, KIRT-generated. |
| Exact dedup | Content-hash detection. 1-result UX with "N copies found" badge. User selects canonical. |
| Human-in-the-loop editing | Inline edit or regenerate sections with new instructions. |
| Deliverable write-back | Generated docs write to configurable SP output folders. `[Client]-[DocType]-[Date].docx` naming. |
| Admin settings | SP paths, directory structures, scoring weights, offerings catalog — tunable per deployment. |

### V2 — Discovery & Scoring (SAF Phase 3)

| Capability | Detail |
|-----------|--------|
| Transcript analysis | Speaker diarization, signal extraction, sentiment analysis |
| Opportunity scoring | Signal-based scoring from transcripts + engagement data, human-confirmed |
| Offering maturity gate | Pass/fail before Phase 4 deliverable generation |
| Near-duplicate detection | Fuzzy matching (SimHash/MinHash or embedding similarity) |
| Version lineage | Distinguish accidental duplication from intentional versioning |
| Cross-format dedup | docx ↔ pdf parity |
| Persona-based views | EA vs. AE vs. Sales Leadership see different dashboards |
| Public company monitoring | SEC EDGAR integration for account intelligence |

### V3 — Client Deliverables (SAF Phase 4)

| Capability | Detail |
|-----------|--------|
| Reframe deck generation | Pulls scored opportunities, past deliverables, offering data |
| Assessment kickoff | Template-driven generation |
| Transformation roadmap | Requires full scored + gated opportunity pipeline |
| EBR/QBR packages | Engagement review deliverables |
| Agentic RAG | Multi-hop retrieval for complex generation tasks |
| Template management | Versioning, selection logic, pull-populate-write cycle |
| Full provenance | Every generated insight shows source data with attribution |

### V4+ — Enablement & Continuous Threads (SAF Phases 5-6)

| Capability | Detail |
|-----------|--------|
| Insight drops | Proactive intelligence delivery based on triggers |
| Win/loss patterns | ML-driven analysis across closed engagements |
| Advancement triggers | Defined events signaling opportunity progression |
| Stakeholder tracking | Persistent relationship maps with change detection |
| Outcome register | Win wires, past performance, institutional memory |
| Multi-agent architecture | Specialized agents per SAF phase |

## Document Classification

Classification happens at ingest, LLM-based with labeled training examples. Drives processing pipeline routing.

| Type | Processing Route |
|------|-----------------|
| Meeting notes | NLP extraction → signals, entities, action items → Phase 3 signal capture |
| Transcript | Speaker diarization → signal extraction → sentiment analysis |
| BSE / account profile | Entity extraction → account linking → white space input |
| SOW / contract | Offerings linkage → engagement tracking → maturity input |
| Deck / presentation | Subtype classification (pitch vs. deliverable vs. internal) → theme extraction |
| Template | Mark as template → feed generation pipeline → never modify original |
| KIRT-generated | Provenance tracking → version control → link to source inputs |

V1: Manual override for misclassification. V2+: Feedback loop to retrain.

## Scoring Models

Three models drive SAF automation. All need v0.1 definitions before the features that consume them.

| Model | Phase | V1 Approach |
|-------|-------|-------------|
| Growth scoring | 1 | Simplified weighted formula (revenue, strategic value, relationship depth, engagement frequency, white space size). Tunable via admin settings. Output: Tier 1/2/3 assignment. |
| Opportunity scoring | 3 | Pre-computed at ingest time, stored in Foundation annotations. Inputs: meeting signals, engagement recency, competitive mentions, stakeholder sentiment. Human confirmation step. |
| Offering maturity | 3 | Manual rating in offerings catalog, surfaced in KIRT. Pass/fail gate before Phase 4 generation. |

## User-Generated Content Flow

```
User uploads file in KIRT
  → KIRT sends to Foundation via POST /v1/ingest/
  → Foundation: parse → store → chunk → embed → index
  → Foundation emits document.ingested event (Redis Streams)
  → KIRT receives event, runs auto-classification
  → KIRT writes classification + enrichments via POST /v1/annotations/
  → Document appears in KIRT search results with enrichment metadata
```

KIRT does not maintain a separate document store. All documents live in Foundation. KIRT owns the intelligence layer: classifications, scores, and enrichments written as annotations.

## Document Generation Flow

```
User selects account / opportunity in KIRT
  → KIRT pulls template from SP (via Foundation's template canonical)
  → KIRT triggers research run (SF data + Foundation knowledge + public data)
  → KIRT generates document via LLM
  → KIRT writes deliverable to SP output folder
  → Foundation records SP URL in derived_docs[]
  → User sees finished document + provenance summary
  → User can edit inline or regenerate sections
```

**SP output conventions:**
- Configurable output folders per account/engagement
- File naming: `[Client]-[DocType]-[Date].docx`
- Shallow folder structures (flat over deep)
- Prose + structured sections over raw tables

## Search Architecture

**Single blended query** — not separate search tabs:

| Modality | Engine | Purpose |
|----------|--------|---------|
| Keyword/phrase | Weaviate BM25 | Known terms, exact matches |
| Semantic | Weaviate vector (HNSW) | Conceptual similarity, natural language |
| Relationship traversal | Weaviate cross-references | Entity connections (account → contacts → engagements) |
| Structured filters | Weaviate + Foundation metadata | doc_type, date range, client, author, status |

**Result ranking (knowledge hierarchy):**
1. Account intelligence dossier / brief
2. Recent meeting notes and transcripts (last 90 days)
3. Salesforce data (account, opportunity, contact)
4. Public research (SEC filings, earnings, news)
5. Playbooks, battle cards, templates

**Dedup in results:** Group by content hash. Show canonical only. "Found in N locations" badge. Prefer editable format (.docx over .pdf).

**Freshness:** Flag anything >30 days as potentially stale.

## Permissions Model

KIRT delegates entirely to Foundation's permission gate:
- Users authenticate via M365/Entra identity
- JWT claims passed to Foundation on every request
- Foundation enforces: user's M365 group memberships must intersect with `sp_permissions[]` on the canonical record
- If a user can't see it in SharePoint, they can't see it in KIRT — no exceptions
- MSA opt-out clients: data never ingested into Foundation; KIRT queries via Graph API passthrough (read-only)
