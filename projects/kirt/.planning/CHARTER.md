# KIRT — CHARTER.md

> Project charter defining non-goals, success metrics, stakeholders, and timeline. Complements PROJECT.md (what it is) and HANDOFF.md (how to build it).

---

## Project Name

**KIRT** — Knowledge In Real Time

## Mission Statement

Build the AI-powered intelligence layer that turns account research, engagement preparation, and deliverable generation from hours of manual work into minutes of AI-assisted generation — working with whatever data exists and clearly communicating what's missing.

---

## Non-Goals

These are things KIRT explicitly does NOT do. If a feature request conflicts with these, push back and reference this charter.

1. **KIRT does not replace Salesforce.** SF stays the system of record for structured CRM data (accounts, pipeline, contacts, forecasting, workflow automation). KIRT handles unstructured analysis, semantic search, AI generation, and pattern intelligence. They complement each other.

2. **KIRT does not store documents.** Foundation is the single document storage layer. Users upload through KIRT, which routes to Foundation via `POST /v1/ingest/`. KIRT stores intelligence (enrichments, classifications, scores, generated insights) — not files.

3. **KIRT does not require SAF adherence.** The SAF's 6 phases organize KIRT's capabilities, but every feature is independently accessible. Users can generate an Engagement Prep without completing Account Intelligence first. The UI is task-oriented, not phase-oriented.

4. **KIRT does not build multi-tenant for V1.** Single-tenant, single AD group, internal Pellera deployment only. Foundation's `tenant_id` scoping means multi-tenancy can be added later without a rewrite, but it is gated by security and legal review.

5. **KIRT does not generate BSE/SOW.** Reframe decks, assessment kickoffs, transformation roadmaps, and EBR/QBR packages are V3+ capabilities. V1 generates Account Briefs, Engagement Prep, Meeting Summaries, Client Deliverables, and Cross-sell Recommendations only.

6. **KIRT does not provide production-scale deployment.** V1 is demo-ready, not enterprise-scale. No SLAs, no multi-region, no Azure migration, no WCAG compliance. Performance targets are soft with progress indicators. Demo reliability (3 consecutive runs) is the bar.

7. **KIRT does not modify source SP files.** KIRT writes deliverables to configured output folders. It never edits source files in their original SP locations. Generated output files are updated in place on regeneration — SP versioning preserves history. All writes require explicit user confirmation.

8. **KIRT does not provide a native mobile app.** Responsive web design via Tailwind handles mobile read flows (search, view briefs, view engagement prep). Generation and admin flows are desktop-only in V1. No iOS/Android app, no Teams tab.

9. **KIRT does not implement ML-based scoring for V1.** Growth scoring uses a simple equal-weight composite with manual override. ML-informed scoring is V2+ after data volume justifies it.

10. **KIRT does not implement cached/degraded mode for V1.** When Foundation is unavailable, KIRT shows an error state. No local cache, no offline mode, no degraded operation. Ensure Foundation stability during demo windows rather than engineering around it.

---

## Success Metrics

| Metric | Target | Measurement Method | Notes |
|--------|--------|--------------------|-------|
| Account Brief generation time | <60 seconds | Timer from user request to fully rendered output | Soft target; hard limit <5 minutes. Progress indicator shows after 60s. |
| Engagement Prep generation time | <90 seconds | Timer from user request to fully rendered output | Soft target; hard limit <5 minutes. |
| Search latency | <3 seconds | API response time for hybrid search query | Hard limit <10 seconds. |
| Page load time | <2 seconds | Time to interactive for main views | Hard limit <5 seconds. |
| Document classification time | <30 seconds | Time from upload to classification annotation written | Hard limit <2 minutes. |
| Demo reliability | 3 consecutive successful runs | End-to-end smoke test: login → search → generate → edit → write to SP | Must pass 3/3 attempts without failure. |
| Data completeness indicator accuracy | Shows correct source count | Manual verification against known test data for demo accounts | Verify each source type (SF, SP, meeting notes, public, stakeholder, catalog) correctly reported. |
| Manual effort reduction | 80%+ for prep tasks | Before/after time comparison: manual Account Brief (2-4 hrs) vs. KIRT (<1 min), manual Engagement Prep (1-2 hrs) vs. KIRT (<2 min) | Based on SAF framework manual effort baselines from spec seed. |
| Account intelligence quality | 50-70% improvement | Qualitative assessment by EA team comparing KIRT-generated briefs to manual equivalents | Subjective metric — measured post-demo via stakeholder feedback. |
| Feature completeness | All V1 delivers | Success Criteria checklist in HANDOFF.md (25+ items) | All checkboxes must be checked before V1 is declared done. |

---

## Stakeholders

| Who | Role | Responsibility | Engagement |
|-----|------|---------------|------------|
| **Roy Bales** | Owner / Technical Lead | Architecture, technical build, Foundation integration, demo execution | Daily — hands-on build |
| **Carlos Marques** | Executive Sponsor | SAF framework owner, organizational alignment, executive buy-in, go/no-go authority | Milestone reviews — after each build layer |
| **Dr. Jonathan Gough** | Co-Architect | AI systems design, governance framework, LLM strategy, generation quality | As-needed — design reviews, AI pipeline decisions |
| **Caleb Crisci** | Content Lead | Deliverable templates, discovery question taxonomy, offerings catalog v0.1 content | As-needed — content delivery, template review |
| **EA Team** | Primary Users (Full SAF) | Account intelligence, deliverable generation, SAF progress tracking, feedback | Demo + UAT — validate Full SAF mode |
| **AEs / Solution Specialists** | Target Users (On-Demand) | Meeting prep, note upload, quick search, client deliverables | Demo + UAT — validate On-Demand mode |
| **Sales Leadership** | Target Users (Consumer) | Brief consumption, cross-sell review, pipeline visibility | Demo — validate Consumer mode value |

### RACI Quick Reference

| Activity | Roy | Carlos | Jonathan | Caleb | EA Team |
|----------|-----|--------|----------|-------|---------|
| Architecture decisions | A/R | I | C | — | — |
| AI/LLM pipeline design | A | I | R | — | — |
| Build execution | R | I | C | — | — |
| Offerings catalog content | C | I | C | R | C |
| Template content | C | — | — | R | C |
| Demo delivery | R | A | C | — | C |
| Go/no-go decision | C | A | C | — | I |

*A = Accountable, R = Responsible, C = Consulted, I = Informed*

---

## Timeline

### Prerequisites (Before Build Starts)

- [ ] Demo data package available (4 accounts per `demo-data-requirements.md`)
- [ ] Offerings catalog v0.1 available (real or synthetic — degrades gracefully without it)
- [ ] Foundation v3.0 confirmed stable and accessible for integration testing
- [ ] AD/SSO test environment available
- [ ] SP test site with read/write permissions confirmed (forgecf.sharepoint.com)

### Build Phases

| Phase | Layers | Focus | Review Gate |
|-------|--------|-------|-------------|
| **Foundation** | 1-2 | Scaffold, auth, Foundation integration | Authenticated app shell, health check, Foundation API client working |
| **Core Experience** | 3-4 | Search, upload, classification | User can search, upload docs, see classifications |
| **Generation** | 5-6 | Engine + all 5 Day-1 outputs | All 5 outputs generating with completeness indicators |
| **Configuration** | 7 | Admin settings, source management | Admin can configure sources, outputs, scoring, LLM provider |
| **Polish** | 8 | Notifications, mobile, observability | Smoke test passes 3/3, error tracking live |

### Milestones

- **M1: App Shell** — Authenticated React+Django app with health endpoint and Foundation connectivity
- **M2: Search Works** — Hybrid search returns results from Foundation with dedup, freshness, and ranking
- **M3: First Generation** — Account Brief generates end-to-end with completeness indicators
- **M4: All Outputs** — All 5 Day-1 outputs generating, editing, and writing to SP
- **M5: Demo Ready** — Admin configured, notifications working, smoke test passing 3/3

### Review Cadence

- After each build layer completion: technical review
- After each milestone: stakeholder review with Carlos
- Demo readiness: full end-to-end walkthrough with EA team before external demo

---

## Dependencies

| Dependency | Owner | Status | Impact if Missing |
|-----------|-------|--------|-------------------|
| Foundation v3.0 (live, accessible) | Foundation team | Deployed on Contabo | Blocker — no Foundation, no KIRT |
| Demo data package (4 accounts) | Roy / Caleb | Not started | Blocks integration testing and demo |
| Offerings catalog v0.1 | Roy | In progress | Will be available (real or synthetic) for V1 — not a blocker |
| Discovery question taxonomy | — | Available | Structured taxonomy in `demo-data/config/discovery-questions.json` |
| Deliverable templates (4 types) | Caleb | Not started | Blocks template-based generation |
| AD/SSO test environment | Roy | TBD | Blocks auth layer |
| SP test site (read + write) | Roy | Available (forgecf.sharepoint.com) | Entra app registration when needed |
| Graph API permissions confirmed | Roy | TBD | Blocks SP integration |

---

## Budget & Resources

V1 is a technical proof-of-value, not a funded program. Resources:

- **Compute:** Foundation v3 on Contabo (existing), KIRT on Contabo CVPS3 (Docker), CI/CD via GitHub Actions
- **LLM costs:** Multi-provider (Gemini, Azure OpenAI, Bedrock) — development and demo usage
- **Time:** Roy (primary build)
- **No additional headcount** for V1 — the build validates the concept for executive buy-in

---

## References

- **Spec Seed:** `kirt-spec-seed.md`
- **Demo Data:** `demo-data-requirements.md`
- **HANDOFF:** `.planning/HANDOFF.md`
- **PROJECT:** `.planning/PROJECT.md`
- **Risks:** `.planning/risks-and-considerations.md`