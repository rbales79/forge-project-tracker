# Research to Dev Handoff

## Project
- Name: devlog
- Research depth: low
- Complexity: medium
- Recommended build engine: ecc-build

## Artifacts Location
- HANDOFF.md: .planning/HANDOFF.md
- Phase contexts: 3 files at .planning/ (phase-1-context.md, phase-2-context.md, phase-3-context.md)
- Market research: .planning/research/market-research.md
- Synthesis: .planning/research/synthesis.md

## Key Decisions (Locked)

| Decision | Choice |
|---|---|
| Language | Go |
| TUI framework | Bubble Tea (Charm) |
| Styling | Lip Gloss |
| Git access | go-git (github.com/go-git/go-git) |
| Config format | YAML |
| Config location | ~/.config/devlog/config.yaml |
| AI integration | LiteLLM proxy, OpenAI-compatible SDK |
| AI model tier | `standard` |
| Offline behavior | Full functionality without AI |

## Deferred Items

These are explicitly v2 — do not build in Phase 1-3:

- Repo tagging/grouping
- Language/LOC breakdown (onefetch-style)
- Per-author contribution stats
- Inline diff viewer
- Export to markdown/JSON
- AI pattern detection (cold repos, file churn)
- Windows support
- GitHub/GitLab API integration

## Notes for Dev

1. **Name risk:** "devlog" may collide with existing packages. Verify `brew search devlog` and `apt-cache search devlog` before publishing. Fallbacks: `glog`, `repolog`, `logr`.

2. **UX contract:** Follow lazygit/gitui keybinding conventions exactly — j/k scroll, Enter select, q quit, / filter, Tab switch panels. These are user expectations for Go TUI tools.

3. **go-git performance:** Use iterator pattern with early termination at date boundary. Default to last 30 days. Never load full commit history into memory.

4. **AI is additive:** Keep `internal/ai/` behind an interface. All failure modes must return a message and fall back gracefully — never crash or hang.

5. **First-run UX:** If no repos are adopted, show "No repos adopted. Run `devlog scan [path]` to get started." — not a blank screen.

6. **Secrets:** LiteLLM API key is env var only (`LITELLM_API_KEY`). Never write it to config.yaml.

7. **Read the synthesis:** `.planning/research/synthesis.md` has tagged `[FROM RESEARCH]` items that map directly to requirements. Review before writing any feature code.

8. **Build order matters:** Phase 1 (TUI shell with fixtures) before Phase 2 (real git data). Don't shortcut. Having a working TUI skeleton first makes integration much cleaner.
