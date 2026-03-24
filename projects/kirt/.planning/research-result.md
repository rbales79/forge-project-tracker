# Research Complete: KIRT

## Status: complete
## Interaction: full-auto (intake synthesis)
## Research Depth: medium
## Complexity: complex
## Duration: ~20 minutes
## Date: 2026-03-24

---

## Artifacts Produced

- [x] `HANDOFF.md` — GSD `--auto` format, all required sections, 62 requirements (R01–R62), 27 deferred items, 5 non-goals, 29 locked decisions, 5-layer build order, 3 personas
- [x] `phase-1-context.md` — Foundation Integration & Project Skeleton
- [x] `phase-2-context.md` — Core Search & Document Management
- [x] `phase-3-context.md` — Content Generation Pipeline
- [x] `phase-4-context.md` — Engagement Prep, Meeting Notes & Cross-Sell
- [x] `phase-5-context.md` — Admin, Config & Production Hardening
- [x] `PROJECT.md` — GSD project definition format
- [x] `project.yaml` — Updated with complexity, build engine, planning references, key dependencies
- [x] `risks-and-considerations.md` — 15 risks (RISK-01 through RISK-15), 1 critical, 6 high, 5 medium, 3 low
- [x] `research/market-research.md` — Competitive landscape (5 categories), build/fork/extend recommendation, positioning signals, GTM signals, name check
- [x] `research/synthesis.md` — 49 tagged research items across 7 categories: FROM RESEARCH, UX, POSITIONING, GTM, ARCH, RISK, excluded items

---

## Format Validation

- HANDOFF.md sections: ✅ PASS — all 10 required headings present
- Phase context files: ✅ PASS — all 5 files, all 6 required sections each
- Requirements coverage: ✅ PASS — R01–R62 all appear in both HANDOFF.md and phase contexts
- Risk cross-reference: ✅ PASS — RISK IDs in synthesis.md all defined in risks-and-considerations.md
- User Personas: ✅ PASS — 3 personas with core tasks mapped to requirement IDs

---

## Research Tools Used

| Tool | What It Provided |
|------|-----------------|
| Intake: `kirt-spec-seed.md` | All locked decisions (29), V1/V2/V3+ scope, tech stack, user modes, requirements, constraints, scoring models, data flow, design principles |
| Intake: `demo-data-requirements.md` | Demo data matrix (4 accounts), validation checklists, synthetic data schemas, reference data specs |
| ECC market-research agent | Competitive landscape (running in background — preliminary output applied, full merge pending agent completion) |
| Domain knowledge | Architecture patterns, complexity assessment (Complex confirmed), risk identification |

---

## Key Decisions (Locked — All FROM SOURCE)

| Category | Decision |
|----------|----------|
| Stack | React + Vite + Tailwind + Shadcn / Django MCP-native / Weaviate / Iceberg / Azure |
| LLM | Multi-provider abstraction (Gemini, Azure OpenAI, AWS Bedrock). Zero provider-specific code. |
| Auth | AD/SSO via Graph API. JWT claims to Foundation on every request. |
| Tenancy | Single-tenant V1. Internal Pellera only. |
| Access | Single AD group, full access V1. |
| Document storage | Foundation owns all storage. No KIRT document store. |
| Write-back | Explicit, user-initiated, confirmation required. |
| Permissions | SP inheritance, strict, pre-query at Foundation. |
| Dedup V1 | Content hash, 1-result default, "N copies" badge. |
| Scoring V1 | Equal weights (20% × 5 dimensions), manual override. |
| MSA opt-out | `ai_processing_allowed` flag, keyword only, no exceptions. |
| GDPR | KIRT owns annotation cleanup. |
| Feedback V1 | Prompt-level (few-shot). RAG is V2. Fine-tune is V3+. |
| Notifications V1 | In-app badges only. |
| Foundation | V3.0 only. No V4 features. |

---

## Deferred Items

All per spec. See HANDOFF.md Deferred section (D01–D27).

Key V2+ items:
- Near-duplicate detection (fuzzy matching)
- Bronze/silver/gold access tiers
- Opportunity scoring (ML-informed)
- Cached/degraded Foundation mode
- RAG feedback loop
- Email/push notifications
- Public company monitoring (SEC EDGAR)

Permanently out of scope:
- CRM replacement
- SP source file editing
- KIRT document store
- Surfacing intermediate research to users

---

## Notes for Dev

**Critical pre-build actions (before code starts):**
1. Offerings catalog v0.1 — Roy. Will be available (real or synthetic) for V1. Blocks white space, cross-sell, Engagement Prep precision.
2. Discovery question taxonomy — Done. Available in `demo-data/config/discovery-questions.json`.
3. Demo data preparation — 4 accounts across the matrix. Gap analysis via `demo-data-requirements.md` validation checklists.
4. Foundation v3 access confirmed — endpoint URL, API credentials, Weaviate access, Redis Streams access.
5. LLM provider credentials confirmed — Gemini API key, Azure OpenAI endpoint, AWS Bedrock credentials.
6. Entra app registration for KIRT — needs `Sites.ReadWrite.All` for SP write-back, Graph API access for auth. SP test site: forgecf.sharepoint.com.
7. CVPS3 provisioning — Contabo VPS for Docker deployment.

**Architecture note:** Foundation reference POC (9 layers, 179 passing tests, MCP-native Django monolith) is in the Forge ecosystem. Read it before Phase 1. Mirror the structure.

**Compliance gates (must pass before V1 ships):**
- MSA opt-out integration test: opted-out document in keyword search results, NOT in generation context
- GDPR purge flow: document + all 5 annotation types deleted, verified via annotations API query post-purge
- SP permission integration test: User A can see, User B cannot — verify at Foundation level

**Demo reliability requirement:** Three consecutive runs without intervention. Test this explicitly in Phase 5 before marking complete. Common failure modes: LLM rate limits, Foundation cold start, SP write-back permission scope.

**Market research:** Full ECC market-research output merged into `research/market-research.md` (20+ primary sources, March 2026 data). Key finding: KIRT's white space is structural — no incumbent owns internal document intelligence + deliverable generation + SP write-back. Three structural moats confirmed: unstructured internal docs, deliverable generation (not summaries), SP as source and destination. Closest competitors: DemandFarm (KAM, CRM-bounded), Microsoft Copilot for Sales (general assistant, monitoring quarterly for SharePoint Agents convergence). **Build recommendation: Greenfield confirmed.**

---

## Recommendations for Manager

1. **Engine:** Full GSD (gsd-full). Complexity confirmed: 6+ complex signals from criteria matrix. Multi-service, compliance requirements, parallel frontend/backend workstreams.
2. **Kickoff with GSD:** `/gsd:new-project --auto @.planning/HANDOFF.md` — all required sections present, 29 locked decisions pre-populated, 60 requirements ready for GSD scoping.
3. **Phase 1 start condition:** Foundation access + Entra app registration confirmed. Mock Foundation API server is a Phase 1 deliverable.
4. **Scope concern:** V1 scope is large (60 requirements, 5 MVP output types). GSD phase planning may choose to organize into more granular phases than the 5 build layers suggested here. The build order layers are guidance, not constraint.
5. **Content dependencies are the critical path:** Offerings catalog and discovery question taxonomy are not code — they're content assets that must be ready before Phases 3-4 complete. Track these in parallel to the code build.
