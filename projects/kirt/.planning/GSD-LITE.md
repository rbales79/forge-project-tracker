# GSD Lite — Execution Protocol

This file governs how AI assistants work in this codebase. Read it at the start of
every session. Follow it for every change — no exceptions.

## Before You Write Code

Every session starts the same way:

1. **Read this file** — you're doing that now
2. **Read `.planning/SESSION_LOG.md`** — understand what happened last session, what's in progress, what's next
3. **Read `.planning/KNOWLEDGE.md`** — know the gotchas, patterns, and decisions already made
4. **Read `.planning/DEVIATIONS.md`** — understand where the codebase differs from the original plan
5. **Understand the change** — read the relevant code before modifying it. No blind edits.

If any of these files don't exist yet, create them using the formats defined below.

## Change Protocols

Use the checklist that matches your change type. Don't skip steps.

### Bug Fix

- [ ] Reproduce the bug — confirm you can see the failure
- [ ] Identify root cause, not just the symptom
- [ ] Check for other instances of the same bug pattern
- [ ] Write a failing test that captures the bug (if testable)
- [ ] Fix the root cause
- [ ] Verify the test passes
- [ ] Run existing tests — confirm nothing else broke
- [ ] Commit: `fix: <what was fixed and why>`
- [ ] Update SESSION_LOG.md
- [ ] Update KNOWLEDGE.md if the bug revealed a gotcha or pattern

### Feature Addition

- [ ] Check scope — is this feature in scope? If not, add to `TODO.md` and stop
- [ ] Read existing code in the area you're modifying
- [ ] Design the approach before writing code — if multiple valid approaches exist, document your choice in SESSION_LOG.md
- [ ] Build incrementally — don't write 500 lines then test. Build a piece, verify, build the next piece
- [ ] Write tests proportional to complexity — business logic always gets unit tests
- [ ] Verify the feature works end-to-end before considering it done
- [ ] Commit: `feat: <what was added>` — one logical change per commit
- [ ] Update SESSION_LOG.md
- [ ] Update KNOWLEDGE.md if new patterns, dependencies, or architectural decisions were introduced

### Refactor

- [ ] Confirm the refactor was requested — don't refactor opportunistically
- [ ] Write characterization tests if none exist for the code being refactored
- [ ] Make changes in small, verifiable steps — each step should pass all existing tests
- [ ] Don't change behavior. If you're changing behavior, that's a feature or a bug fix, not a refactor
- [ ] Commit per logical step: `refactor: <what changed and why>`
- [ ] Update SESSION_LOG.md
- [ ] Update KNOWLEDGE.md if the refactor changes patterns or conventions

### Dependency / Config Change

- [ ] Document why the change is needed
- [ ] Check for breaking changes or version conflicts
- [ ] Verify the dependency is actively maintained (if new)
- [ ] Update relevant config or environment documentation
- [ ] Run full test suite after the change
- [ ] Commit: `chore: <what changed>` or `build: <what changed>`
- [ ] Update SESSION_LOG.md

### Performance Optimization

- [ ] Measure current performance — establish a baseline with numbers
- [ ] Identify the actual bottleneck (profile, don't guess)
- [ ] Fix the bottleneck
- [ ] Measure again — confirm improvement with numbers
- [ ] Verify no behavioral changes — run existing tests
- [ ] Commit: `perf: <what was optimized and the improvement>`
- [ ] Update SESSION_LOG.md
- [ ] Update KNOWLEDGE.md with the performance characteristics discovered

## Execution Principles

These apply to every change, regardless of type.

1. **One change at a time.** Complete and verify before starting the next. Don't interleave unrelated changes.
2. **Atomic commits.** Each commit is one logical change. Conventional commit prefix (`feat:`, `fix:`, `refactor:`, `chore:`, `perf:`, `docs:`, `test:`). If you can't describe the commit in one line, it's too big.
3. **Verify behaviorally.** After every change, run it. Check that it works. "It should work" is not verification.
4. **Read before write.** Understand existing code before modifying it. Understand existing patterns before introducing new ones.
5. **No scope creep.** If it's not what you're here to do, capture it in `TODO.md` and move on. Don't "fix" unrelated things while you're in a file.
6. **Tests are part of the change.** Not "I'll add tests later." Business logic gets unit tests. Integration points get integration tests. If you're changing behavior, update the tests.
7. **Track state.** SESSION_LOG.md is always current. If you made a change, it's logged.
8. **Deviations are fine. Undocumented deviations aren't.** If you diverge from the established architecture, patterns, or original plan — log it in DEVIATIONS.md with the rationale.
9. **Don't refactor during a fix.** Build it, ship it, then refactor. Resist the urge to "improve" code you're touching for a different reason.
10. **Ask when uncertain.** If you're unsure about an approach, surface it. Don't guess and hope.

## Scope Enforcement

**In scope (V1 — build these):**
- R01-R04: SSO/AD auth, single group full access, JWT to Foundation, MSA opt-out flag
- R05-R12: Hybrid search (BM25 + semantic), filters, relationship traversal, dedup, ranking, freshness
- R13-R18: Document upload → Foundation ingest, auto-classification (7 types), job tracking, manual override, notifications, admin SP site ingestion
- R19-R23: Account Brief (<60s), 9 sections, partial data support, completeness indicators, SP write-back
- R24-R27: Engagement Prep (prior work, stakeholder map, discovery questions, next-best-question)
- R28-R30: Meeting Summary & Follow-Up (notes → recap + action items + email draft)
- R31-R33: Client Deliverable (name + keywords → client-ready doc with account history)
- R34-R36: Cross-sell recommendations (pattern-based, triggered after ingestion)
- R37-R43: Generation infrastructure (progress indicators, inline editing, feedback learning, concurrent handling, versioning, SP write-back conventions)
- R44-R46: SAF overlay (optional for EA users), growth scoring (equal weights, manual override)
- R47-R49: Canonical selection (location-ranking config), archival
- R50-R52: Admin config (SP paths, scoring weights, LLM provider, offerings catalog, GDPR purge, input/output separation)
- R53-R55: Health endpoint, error tracking (Sentry), usage logging
- R56-R58: Responsive mobile read flows, task-oriented UI, desktop-only generation/admin
- R59-R60: Error state for Foundation unavailable, graceful degradation everywhere

**Out of scope — do not build:**
- D01-D16: V2 features (near-duplicate detection, access tiers, opportunity scoring, maturity gate, email/push notifications, cached mode, RAG feedback, SF narrative pipeline, SEC EDGAR, APM, quality scoring, retention policies, transcript analysis, stale alerts)
- D17-D20: V3 features (reframe decks, agentic RAG, template management, full provenance)
- D21-D27: V4+ and post-V1 (multi-tenancy, SF direct API, email/calendar, multi-agent, Teams tab, WCAG, Azure migration)
- KIRT is not a CRM replacement
- KIRT does not edit SP source files
- KIRT does not maintain a document store
- KIRT does not surface intermediate research/generation process
- KIRT does not decide which copy is authoritative — user decides

**Rule:** When working on a feature request, check it against this list. If it's
in the "out of scope" list, capture it in `TODO.md` with a note and move on.
If it's genuinely needed for something in scope to work, document the rationale
in DEVIATIONS.md and proceed.

## Test Commands

```bash
# Backend (Django)
pytest                           # All tests
pytest tests/unit/               # Unit tests only
pytest tests/integration/        # Integration tests (requires Foundation v3)
python manage.py test            # Django test runner alternative

# Frontend (React/Vite)
npm test                         # Vitest
npm run test:unit                # Unit tests only
npm run test:e2e                 # E2E (if configured)

# Linting
ruff check .                     # Python linting
npm run lint                     # ESLint + Prettier

# Type checking
mypy .                           # Python type checking
npm run typecheck                # TypeScript checking
```

## Project-Specific Notes

### Key Decisions (Locked — do not revisit)

| Decision | Choice |
|----------|--------|
| Frontend | React + Vite + Tailwind + Shadcn |
| Backend | Django, MCP-native |
| Vector DB | Weaviate via Foundation API (not direct) |
| Storage | Apache Iceberg / Parquet on Azure Blob (Foundation manages) |
| Auth | AD/SSO via Graph API, JWT claims to Foundation |
| LLM | Multi-provider abstraction (Gemini, Azure OpenAI, Bedrock). No provider-specific code. Admin-configurable. |
| Form factor | Web app, responsive. No native app. No Teams tab in V1. |
| Tenancy | Single-tenant (Pellera only) in V1 |
| Access | Single AD group, full access in V1 |
| Document storage | Foundation owns all storage. KIRT routes uploads via `POST /v1/ingest/` |
| SP write policy | Explicit, user-initiated, confirmation required |
| Permissions | SP permission inheritance, strict, pre-query enforcement |
| Dedup | Content hash, 1 result per doc, location-ranking config for canonical |
| Scoring | Equal weights (5 dimensions, 20% each), manual override, tunable via admin |
| MSA opt-out | `ai_processing_allowed` flag, keyword search only for opted-out content |
| GDPR | KIRT manages purge. Must explicitly delete own annotations. Foundation cascade covers docs/chunks only. |
| Feedback | Prompt-level (few-shot from edits) in V1. RAG loop V2. Fine-tune V3+. |
| Concurrent gen | Both complete, both write SP, user picks winner. Merge is V3+. |

### Critical Dependencies

- **Foundation v3.0** — 13 REST API endpoints. KIRT cannot function without it. Live on Contabo.
- **Offerings catalog v0.1** — WIP. Cross-sell, white space, scoring all degrade without it. Not a blocker — features degrade gracefully.
- **Discovery question taxonomy** — 200+ questions, may need formalization from informal notes during build.
- **Demo data (4 accounts)** — Required for integration testing and demo. Not yet ingested into Foundation.
- **AD/SSO test environment** — Required for auth layer.
- **SP test site with read/write** — Required for upload, write-back, source management.

### Development Strategy

**Local first, Foundation last.**
- Unit tests and UI development: local, mock Foundation API responses
- Integration tests: against live Foundation v3 on Contabo
- Foundation flaws: file issues in Foundation repo, don't work around them in KIRT
- KIRT bugs are KIRT issues. Foundation API/search/ingestion issues are Foundation issues. Track independently.

### Performance Targets (soft, V1)

| Operation | Target | Hard Limit |
|-----------|--------|------------|
| Search | <3s | <10s |
| Page load | <2s | <5s |
| Account Brief | <60s | <5min |
| Engagement Prep | <90s | <5min |
| Classification | <30s | <2min |

All generation operations show real-time progress via Foundation's `GET /v1/jobs/<id>/`.
