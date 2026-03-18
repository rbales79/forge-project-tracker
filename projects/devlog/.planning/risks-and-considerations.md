# Risks and Considerations: devlog

**Date:** 2026-03-17

---

## Risk Register

### R1 — Binary Name Collision
**Severity:** Medium
**Source:** [FROM RESEARCH: RISK]
"devlog" is a generic term. It may already exist in Homebrew, apt, or other package managers as developer blog tooling or log utilities.

**Mitigation:** Run `brew search devlog`, `apt-cache search devlog`, `pip search devlog` before finalizing the name. Reserve the GitHub repo name early. If collision found, fallback names: `glog`, `repolog`, `logr`, `gitlog-tui`.

---

### R2 — Scope Creep Toward Full Git Client
**Severity:** High
**Source:** [FROM RESEARCH: RISK]
lazygit has a gravitational pull. Once you have a TUI with commit list and keyboard nav, it's tempting to add staging, diff, branch switching. This would make devlog a worse lazygit.

**Mitigation:** HANDOFF explicitly marks these as Non-Goals. Phase contexts do not include staging/commit features. Code reviewer should flag any PR that touches git write operations.

---

### R3 — mani Feature Overlap
**Severity:** Low
**Source:** [FROM RESEARCH: RISK]
If repo management features (sync, run commands across repos) get added, devlog becomes a worse mani.

**Mitigation:** devlog is read-only on repos. Config management (adopt/unadopt) is the only write operation. No task execution, no sync, no clone.

---

### R4 — Performance with Large Repos
**Severity:** Medium
**Source:** Internal assessment
Repos with 100k+ commits (Linux kernel, large monorepos) could cause go-git to hang on full log iteration.

**Mitigation:** Use go-git iterator with early termination at date boundary. Never load full commit history. Default filter to last 30 days. Document known limitation for very large repos.

---

### R5 — go-git Completeness Gaps
**Severity:** Low
**Source:** Internal assessment
go-git is a pure Go reimplementation and doesn't cover every git edge case: shallow clones, worktrees, git worktree commands, some pack formats.

**Mitigation:** Wrap go-git calls in error handlers that log and skip problematic repos. Consider subprocess fallback (`git log`) for edge cases where go-git fails.

---

### R6 — LiteLLM API Changes
**Severity:** Low
**Source:** Internal assessment
LiteLLM proxy API may change. But it's OpenAI-compatible, so changes are unlikely to break the `/v1/chat/completions` endpoint.

**Mitigation:** Keep AI client behind an interface. Version pin the openai SDK dependency.

---

## Considerations

### Config File Location
`~/.config/devlog/config.yaml` follows XDG base directory spec. On macOS, `~/.config` may not exist by default. devlog must create the directory if missing.

### Secrets in Config
LiteLLM API key must NOT be written to config.yaml. Use environment variables only. The config file may end up in dotfiles repos (it's in `~/.config`). Document this clearly.

### First-Run Experience
A user who runs `devlog` with no config should see a helpful message: "No repos adopted. Run `devlog scan [path]` to get started." Not a blank/broken TUI.

### Testing Strategy
Bubble Tea has `teatest` package for TUI testing. Use it for component render tests. For git integration, use fixture repos (small bare repos checked into testdata/). Don't rely on the user's actual repos in CI.
