# Phase 1 Context: Core Shell

**Layer:** 1 of 3
**Label:** Go module + Bubble Tea skeleton + TUI layout
**Estimated effort:** 1 session

---

## Objective

Stand up the Go project with a working Bubble Tea TUI that shows a two-panel layout: repo list on the left, commit log on the right. Navigation works. No real data yet — hardcoded fixtures are fine.

---

## Deliverables

- `go.mod` + `go.sum` with Bubble Tea + Lip Gloss dependencies
- `main.go` entry point, Cobra CLI skeleton (`devlog` command)
- `internal/ui/` — Bubble Tea app, model, update, view
- Two-panel layout: left pane (repo list), right pane (log list)
- Keyboard: j/k navigation, Enter to select repo, q to quit, Tab to switch panels
- Hardcoded sample data (3 repos, 10 commits each)

---

## Research Context

[FROM RESEARCH: UX] lazygit's split-pane layout is the dominant TUI pattern for git tools. Two-panel (repo list left, log right) is the correct layout choice — familiar to terminal power users.

[FROM RESEARCH: UX] gitui and lazygit both use j/k for scroll, Enter to expand, q to quit, / for search. This is the expected keybinding contract. Follow it exactly.

[FROM RESEARCH: ARCH] Bubble Tea is the modern replacement for gocui (what lazygit uses). Charm's tutorial and bubbletea examples repo are the implementation guide. Start with the `list` and `viewport` components from the `bubbles` library.

---

## Dependencies

- github.com/charmbracelet/bubbletea
- github.com/charmbracelet/bubbles (list, viewport)
- github.com/charmbracelet/lipgloss
- github.com/spf13/cobra

---

## Success Gate

`devlog` launches, shows two-panel layout with sample data, keyboard navigation works, no panics.

---

## Notes

- Keep UI code in `internal/ui/` separate from domain logic from day one
- Model should be a pure struct (no mutation in place) — use Bubble Tea's Msg/Cmd pattern correctly
- Don't wire real git data yet — that's Phase 2
