# KIRT — Content Dependencies

> Non-code dependencies that must be ready before demo. Tracked separately from GSD phases.
> Updated: 2026-03-24

## Status Board

| # | Dependency | Status | Owner | Blocks | Notes |
|---|-----------|--------|-------|--------|-------|
| C1 | Offerings catalog v0.1 | In Progress | Roy | White space, cross-sell, scoring | Real extraction from practice materials preferred; synthetic fallback acceptable |
| C2 | Discovery question taxonomy | Done | — | Engagement Prep precision | Generated: demo-data/config/discovery-questions.json (200+ questions, tree structure) |
| C3 | Demo data (4 accounts) | In Progress | Roy | Integration testing, demo | Being prepared separately. Synthetic SF CSVs + SP documents. |
| C4 | Deliverable templates | Done | — | Generation pipeline | Generated: demo-data/templates/ (Pellera-branded Word/PPT) |
| C5 | SP test site | Available | Roy | SP write-back testing | forgecf.sharepoint.com |
| C6 | Entra app registration | Not Started | Roy | AD/SSO, Graph API | Will create when Phase 1 auth work begins |
| C7 | Foundation v3 access | Operational | — | All integration | Running on Contabo |
| C8 | CVPS3 provisioning | Not Started | Roy | Deployment | New Contabo VPS for KIRT Docker deployment |
| C9 | LLM provider credentials | Not Started | Roy | Generation pipeline | Gemini, Azure OpenAI, or Bedrock API keys |
| C10 | Sentry DSN | Not Started | Roy | Error tracking | Create Sentry project for KIRT |

## Completed
- C2: Discovery question taxonomy (2026-03-24)
- C4: Deliverable templates (2026-03-24)
- C5: SP test site identified (forgecf.sharepoint.com)
- C7: Foundation v3 operational
