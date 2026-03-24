# KIRT Beyond SAF — Nighthawk Bridge

> How KIRT serves users outside the EA team who won't follow the Strategic Account Framework. Addresses the Project Nighthawk use case (individual sales intelligence) within KIRT's architecture rather than as a separate tool.

---

## The Problem

The SAF is an EA-team process. It has 6 phases, scoring models, maturity gates, and structured deliverable types. KIRT was designed around it.

But most of the people who would use KIRT are not EAs and will never follow the SAF:
- **AEs** need pre-meeting prep and account context — they don't care about white space grids or maturity gates
- **Solution Specialists** need competitive positioning and technical context — they skip straight to the content they need
- **Sales Managers** need pipeline visibility and team patterns — they consume intelligence, they don't generate it
- **Delivery/PMs** need engagement context and handoff docs — they enter mid-cycle after the EA work is done
- **Individual contributors** (the Nighthawk user) want a personal sales brain they can query — they don't want a framework, they want answers

Even EAs skip phases. An architect preparing for a follow-up meeting with a long-standing client doesn't need Phase 1 account intelligence from scratch — they need Phase 2 engagement prep with the assumption that context already exists.

## The Architecture Implication

KIRT cannot be built as a linear SAF pipeline where users must start at Phase 1 and work through to Phase 6. The SAF phases should be the **organizing model for capabilities**, not a required workflow.

This means:
- Every KIRT feature must be **independently accessible** — you can generate an Engagement Prep without having run Account Intelligence first
- The SAF phases describe **what KIRT can do**, not **what users must do**
- The UI should be **task-oriented** (what do you need right now?) not **phase-oriented** (which SAF step are you on?)
- Users who follow the full SAF get compounding value (each phase enriches the next), but users who cherry-pick still get standalone value from any single capability

## User Modes

Instead of forcing all users through the SAF, KIRT should recognize different usage patterns:

### Mode 1: Full SAF (EA Team)

The structured path. EAs work through phases, build cumulative account intelligence, generate all deliverable types, manage the scoring pipeline. This is what the spec seed describes today.

**Who:** Enterprise Architects, strategic account leads
**How they use KIRT:** Sequentially through phases for target accounts. Dashboard shows SAF progress per account.

### Mode 2: On-Demand Intelligence (Nighthawk Replacement)

The personal sales brain. User asks a question, KIRT answers it using whatever data exists. No framework, no phases, no scoring — just search, context, and generation.

**Who:** AEs, Solution Specialists, individual contributors
**How they use KIRT:**
- "Prepare me for my meeting with Acme Corp tomorrow" → Engagement Prep (pulls whatever context exists, doesn't require Phase 1 to have been completed)
- "What do we know about this company?" → Account Brief (auto-generated from available data, clearly indicates data completeness)
- "Upload these meeting notes" → Ingest and enrich (signals extracted, account context updated, no Phase 3 scoring required)
- "What are our healthcare clients' common objections?" → Cross-account pattern search
- "Generate a client-ready summary for this prospect" → Client Deliverable from minimal input

**Key difference from SAF mode:** No phase tracking. No scoring gates. No white space grid (unless the offerings catalog happens to be populated for that account). KIRT does the best it can with what it has and tells the user what's missing.

### Mode 3: Consumer (Read-Only)

Users who consume intelligence but don't generate or upload. They read Account Briefs, search for documents, view engagement context.

**Who:** Sales Leadership, Delivery/PM, extended team
**How they use KIRT:** Search, browse, read. No generation, no upload, no scoring.

## What This Means for the Build

### Features That Work in All Modes

These capabilities should function regardless of whether the user is following the SAF:

| Feature | SAF Context | Standalone Context |
|---------|-------------|-------------------|
| Search | Finds documents across all phases | Finds documents, period |
| Account Brief | Phase 1 output with scoring and white space | Best-effort summary from available data, flags gaps |
| Engagement Prep | Phase 2 output with full discovery taxonomy | Pre-meeting package from whatever context exists |
| Meeting note upload | Feeds Phase 3 signal capture pipeline | Enriches account knowledge, extracts action items |
| Client Deliverable | Phase 4 gated output | Generated from minimal input, no gate required |
| Cross-sell recommendations | Driven by white space grid and offerings catalog | Pattern-based suggestions from engagement history, works without catalog (just less precise) |
| Document classification | Routes to phase-specific processing | Auto-classifies for search and retrieval |
| Dedup | Supports SAF data quality | Saves everyone time finding the right version |
| Human-in-the-loop editing | Refines SAF deliverables | Refines any generated content |

### Features That Only Apply in SAF Mode

These are Phase 3+ capabilities that require the full SAF context:

| Feature | Why SAF-Only | Version |
|---------|-------------|---------|
| Growth scoring / tier assignment | Requires offerings catalog + composite scoring model | V1 |
| White space grid | Requires offerings catalog as row axis | V1 |
| Opportunity scoring | Requires signal taxonomy and scoring model | V2 |
| Maturity gate | Requires offering maturity ratings | V2 |
| Reframe deck generation | Requires scored + gated opportunity pipeline | V3 |
| EBR/QBR packages | Requires engagement history + scoring data | V3 |
| Win/loss pattern analysis | Requires historical scoring data volume | V4+ |
| Advancement triggers | Requires defined trigger model | V4+ |

### Data Completeness Indicators

When a user in on-demand mode generates an Account Brief, KIRT should clearly indicate what data it had and what it didn't:

```
Account Brief: Apex Industries
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Data sources used:
  ✓ Salesforce account record
  ✓ 3 SharePoint documents
  ✓ 2 meeting notes
  ✓ Public data (10-K, earnings)
  ✗ No offerings catalog coverage (white space unavailable)
  ✗ No stakeholder map on file
  ✗ No discovery questions completed

Brief quality: ████████░░ 75%
To improve: upload a stakeholder map, complete discovery session
```

This lets non-SAF users understand the value gap without requiring them to follow the framework.

### UI Implications

**Don't do this:**
```
SAF Phase 1 → SAF Phase 2 → SAF Phase 3 → SAF Phase 4
```

**Do this:**
```
What do you need?
  [Search]  [Prepare for a Meeting]  [Generate a Brief]  [Upload Documents]

Your accounts:
  Apex Industries — rich context (12 docs, 5 meetings, SF data)
  Terraform Logistics — thin context (1 meeting note)
```

The SAF phase progression can exist as an **optional overlay** for EA users — a progress tracker on the account detail page showing which phases have been completed and what's available next. Non-EA users never see it unless they opt in.

## Spec Seed Implications

These changes should be reflected in the spec seed:

1. **Reframe the SAF mapping** — SAF phases describe capabilities, not required workflow. Add a note that features are independently accessible.

2. **Add data completeness indicators** — every generation output should show what data was available and what was missing. This is the bridge between "full SAF" and "on-demand intelligence."

3. **Task-oriented UI** — the spec should describe the UI as task-oriented (what do you need?) not phase-oriented (which step are you on?). SAF progress tracking is an optional view.

4. **Graceful degradation** — every feature should work with partial data and clearly communicate what's missing. An Account Brief with no SF data is still useful if there are SP docs and public data. An Engagement Prep with no prior meetings still generates questions and stakeholder placeholders.

5. **Discovery questions without the taxonomy** — on-demand users should be able to get AI-generated discovery questions even without the formal tree taxonomy. The taxonomy improves precision, but KIRT should generate reasonable questions from account context alone.

6. **Cross-sell without the catalog** — pattern-based recommendations should work from engagement history alone. The offerings catalog makes them more precise, but "accounts like this also engaged with security assessments" doesn't require a catalog.

## Nighthawk Feature Mapping

What Nighthawk users build manually that KIRT replaces:

| Nighthawk (Manual) | KIRT (Automated) | Requires SAF? |
|--------------------|--------------------|--------------|
| OneDrive folder hierarchy per client | Account-level document organization (automatic via Foundation) | No |
| Manual SF data exports to Word docs | SF CSV import → auto-generated account summaries | No |
| Client Intelligence Dossier (living Word doc) | Account Brief (auto-generated, auto-updated) | No |
| Pre-meeting Copilot queries across files | Engagement Prep generation | No |
| Post-meeting note capture in OneDrive | Meeting note upload → auto-classification → enrichment | No |
| Weekly manual SF export refresh | Foundation ingest pipeline (continuous or scheduled) | No |
| Copilot declarative agent grounded in folder | KIRT's search with knowledge hierarchy ranking | No |
| SEC EDGAR manual research | AI research pipeline (runtime, V2) | No |
| Win/loss analysis Word docs | Structured win/loss capture | V3 (SAF Phase 5) |
| Cross-client pattern recognition via Copilot prompts | Cross-account analytics | V3+ |

**The takeaway:** 80% of what Nighthawk users want doesn't require the SAF at all. KIRT gives them the enterprise version of their personal duct-tape system, whether or not they ever touch growth scoring or maturity gates.
