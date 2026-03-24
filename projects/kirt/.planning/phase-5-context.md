# Phase 5: Admin, Config & Production Hardening — Context

## Phase Goal

Complete the admin configuration surface, enforce all compliance requirements (MSA opt-out, GDPR purge), polish the responsive frontend, ingest all demo data, and validate the system end-to-end against live Foundation v3. By end of phase, the demo runs reliably 3 consecutive times and all non-technical stakeholders understand the value without explanation.

## What to Build

### Admin Configuration UI
Full admin surface (authenticated admin users only):

**Data Source Configuration:**
- Add/remove SP sites and document libraries as input sources
- Configure output SP folders per account, engagement, or deliverable type
- **Input/output path overlap validation:** Admin config must reject configurations where an output path equals, is a parent of, or is a child of any input source path. Same SP site with different folders is allowed. This prevents KIRT from re-ingesting its own output on the next crawl.
- Set crawl schedules and delta sync frequency
- Define location-ranking for canonical selection (ordered list of SP location patterns by authority)

**Scoring Configuration:**
- Growth scoring dimension weights (5 sliders, must sum to 100%)
- Tier thresholds (Tier 1/2/3 score cutoffs)
- Manual tier override history log

**LLM Provider Configuration:**
- Default provider selection (Gemini / Azure OpenAI / AWS Bedrock / Custom OpenAI-compatible)
- Fallback provider
- Model selection per task type (generation, classification, cross-sell)
- Provider health check display

**Offerings Catalog Management:**
- Upload/edit offerings catalog v0.1 (YAML/JSON)
- View solution domains and offerings
- Set maturity ratings (ready/emerging/sunset)
- Map offerings to discovery question tags

**System Settings:**
- KIRT-generated file naming convention (configurable template)
- SP output folder naming convention
- Data freshness threshold (default: 30 days)
- In-app notification preferences (global defaults)

### MSA Opt-Out Enforcement Audit
- Audit every LLM pipeline entry point for `ai_processing_allowed` flag check
- Confirm that opted-out content appears in keyword search results but never in:
  - Generation context
  - AI classification
  - Summarization
  - Cross-sell pattern matching
- Write explicit integration tests for opt-out enforcement (not unit tests — must test against Foundation's actual flag behavior)
- Document opt-out enforcement as a named invariant in CLAUDE.md / KNOWLEDGE.md

### GDPR Purge Workflow
- Admin-triggered purge action for a specific canonical document
- Purge flow:
  1. Admin selects document → KIRT shows all related annotations (classifications, scores, feedback pairs, stakeholder map entries, generation provenance)
  2. Admin confirms purge
  3. KIRT calls `DELETE /v1/delete/<id>/` with GDPR hard purge option on Foundation
  4. KIRT explicitly calls Foundation annotations API to delete all KIRT-owned annotations for that document
  5. KIRT verifies deletion (no orphaned annotations remain)
  6. Audit log entry written
- Test: run purge on a document with full annotation set, verify all 5 annotation types deleted

### In-App Notification System
- Dashboard notification badges: new data available, brief is stale (>30 days), generation complete, classification failed
- Notification feed in UI sidebar or header (click to see recent events)
- Mark as read / dismiss
- Driven by Foundation Redis Streams `document.ingested` events + KIRT's own generation completion events
- No email or push notifications in V1

### Responsive Frontend Polish
- Mobile read flows: search results, Account Brief view, Engagement Prep view (phone-comfortable)
- All read-only content fully accessible on mobile via Tailwind responsive classes
- Generation UI: desktop-only (mobile sees "generation available on desktop" message)
- Admin: desktop-only
- Loading states on all async operations (search, generation, upload, classification)
- Error states: Foundation unavailable, LLM provider error, upload failed
- Empty states: no search results, no account data, no documents for an account
- Performance: <2 second page load, <3 second search response (soft targets)

### Demo Data Ingestion
Full 4-account demo data package ingested into live Foundation v3:

**Account A (public company, existing client):**
- Full SF data set: account record, 5+ contacts, 3+ opportunities, 8+ activities, 3+ notes
- SP documents: account profile, SOW, 2x meeting notes, security assessment deck, cloud strategy proposal
- Dedup test data: proposal in 3 locations (Official-Library, Team-Site, Personal) with different filenames
- Stakeholder map: 5 stakeholders with roles and relationships

**Account B (public company, new prospect):**
- Bare SF account record
- No internal SP documents (tests greenfield research + public data flow)
- Cross-format dedup test: AI readiness assessment as .docx + .pdf

**Account C (private company, existing client):**
- SF data: account, contacts, opportunities
- SP documents: BSE, SOW (SOC services), meeting notes, pitch deck
- MSA opt-out flag set on one document (tests opt-out enforcement)

**Account D (private company, new prospect):**
- Bare SF account record
- 1 initial meeting note only
- No public data available (private company, thin data stress test)

Note: Accounts may be real (anonymized) or synthetic. Names and details finalized during demo data preparation.

**Config files ingested:**
- offerings-catalog.json (v0.1, 5-8 domains, 20-40 offerings)
- discovery-questions.json (200+ questions, tree structure)
- scoring-weights.json (equal weights, tier thresholds)
- location-ranking.json (5 location patterns with authority ranking)

**Templates in Foundation (doc_type: template):**
- Account Brief Template.docx
- Engagement Prep Template.docx
- Client Deliverable Template.docx
- White Space Grid template (KIRT renders as UI component; template defines schema)

### Integration & Demo Validation
- Run all 5 MVP output types against live Foundation v3 with demo data
- Verify each against its spec requirements:
  - Account Brief: all 9 sections, <60 seconds, completeness indicator, SP write-back
  - Engagement Prep: filtered discovery questions, stakeholder map, <90 seconds
  - Meeting Summary: upload → recap + action items + email in <2 minutes
  - Client Deliverable: name + keywords → generated + SP write-back
  - Cross-Sell: recommendations triggered after ingestion with offerings catalog
- Thin-data demo (Account D): every feature operates in graceful degradation mode
- Public data demo (Account B): research pipeline executes live against public sources
- Dedup demo: search shows 1 result with "N copies found" badge
- Opt-out demo: opted-out document appears in keyword search, excluded from generation
- SF CSV import validation: synthetic CSVs loaded and verified

### CI/CD Pipeline Completion
- GitHub Actions workflow: build → test → deploy to Contabo CVPS3
- Docker production deployment config (production docker-compose.yml, environment config)
- Automated deployment on merge to main branch

### Playwright E2E Test Infrastructure
- E2E test framework setup (Playwright)
- Critical flow tests automated
- Run against deployed/staging environment with demo data loaded

### E2E Test Suite
- Critical flow 1: Account Brief generation (Account A — rich data)
- Critical flow 2: Meeting notes upload → Follow-up email (Account D — thin data)
- Critical flow 3: Hybrid search with freshness filter and dedup UX
- Security test: Opt-out enforcement (opted-out content never in generation context)
- Security test: Permission inheritance (unauthorized document not surfaced)
- GDPR test: Purge with annotation cleanup verification

## Requirements Covered

- R50 — Admin-configurable settings (full surface)
- R51 — Admin-triggered GDPR purge with annotation cleanup
- R52 — Output vs. input separation (configured and enforced)
- R04 — MSA opt-out enforcement (audited at every LLM entry point)
- R17 — In-app notification badges and dashboard indicators
- R56 — Mobile read access (responsive polish)
- R57 — Task-oriented UI (final UX review and polish)
- R58 — Generation and admin: desktop-only
- R59 — Error state when Foundation unavailable
- R53, R54, R55 — Health endpoint, Sentry, usage logging (final verification)
- All R01–R62: end-to-end integration validation against live Foundation

## Key Constraints

- Demo must run reliably 3 consecutive times. This is the V1 definition of done. Not "it worked once." Three times, no intervention.
- MSA opt-out audit is mandatory before marking this phase complete. Opt-out enforcement is a compliance requirement, not a nice-to-have.
- GDPR purge must verify annotation deletion. Do not mark complete without running the full purge flow on a test document and confirming zero orphaned annotations.
- Demo data must cover the full account matrix. A demo without Account D (thin data) doesn't prove graceful degradation. A demo without Account B (public company, no internal history) doesn't prove the research pipeline.
- All Foundation bugs encountered during integration testing go to the Foundation repo as issues — do not work around them in KIRT.
- Performance targets are soft (not hard SLAs) — but if search takes >10 seconds or Account Brief takes >5 minutes, that's a hard failure.

## Tech Stack

- **Admin UI:** React admin panel (authenticated route, desktop-only)
- **Config storage:** Django admin models + database (all settings from DB, not files)
- **Notifications:** Django Channels or Server-Sent Events for real-time badge updates from Redis Streams
- **Demo data:** Python ingestion scripts (idempotent, can be re-run) that call Foundation `POST /v1/ingest/` for each asset
- **E2E tests:** Playwright (or similar) against deployed/staging environment with demo data loaded
- **Responsive:** Tailwind CSS responsive classes (already in stack); review with browser devtools mobile viewport

## Dependencies

- Phases 1-4 complete
- Live Foundation v3 accessible and stable (schedule demo window around Foundation stability)
- All demo data assets created (4 account packages, config files, templates)
- Offerings catalog v0.1 finalized (real or synthetic, will be available for V1)
- Discovery question taxonomy available (`demo-data/config/discovery-questions.json`)
- LLM provider credentials and quotas confirmed for demo volume
- SP output folders created and permissions configured for KIRT write-back

## Research Context

**Demo reliability:** The 3-consecutive-run requirement is the forcing function for reliability engineering. Common failure modes: LLM provider rate limits during sustained demo load (mitigate: cache generation results during demo, or increase provider quota), Foundation response times under cold start (mitigate: warm Foundation before demo), SP write-back latency (mitigate: show progress indicator, don't block UI). Run 3 consecutive demos during this phase as an explicit test, not just an assumption.

**Admin UX pattern:** The admin config surface should be discoverable but not in the primary navigation. A settings gear icon or a separate `/admin` route is sufficient. The key usability requirement is that all configurable parameters are visible in one place — no hunting through code or environment files. Each setting should have a description explaining what it controls and what impact changing it has.

**Demo data quality principle (from spec):** Clean-first, not intentionally messy. Real-world extraction will produce some organic messiness — that's fine and is actually good for demonstrating robustness. Don't engineer in bad data, but don't sanitize away realistic messiness either. Account D (thin data) is the intentionally thin scenario; the other accounts should be as rich as realistic extraction allows.
