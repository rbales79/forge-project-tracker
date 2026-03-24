# KIRT — Research Synthesis

> Intake synthesis from `kirt-spec-seed.md` and `demo-data-requirements.md`. Research tool: intake (spec-seed documents). Market research tool: ECC market-research (running in parallel). All items tagged for downstream phase consumption.

---

## Requirements Signals `[FROM RESEARCH]`

Items extracted from spec seed that inform must-have requirements. These were confirmed in the spec and carried directly to HANDOFF.md Requirements Signals.

- `[FROM RESEARCH]` Three distinct user modes must be supported from V1: Full SAF, On-Demand Intelligence, Consumer. UI must be task-oriented, not phase-oriented. → R57
- `[FROM RESEARCH]` Graceful degradation is not optional. Every feature must work with partial data and communicate what's missing. This is a differentiating product behavior, not a nice-to-have. → R60, R22
- `[FROM RESEARCH]` Meeting Summary & Follow-Up is the lowest-friction entry point. Upload notes → get something sendable. Single action, immediate value. Priority entry point for on-demand users. → R28, R29, R30
- `[FROM RESEARCH]` Discovery question taxonomy (200+ questions, tree structure) is a content dependency, not just a code feature. The questions may not exist in structured form — creating them is part of the build. → Phase 4 dependency
- `[FROM RESEARCH]` Offerings catalog is the single largest content dependency. It does not exist. Everything depends on it (cross-sell, white space, scoring, question mapping). → RISK-01
- `[FROM RESEARCH]` Public data for public company accounts is gathered at runtime via AI research workflow — not pre-staged files. Demo accounts must be real public companies (or realistic synthetics) so live research can execute. → R19 (Account Brief), demo data design
- `[FROM RESEARCH]` SP write-back uses output folders separate from input source folders. This is the normalization strategy. Input and output SP locations must never mix. → R52
- `[FROM RESEARCH]` Prompt-level feedback learning (store edits as few-shot examples) is V1. RAG feedback loop is V2. Fine-tuning is V3+. This is a clear progression — don't over-engineer V1 feedback. → R39
- `[FROM RESEARCH]` Redis Streams `document.ingested` event is the near-real-time trigger for cross-sell, notification badges, and account intelligence updates. Foundation provides this — KIRT subscribes. → R17, R35

---

## UX Patterns `[FROM RESEARCH: UX]`

- `[FROM RESEARCH: UX]` Data completeness indicator is a first-class UI component, not a tooltip or footnote. Shows: sources used, what's missing, quality % score, specific improvement actions. Reused across all 5 MVP output types. → Phase 3 component, Phase 4 reuse
- `[FROM RESEARCH: UX]` Task-oriented entry UI: "What do you need?" — not "Which SAF step are you on?" Non-SAF users should never feel like they're skipping a step. → Phase 5 UX review
- `[FROM RESEARCH: UX]` SAF progress overlay is off by default. Non-EA users never see it. EA users opt in. The overlay is informational only — no workflow gates in V1. → R44
- `[FROM RESEARCH: UX]` Search result ranking as knowledge hierarchy (account dossier → meeting notes → SF → public → playbooks) is a UX decision, not just a technical one. Users expect more relevant results first. Implement as document-type weighting in Foundation query. → R10
- `[FROM RESEARCH: UX]` Generation progress: show "Still working — pulling data from N sources" if >60 seconds. Users will abandon if they see just a spinner. The N sources shown builds trust in the depth of research. → R37
- `[FROM RESEARCH: UX]` "N copies found" badge: visible on result card, expandable. The default experience hides the complexity; power users can drill in. Dedup is invisible to the standard user flow. → R08, R09
- `[FROM RESEARCH: UX]` Inline editing + section regeneration must feel immediate. If a user edits a section and hits "regenerate", the result should replace the section in-place — not redirect to a new page or start a full document regeneration.
- `[FROM RESEARCH: UX]` White space grid as visual anchor: the offerings × engagement status matrix is a high-value visual for both Account Brief and standalone. Rich-data accounts (Account A) show populated grids. Thin-data accounts (Account D) show mostly empty with completeness indicator.
- `[FROM RESEARCH: UX]` Mobile: search and read flows only. Generation and admin are desktop. Don't try to make generation work on mobile — it's a deliberate scope decision, not a gap.

---

## Positioning `[FROM RESEARCH: POSITIONING]`

- `[FROM RESEARCH: POSITIONING]` KIRT's positioning: "turns the organization's own knowledge into structured account intelligence." Not a public data aggregator (ZoomInfo), not a meeting recorder (Gong), not a generic AI assistant (Copilot). Internal knowledge + public enrichment.
- `[FROM RESEARCH: POSITIONING]` The Nighthawk replacement angle: on-demand users (AEs, individual contributors) are currently using KIRT's predecessor (Nighthawk). KIRT must match or exceed Nighthawk for this use case while adding the SAF layer for EA users. Nighthawk replacement is table stakes, not a differentiator.
- `[FROM RESEARCH: POSITIONING]` Graceful degradation with transparency is a competitive differentiator. Competitors generally fail silently or require complete setup. KIRT tells the user exactly what's missing.
- `[FROM RESEARCH: POSITIONING]` "Baby, not the baby-making" — research, scoring, and intermediate analysis live in the data lake. Final deliverables write to SP. End users see the deliverable in SP and the full context in KIRT. This clean separation is a trust-building UX principle.
- `[FROM RESEARCH: POSITIONING]` Multi-provider LLM from day one differentiates from tools locked to a single provider (OpenAI, Azure). Pellera can use approved vendors without changing KIRT code.

---

## GTM Signals `[FROM RESEARCH: GTM]`

- `[FROM RESEARCH: GTM]` Primary demo audience is internal: leadership (cost-of-sale reduction, strategic alignment) + go-to-market teams (meeting prep, time savings). Two emphasis modes in one demo.
- `[FROM RESEARCH: GTM]` GTM metric targets: 80%+ reduction in document prep time, 50-70% improvement in account intelligence quality. These are the ROI numbers for the leadership audience.
- `[FROM RESEARCH: GTM]` Demo reliability is a GTM requirement, not just a quality requirement. Leadership demos that fail to run are remembered. Three consecutive reliable runs is the bar.
- `[FROM RESEARCH: GTM]` The demo must work without explanation for non-technical stakeholders. "If you have to explain what just happened, the demo failed."
- `[FROM RESEARCH: GTM]` Foundation-native positioning (vs. public data aggregators) resonates for enterprise: "This is your organization's intelligence, not someone else's data."

---

## Architecture Signals `[FROM RESEARCH: ARCH]`

- `[FROM RESEARCH: ARCH]` MCP-native architecture from Phase 1. Mirror Foundation reference POC (9 layers, 179 tests, monolith design). Not a microservices architecture — monolith with clear module boundaries.
- `[FROM RESEARCH: ARCH]` LLM abstraction layer is architecturally critical. Zero provider-specific code in application logic. All provider differences encapsulated in the abstraction. This enables admin-configurable provider switching without code deployment.
- `[FROM RESEARCH: ARCH]` Mock Foundation API server is a Phase 1 deliverable, not a nice-to-have. It enables full unit test coverage without live infrastructure. Mock must return realistic response shapes — empty stubs cause false-positive test passes.
- `[FROM RESEARCH: ARCH]` Permission enforcement is pre-query at Foundation. KIRT trusts Foundation's response. Post-filtering in KIRT is an anti-pattern — it introduces a gap between what Foundation returns and what KIRT shows.
- `[FROM RESEARCH: ARCH]` All admin-configurable settings stored in database, surfaced in admin UI. No configuration constants in code. This is a deployment flexibility requirement, not just a UX preference.
- `[FROM RESEARCH: ARCH]` Foundation `derived_docs[]` field links generated deliverables to their source research records. KIRT must update this on every SP write-back. This is how provenance is tracked without building a separate provenance system.
- `[FROM RESEARCH: ARCH]` Redis Streams subscription should be registered in Phase 1 (as a stub/handler) so the event pipeline is wired up before Phase 2 classification and Phase 4 cross-sell depend on it.
- `[FROM RESEARCH: ARCH]` Discovery question taxonomy: in-memory lookup on small dataset (200+ questions). No separate database or vector store needed. Load at startup from structured YAML/JSON config.

---

## Risk Items `[FROM RESEARCH: RISK]`

- `[FROM RESEARCH: RISK]` Offerings catalog: CRITICAL. Does not exist. → RISK-01
- `[FROM RESEARCH: RISK]` Foundation unavailability during demo: HIGH. Error state only in V1. → RISK-02
- `[FROM RESEARCH: RISK]` Discovery question taxonomy as informal notes: HIGH. Must formalize. → RISK-03
- `[FROM RESEARCH: RISK]` MSA opt-out enforcement gap: HIGH. Legal liability. → RISK-06
- `[FROM RESEARCH: RISK]` GDPR annotation cascade: HIGH. Compliance gap. → RISK-07
- `[FROM RESEARCH: RISK]` SP permission enforcement gap: HIGH. Security. → RISK-05
- `[FROM RESEARCH: RISK]` LLM rate limits during demo: MEDIUM. Operational mitigation needed. → RISK-11

---

## Items Excluded from Requirements (With Rationale)

The following items from the source material were considered but excluded from V1 must-have requirements:

| Item | Disposition | Rationale |
|------|-------------|-----------|
| Teams tab wrapper | Deferred (D25) | Future if org pressure requires. Responsive web covers mobile access. |
| Transcript speaker diarization | Deferred (D15, V2) | Meeting notes upload works V1. Full transcript analysis is more complex. |
| Near-duplicate detection (fuzzy) | Deferred (D01, V2) | Exact dedup (content hash) is V1. Fuzzy matching is V2+. |
| SF direct API | Deferred (D22, V4+) | CSV exports sufficient for V1 demo. Direct API requires additional auth setup. |
| Email/push notifications | Deferred (D07, V2) | In-app badges sufficient for V1. |
| ML-informed scoring | Deferred (V2+) | Equal-weight formula + manual override is V1. ML needs data volume to justify. |
| WCAG compliance | Post-V1 | Important but not a demo blocker. |
| Multi-tenancy | Deferred (D21, V4+) | Foundation `tenant_id` supports it when business requires. |
