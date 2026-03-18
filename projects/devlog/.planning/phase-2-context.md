# Phase 2 Context: Git Integration + Config

**Layer:** 2 of 3
**Label:** go-git integration, multi-repo fan-out, scan command, config persistence
**Estimated effort:** 1-2 sessions

---

## Objective

Wire real git data into the TUI. Implement `devlog scan` to discover repos. Persist adopted repos to YAML config. Support date range and author filtering.

---

## Deliverables

- `internal/git/` — go-git wrapper: open repo, read log, return []Commit structs
- `internal/scanner/` — recursive filesystem walk, find `.git` dirs, return []RepoCandidate
- `internal/config/` — read/write `~/.config/devlog/config.yaml` (adopted repos list)
- `devlog scan [path]` CLI command — find repos, interactive adopt prompt (TUI list with checkboxes)
- Multi-repo fan-out: read commits from all adopted repos, merge by date descending
- Filters: date range (today / yesterday / last 7 days / custom) and author string
- TUI updated to show real data; loading state while scanning

---

## Research Context

[FROM RESEARCH] git-standup uses recursive directory walk + git subprocess calls. devlog improves on this by using go-git for pure Go reads and presenting results in a TUI rather than plain text.

[FROM RESEARCH] git-standup's `-m` (max depth), `-a` (author), `-d` (days ago) flags are the proven filter dimensions. Implement these as TUI interactive filters (not CLI flags) for the log view.

[FROM RESEARCH] mani uses `mani.yaml` for declarative repo lists — the XDG config location (`~/.config/devlog/config.yaml`) and YAML format is the right pattern.

[FROM RESEARCH: ARCH] go-git (github.com/go-git/go-git) is the pure Go git library. Use `repository.PlainOpen(path)` and `repo.Log(&git.LogOptions{})` for commit iteration. Handle shallow clones gracefully (they return limited history, not errors).

[FROM RESEARCH: ARCH] For repos with large commit histories (10k+), use go-git's iterator pattern with early termination on date boundary — don't load all commits into memory.

---

## Key Data Structures

```go
type Repo struct {
    Path    string
    Name    string
    Adopted bool
}

type Commit struct {
    Hash      string
    Author    string
    Timestamp time.Time
    Subject   string
    RepoName  string
}

type Config struct {
    AdoptedRepos []string // absolute paths
    DefaultAuthor string
    LiteLLMBaseURL string
    LiteLLMAPIKey  string
}
```

---

## Success Gate

`devlog scan ~/repos` finds all repos and offers adopt prompt. `devlog` shows real commits from adopted repos, filtered by date. Author filter narrows results. Config persists across restarts.

---

## Notes

- go-git repo open errors (not a git repo, corrupt, etc.) must be handled gracefully — log and skip, never crash
- Treat config write as immutable update: read → modify copy → write (no in-place mutation)
- Scanner should skip `.git` directories, `node_modules`, and other non-repo noise
