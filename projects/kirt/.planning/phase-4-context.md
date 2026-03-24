# Phase 4: Engagement Prep, Meeting Notes & Cross-Sell — Context

## Phase Goal

Deliver Client Deliverable generation, Cross-Sell recommendations, and SAF overlay features (growth scoring, white space grid, SAF progress tracking). Engagement Prep and Meeting Summary have moved to Phase 3. By end of phase, all five Day-1 MVP outputs are working and all three user modes (Full SAF, On-Demand, Consumer) are supported.

## What to Build

### Client Deliverable Generation
- Input: company name + a few keywords (lowest friction — no prior account context required)
- KIRT pulls applicable template from SP (via Foundation's template canonical)
- All deliverable templates use Pellera format (from `demo-data/templates/`). Template-driven generation.
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

- R33 — Client Deliverable generation from name + keywords (Pellera template format)
- R34 — Template pulled from SP via Foundation
- R35 — Research run → generation → SP write-back → provenance
- R36 — Cross-sell recommendations (pattern-based)
- R37 — Triggered after ingestion/upload
- R38 — Offerings catalog as reference frame
- R46 — SAF progress overlay (optional for EA users)
- R47 — Growth scoring (5-dimension, equal weights, admin-tunable)
- R48 — Manual tier override
- R22 — Data completeness indicators (reused from Phase 3 component)
- R62 — Graceful degradation (same pattern as Phase 3)

## Key Constraints

- Offerings catalog v0.1 must exist before white space grid and cross-sell are meaningful. Without it, these features operate in degraded mode — generate without precise catalog matching, display "offerings catalog not configured" in completeness indicator.
- Growth scoring weights must be admin-configurable without code changes. Store in settings, not constants.
- SAF overlay is off by default. User must explicitly enable it. Non-EA users: never shown.
- All generation types (Client Deliverable) go through the Phase 3 LLM abstraction layer — no direct provider calls.
- Cross-sell triggered automatically after ingestion — this is an async Celery task, not a blocking call.
- All deliverable templates use Pellera format from `demo-data/templates/`.

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
- Phase 2 complete: search, upload, classification, SF CSV import
- Phase 3 complete: LLM abstraction layer, Account Brief generation, Engagement Prep, Meeting Summary, SP write-back, data completeness indicators
- Offerings catalog v0.1 (content dependency — real or synthetic, will be available for V1)
- Discovery question taxonomy available (`demo-data/config/discovery-questions.json`)
- Demo data: Accounts A, B, C, D all ingested into Foundation (integration testing requires realistic account data)
- SF CSV exports (accounts, contacts, opportunities, activities, notes) ingested into Foundation

## Research Context

**Discovery question taxonomy tree:** The business → industry → solution tree structure enables contextual filtering. Available in `demo-data/config/discovery-questions.json`. For a Financial Services account with a data analytics opportunity, the system should surface: all business module questions + FS-specific industry questions + data analytics solution questions. The tree traversal is straightforward lookup — no ML required. The value is in the question content, not the filtering algorithm.

**White space grid as visual anchor:** The white space grid (offerings x engagement status matrix) is a high-value visual for both the Account Brief and standalone. An existing client with rich data should show a clearly populated grid with active, past, and identified opportunity statuses. A thin-data account should show mostly "no engagement" with completeness indicator noting data gaps. This contrast is a key demo moment.

**Cross-sell pattern matching:** For V1, "accounts like this also bought" can be simple: retrieve all accounts where `industry = target_account.industry`, look at their SF opportunities/SOWs via Foundation, identify common offering patterns. The LLM generates the recommendation narrative from these signals. Precise vector similarity matching is V2+ — the pattern lookup is sufficient for V1 demo quality.

**Client Deliverable templates:** All templates use Pellera format. Templates are pre-generated in `demo-data/templates/` and loaded into Foundation as `doc_type: template`. Template-driven generation: KIRT pulls the template, populates with account data, and generates via LLM.
