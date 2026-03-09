# DevLog - Builder Handoff

## What to Build
A terminal UI application that aggregates git commits across multiple repos, groups them by time period, and generates AI-powered summaries/changelogs.

## Core Features (MVP)
1. **Multi-repo scanning** - Point at a directory, discover all git repos, aggregate commits
2. **TUI interface** - Interactive terminal UI with repo list, commit browser, summary panel
3. **AI summaries** - Send grouped commits to LLM via LiteLLM proxy, get human-readable changelogs
4. **Time filtering** - Daily, weekly, sprint-based grouping
5. **Export** - Markdown changelog export

## Technical Decisions
- **Language:** Go (for single-binary distribution, TUI ecosystem)
- **TUI framework:** Bubble Tea (charmbracelet/bubbletea)
- **Git integration:** go-git (pure Go, no libgit2 dependency)
- **LLM integration:** OpenAI-compatible SDK pointed at LiteLLM proxy
- **Config:** YAML config file (~/.devlog.yaml)

## Architecture
- `cmd/` - CLI entry point (cobra)
- `internal/git/` - Git scanning and commit aggregation
- `internal/tui/` - Bubble Tea models and views
- `internal/ai/` - LLM summarization client
- `internal/config/` - Config loading

## Acceptance Criteria
- [ ] Scan N repos and display commits in TUI
- [ ] Filter by date range
- [ ] Generate AI summary of selected commits
- [ ] Export summary as markdown
- [ ] Single binary, no runtime dependencies
