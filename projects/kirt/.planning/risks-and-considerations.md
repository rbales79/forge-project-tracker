# KIRT — Risks & Considerations

> Generated via intake synthesis from `kirt-spec-seed.md`. All risks validated against spec content. Decisions marked `[FROM SOURCE]` were pre-resolved in the spec.

---

## Critical Risks

### RISK-01: Offerings Catalog Does Not Exist
- **Category:** Dependency
- **Severity:** CRITICAL
- **Description:** The offerings catalog v0.1 is the single largest content dependency. White space grid, growth scoring, cross-sell recommendations, and Engagement Prep discovery question mapping all depend on it. It does not exist today — it must be extracted from PPT decks and practice collateral.
- **Evidence:** Spec explicitly states: "The catalog does not exist yet — this is the single biggest content dependency."
- **Mitigation:** Best-effort v0.1 extraction from PPT decks and practice collateral before demo. Assign ownership immediately (Jonathan action item per spec). Features operate in graceful degradation mode without it — completeness indicator flags "offerings catalog not configured."
- **Blast radius:** Without catalog: white space grid empty, cross-sell imprecise, Engagement Prep question mapping untargeted.
- **Reversibility:** Can be added post-build without code changes (admin config upload).
- **Decision:** `[FROM SOURCE]` Extract v0.1 from existing materials. Don't defer with a placeholder. Don't build from scratch.

---

## High Risks

### RISK-02: Foundation Availability for Development & Testing
- **Category:** Dependency / Infrastructure
- **Severity:** HIGH
- **Description:** Foundation v3.0 is the only supported data platform. If Foundation is down during a demo, KIRT shows an error state (no degraded mode until V2). Development and integration testing require Foundation to be live.
- **Evidence:** "Foundation is running. v3.0 is deployed and operational on Contabo. No seed data exists yet."
- **Mitigation:** Mock Foundation API server for unit test coverage (build in Phase 1). Integration tests against live Foundation v3. Ensure Foundation stability during demo windows rather than engineering around it (V1 decision).
- **Blast radius:** Demo failure if Foundation is unavailable during live demo.
- **Reversibility:** Cached/degraded mode is V2+ — deferred intentionally.
- **Decision:** `[FROM SOURCE]` No degraded mode in V1. Error state only.

### RISK-03: Discovery Question Taxonomy as Informal Notes
- **Category:** Dependency / Content
- **Severity:** HIGH
- **Description:** The 200+ question set exists across 20+ variants, possibly as informal notes rather than a structured taxonomy document. If no formalized file exists, one must be created during Phase 2/4.
- **Evidence:** Spec states: "Questions may exist as informal notes rather than a structured document — if no formalized taxonomy file exists, one will need to be created during build."
- **Mitigation:** Formalize during Phase 2 (content deliverable, not code). Tree structure: business → industry → solution. Tag each question with module, applicable industries, related offerings. Block Engagement Prep integration tests until taxonomy is structured.
- **Blast radius:** Engagement Prep loses question filtering precision until taxonomy is formalized. Feature still ships but with lower quality.
- **Reversibility:** Taxonomy can be refined iteratively after V1.

### RISK-04: LLM Generation Quality Below 80-90% Threshold
- **Category:** Technical / Quality
- **Severity:** HIGH
- **Description:** The spec targets "80-90% quality" for generated deliverables. Achieving this requires well-crafted prompt templates, few-shot examples, and sufficient context from Foundation. If generation quality is poor, users won't adopt the features.
- **Evidence:** "Drafts all formats at 80-90% quality; human refines via inline editing and section regeneration."
- **Mitigation:** Prompt-level feedback learning (store edits as few-shot context). Data completeness indicators surface quality confidence to users. Human-in-the-loop editing for refinement. Graceful degradation messaging sets expectations.
- **Blast radius:** If generation is poor, adoption fails even if the platform is technically complete.
- **Reversibility:** Prompt quality can be improved post-launch without code changes.

### RISK-05: SP Permission Enforcement Gaps
- **Category:** Security / Compliance
- **Severity:** HIGH
- **Description:** KIRT must never surface a document a user couldn't access in SharePoint. This enforcement happens at Foundation pre-query time via JWT claims. Any gap means KIRT exposes unauthorized content.
- **Evidence:** "Permission inheritance is strict. KIRT never surfaces a document the user couldn't access in its source system. Enforced pre-query in Foundation, not post-fetch in KIRT."
- **Mitigation:** KIRT never post-filters — it passes JWT and trusts Foundation's response. Explicit integration test: create a document accessible to User A but not User B; verify B's KIRT search returns no result. This test must pass before V1 ships.
- **Blast radius:** Data exposure. Critical compliance failure.
- **Reversibility:** Architectural fix required if post-filtering is discovered — high cost.

### RISK-06: MSA Opt-Out Enforcement Gap
- **Category:** Compliance / Legal
- **Severity:** HIGH
- **Description:** MSA opt-out means `ai_processing_allowed: false` → keyword search only, no LLM processing. If any LLM pipeline entry point misses the flag check, opted-out content gets AI-processed in violation of the opt-out agreement.
- **Evidence:** "Opt-out means keyword search only — documents are ingested and indexed for BM25 keyword/phrase search, but no LLM processing (no summarization, no AI classification, no generation using that content)."
- **Mitigation:** Explicit audit of every LLM pipeline entry point during Phase 5. Named invariant in codebase. Integration test with opt-out flagged document: verify it appears in keyword search, does not appear in generation context, is not AI-classified.
- **Blast radius:** Breach of client agreement. Legal and contractual liability.
- **Reversibility:** Code fix, but the violation may have already occurred.

### RISK-07: GDPR Annotation Cascade Not Triggered
- **Category:** Compliance / Legal
- **Severity:** HIGH
- **Description:** Foundation's GDPR purge cascade covers documents and chunk records only. It does not cascade to annotations. KIRT owns annotations (classifications, scores, feedback pairs, stakeholder map, generation provenance). If KIRT doesn't explicitly delete its annotations, orphaned data remains after GDPR purge.
- **Evidence:** "Annotation cleanup is KIRT's responsibility. Foundation's purge cascade only covers documents and chunk records — it does not cascade to annotations."
- **Mitigation:** KIRT purge workflow explicitly enumerates and deletes all annotation types for the purged document. Verification step: query annotations API after purge to confirm zero orphaned records. Audit log entry written per purge.
- **Blast radius:** GDPR non-compliance. Regulartory risk.
- **Reversibility:** Annotations can be retroactively deleted if the gap is caught, but the violation may have already occurred.

---

## Medium Risks

### RISK-08: Demo Data Extraction Quality
- **Category:** Content / Dependency
- **Severity:** MEDIUM
- **Description:** The 4-account demo matrix requires realistic data across SF (CSV exports) and SP (documents). Real-world extraction may produce gaps or inconsistencies.
- **Evidence:** "Real-world extraction will produce some messiness organically. Clean-first plan — don't intentionally introduce bad data."
- **Mitigation:** Use the gap decision matrix from `demo-data-requirements.md`. For critical gaps (no SP documents for an account, missing entire SF object): synthesize. For minor gaps (missing SF field): synthesize the field. Account D is intentionally thin — other accounts should aim for richness.
- **Blast radius:** Demo scenarios don't cover the full matrix without complete data across 4 accounts.
- **Reversibility:** Data can be enriched iteratively before demo date.

### RISK-09: Concurrent Generation Conflicts
- **Category:** Technical / UX
- **Severity:** MEDIUM
- **Description:** Two users generating the same account's brief simultaneously both write to SP. The resulting "2 versions generated" UX is functional but may confuse users.
- **Evidence:** "If two users generate deliverables for the same account simultaneously, both complete."
- **Mitigation:** Clearly surface both versions with timestamps and generating user. User picks winner. Content merging is V3+.
- **Blast radius:** User confusion. SP has duplicate generated files for the same account.
- **Reversibility:** V3 content merge resolves. Low urgency.

### RISK-10: Weaviate Cross-Reference Traversal Complexity
- **Category:** Technical
- **Severity:** MEDIUM
- **Description:** Relationship traversal queries (e.g., "all MSAs related to this client") rely on Weaviate cross-references. Complex graph traversals may have performance implications.
- **Evidence:** Foundation uses Weaviate HNSW + hybrid search + relationship traversal.
- **Mitigation:** V1 limits traversal to basic 1-hop cross-references. Complex multi-hop graph queries deferred to V2+. Profile search response times against Foundation during Phase 2.
- **Blast radius:** Search performance degradation if complex traversals are added without profiling.
- **Reversibility:** Can limit query depth without breaking changes.

### RISK-11: LLM Provider Rate Limits During Demo
- **Category:** Operational / Infrastructure
- **Severity:** MEDIUM
- **Description:** Running 3 consecutive demos with live LLM generation may hit provider rate limits, causing generation failures or latency spikes.
- **Evidence:** V1 soft target: Account Brief <60 seconds, Engagement Prep <90 seconds.
- **Mitigation:** Increase provider quotas before demo. Multi-provider fallback in the abstraction layer. Consider generation result caching during demo window (serve same result for identical inputs within 1 hour). Monitor provider health via admin config display.
- **Blast radius:** Demo failure if LLM is rate-limited during live presentation.
- **Reversibility:** Quota increase is fast; caching layer is more involved.

### RISK-12: Admin Configuration Surface Coverage
- **Category:** Scope / UX
- **Severity:** MEDIUM
- **Description:** The spec requires all configurable parameters (SP paths, scoring weights, LLM provider, offerings catalog) to be tunable without code changes. If any parameter is hardcoded, it violates this constraint.
- **Evidence:** "Admin-configurable deployment settings. Directory structures, SP output paths, scoring model weights, offerings catalog, LLM provider selection, and data source configurations must be tunable per deployment without code changes."
- **Mitigation:** Explicit audit in Phase 5 of all configurable parameters against the admin UI. No configuration constant in code — all settings stored in database, surfaced in admin UI.
- **Blast radius:** Deployment inflexibility. Can't adapt to different SP structures without code changes.
- **Reversibility:** Configuration refactor can be done post-launch but requires database migration.

---

## Low Risks

### RISK-13: SP Write-Back Permission Scope
- **Category:** Technical / Security
- **Severity:** LOW
- **Description:** KIRT's Entra app registration needs `Sites.ReadWrite.All` permission for SP write-back to output folders. Over-broad permission scope is a security concern.
- **Mitigation:** Scope permission to designated output sites only if Microsoft Graph supports site-specific write scopes. Document permission model in CLAUDE.md. Review during security assessment.
- **Reversibility:** Permission scope can be narrowed post-launch.

### RISK-14: Foundation v4 Feature Creep
- **Category:** Scope
- **Severity:** LOW
- **Description:** Foundation v4 capabilities (automated reconciliation, Spark, advanced lineage, OpenMetadata) are useful for V2+ KIRT features. Risk of designing KIRT around V4 before it's available.
- **Mitigation:** `[FROM SOURCE]` Build KIRT V1 on Foundation v3 only. Any V4 dependency in V1 code is a scope violation.
- **Reversibility:** V4 dependencies can be removed, but causes rework.

### RISK-15: Deployment Drift (Contabo → Azure)
- **Category:** Infrastructure
- **Severity:** LOW
- **Description:** Foundation is on Contabo; KIRT targets Azure. Configuration differences between environments may cause integration issues.
- **Mitigation:** All Foundation endpoint URLs and credentials via environment config only. No hardcoded Contabo URLs. Azure migration is post-V1; design for it, don't block on it.
- **Reversibility:** Environment-based config makes migration straightforward.

---

## Risk Summary Table

| Risk | Severity | Category | Mitigation Status |
|------|----------|----------|-------------------|
| RISK-01: Offerings catalog missing | CRITICAL | Dependency | Extraction underway (WIP) |
| RISK-02: Foundation unavailable | HIGH | Infrastructure | Mock API in Phase 1 |
| RISK-03: Taxonomy as informal notes | HIGH | Content | Formalize in Phase 2 |
| RISK-04: Generation quality below threshold | HIGH | Technical | Prompt feedback loop in Phase 3 |
| RISK-05: SP permission enforcement gap | HIGH | Security | Integration test required |
| RISK-06: MSA opt-out gap | HIGH | Compliance | Phase 5 audit + integration test |
| RISK-07: GDPR annotation cascade | HIGH | Compliance | Phase 5 purge workflow |
| RISK-08: Demo data extraction quality | MEDIUM | Content | Gap matrix in demo-data-requirements.md |
| RISK-09: Concurrent generation conflicts | MEDIUM | UX | "2 versions" UX in Phase 3 |
| RISK-10: Weaviate traversal complexity | MEDIUM | Technical | 1-hop limit in V1 |
| RISK-11: LLM rate limits during demo | MEDIUM | Operational | Quota increase + caching |
| RISK-12: Admin config coverage | MEDIUM | Scope | Phase 5 audit |
| RISK-13: SP write-back permission scope | LOW | Security | Review during build |
| RISK-14: Foundation v4 feature creep | LOW | Scope | V3-only constraint enforced |
| RISK-15: Contabo → Azure drift | LOW | Infrastructure | Environment config only |
| RISK-16: Adoption beyond EA team | LOW (V1) | Organizational | Three user modes designed from V1 |
| RISK-17: Template ground truth collection stalls | MEDIUM | Organizational | Escalation path through Carlos |
| RISK-18: Executive alignment not yet obtained | MEDIUM | Organizational | Demo scoped as "demo-ready MVP" |
| RISK-19: Content owner bandwidth | MEDIUM | Organizational | All dependencies have degradation paths |

---

## Organizational Risks

### RISK-16: Adoption Beyond EA Team
- **Category:** Organizational
- **Severity:** LOW (V1) — actual adoption risk is post-V1
- **Description:** V1 is an internal demo. The demo must be convincing to both EA-audience (Full SAF value) and AE/GTM audience (On-Demand value) simultaneously. If the demo audience doesn't see themselves in the product, leadership approves but GTM team doesn't adopt post-V1.
- **Evidence:** Three user modes (Full SAF, On-Demand, Consumer) designed into V1 architecture.
- **Mitigation:** Meeting Summary & Follow-Up is the lowest-friction entry point for on-demand users (upload notes → get sendable email). Task-oriented UI ("What do you need?") speaks to all personas, not just SAF-fluent EAs. Demo has two emphasis modes: leadership (cost-of-sale reduction) and GTM (time savings before tomorrow's meeting).
- **Blast radius:** Leadership approves but actual adoption stalls post-V1 if GTM users don't see value.
- **Reversibility:** Post-V1 iteration can adjust user modes based on feedback.

### RISK-17: Template Ground Truth Collection Stalls
- **Category:** Organizational
- **Severity:** MEDIUM
- **Description:** Deliverable templates and the offerings catalog require content owner involvement (Caleb). If content collection stalls due to bandwidth, priorities, or unclear ownership, generation quality suffers. Generation produces placeholder-quality output and demo shows structure but not substance.
- **Evidence:** Three content dependencies (offerings catalog, discovery taxonomy, deliverable templates) all route through a single content owner.
- **Mitigation:** Start with available samples — any existing Account Brief, Engagement Prep, or proposal serves as a template seed. Templates can iterate rapidly once generation works — first pass is structural, not final. Escalation path through Carlos if content bandwidth is the bottleneck. KIRT can generate without templates (unstructured output) — templates improve format, not feasibility.
- **Blast radius:** Multiple features produce unstructured output instead of formatted deliverables matching expected output format.
- **Reversibility:** Templates can be added iteratively post-launch without code changes.

### RISK-18: Executive Alignment Not Yet Obtained
- **Category:** Organizational
- **Severity:** MEDIUM
- **Description:** KIRT is in pre-build ideation. Executive buy-in (Carlos and broader leadership) depends on a convincing demo. The demo itself depends on successful build execution. Build may complete but organizational support for production deployment doesn't materialize.
- **Evidence:** V1 is explicitly scoped as "demo-ready MVP" not "production deployment."
- **Mitigation:** V1 scoped as "demo-ready MVP" — lower the bar for executive decision. Demo audience includes both leadership (strategic alignment) and GTM (tactical value) to build grassroots support. The "credible demo" success criterion is designed for this exact scenario — stakeholder watches and understands the value without explanation.
- **Blast radius:** Build completes but production deployment doesn't get organizational support.
- **Reversibility:** Demo can be re-run for different audiences; scope can be adjusted based on feedback.

### RISK-19: Content Owner Bandwidth
- **Category:** Organizational
- **Severity:** MEDIUM
- **Description:** Three content dependencies (offerings catalog, discovery taxonomy, deliverable templates) all route through Caleb. If Caleb is pulled to other priorities, all three are at risk simultaneously, causing multiple features to degrade at the same time.
- **Evidence:** Single content owner for three parallel content deliverables.
- **Mitigation:** All three dependencies have graceful degradation paths (by design). Offerings catalog: cross-sell works from engagement patterns, white space shows "not configured." Discovery taxonomy: AI-generates questions from context. Templates: generation works without templates (unstructured output). Priority order: templates (most visible impact) > offerings catalog > discovery taxonomy. Roy can create minimal versions of all three if Caleb is unavailable — spec defines exact requirements.
- **Blast radius:** Multiple features degrade simultaneously. Demo shows the "partial data" experience more than intended.
- **Reversibility:** All three content dependencies can be delivered and integrated iteratively without code changes.
