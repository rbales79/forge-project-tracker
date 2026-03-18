# Research Synthesis: devlog

**Date:** 2026-03-17
**Purpose:** Extract build-relevant insights from market research. Tagged for Dev Agent consumption.

---

## Features Competitors Have — Candidate Requirements

[FROM RESEARCH] git-standup's directory-scan model (configurable depth via `-m`, author filter via `-a`, date range via `-d`) is proven for multi-repo discovery. devlog should adopt the same scanning approach with interactive selection rather than static output.

[FROM RESEARCH] mani's declarative config file (mani.yaml) for persisting repo groups is a proven pattern. devlog should support a config file (devlog.yaml or ~/.config/devlog/config.yaml) for persisting adopted repo lists so users don't re-scan every session.

[FROM RESEARCH] git-standup supports `.git-standup-whitelist` for path inclusion filtering. devlog should support ignore/include patterns (`.devlogignore` or inline config).

[FROM RESEARCH] lazygit and gitui both show per-repo commit graph/log with diff inline. devlog should offer a detail pane for selected commits showing diff and full message.

[FROM RESEARCH] onefetch displays language breakdown, top contributors, creation date, LOC per repo. These are candidate "repo summary card" fields for devlog's adopt/overview screen.

[FROM RESEARCH] git-quick-stats supports author filtering and time-range filtering (day, week, month). devlog should expose these as TUI filters rather than flags.

[FROM RESEARCH] mani supports repo tagging/grouping (project, team, language). devlog should let users tag adopted repos for filtered views.

---

## UX / Interaction Patterns

[FROM RESEARCH: UX] lazygit uses a multi-pane layout: left column (file tree / branch list), right column (log / diff). This split-pane model is the dominant TUI pattern for git tools. devlog should adopt a two-panel layout: left = repo list, right = log/detail.

[FROM RESEARCH: UX] gitui is keyboard-driven with vim-like navigation (j/k for up/down, Enter to expand, q to quit). This is the expected UX contract for terminal power users. devlog must follow similar conventions.

[FROM RESEARCH: UX] tig uses a top/bottom split: top = commit list, bottom = diff/detail. This is a valid alternative to left/right split, especially for log-heavy views. Consider as a secondary layout mode.

[FROM RESEARCH: UX] lazygit and gitui both support filtering/search via `/` keybinding — standard convention devlog should implement.

[FROM RESEARCH: UX] mani's TUI includes tabbed views for different report types. devlog can use tabs or switchable panels: "All Repos", "Today", "This Week", "By Author".

---

## Positioning

[FROM RESEARCH: POSITIONING] The killer differentiator is the combination of multi-repo scan + TUI navigation + AI summaries in Go. No existing tool does all three. git-standup does multi-repo but outputs plain text. lazygit does TUI but is single-repo. This is a clear whitespace.

[FROM RESEARCH: POSITIONING] "Daily standup prep" is the strongest entry-point use case — it's a daily pain point, the value is immediately obvious, and it differentiates from generic git clients. Lead with this in the README.

[FROM RESEARCH: POSITIONING] "AI-optional" is a trust differentiator. Users distrust tools that fail without internet. Lean into "works offline, AI makes it better" — not "AI-powered" framing.

---

## Risks

[FROM RESEARCH: RISK] The name "devlog" is generic. Search for "devlog" returns developer blog tooling, not git tooling. Binary name collision risk: check that `devlog` isn't an existing package in Homebrew, apt, or common Linux distros. Fallback names: `glog`, `gitlog-tui`, `repolog`, `logr`.

[FROM RESEARCH: RISK] git-standup already solves the multi-repo standup use case adequately for non-TUI users. devlog must have a clear TUI value-add story, not just be "git-standup with a pretty interface."

[FROM RESEARCH: RISK] mani is also written in Go and has a built-in TUI for multi-repo management. There's partial feature overlap. devlog must stay focused on log aggregation, not repo management — don't drift into mani territory.

[FROM RESEARCH: RISK] lazygit community is large and active. If devlog tries to compete as a full git client it will lose. Stay in the log aggregation / standup lane.

---

## Architecture Patterns

[FROM RESEARCH: ARCH] lazygit uses gocui for its TUI layer. Bubble Tea (Charm) is the modern successor — more composable, better maintained, growing ecosystem. Use Bubble Tea + Lip Gloss for devlog. This is the right call.

[FROM RESEARCH: ARCH] mani is written in Go with a declarative YAML config. Go + YAML config is the proven stack for multi-repo CLI tools. devlog should follow the same pattern.

[FROM RESEARCH: ARCH] onefetch runs as a single-pass read tool (no state mutation). devlog should treat repo scanning as read-only enumeration, with adopt/unadopt being explicit config mutations — never silently modify repos.

[FROM RESEARCH: ARCH] git-standup uses recursive directory walk + `git log` subprocess calls. This is the practical approach — don't re-implement git, shell out to it or use `go-git` library for pure Go parsing. go-git is the leading Go git library (https://github.com/go-git/go-git).

[FROM RESEARCH: ARCH] For AI integration, use the OpenAI-compatible SDK against LiteLLM proxy. This is already the house standard. Don't import provider-specific SDKs.

---

## Summary of Must-Have Signals

From research, the following belong in Must-Have:
1. Multi-repo scan (recursive directory walk, configurable depth)
2. Repo adopt/unadopt with persistent config
3. TUI with two-panel layout (repo list + log/detail)
4. Standard keyboard navigation (j/k, /, q, Enter)
5. Date range + author filters
6. Works fully offline (no AI required)
7. AI summary of commit log (optional, LiteLLM proxy)

From research, the following belong in Deferred (v2):
1. Repo tagging/grouping
2. Language/LOC breakdown (onefetch-style)
3. Per-author contribution stats
4. Diff viewer inline
5. Export to markdown/JSON
