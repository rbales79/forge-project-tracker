# Phase 4: Engagement Prep, Meeting Notes & Cross-Sell — Context

## Phase Goal

Deliver the remaining generation features: Engagement Prep packages, Meeting Summary & Follow-Up, Client Deliverable generation, Cross-Sell recommendations, and SAF overlay features (growth scoring, white space grid, SAF progress tracking). By end of phase, all five Day-1 MVP outputs are working and all three user modes (Full SAF, On-Demand, Consumer) are supported.

## What to Build

### Discovery Question Taxonomy
- Formalize the 200+ question set into structured tree format: business → industry → solution modules
- Each question tagged with: module, applicable industries, related offerings
- Storage: Foundation annotations or KIRT-managed reference data (admin-configurable)
- Filtering logic: given an account, filter questions by industry + relevant offerings
- Next-best-question suggestions: select questions from the filtered set based on account context signals

**Content note:** If no formalized taxonomy file exists, construct it from available question variants during this phase. The 200+ question set is a content deliverable, not just a code feature.

### Engagement Prep Generation
Full Engagement Prep package:
- Prior work with this account (from Foundation knowledge + SP docs)
- Known pain points (from previous meeting notes, signals)
- Stakeholder map (pre-populated from SF contacts + prior meetings, user-editable)
- Recommended talking points (from account context)
- Discovery questions (filtered 200+ taxonomy by business/industry/solution, mapped to offerings)
- Next-best-question suggestions based on account context signals

**Works without Phase 1 completion** — generates from whatever context exists, follows graceful degradation + completeness indicator pattern from Phase 3.
**Target:** <90 seconds (soft target). Hard limit: <5 minutes.

### Stakeholder Map
- Pre-populated from SF contacts + prior meeting notes
- User-editable inline
- Fields: name, title, role (decision maker, champion, blocker, influencer, technical lead), relationship status, reporting relationships, notes
- Stored as Foundation annotations per account
- Displayed in Engagement Prep and accessible standalone

### Meeting Summary & Follow-Up
The on-demand, lowest-friction pipeline:
- User uploads meeting notes (any format: .docx, .txt, .pdf)
- KIRT generates immediately:
  - Polished meeting recap (what was discussed, key takeaways, decisions made) — prose, not bullet lists
  - Structured action items with owners and dates (extracted from raw notes)
  - Draft follow-up email suitable for sending to the client
  - Account intelligence update: new pain points, competitive mentions, stakeholder signals extracted and fed back into account context via Foundation annotations
- Works from notes alone; enriched by existing account data if available
- Single action: upload → something sendable (this is the Marcus persona's primary entry point)
- Auto-triggers Cross-Sell signal check after ingestion

### Client Deliverable Generation
- Input: company name + a few keywords (lowest friction — no prior account context required)
- KIRT pulls applicable template from SP (via Foundation's template canonical)
- Triggers research run (SF data + Foundation knowledge + public data)
- Generates client-ready document at 80-90% quality
- User refines via inline editing + section regeneration (same mechanism as Account Brief, Phase 3)
- Writes deliverable to SP (same write-back pipeline as Phase 3)

### Cross-Sell Recommendations
- Pattern-based: "Accounts like this also bought..." from engagement history
- Reference frame: offerings catalog (less precise without it, still generates)
- Triggered after: data ingestion, meeting note upload
- Displayed on: account dashboard, after generating Account Brief
- Requires offerings catalog v0.1 to be meaningful; operates in degraded mode without it

### Growth Scoring (SAF Phase 1 Feature)
- Account-level score driving tier assignment (Tier 1/2/3)
- 5 dimensions: revenue, strategic value, relationship depth, engagement frequency, white space size
- Starting weights: equal (20% each) — tunable via admin settings
- Tier thresholds: Tier 1 ≥ 80, Tier 2 ≥ 50, Tier 3 < 50 (admin-configurable)
- Manual tier override: user can adjust tier when formula gets it wrong
- Score stored as Foundation annotation; recalculated on data changes
- Surfaced on account list view and account dashboard

### White Space Grid
- Matrix: solution domains/offerings (rows) × account engagement status (columns: active, past, identified opportunity, no engagement)
- Populated per account by cross-referencing offerings catalog against SF opportunities + SP engagement docs
- Core Phase 1 output that feeds cross-sell recommendations and Account Brief white space section
- Display: visual grid in KIRT UI; exportable as structured data (JSON)
- Template-driven: offerings catalog defines the row axis

### SAF Progress Overlay
- Optional overlay for EA users (not default view — off by default)
- Shows SAF 6-phase progress per account: which phases have artifacts, which are incomplete
- Non-EA users never see it unless they opt in
- Does not enforce linear workflow — overlay only, not a gate

## Requirements Covered

- R24 — Engagement Prep package
- R25 — Stakeholder map (pre-populated, user-editable)
- R26 — Discovery questions from 200+ taxonomy, filtered by context
- R27 — Engagement Prep works without prior Phase 1 completion
- R28 — Meeting Summary: recap + action items + follow-up email + intelligence update
- R29 — Works from notes alone, enriched by account context
- R30 — Lowest-friction entry point (upload → sendable)
- R31 — Client Deliverable generation from name + keywords
- R32 — Template pulled from SP via Foundation
- R33 — Research run → generation → SP write-back → provenance
- R34 — Cross-sell recommendations (pattern-based)
- R35 — Triggered after ingestion/upload
- R36 — Offerings catalog as reference frame
- R44 — SAF progress overlay (optional for EA users)
- R45 — Growth scoring (5-dimension, equal weights, admin-tunable)
- R46 — Manual tier override
- R22 — Data completeness indicators (reused from Phase 3 component)
- R60 — Graceful degradation (same pattern as Phase 3)

## Key Constraints

- Discovery question taxonomy must be complete (200+ questions, tagged) before Engagement Prep integration tests can pass. This is a content dependency, not a code dependency.
- Offerings catalog v0.1 must exist before white space grid and cross-sell are meaningful. Without it, these features operate in degraded mode — generate without precise catalog matching, display "offerings catalog not configured" in completeness indicator.
- Growth scoring weights must be admin-configurable without code changes. Store in settings, not constants.
- SAF overlay is off by default. User must explicitly enable it. Non-EA users: never shown.
- Meeting Summary must use graceful degradation + completeness indicators (same Phase 3 component) — even though it works without prior account context, it should indicate when enrichment is thin.
- All new generation types (Engagement Prep, Meeting Summary, Client Deliverable) go through the Phase 3 LLM abstraction layer — no direct provider calls.
- Cross-sell triggered automatically after ingestion — this is an async background operation, not a blocking call.

## Tech Stack

- **Discovery questions:** Structured YAML/JSON taxonomy loaded at startup, queried in-memory (small dataset)
- **Stakeholder map:** Foundation annotations API (`POST/GET /v1/annotations/`) with `type: stakeholder_map`
- **Growth scoring:** Python formula (weighted sum), stored as Foundation annotation, recalculated via Django signal on account data change
- **White space grid:** Django computation from offerings catalog + SF opportunity annotations; rendered as React grid component
- **Cross-sell:** Embedding similarity or pattern matching against engagement history in Foundation; async Celery task or Django Q
- **LLM:** Phase 3 abstraction layer (all generation goes through it)
- **Frontend:** React components for Engagement Prep, Meeting Summary, Client Deliverable, SAF overlay, growth score badge, white space grid

## Dependencies

- Phase 1 complete: Foundation client, auth
- Phase 2 complete: search, upload, classification
- Phase 3 complete: LLM abstraction layer, Account Brief generation, SP write-back, data completeness indicators
- Offerings catalog v0.1 (content dependency — must be created before this phase)
- Discovery question taxonomy formalized (200+ questions in tree structure)
- Demo data: Accounts A, B, C, D all ingested into Foundation (integration testing requires realistic account data)
- SF CSV exports (accounts, contacts, opportunities, activities, notes) ingested into Foundation

## Research Context

**Meeting Summary as entry point:** The meeting notes upload → polished output flow is the highest-value, lowest-friction feature for on-demand users (Marcus persona). It should require zero prior setup — no account must exist, no SF data required. The output (recap + action items + follow-up email) must be immediately useful. Test this flow first in this phase.

**Discovery question taxonomy tree:** The business → industry → solution tree structure enables contextual filtering. For a Financial Services account with a data analytics opportunity, the system should surface: all business module questions + FS-specific industry questions + data analytics solution questions. The tree traversal is straightforward lookup — no ML required. The value is in the question content, not the filtering algorithm.

**White space grid as visual anchor:** The white space grid (offerings × engagement status matrix) is a high-value visual for both the Account Brief and standalone. Account A (Apex Industries — existing client with rich data) should show a clearly populated grid with active, past, and identified opportunity statuses. Account D (Terraform Logistics — thin data) should show mostly "no engagement" with completeness indicator noting data gaps. This contrast is a key demo moment.

**Cross-sell pattern matching:** For V1, "accounts like this also bought" can be simple: retrieve all accounts where `industry = target_account.industry`, look at their SF opportunities/SOWs via Foundation, identify common offering patterns. The LLM generates the recommendation narrative from these signals. Precise vector similarity matching is V2+ — the pattern lookup is sufficient for V1 demo quality.
