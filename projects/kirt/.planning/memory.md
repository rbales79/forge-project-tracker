# KIRT — Session Memory

> Intake synthesis session — 2026-03-24. All decisions from `kirt-spec-seed.md`. No interactive Q&A.

---

## Session State

- **Mode:** Intake Synthesis (auto)
- **Date:** 2026-03-24
- **Status:** Artifacts complete, validation in progress
- **Source documents:** `kirt-spec-seed.md`, `demo-data-requirements.md`

---

## Active Decisions

All decisions are `[FROM SOURCE]` — locked from spec seed. None deferred to user for confirmation.

| Decision | Value | Source |
|----------|-------|--------|
| Frontend stack | React + Vite + Tailwind + Shadcn | spec-seed locked |
| Backend stack | Django, MCP-native | spec-seed locked |
| Vector DB | Weaviate | spec-seed locked |
| Storage | Apache Iceberg / Parquet on Azure Blob | spec-seed locked |
| Auth | AD/SSO via Graph API | spec-seed locked |
| Cloud target | Azure (target), Contabo (current) | spec-seed locked |
| LLM providers | Gemini, Azure OpenAI, AWS Bedrock + OpenAI-compatible | spec-seed locked |
| LLM architecture | Multi-provider abstraction, no provider-specific code | spec-seed locked |
| Form factor | Web app, responsive | spec-seed locked |
| Tenancy | Single-tenant V1 | spec-seed locked |
| Access control V1 | Single AD group, full access | spec-seed locked |
| Document storage | Foundation owns all storage | spec-seed locked |
| SP write policy | Explicit, user-initiated, confirmation required | spec-seed locked |
| Permission model | SP inheritance, strict, pre-query at Foundation | spec-seed locked |
| Dedup V1 | Content hash exact dedup, 1-result default | spec-seed locked |
| Canonical selection | Admin-configurable location-ranking | spec-seed locked |
| Scoring V1 | Equal weights (20% each, 5 dimensions), manual override | spec-seed locked |
| MSA opt-out | ai_processing_allowed flag, keyword only, no exceptions | spec-seed locked |
| GDPR purge | KIRT owns annotation cleanup, Foundation cascade insufficient | spec-seed locked |
| Feedback V1 | Prompt-level (store edits as few-shot context) | spec-seed locked |
| Concurrent generation | Both complete, both write SP, user picks winner | spec-seed locked |
| Error state | Show error when Foundation unavailable (no degraded mode V1) | spec-seed locked |
| Offerings catalog | v0.1 real or synthetic, will be available for V1 | spec-seed locked |
| Discovery questions | 200+ questions, tree structure, available in demo-data/config/discovery-questions.json | spec-seed locked |
| Demo data | 4 accounts (anonymized), public data at runtime | spec-seed locked |
| Notifications V1 | In-app badges only | spec-seed locked |
| Foundation version | V3.0 only | spec-seed locked |
| Complexity | Complex → Full GSD | derived from signals |

---

## Completed Phases

| Phase | Status | Date |
|-------|--------|------|
| Phase 0: Project Setup | Complete | 2026-03-24 |
| Phase 1: Concept Capture | Complete (via intake synthesis) | 2026-03-24 |
| Phase 2: Market Research | Complete (ECC agent running, web synthesis applied) | 2026-03-24 |
| Phase 3: Domain Selection & Q&A | Complete (auto-synthesized from spec seed) | 2026-03-24 |
| Phase 4: Risk Review | Complete | 2026-03-24 |
| Phase 5: Artifact Generation | Complete | 2026-03-24 |
| Phase 6: Validation | In progress | 2026-03-24 |

---

## Research Sources

| Tool | What It Provided |
|------|-----------------|
| Intake: `kirt-spec-seed.md` | All locked decisions, V1/V2/V3 scope, tech stack, user modes, requirements, constraints, scoring models, data flow, key design decisions |
| Intake: `demo-data-requirements.md` | Demo data matrix (4 accounts), validation checklists, synthetic data schemas, reference data specs, directory structure |
| ECC market-research | Competitive landscape, positioning signals (running in background — output to market-research.md) |
| Domain knowledge | Architecture patterns, complexity assessment, risk identification |

---

## Prior Art (in Forge Ecosystem)

- **Foundation reference POC:** MCP-native Django monolith, 9 layers, 179 passing tests. KIRT mirrors this architecture. Available as direct reference for Phase 1 project setup.
- **KIRT spec corpus:** `~/repos/KIRT/` — locked decisions, architecture docs. Read-only spec reference. Architecture docs and spec seed live in `forge-project-tracker/projects/kirt/`.

---

## Deferred State

Session is not paused. Artifacts complete.

Next action: **Return to Manager** → Manager creates project repo and initiates `/gsd:new-project --auto @.planning/HANDOFF.md`

---

## Assumptions Made During Synthesis

| Assumption | Basis | Risk if Wrong |
|------------|-------|---------------|
| [ASSUMPTION] Build team has Django expertise (MCP-native pattern) | Foundation POC as prior art | Phase 1 slower if pattern is unfamiliar |
| [ASSUMPTION] Foundation v3 API is stable and won't have breaking changes before KIRT build | Spec says v3 is running and operational | Integration tests would fail |
| [ASSUMPTION] KIRT builds its own multi-provider LLM abstraction layer (not LiteLLM) | Spec seed defines OpenAI-compatible abstraction, Roy explicitly rejected LiteLLM for KIRT | Direct provider SDKs required for Gemini, Azure OpenAI, Bedrock |
| [ASSUMPTION] 5-phase build order is achievable as a sequential GSD milestone structure | Spec V1 scope | GSD may reorganize phases differently |
| [ASSUMPTION] Complexity is `Complex` (not `Moderate`) | 6+ complex signals from criteria matrix | GSD may choose different phase depth |
