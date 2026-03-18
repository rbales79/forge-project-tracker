# Phase 3 Context: AI Integration + Polish

**Layer:** 3 of 3
**Label:** LiteLLM summarization, graceful degradation, distribution
**Estimated effort:** 1 session

---

## Objective

Add AI-assisted commit summarization as an optional enhancement. Wire LiteLLM proxy. Handle all failure modes gracefully. Ship a distributable binary.

---

## Deliverables

- `internal/ai/` — LiteLLM client: send commit log slice, return summary string
- TUI: `s` keybinding to trigger AI summary for current view (selected date range + repos)
- AI status indicator: loading spinner while request in flight
- Graceful degradation: if LiteLLM unreachable, show "AI unavailable — configure LITELLM_BASE_URL" message, do not crash
- AI config: `LITELLM_BASE_URL` + `LITELLM_API_KEY` env vars (fallback to config.yaml fields)
- `Makefile` with `build`, `install`, `test` targets
- Basic README: install, quick start, config, keybindings reference

---

## Research Context

[FROM RESEARCH: ARCH] LiteLLM proxy exposes an OpenAI-compatible API. Use the `openai` Go SDK or plain HTTP client with the standard `/v1/chat/completions` endpoint. Use the `standard` tier model alias for commit summarization — it doesn't need premium reasoning.

[FROM RESEARCH: POSITIONING] "AI-optional" is a trust differentiator. The README should lead with offline capability and frame AI as an enhancement, not a requirement. Users who've been burned by tools that phone home will appreciate this framing.

---

## AI Prompt Pattern

```
You are a developer assistant. Summarize the following git commits in 2-3 sentences.
Focus on what changed, why it matters, and any patterns you notice.
Be concise and technical.

Commits:
<formatted commit list>
```

---

## Error Handling Contract

| Condition | Behavior |
|---|---|
| LiteLLM not configured | `s` key shows "AI not configured. Set LITELLM_BASE_URL." |
| LiteLLM unreachable | Shows "AI unavailable (connection refused)" |
| LiteLLM returns error | Shows error message from API response |
| Request timeout (>10s) | Cancel, show "AI request timed out" |
| No commits in view | `s` key disabled / shows "No commits to summarize" |

---

## Success Gate

`s` key triggers summary when LiteLLM is configured. Summary displays in a modal or status pane. All failure modes show a message and return to normal TUI — no crash, no hang.

Binary built with `make build`, runs on target machines without Go installed.

---

## Notes

- AI call must be non-blocking: use Bubble Tea's `tea.Cmd` async pattern to fire HTTP request and return result as a Msg
- Never store LiteLLM API key in config.yaml — env var only for secrets
- Keep `internal/ai/` behind an interface so it can be swapped or mocked in tests
