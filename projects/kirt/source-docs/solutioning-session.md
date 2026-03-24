> **STATUS: HISTORICAL** — All items from this session have been resolved and merged into `kirt-spec-seed.md`. This document is retained for decision archaeology only. Do not treat any items here as open or blocking.

# KIRT — Gough/R Solutioning Session

> Items extracted from the spec seed and roadmap analysis that require design sessions before they can be locked in. Once resolved, decisions feed back into `kirt-spec-seed.md`.
>
> **Status:** Roy's initial positions documented below. Jonathan to review, challenge, and co-sign before locking into the spec seed.

---

## 1. Decisions Needed Before V1 Build

These are content and business decisions that gate V1 development. They are not development tasks — they are inputs the build depends on.

### 1.1 Offerings Catalog v0.1

| | |
|---|---|
| **Options** | Build from scratch / extract from existing materials / defer with placeholder |
| **Impact** | Gates white space grid, scoring, cross-sell — entire Phase 1 |
| **Roy's Position** | Extract from existing materials — best effort to capture what we actually have today. PPT decks and practice collateral are the likely source. |
| **Open** | Who owns this extraction? Caleb + Carlos are the natural candidates. Need to identify what source materials exist and how much effort this is. |
| **Scope** | V1 (required before build) |

> **Jonathan:** Does your gap analysis tool or any existing practice documentation give us a head start here? Who has the most complete view of offerings across practices?

### 1.2 Growth Scoring Model v0.1

| | |
|---|---|
| **Options** | Simple weighted formula / ML-based / manual tier assignment for MVP |
| **Impact** | Determines account prioritization UX |
| **Roy's Position** | Simple weighted formula for V1 with manual override capability. ML-based scoring is a future version concern. Users must be able to manually adjust tier assignments when the formula gets it wrong. |
| **Scope** | V1 (weighted formula + manual override), V2+ (ML-informed) |

> **Validation:** A weighted formula ships fast, manual override builds trust with users who know their accounts better than any model. ML requires data volume we won't have at MVP scale (3-4 synthetic accounts).

> **Jonathan:** Any thoughts on initial weight distribution? Revenue vs. strategic value vs. relationship depth — what's the starting ratio?

### 1.3 Tier Assignment Criteria

| | |
|---|---|
| **Options** | Revenue / strategic value / relationship depth / composite |
| **Impact** | Drives which accounts get full treatment |
| **Roy's Position** | Composite — no single dimension is sufficient. |
| **Scope** | V1 (carries through all versions) |

> **Validation:** Pure revenue tiers miss strategic accounts. Pure relationship depth misses greenfield opportunities. Composite with tunable weights (via admin settings, per spec seed constraint #9) is the right architecture.

> **Jonathan:** Agree with composite? Should we define the initial dimensions now, or iterate once we see the demo account data?

### 1.4 Discovery Question Taxonomy Structure

| | |
|---|---|
| **Options** | Flat list / tree (business→industry→solution) / offering-mapped |
| **Impact** | Shapes the engagement prep generation |
| **Roy's Position** | Tree structure (business→industry→solution). |
| **Scope** | V1 (carries through) |

> **Validation:** Aligns with existing materials — the 200+ questions already exist across business, industry, and solution modules. Tree structure lets KIRT filter contextually: show industry questions for healthcare accounts, solution questions for specific offerings. Offering-mapping can be layered on top of the tree in V2 once the offerings catalog matures.

> **Jonathan:** Agree? The tree maps directly to the three modules that already exist. Offering-mapping becomes possible once we have the catalog.

### 1.5 Canonical Document Selection Rules

| | |
|---|---|
| **Options** | Most recent / most authoritative location / user-selected |
| **Impact** | Affects dedup UX and trust |
| **Roy's Position** | Default to most authoritative location (e.g., official SP library > personal OneDrive), with manual override so users can set canonical. |
| **Scope** | V1 (authoritative + manual override), V2 (automated reconciliation) |

> **Validation:** "Most authoritative location" needs a definition — likely means official SP document libraries outrank personal OneDrive, which outranks email attachments. The manual override is critical because KIRT doesn't always know which copy the user considers definitive (per spec seed: "KIRT does not decide which copy is authoritative — the user does").

> **Jonathan:** How do we define "authoritative"? Is it by SP site hierarchy, or do we need a location-ranking config?

### 1.6 Demo Account Selection

| | |
|---|---|
| **Options** | 3-4 accounts covering public/private × new/existing matrix |
| **Impact** | Shapes synthetic data build |
| **Roy's Position** | Same approach as offerings catalog — pick 4+ real accounts and extract as close to reality as possible. |
| **Scope** | V1 (demo data) |

> **Validation:** Using real accounts (anonymized/obfuscated) will be more credible than fully synthetic data. Need to ensure we cover the matrix: at least one public company (SEC data, earnings available), one private company (tests thin-data behavior), one existing client (rich internal history), one new prospect (tests greenfield research value).

> **Jonathan:** Any accounts you'd recommend that have good data coverage across SF + SP + public sources?

### 1.7 Data Quality Strategy for Demo

| | |
|---|---|
| **Options** | Clean only / mixed (clean + messy) |
| **Impact** | Affects demo narrative |
| **Roy's Position** | Clean-first approach. Plan for clean data, but acknowledge some of it will naturally be messy once we extract from real sources. Don't intentionally introduce bad data for demo. |
| **Scope** | V1 (demo only) |

> **Validation:** Pragmatic. Aim for clean, accept that real-world extraction will introduce some messiness organically. We don't need to stage bad data to prove the point.

> **Jonathan:** Should we have a narrative ready for when messy data shows up naturally? ("Here's what KIRT does when data isn't perfect" is actually a strong demo moment.)

### 1.8 Demo Audience

| | |
|---|---|
| **Options** | Internal leadership / technology partners / prospective clients |
| **Impact** | Shapes fidelity level and emphasis |
| **Roy's Position** | Internal leadership AND go-to-market teams. |
| **Scope** | V1 (demo) |

> **Validation:** Two audiences means two emphasis modes. Internal leadership cares about cost-of-sale reduction and strategic alignment. GTM teams care about "does this save me time before my meeting tomorrow." May need two demo scripts or at least two narrative tracks through the same demo.

> **Jonathan:** Agree with dual audience? Should we plan distinct demo flows or one flow that hits both?

---

## 2. Architecture & Platform Questions

### 2.1 Application Form Factor

| | |
|---|---|
| **Question** | Web app, Teams tab, or SP add-in? |
| **Roy's Position** | Web app. |
| **Scope** | V1 (web app), future (Teams tab wrapper possible) |

> **Validation:** React/Vite is already confirmed in the stack. A Teams tab is just an iframe wrapper around a web app — it can be added later without architectural changes. Starting with web app gives maximum flexibility.

> **Jonathan:** Agree? Any pressure from the org to surface this inside Teams from day one?

### 2.2 Offline / Degraded Mode

| | |
|---|---|
| **Question** | What happens when Foundation is down? |
| **Roy's Position** | Error state for V1. Cached/degraded mode is a future version concern. |
| **Scope** | V1 (error state), V2+ (cached results consideration) |

> **Validation:** Right for MVP. Caching adds significant complexity (cache invalidation, staleness, partial data). Not worth it for V1 with <20 users. For demo reliability, ensure Foundation is stable during demo windows rather than engineering around it.

> **Jonathan:** Agree?

### 2.3 Multi-Tenancy Model

| | |
|---|---|
| **Question** | Single-tenant (Pellera only) or multi-tenant from the start? |
| **Roy's Position** | Internal single-tenant only. Multi-tenancy is a future version problem, likely gated by security and legal review. |
| **Scope** | V1 (single-tenant), V2+ (multi-tenant if needed) |

> **Validation:** Foundation already has `tenant_id` scoping, so the data layer supports multi-tenancy. KIRT can be designed with tenant awareness in the data model without building the full multi-tenant UX/isolation. Don't over-engineer for a scenario that hasn't been validated by the business.

> **Jonathan:** Any signal from Carlos or Rusk about KIRT as a client-facing product vs. internal tool only?

### 2.4 Content Versioning

| | |
|---|---|
| **Question** | When KIRT regenerates a brief, what happens to the previous version? |
| **Roy's Position** | Lightweight dual approach — store generation metadata as Foundation annotations (timestamp, input hash, generation count) while SP handles actual document versions. KIRT shows "Generated v3, last updated 2026-03-20" as a badge, not a full diff view. |
| **Scope** | V1 |

> **Validation:** This gets the benefit (users see regeneration history) without the cost (no separate version store in KIRT). Foundation annotations are the right place for generation metadata — they're already in the API contract.

> **Jonathan:** Sufficient for V1, or do we need full version comparison in KIRT?

### 2.5 LLM Provider Strategy

| | |
|---|---|
| **Question** | Which LLM providers to support? |
| **Roy's Position** | Multi-provider from the start. Focus on Gemini, Azure OpenAI, and AWS Bedrock, plus any OpenAI-compatible endpoint. Build a provider abstraction layer so KIRT never has provider-specific code. Provider selection configurable via admin settings. |
| **Scope** | V1 (multi-provider abstraction), carries through |

> **Validation:** OpenAI-compatible is the right interface to standardize on — all three named providers support it. The abstraction layer means swapping providers is a config change, not a code change. Aligns with spec seed constraint #9 (admin-configurable deployment settings).

> **Jonathan:** Any enterprise governance concerns with specific providers? Does Pellera have an approved vendor list for AI services?

---

## 3. UX & Data Questions

### 3.1 Permission Model

| | |
|---|---|
| **Question** | All users see everything, or role-based within KIRT? |
| **Roy's Position** | Role-based, tied to AD/SSO groups. Mapped to tiers: bronze, silver, gold. |
| **Scope** | V1 (AD-group-to-role mapping), V2 (per-account filtering) |

> **Validation:** Foundation already enforces document-level permissions via `sp_permissions[]` and M365 groups — that's the baseline. Adding KIRT role tiers on top is a new but manageable layer:
> - **V1:** Map AD groups to KIRT roles at login. "If user is in AD group X, they get gold access." Just a group-to-role mapping table in admin config. Bronze/silver/gold tiers control which KIRT features are visible (gold sees admin settings, silver sees full search, bronze sees basic briefs).
> - **V2:** Fine-grained per-account access (AE sees only their accounts). Requires account-ownership data from SF and a filtering layer on every query.

> **Jonathan:** Does the AD-group-to-role mapping approach work for V1? What groups already exist that we could map to?

### 3.2 Persona Definitions

| | |
|---|---|
| **Question** | Who are the personas, what do they see? |
| **Roy's Position** | Tie to AD/SSO groups and the bronze/silver/gold permission tiers. |
| **Scope** | V1 (simplified), V2 (full persona views) |

> **Validation:** Merging personas with permission tiers solves both problems at once. Personas aren't just UX sugar — they're permission scopes.

> **Proposed mapping (for discussion):**
>
> | Tier | AD Group Pattern | Personas | What They See |
> |------|-----------------|----------|---------------|
> | Gold | EA team, leadership | Architect/EA, Sales Leadership | Full platform — all accounts, admin settings, scoring config, analytics |
> | Silver | GTM, AEs, Solution Specialists | AE, Solution Specialist | Search, briefs, engagement prep, cross-sell for accessible accounts |
> | Bronze | Delivery, PM, extended team | Delivery/PM | Read-only briefs, engagement context, knowledge export |

> **Jonathan:** Does this tier mapping make sense? Which AD groups map to which tier?

### 3.3 Feedback Loop / Learning

| | |
|---|---|
| **Question** | Does KIRT learn from user edits to generated deliverables? |
| **Roy's Position** | Yes, phased approach: |

> **Phasing:**
> 1. **V1 — Prompt-level feedback:** Store edited outputs alongside originals. When generating for the same account/type, include the user's prior edits as few-shot examples in the prompt. No model training required. Low cost.
> 2. **V2 — RAG feedback loop:** User edits get ingested back into Foundation as "KIRT-refined" documents. Future generations retrieve the refined version as context.
> 3. **V3+ — Fine-tuning evaluation:** Collect edit pairs (generated vs. user-corrected) as training data. Fine-tune if edit volume justifies it.

> **Jonathan:** Agree with the phased approach? Anything else that should feed the learning loop?

### 3.4 Notification Model

| | |
|---|---|
| **Question** | How does KIRT notify users of new data, stale briefs, etc.? |
| **Roy's Position** | All channels (push, dashboard, email, in-app badges), configurable per user. V1 ships in-app badges and dashboard indicators. Email digest and push notifications are V2. |
| **Scope** | V1 (in-app), V2 (full notification suite) |

> **Validation:** In-app indicators come nearly free with the React UI. Email and push require a notification service, email integration, and user preference management — V2 scope.

> **Jonathan:** Agree that in-app indicators are sufficient for V1? Or is email digest critical for the GTM audience who may not check KIRT daily?

### 3.5 Concurrent Editing

| | |
|---|---|
| **Question** | Two users generate deliverables for the same account simultaneously? |
| **Roy's Position** | Allow both to complete. Write both to SP (different filenames or SP versions). Surface both in KIRT with a "2 versions generated" indicator. User picks which to keep. Content merging is V3+ (requires semantic diff). |
| **Scope** | V1 (write both, pick winner), V3+ (merge) |

> **Jonathan:** Sufficient for V1? Or do we expect enough concurrent generation that this needs a queue?

### 3.6 Mobile Experience

| | |
|---|---|
| **Question** | Mobile requirements? |
| **Roy's Position** | Yes — responsive web app (not native). V1 ensures the web app is responsive via Tailwind. Key mobile flows: search, view Account Brief, view Engagement Prep. Generation and admin are desktop-only in V1. |
| **Scope** | V1 (responsive web), future (native if needed) |

> **Jonathan:** Agree with responsive web as the mobile strategy? Any use cases that require offline mobile access?

---

## 4. Compliance & Operations Questions

### 4.1 MSA Opt-Out Scope

| | |
|---|---|
| **Question** | Does opt-out mean no AI processing, or no ingest at all? |
| **Roy's Position** | Opt-out means keyword search only — documents can be ingested and indexed for keyword/phrase search, but no LLM processing (no summarization, no AI classification, no generation using that content). Requires `ai_processing_allowed` flag on canonical records that KIRT checks before running any LLM pipeline. |
| **Scope** | V1 (architecture must support this from day one) |

> **Jonathan:** Align with this interpretation?

### 4.2 Data Retention / GDPR Purge

| | |
|---|---|
| **Question** | Does KIRT need its own purge, or does Foundation's purge cascade? |
| **Roy's Position** | KIRT manages its own purge workflow. Foundation provides the mechanism (`DELETE /v1/delete/<id>/` with GDPR hard purge), but triggering is an external action — KIRT owns the business logic for what to purge and when. V1: admin-triggered. V2+: automated retention policies. |
| **Scope** | V1 (admin-triggered purge), V2+ (automated retention) |

> **Open:** KIRT's annotations in Foundation (classifications, scores, tags via `POST /v1/annotations/`) also need to be purged when source documents are purged. Need to confirm if Foundation's purge cascades to annotations or if KIRT must explicitly delete them.

> **Jonathan:** Does Foundation's GDPR purge cascade to annotations, or does KIRT need to clean those up separately?

### 4.3 Monitoring and Observability

| | |
|---|---|
| **Question** | What monitoring does KIRT need? |
| **Roy's Position** | Full observability, phased rollout: |
| **Scope** | V1 (basics), progressive enhancement |

> **V1:**
> - **Error tracking:** Django error logging + Sentry (or equivalent). Non-negotiable.
> - **Health endpoint:** `/health/` for KIRT (is the app up, can it reach Foundation).
> - **Usage analytics:** Basic request logging (who searched, who generated, when). Foundation's job tracking covers ingestion.
>
> **V2+:**
> - APM (request latency percentiles)
> - Generation quality scoring (user satisfaction from edit rates)
> - Admin dashboard for visibility

> **Jonathan:** Agree with V1 scope? Any specific metrics the leadership demo audience will expect?

### 4.4 Accessibility (WCAG)

| | |
|---|---|
| **Question** | WCAG compliance requirement? |
| **Roy's Position** | Defer to V2+. Shadcn components provide reasonable baseline accessibility (keyboard navigation, ARIA labels, focus management) out of the box. |
| **Scope** | V2+ |

> **Jonathan:** Any awareness of a Pellera accessibility mandate?

### 4.5 Performance Targets

| | |
|---|---|
| **Question** | Performance targets beyond Account Brief <60s? |
| **Roy's Position** | Tiered soft targets for V1 with progress indicators on all generation operations. No hard SLAs until V2. |
| **Scope** | V1 (soft targets + UX feedback), V2 (hard SLAs) |

> **V1 targets:**
>
> | Operation | Target | Hard Limit |
> |-----------|--------|------------|
> | Search | <3 seconds | <10 seconds |
> | Page load | <2 seconds | <5 seconds |
> | Account Brief generation | <60 seconds | <5 minutes |
> | Engagement Prep generation | <90 seconds | <5 minutes |
> | Document classification (on upload) | <30 seconds | <2 minutes |
>
> All generation operations show real-time progress via Foundation's `GET /v1/jobs/<id>/`. If generation exceeds 60 seconds, UI shows "Still working — pulling data from N sources" rather than just spinning.

> **Jonathan:** Do these targets seem right? Any operations missing?

---

## Session Format

For each item above, the session should produce:
1. **Decision** — what we're going with
2. **Rationale** — why (1-2 sentences)
3. **Scope** — V1 only, or carries through all versions
4. **Action** — who does what to make it real (if it's a content dependency like the offerings catalog)

**Status legend:**
- Items with Roy's position + validation = ready for Jonathan's review
- Items marked **Open** = need discussion before locking

Resolved decisions get written back into `kirt-spec-seed.md`. Items that can't be resolved get tagged with what's needed to unblock them.
