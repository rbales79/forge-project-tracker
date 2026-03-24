# KIRT — Project Knowledge

Accumulated understanding of the KIRT codebase. Updated as patterns, gotchas, and architectural decisions emerge during development. Seeded from HANDOFF.md.

---

## Architecture

### Two-Product Model
KIRT is the intelligence layer. Foundation is the data layer. They communicate via Foundation's 13 REST API endpoints and Redis Streams events. KIRT never accesses Weaviate, Iceberg, or PostgreSQL directly.

### Source-of-Truth Model
- **SharePoint** = document source-of-truth (files live here)
- **Foundation** = raw data source-of-truth (ingested, normalized, searchable)
- **KIRT** = intelligence source-of-truth (enrichments, classifications, scores, generated insights)

### Data Flow Direction
```
Source SP sites/SF CSV/uploads → Foundation ingests → KIRT enriches/generates
  → Deliverables write to designated SP output folders (never back to source)
  → Source docs marked stale when superseded (user-initiated archival)
```

Input sources and output targets are always separate SP locations. This separation IS the normalization strategy.

### Three User Modes
All features are independently accessible. SAF is a capability model, not a required workflow.
- **Full SAF** — EAs following the framework sequentially
- **On-Demand** — AEs/specialists who need answers now, no framework
- **Consumer** — leadership/delivery who read, don't generate

### Five Day-1 Outputs
1. Account Brief (<60s, 9 sections, completeness indicators)
2. Engagement Prep (stakeholder map, discovery questions, talking points)
3. Meeting Summary & Follow-Up (notes → recap + action items + email draft)
4. Client Deliverable (name + keywords → client-ready doc)
5. Cross-sell Recommendations (pattern-based from engagement history)

## Patterns & Conventions

### Foundation API Client
- All Foundation calls go through a centralized API client class
- JWT claims attached to every request (AD/SSO identity)
- Error handling: Foundation unavailable → error state in UI, no silent failures
- Job tracking for async operations via `GET /v1/jobs/<id>/`
- Redis Streams subscription for `document.ingested` events

### Document Classification
- 7 types: meeting notes, transcript, BSE, SOW, deck, template, KIRT-generated
- LLM-based classification at ingest time
- Manual override available for misclassification (V1)
- Classification stored as Foundation annotations via `POST /v1/annotations/`

### Generation Pipeline
- Template pulled from SP → research run (SF + Foundation + public data) → LLM generation → SP write-back
- Content versioning: generation metadata (timestamp, input hash, count) stored as Foundation annotations
- SP handles actual document versions
- Concurrent generation: both complete, both write SP, user picks winner
- Feedback learning: store edits alongside originals, use as few-shot context

### Search
- Single blended query (BM25 + semantic + relationship traversal + filters)
- Result ranking: account dossier > recent meetings > SF data > public research > playbooks
- Dedup: content hash, 1 result per logical document, "N copies" badge
- Canonical selection: admin-configurable location-ranking
- Freshness: flag >30 days as potentially stale

### Admin Configuration
- All tunable settings are deployment-configurable without code changes
- SP input sources, SP output folders, crawl schedules, location-ranking
- Scoring weights, LLM provider/model, offerings catalog
- MSA opt-out flag management, GDPR purge triggers

## Gotchas

- **Foundation purge does NOT cascade to annotations.** When a source document is purged, KIRT must explicitly delete its own annotations. This is easy to forget and will cause orphaned data.
- **MSA opt-out flag must be checked before ANY LLM pipeline.** Keyword search is fine, but no summarization, classification, or generation context for opted-out content.
- **Offerings catalog doesn't exist yet.** Cross-sell, white space, and scoring degrade gracefully without it, but engagement prep question mapping won't work without the taxonomy-to-offering linkage.
- **Discovery questions may be informal notes.** The 200+ question set may not be in a structured file. May need formalization during build.
- **SP write-back requires explicit user confirmation.** Never silently write to SP.
- **No seed data in Foundation.** Demo data must be ingested before integration testing.
- **Weaviate is V1 only.** If relationship traversal performance is insufficient, Neo4j evaluation is the path — but don't preemptively add it.

## Dependencies

| Dependency | Status | Impact |
|-----------|--------|--------|
| Foundation v3.0 (13 endpoints) | Live on Contabo | Blocker — no Foundation, no KIRT |
| AD/SSO test environment | TBD | Blocks auth layer |
| SP test site (read + write) | TBD | Blocks upload, write-back, source management |
| Offerings catalog v0.1 | WIP | Degrades cross-sell, white space, scoring |
| Discovery question taxonomy | Needs formalization | AI-generates from context without it |
| Demo data (4 accounts) | WIP | Blocks integration testing and demo |
| Deliverable templates (4 types) | Not started | Blocks template-based generation |

## Performance

| Operation | Target | Hard Limit |
|-----------|--------|------------|
| Search | <3s | <10s |
| Page load | <2s | <5s |
| Account Brief | <60s | <5min |
| Engagement Prep | <90s | <5min |
| Classification | <30s | <2min |

All generation operations must show real-time progress indicators. If >60s, show "Still working — pulling data from N sources."

## Testing

- **Backend:** `pytest` (Django)
- **Frontend:** `vitest` (React/Vite)
- **Linting:** `ruff` (Python), ESLint (JS/TS)
- **Integration:** Against live Foundation v3 — tests the real API contract
- **Smoke:** 3 consecutive successful end-to-end runs (login → search → generate → edit → SP write-back)
- **Foundation bugs → Foundation issues.** Don't work around them in KIRT.
