# Market Research: devlog

**Research depth:** Low (single pass)
**Date:** 2026-03-17

---

## Problem Space

Developers working across multiple git repositories have no unified TUI tool that aggregates commit history, provides AI-assisted summaries, and works offline-first. Existing tools either focus on a single repo, require internet access, or are read-only stat dashboards rather than interactive aggregators.

---

## Competitive Landscape

### Category 1: Multi-Repo Aggregators / Standup Tools

**git-standup**
- URL: https://github.com/kamranahmedse/git-standup
- What it does: Scans a directory tree of repos, shows commits from last working day per repo. Useful for daily standup prep.
- Strengths: Multi-repo scan, configurable depth, author filtering, date range
- Weaknesses: CLI output only (no TUI), no AI, no interactive navigation, output is static text
- Key flags: `-a` (author), `-d` (days ago), `-m` (max dir depth), `-F` (force recursion)

**git-quick-stats**
- URL: https://github.com/git-quick-stats/git-quick-stats | https://git-quick-stats.sh/
- What it does: Statistical summaries for a single git repo — commits by author, time distribution, file churn
- Strengths: Rich statistics, bash-based so portable
- Weaknesses: Single repo only, no TUI navigation, no AI, output is terminal-printed stats

**mani**
- URL: https://github.com/alajmo/mani | https://manicli.com/
- What it does: CLI tool to manage multiple repos — clone, sync, run commands across all repos. Has a built-in TUI.
- Strengths: Declarative config (mani.yaml), repo grouping/tagging, custom tasks, built-in TUI, written in Go
- Weaknesses: Focused on repo management/command execution, not log aggregation or commit history. Not AI-enabled.
- Tech: Go

### Category 2: Full Git TUI Clients

**lazygit**
- URL: https://github.com/jesseduffield/lazygit
- What it does: Full-featured TUI git client — staging, branching, rebasing, log view, diff view
- Strengths: Single-repo interactive git operations, excellent UX, Go + gocui library, huge community
- Weaknesses: Single repo focus, no multi-repo aggregation, no AI summaries, not a log aggregator
- Tech: Go, gocui

**gitui**
- URL: https://github.com/gitui-org/gitui
- What it does: Blazing fast terminal UI for git written in Rust
- Strengths: Performance, full git operations (commit, stage, diff), keyboard-driven
- Weaknesses: Single repo, no AI, no multi-repo, Rust not Go
- Tech: Rust

**tig**
- URL: https://jonas.github.io/tig/
- What it does: Text-mode interface for git — browse log, diff, blame, refs
- Strengths: Lightweight, fast, widely available, excellent log browsing
- Weaknesses: Single repo, no AI, limited to browsing (no multi-repo), curses-based (not Bubble Tea)
- Tech: C

**grv (Git Repository Viewer)**
- URL: https://github.com/rgburke/grv
- What it does: Terminal interface for viewing git repositories — commits, diffs, refs
- Strengths: Multi-window layout, vim-like keybindings
- Weaknesses: Single repo, abandoned/low maintenance, no AI
- Tech: Go

### Category 3: Stats / Summary Tools

**onefetch**
- URL: https://github.com/o2sh/onefetch
- What it does: Displays a summary card for a single git repo — language breakdown, top contributors, LOC, license
- Strengths: Beautiful output, language detection, offline
- Weaknesses: Single repo, display-only (no interaction), no multi-repo, Rust

**git-fame**
- What it does: Per-file/author contribution breakdown
- Weaknesses: Single repo, Python, no TUI

**GitStats**
- URL: https://gitstats.sourceforge.net/
- What it does: Generates HTML statistics reports for a repo
- Weaknesses: HTML output, not terminal, single repo, old project

### Category 4: Multi-Repo Management (adjacent)

**gita** (Python)
- Mentioned on HN — manages groups of repos, run git commands across all
- Weaknesses: Python, no TUI, no AI

**mu-repo**
- URL: https://fabioz.github.io/mu-repo/
- What it does: Run git commands across multiple repos
- Weaknesses: Python, no TUI, no log aggregation focus

---

## Gap Analysis

| Feature | git-standup | lazygit | tig | mani | devlog target |
|---|---|---|---|---|---|
| Multi-repo | Yes | No | No | Yes | Yes |
| TUI | No | Yes | Yes | Partial | Yes |
| AI summaries | No | No | No | No | Yes |
| Log aggregation | Partial | No | Yes (single) | No | Yes |
| Scan & adopt repos | No | No | No | Yes (manual config) | Yes |
| Offline-first | Yes | Yes | Yes | Yes | Yes |
| Go | No | Yes | No | Yes | Yes |

---

## Key Takeaway

No existing tool combines: multi-repo scan + TUI log aggregation + AI-assisted summaries + Go + Bubble Tea. The closest is git-standup (multi-repo scan, no TUI) + lazygit (excellent Go TUI, single repo). devlog fills the intersection.
