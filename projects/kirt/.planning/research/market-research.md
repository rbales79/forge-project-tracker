# KIRT — Market Research

> Research tool: ECC market-research (running in background) + synthesis from spec seed positioning signals.
> Status: Preliminary synthesis complete. Full ECC output will be merged when agent completes.
> Date: 2026-03-24

---

## Overview

KIRT operates at the intersection of three market categories: **account intelligence**, **sales enablement**, and **revenue intelligence**. None of the established players in these categories match KIRT's positioning as an internal-first, Foundation-native intelligence layer that generates written deliverables and writes them back to SharePoint.

**Build / Fork / Extend Recommendation: Build (Greenfield)**

No existing open-source or commercial platform provides:
1. Foundation-native data layer integration
2. SAF-aligned workflow with on-demand fallback
3. SharePoint write-back for generated deliverables
4. Multi-provider LLM abstraction with admin-configurable provider switching
5. Graceful degradation with data completeness indicators

The tech stack (Django, React, Weaviate, Foundation API) has no skeleton project that covers more than 30% of the required surface. Build from scratch, using Foundation reference POC as architectural prior art.

---

## Competitive Landscape

### Category 1: Revenue Intelligence Platforms

**Gong**
- Core value: Call recording, transcription, deal intelligence from conversation data
- Key features: AI-generated call summaries, deal risk scoring, pipeline forecasting, coaching insights
- Pricing: Enterprise ($5,000-$10,000+/month range, per seat)
- What KIRT does that Gong doesn't: Generates written client-facing deliverables (Account Briefs, Engagement Prep packages). Processes SP documents and unstructured internal knowledge, not just call recordings. SAF framework alignment. SP write-back.
- What Gong does well: Conversation analysis, objection detection, rep coaching. KIRT should consider: Gong's approach to surfacing competitive mentions from transcripts is a useful pattern for KIRT's V2 transcript analysis feature.

**Clari**
- Core value: Revenue forecasting and pipeline inspection using CRM + activity data
- Key features: Pipeline analytics, deal inspection, forecast accuracy, revenue collaboration
- Pricing: Enterprise ($1,500-$3,000/user/year)
- What KIRT does that Clari doesn't: Account-level intelligence generation. Document management and search. Deliverable generation. Works without a clean, structured CRM.
- What Clari does well: Structured CRM data analysis and forecasting. KIRT complements Clari for orgs that have it.

**People.ai**
- Core value: Revenue intelligence from CRM + email/calendar data enrichment
- Key features: Activity capture, contact coverage, account engagement scoring, pipeline hygiene
- Pricing: Enterprise
- What KIRT does that People.ai doesn't: Unstructured document analysis, deliverable generation, SAF workflow, SP integration.
- What People.ai does well: Automated CRM hygiene and activity enrichment. Not a competitor — potential complement if SF data quality improves.

### Category 2: Sales Enablement Platforms

**Seismic**
- Core value: Sales content management and personalization at scale
- Key features: Content library, content analytics, in-context coaching, playbooks, guided selling
- Pricing: Enterprise ($30,000-$100,000+/year)
- What KIRT does that Seismic doesn't: AI-generated account-specific content (not just search/deliver existing content). Unstructured document analysis. Discovery question intelligence. Meeting Summary generation. Foundation-layer data integration.
- What Seismic does well: Content governance, content analytics (what content wins deals), content delivery at scale. KIRT generates where Seismic organizes and distributes.

**Highspot**
- Core value: Sales content effectiveness and readiness platform
- Key features: Content search, pitch management, analytics, training integration
- Pricing: Enterprise ($50,000+/year)
- Similar positioning to Seismic. Strong content analytics and governance. KIRT is additive — generated deliverables could feed into Highspot for distribution.

### Category 3: Account Intelligence / Intent Data

**ZoomInfo**
- Core value: Public B2B data — contact and company data, intent signals, technographics
- Key features: Contact database, buying intent signals, sales automation, website visitor intelligence
- Pricing: $15,000-$50,000+/year
- What KIRT does that ZoomInfo doesn't: Internal document analysis. SP integration. SAF workflow. Deliverable generation. KIRT uses internal knowledge first; public data enriches it.
- Relationship: ZoomInfo enriches KIRT's public data layer — they're complementary, not competing.

**6sense**
- Core value: Account-based marketing and intent data
- Key features: Predictive analytics, intent signals, buying stage identification, orchestration
- Pricing: Enterprise
- KIRT and 6sense serve different buyers (sales vs. marketing). Complementary in an ABM context.

**Demandbase**
- Core value: Account-based experience platform (ABM + sales intelligence)
- Key features: Intent data, account scoring, advertising, sales intelligence
- Similar to 6sense. Not a direct competitor for KIRT's core use case.

### Category 4: AI Meeting Assistants

**Otter.ai / Fireflies.ai / Avoma**
- Core value: Meeting transcription, notes, action items
- Key features: Auto-transcription, meeting summaries, action items, CRM sync
- Pricing: $10-$40/user/month
- What KIRT does that they don't: Account Brief and Engagement Prep generation. Full SAF workflow. SP integration and write-back. Foundation data layer. Works with uploaded notes (not just live recordings).
- What they do well: Real-time transcription during meetings. KIRT's Meeting Summary works from uploaded notes — not real-time. V2+ could add real-time transcription as an optional integration.

### Category 5: Microsoft Copilot for Sales

**Microsoft Copilot for Sales (formerly Viva Sales)**
- Core value: AI-powered CRM updates, email drafting, meeting preparation within M365 context
- Key features: CRM summary in Teams/Outlook, email drafting, meeting preparation, deal room
- Pricing: $50/user/month (requires Microsoft 365)
- **Closest analog** in terms of M365/Entra integration and SharePoint context
- What KIRT does that Copilot doesn't: Strategic Account Framework alignment. Foundation-native data platform (ingestion, normalization, versioned storage). Multi-provider LLM (not just Azure OpenAI). Hybrid semantic + keyword search across all content. Graceful degradation with completeness indicators. Cross-sell pattern matching. Growth scoring. White space grid.
- What Copilot does well: Tight M365 integration, simple CRM updates, email drafting. Lower lift to adopt for organizations already on M365.
- **Key insight:** Copilot is generic. KIRT is purpose-built for strategic account management. The positioning story is "Copilot for anyone, KIRT for account teams who have a methodology."

---

## Differentiation Analysis

| Differentiator | KIRT | Gong | Seismic | ZoomInfo | Copilot |
|---------------|------|------|---------|---------|---------|
| Internal document analysis | ✅ | ❌ | ⚠️ | ❌ | ⚠️ |
| SAF framework alignment | ✅ | ❌ | ❌ | ❌ | ❌ |
| SharePoint write-back | ✅ | ❌ | ❌ | ❌ | ✅ |
| Deliverable generation | ✅ | ❌ | ❌ | ❌ | ⚠️ |
| Graceful degradation | ✅ | ❌ | ❌ | ❌ | ❌ |
| Multi-provider LLM | ✅ | ❌ | ❌ | ❌ | ❌ |
| Hybrid semantic search | ✅ | ❌ | ⚠️ | ❌ | ❌ |
| Public data enrichment | ⚠️ | ❌ | ❌ | ✅ | ❌ |
| Call recording | ❌ | ✅ | ❌ | ❌ | ❌ |
| Content governance | ❌ | ❌ | ✅ | ❌ | ❌ |
| CRM forecasting | ❌ | ✅ | ❌ | ❌ | ⚠️ |

✅ Strong | ⚠️ Partial | ❌ None

---

## Market Positioning Signals

**Primary positioning:** Internal-first account intelligence. "This is your organization's knowledge, structured and searchable." Positions against public data aggregators (ZoomInfo, 6sense) by emphasizing that internal context is richer and more actionable than external intent signals.

**Secondary positioning:** Framework-aligned for EAs, frictionless for everyone else. The same platform serves both the structured SAF user and the AE who just needs to prep for a call. This dual-mode positioning is unique — other tools optimize for one or the other.

**ROI positioning (for leadership audience):** 80%+ reduction in document prep time. 50-70% improvement in account intelligence quality. These are the numbers that justify the build investment and drive adoption in leadership demos.

**Nighthawk replacement positioning (for GTM audience):** KIRT does everything Nighthawk did, plus account-level intelligence, engagement prep, and cross-sell. Zero workflow disruption — the on-demand entry point (Meeting Summary) is even lower friction than Nighthawk.

---

## Name & Namespace Check

**"KIRT":**
- No major commercial product found under this exact name in the enterprise sales/account intelligence space
- Acronym (Knowledge In Real Time) is unique to this project
- No npm/PyPI/GitHub conflicts at product level
- Proceed with KIRT as the product name

**"Knowledge In Real Time":**
- Generic phrase, not trademarked by any identifiable competitor in this space
- Internal branding only for V1 — no external trademark risk

---

## GTM Signals

**Internal first.** V1 is single-tenant, internal Pellera deployment. GTM is internal adoption, not external sales. Success metric: EA and AE adoption within Pellera + demo reliability for external stakeholder presentations.

**Demo-driven GTM.** The demo is the GTM artifact for V1. Non-technical stakeholder watches it and understands the value. This drives exec sponsorship and budget for V2+ scope.

**Two-audience demo structure:**
- Leadership track: Cost-of-sale reduction (80%+ time savings), strategic intelligence quality (50-70% improvement), consistency of account knowledge across the team
- GTM track: "Does this save me time before my meeting tomorrow?" — Meeting Summary, Account Brief, discovery questions in 10 minutes

**Land-and-expand pattern:** On-demand users (Meeting Summary, quick Account Brief) have lowest adoption friction. Once they see value, SAF users follow. Start with Marcus persona (AE), expand to Elena persona (EA), sell to Sarah persona (Sales Director).

---

## Competitive Context for Phase Planning

From competitive analysis, these architectural patterns are confirmed by market evidence:

| Pattern | Market Validation | KIRT Approach |
|---------|-------------------|---------------|
| Hybrid search (keyword + semantic) | Used by Glean, Guru, Notion AI — enterprise knowledge search standard | Phase 2: Foundation hybrid query |
| Few-shot feedback loop | Used in document generation tools (Jasper, Copy.ai) | Phase 3: Store edits as few-shot context |
| Data freshness indicators | Used in analytics dashboards (Tableau, PowerBI) | Phase 2/3: >30 days flag + completeness indicator |
| Multi-provider LLM abstraction | Used by LiteLLM, Langchain, Vercel AI SDK | Phase 3: Custom abstraction layer |
| Document-type search ranking | Used by Confluence, Notion — knowledge hierarchy | Phase 2: Doc type boost in Foundation query |

*Note: Full ECC market-research output will be merged into this file when the background agent completes.*
