# HANDOFF: devlog

**Build Engine:** ECC Build
**Research Depth:** Low
**Complexity:** Medium
**Date:** 2026-03-17

---

## What This Is

devlog is a local CLI tool — a TUI git log aggregator written in Go with Bubble Tea. It scans your filesystem for git repositories, lets you adopt them into a persistent config, and aggregates commit history across all adopted repos in an interactive terminal UI. AI-assisted summarization and pattern detection are available via LiteLLM proxy but are never required.

---

## Core Value

One command to see what happened across every repo you care about, with an interactive TUI to slice it by date, author, or repo. No server, no internet required, no account — just your local repos.

---

## Context

- Field CTO tooling — personal productivity, daily standup prep, week reviews
- Runs on macOS and Linux (Windows stretch goal)
- Deployed as a single Go binary, installed via `go install` or Homebrew
- AI features connect to `https://litellm.roybales.com` (multimode persona) or `http://localhost:4000`
- Config lives at `~/.config/devlog/config.yaml`

---

## Constraints

- Go + Bubble Tea — no framework negotiation
- Local only — no server component, no remote API calls except optional LiteLLM
- AI is never a blocker — all features must work without LiteLLM configured
- Single binary — no install script, no runtime dependencies
- Must handle repos with 10k+ commits without hanging (pagination or lazy-load required)
- No mutation of git repos — devlog is read-only on the repos it watches

---

## Requirements Signals

### Must-Have

- [ ] Recursive filesystem scan for git repos (configurable root path, configurable depth)
- [ ] Adopt / unadopt repos into persistent config (~/.config/devlog/config.yaml)
  - [FROM RESEARCH] mani's declarative YAML config pattern — proven for multi-repo tools
- [ ] TUI with two-panel layout: repo list (left) + commit log (right)
  - [FROM RESEARCH: UX] lazygit's split-pane is the dominant TUI pattern for git tools
- [ ] Standard keyboard navigation: j/k scroll, Enter expand, q quit, / filter
  - [FROM RESEARCH: UX] gitui and lazygit establish this as the expected UX contract
- [ ] Commit log view: hash, author, date, subject — per selected repo or aggregated across all
- [ ] Date range filter: today, yesterday, this week, last 7 days, custom
  - [FROM RESEARCH] git-standup's `-d` flag proves this is the primary query dimension
- [ ] Author filter
  - [FROM RESEARCH] git-standup's `-a` flag — multi-repo author filtering is a core need
- [ ] Fully functional offline (no AI required)
- [ ] AI summary of commit log (optional — requires LiteLLM config)
  - [FROM RESEARCH: ARCH] OpenAI-compatible SDK against LiteLLM proxy — house standard
- [ ] `devlog scan [path]` — discover repos in a path, show list, prompt to adopt
- [ ] `devlog` — launch TUI directly

### Deferred

- [ ] Repo tagging/grouping (v2)
  - [FROM RESEARCH] mani supports this; useful but not Day 1
- [ ] Language/LOC breakdown per repo (v2)
  - [FROM RESEARCH] onefetch-style repo summary cards
- [ ] Per-author contribution stats view (v2)
  - [FROM RESEARCH] git-quick-stats coverage
- [ ] Inline diff viewer (v2)
  - [FROM RESEARCH: UX] lazygit/gitui do this; adds complexity, not core to log aggregation
- [ ] Export to markdown/JSON (v2)
- [ ] Pattern detection (which repos are going cold, file churn) (v2)
- [ ] Windows support (v2)
- [ ] GitHub/GitLab API integration (v3)

### Non-Goals

- Full git client (staging, committing, branching, push/pull) — that's lazygit
- Server component or web UI
- Real-time file watching / daemon mode
- Remote repo cloning / management — that's mani
- Team features (shared config, org-wide rollup)

---

## Key Decisions (Locked)

| Decision | Choice | Rationale |
|---|---|---|
| Language | Go | Single binary, performance, good TUI ecosystem |
| TUI framework | Bubble Tea (Charm) | Modern Go TUI standard; lazygit uses older gocui |
| Styling | Lip Gloss | Charm ecosystem, pairs with Bubble Tea |
| Git access | go-git (github.com/go-git/go-git) | Pure Go, no subprocess for reads |
| Config format | YAML | Proven in mani, human-readable, simple |
| Config location | ~/.config/devlog/config.yaml | XDG-compliant, expected location |
| AI integration | LiteLLM proxy, OpenAI-compatible SDK | House standard; multimode persona |
| AI model | `standard` tier alias | Commit summarization is not complex reasoning |
| Offline behavior | Full functionality without AI | Trust and reliability non-negotiable |

---

## Risk Summary

- [FROM RESEARCH: RISK] **Name collision** — "devlog" binary name may conflict with existing tools or packages. Verify with `brew search devlog`, `apt search devlog` before finalizing. Fallbacks: `glog`, `repolog`, `logr`.
- [FROM RESEARCH: RISK] **Scope creep toward git client** — lazygit has a gravitational pull. Enforce the log-aggregation boundary; don't implement staging/commit/push.
- [FROM RESEARCH: RISK] **mani overlap** — if repo management features get added, devlog becomes a worse mani. Stay in the log lane.
- **Performance with large repos** — repos with 100k+ commits need lazy loading or pagination. go-git must be profiled against the Linux kernel repo scale before shipping.
- **go-git completeness** — go-git doesn't implement all git features. If edge cases appear (shallow clones, worktrees), may need subprocess fallback.

---

## Complexity Assessment

**Overall: Medium**

- Go + Bubble Tea is well-documented with examples (Charm tutorial is excellent)
- go-git covers the core log/commit reading use case cleanly
- Multi-repo aggregation is conceptually simple (fan-out, merge, sort)
- AI integration is additive and isolated (one call, one display toggle)
- TUI layout is the most complex element — managing state across panels, filters, lazy load
- Config management is straightforward YAML read/write

Estimated build time at Low ECC depth: 2-3 sessions.

---

## Execution Guide

### Build Order

**Phase 1 — Core Shell**
- Go module init, Bubble Tea app skeleton
- Two-panel TUI layout (repo list + log pane)
- Hardcoded repo list for development
- Keyboard navigation (j/k, q, Enter)

**Phase 2 — Git Integration**
- go-git integration: read commits from a single repo
- Multi-repo fan-out: read commits from all adopted repos, merge/sort by date
- Date range filtering (today, yesterday, last 7 days)
- Author filtering
- `devlog scan` command: walk directories, find .git, offer adopt prompt
- Config YAML read/write

**Phase 3 — AI + Polish**
- LiteLLM integration: summarize selected date range commit log
- AI toggle (enabled/disabled, graceful degradation if unreachable)
- Install/distribution: `go install`, Makefile, basic README

### Success Criteria

- [ ] `devlog scan ~/repos` finds all git repos and offers adopt prompt
- [ ] `devlog` launches TUI, shows adopted repos on left, commits on right
- [ ] Filtering by date and author works without AI configured
- [ ] AI summary generates and displays when LiteLLM is available
- [ ] AI failure (timeout, not configured) shows graceful fallback message, does not crash
- [ ] Single binary, no external runtime dependencies
- [ ] Works against repos with 10k+ commits without hanging

### Test Expectations

- Unit tests: repo scanner, config read/write, commit merge/sort logic
- Integration tests: TUI component render tests (Bubble Tea has test utilities)
- Manual E2E: run against ~/repos on dev machine, validate output matches `git log` ground truth

---

## Market Positioning

devlog occupies the intersection of multi-repo log aggregation + TUI + AI-optional. No existing tool covers all three:

- git-standup: multi-repo, no TUI, no AI
- lazygit: excellent TUI, single repo, no AI
- tig: excellent log TUI, single repo, no AI
- mani: multi-repo TUI, management focus, no log aggregation, no AI

Lead with the standup use case in the README. That's the daily pain point that makes the value proposition immediately obvious.

---

## User Personas

**Primary: Roy (Field CTO)**
- 10-20 active repos across personal projects, clients, homelab
- Daily standup prep is a real time sink
- Comfortable in terminal, expects vim-like keybindings
- Wants AI to help, not to be required

**Secondary: Polyrepo Developer**
- Microservices shop, 5-50 repos
- Needs "what changed this week" across repos for sprint reviews
- Values offline reliability

**Tertiary: New Machine Setup**
- Just cloned everything to a new laptop
- Wants to scan and get oriented fast
- devlog scan + adopt is the entry point
