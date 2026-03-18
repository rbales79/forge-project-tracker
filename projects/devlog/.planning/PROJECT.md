# devlog

**Type:** CLI Tool (local, offline-first)
**Stack:** Go, Bubble Tea, Lip Gloss, go-git
**Status:** Ideation → Ready for Dev
**Date:** 2026-03-17

---

## What It Is

devlog is an AI-assisted TUI git log aggregator. It scans your filesystem for git repositories, lets you adopt repos into a persistent workspace, and shows aggregated commit history across all of them in an interactive terminal UI. AI summarization and pattern detection are available when a LiteLLM proxy is configured, but the tool is fully functional without it.

## Why It Exists

Developers working across multiple repos — microservices, polyrepos, personal projects — have no single tool to answer "what happened across all my repos today/this week?" git-standup is close but outputs plain text. lazygit is excellent but single-repo only. devlog fills the gap: multi-repo + TUI + AI-optional.

## Core Use Cases

1. **Daily standup prep** — "Show me everything I committed yesterday across all repos"
2. **Week review** — "Summarize this week's commits, grouped by repo"
3. **Pattern detection** — "Which repos am I not touching? Which files change most often?"
4. **Onboarding** — Scan a new machine, adopt all repos, get oriented fast

## Stack Decisions

| Concern | Choice | Rationale |
|---|---|---|
| Language | Go | Performance, single binary, no runtime dep, good TUI ecosystem |
| TUI framework | Bubble Tea (Charm) | Modern, composable, active community, replaces gocui |
| Styling | Lip Gloss | Pairs with Bubble Tea, consistent theming |
| Git access | go-git | Pure Go, no subprocess dependency for reads |
| Config | YAML (~/.config/devlog/config.yaml) | Proven pattern (mani), portable |
| AI | LiteLLM proxy (OpenAI-compatible) | House standard, offline-first |

## Out of Scope (v1)

- Remote repo management (clone, push, pull)
- Full git client (staging, committing, branching)
- Server component or web UI
- Real-time file watching
- GitHub/GitLab API integration
