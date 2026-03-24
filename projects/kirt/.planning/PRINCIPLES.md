# KIRT — Principles

*Project-specific principles that guide decisions. These override or extend workspace-level defaults.*

1. **Foundation owns all storage** — KIRT never stores documents. Uploads route through `POST /v1/ingest/`. No separate document store.
2. **Source SP files are never modified** — KIRT does not edit, move, or delete source files in their original SharePoint locations. Generated deliverables in output folders ARE updated in place on regeneration — SP versioning preserves history.
3. **Input and output paths never overlap** — Output folder must not equal, be a parent of, or be a child of any configured input source path. Same SP site with different folders is fine. Enforced at admin config time. Prevents re-ingestion feedback loops and keeps source vs. generated content distinguishable by folder structure.
4. **Permission inheritance is strict** — If you can't see it in SharePoint, you can't see it in KIRT. Enforced pre-query in Foundation, not post-fetch in KIRT.
5. **Dedup is invisible by default** — 1 result per logical document. The "N copies" detail is accessible but not the default view.
6. **Users never see the research process** — Iceberg snapshots are internal. Users see the output document and provenance summary, not intermediate data.
7. **Write-back is explicit and user-initiated** — Every write to SharePoint is a deliberate user action with a confirmation step. Never silently write.
8. **Graceful degradation with transparency** — Every feature works with partial data and tells the user exactly what's missing. No silent failures, no empty screens.
9. **MSA opt-out is absolute** — `ai_processing_allowed: false` means keyword search only. No LLM processing, no summarization, no classification. No exceptions.
10. **Admin-configurable, not hardcoded** — SP paths, scoring weights, LLM providers, offerings catalog, and data source configs must be tunable without code changes.
11. **KIRT complements Salesforce, never replaces it** — SF stays as system of record for structured CRM data. KIRT handles unstructured analysis and AI generation.
