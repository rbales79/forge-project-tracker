# Phase 3: Content Generation Pipeline — Context

## Phase Goal

Build the LLM provider abstraction layer and deliver Account Brief, Engagement Prep, and Meeting Summary & Follow-Up generation with full data completeness indicators, graceful degradation, SP write-back, inline editing, and feedback learning. By end of phase, the core generation loop is proven end-to-end: research pull → LLM generation → SP write-back → user edits → edits stored as future few-shot context. Engagement Prep and Meeting Summary move here from Phase 4 to consolidate all generation work.

## What to Build

### LLM Provider Abstraction Layer
- Provider abstraction interface: single API surface for all LLM calls, regardless of provider
- Implementations for: Gemini, Azure OpenAI, AWS Bedrock, and any OpenAI-compatible endpoint
- No provider-specific code anywhere in KIRT application logic — all provider details encapsulated in the abstraction layer
- Admin-configurable: default provider, fallback provider, model selection per task type
- Provider selection logic: read from admin config at generation time (no code redeploy required)
- Prompt template management: versioned prompt templates per generation type (Account Brief, Engagement Prep, Meeting Summary, Client Deliverable)
- Error handling: provider timeout, rate limit, unavailable — surface clear error state, do not retry silently

### Account Brief Generation
Full 9-section Account Brief:
1. Company overview (industry, size, revenue, headquarters)
2. Key contacts (names, titles, roles in buying process, relationship status)
3. Business priorities (from earnings, 10-K, conversations)
4. Relationship history (engagement timeline, past deals, current opportunities)
5. Open opportunities (pipeline with stage, value, next steps, risks)
6. Competitive landscape (who else, strengths/weaknesses)
7. Strategic notes (personal observations, dynamics, sentiment)
8. White space analysis (what we sell vs. what this account buys)
9. Action items

**Research pull:** SF data (CSV) + Foundation knowledge (SP docs, meeting notes, enrichments) + public data (SEC filings, earnings — gathered at runtime via AI research workflow)
**Target:** <60 seconds generation time (soft target)
**Hard limit:** <5 minutes. Progress indicator shown throughout.

### Data Completeness Indicators
On every generated output surface:
- Which data sources were available (SF record, SP documents, meeting notes, public data, stakeholder map, offerings catalog)
- What's missing and how it affects output quality
- Specific improvement suggestions ("upload a stakeholder map", "complete discovery session")
- Quality indicator (e.g., "75% — based on data coverage, not content quality")

### Graceful Degradation
- Account Brief without SF data → generates from SP docs + public data, flags SF gap
- Account Brief with no data at all → tells user what to upload/connect, does not generate empty content
- Every partial-data scenario: generates best-effort, flags exactly what's missing
- Never fail silently. Always communicate the degradation to the user.

### SP Write-Back
- Deliverables write to admin-configurable output folders per account/engagement
- File naming: `[Client]-[DocType]-[Date].docx` (e.g., `Acme-Corp-Account-Brief-2026-Q1.docx`)
- **Regeneration updates the existing file in place** — SP versioning automatically preserves previous versions. No new files created per regeneration.
- Foundation records SP URL in `derived_docs[]` on the research canonical record
- User sees: output document link (in SP) + provenance summary ("Generated from [template name], [date], research run #N")
- Write-back is explicit and user-initiated — confirmation step required
- Generation metadata stored as Foundation annotations (timestamp, input hash, generation count)
- Content versioning badge: "Generated v3, last updated 2026-03-20"
- **Source files are never modified** — only output folder files are written/updated
- **Path overlap prevention:** Output folder must not equal or overlap with any input source path. Validated at admin config time (Phase 5), enforced at write time as a safety check. Same SP site, different folder is allowed.

### Human-in-the-Loop Editing
- Inline edit: user can edit any section of generated output in the KIRT UI
- Section regeneration: user can regenerate a specific section with new instructions
- Both paths store the user's changes as Foundation annotations

### Prompt-Level Feedback Learning
- Store edited outputs (generated vs. user-corrected) in Foundation annotations
- At generation time: retrieve prior edits for same account + same document type
- Include retrieved edits as few-shot examples in the prompt
- No model training or RAG loop in V1 — prompt-level only

### Concurrent Generation Handling
- Two users generating the same account's brief simultaneously → both complete
- Both write to SP (different filenames or SP versions)
- KIRT surfaces "2 versions generated" indicator with both results
- User picks which to keep — no automatic merge

### Generation Progress UI
- Real-time progress via Foundation `GET /v1/jobs/<id>/`
- If >60 seconds: "Still working — pulling data from N sources"
- Progress bar or step indicator in UI
- Hard limit display: if >5 minutes → error state with retry option

### Engagement Prep Generation (moved from Phase 4)
Full Engagement Prep package:
- Prior work with this account (from Foundation knowledge + SP docs)
- Known pain points (from previous meeting notes, signals)
- Stakeholder map (pre-populated from SF contacts + prior meetings, user-editable)
- Recommended talking points (from account context)
- Discovery questions (filtered 200+ taxonomy by business/industry/solution, mapped to offerings)
- Next-best-question suggestions based on account context signals
- Works without prior Phase 1 completion — generates from whatever context exists
- Target: <90 seconds (soft target). Hard limit: <5 minutes.

### Stakeholder Map
- Pre-populated from SF contacts + prior meeting notes
- User-editable inline
- Fields: name, title, role (decision maker, champion, blocker, influencer, technical lead), relationship status, reporting relationships, notes
- Stored as Foundation annotations per account

### Meeting Summary & Follow-Up (moved from Phase 4)
- User uploads meeting notes (any format: .docx, .txt, .pdf)
- KIRT generates: polished recap, structured action items (with owners/dates), draft follow-up email, account intelligence update
- Works from notes alone; enriched by existing account data if available
- Single action: upload → something sendable (lowest-friction entry point)
- Auto-triggers Cross-Sell signal check after ingestion

### Public Data Research Pipeline
- SEC EDGAR integration for public company filings (10-K, 10-Q, proxy statements)
- News feed aggregation for company mentions
- Company profile data (industry, revenue, headquarters, key executives)
- Used by Account Brief generation and Engagement Prep

### Prompt Engineering
- Prompt design, testing, and iteration for each generation type
- Account Brief, Engagement Prep, Meeting Summary, Client Deliverable each need dedicated prompt templates
- Few-shot example curation for each generation type
- All templates use Pellera format (from `demo-data/templates/`)

## Requirements Covered

- R19 — Account Brief generation (<60 seconds target)
- R20 — All 9 Account Brief sections
- R21 — Works with partial data
- R22 — Data completeness indicators
- R23 — Deliverable write-back to SP with generation metadata annotation
- R24 — Engagement Prep package (moved from Phase 4)
- R25 — Stakeholder map (pre-populated, user-editable)
- R26 — Discovery questions from 200+ taxonomy, filtered by context
- R27 — Engagement Prep works without prior Phase 1 completion
- R28 — Meeting Summary: polished recap (moved from Phase 4)
- R29 — Structured action items with owners and dates
- R30 — Draft follow-up email
- R31 — Account intelligence update from meeting notes
- R32 — Works from notes alone, lowest-friction entry point
- R39 — Generation progress indicators
- R40 — Human-in-the-loop editing (inline edit, section regeneration)
- R41 — Prompt-level feedback learning
- R42 — Concurrent generation handling
- R43 — Content versioning badge
- R44 — SP write-back conventions (output folders, naming)
- R45 — Write-back explicit, user-initiated, confirmation required
- R62 — Graceful degradation (partial data → generates + flags)

## Key Constraints

- **Zero provider-specific code outside the abstraction layer.** This is non-negotiable. Reviewers: if you see `import anthropic` or `import google.generativeai` in any non-provider module, that's a violation.
- Provider selection must work without code changes. Admin config only.
- Prompt templates are versioned — store version in generation metadata so future debugging can reconstruct what prompt was used.
- SP write-back requires user confirmation. No automatic writes.
- Foundation's `derived_docs[]` must be updated on every successful write-back.
- Re-running research creates a new Iceberg snapshot on the same research record + generates a new SP document revision.
- Previous SP versions remain in SP version history.
- Intermediate research data (Iceberg snapshots, raw research runs) is **never** surfaced to users. Users see output document + provenance summary only.

## Tech Stack

- **LLM abstraction:** Custom provider interface (Python abstract base class or Protocol), implementations per provider. Use `openai` SDK for OpenAI-compatible endpoints (Azure OpenAI, Bedrock via proxy); `google-generativeai` for Gemini.
- **SP write-back:** Microsoft Graph API (MS Graph SDK for Python or direct REST calls) — write to SharePoint document libraries
- **Prompt management:** Versioned prompt templates (YAML or JSON), loaded at generation time
- **Generation progress:** Foundation `GET /v1/jobs/<id>/` polled from frontend via Django API endpoint
- **Frontend:** React generation UI, progress indicators, inline editor, completeness indicator component

## Dependencies

- Phase 1 complete: Foundation client, auth
- Phase 2 complete: search (for pulling account context), document upload (classification writes used in generation context)
- LLM provider credentials: Gemini API key, Azure OpenAI endpoint + key, AWS Bedrock credentials
- Microsoft Graph API access for SP write-back
- SP output folder structure configured (admin config)
- Offerings catalog v0.1 must exist before white space section of Account Brief is meaningful
- Demo data for at least Account A (Apex Industries) in Foundation for end-to-end generation testing

## Research Context

**Provider abstraction pattern:** Build KIRT's own abstraction layer. Azure OpenAI + Bedrock via OpenAI-compatible SDK. Gemini via native SDK. The abstraction layer should normalize: streaming vs. non-streaming, token limits, error codes, and retry behavior across all providers. Provider selection is admin-configurable — no provider-specific code outside the abstraction layer.

**Graceful degradation UX:** The data completeness indicator (% score, sources used, missing sources, improvement suggestions) is a differentiating UX pattern. Competitors generally fail silently or require complete setup. This indicator should be a first-class UI component, not an afterthought. Design it early, reuse it across all generated output types.

**Prompt-level feedback:** The few-shot approach (store edits, include in next generation for same account/type) is the simplest feedback mechanism that delivers real improvement. Store (generated, edited) pairs as Foundation annotations with type `kirt_feedback`. At generation time, query annotations for this account + doc_type, retrieve last 3 pairs, include as examples in the prompt. Test this loop explicitly — it's easy to implement but easy to break silently.

**SP write-back via Graph API:** Use the Microsoft Graph API to write to SharePoint document libraries. This requires KIRT's Entra app registration to have `Sites.ReadWrite.All` permission scoped to the designated output sites. Output folder paths must be configurable per deployment.
