# Deploy Plan: add-repo-context-engine

**When:** 2026-03-15
**Strategy:** Big Bang (low-risk, prompt-only changes, no runtime)

---

## 1. Pre-Deployment Checklist

- [x] Code review approved — APPROVED with 0 blocking issues
- [x] Security audit passed — PASSED with 0 critical/high/medium
- [ ] CLAUDE.md updated — add `/repo-map` to Quick Reference Commands section
- [ ] README.md patched — add `/repo-map` to available commands (if listed)
- [ ] Git commit created with all 4 changed files
- [ ] Pushed to main branch
- N/A: No tests (prompt-engineering project), no CI, no database, no env vars, no feature flags

---

## 2. Rollout Strategy

**Big Bang** — all changes deploy in a single commit.

**Justification:**
- Zero runtime risk — all changes are Markdown prompt files
- No servers to restart, no containers to rebuild
- Changes take effect immediately on the next Claude Code session
- Fully backward compatible — existing workflows work identically without the new feature

**Stages:**
1. Commit all 4 files to a feature branch
2. Push to `main` (or merge via PR if required)
3. Verify by running `/repo-map` and `/discover` in a new session

---

## 3. Database Migration Plan

**N/A** — no database, no state migration. All artifacts are Markdown files under `.claude/`.

---

## 4. Rollback Playbook

### Triggers
- `/repo-map` produces incorrect or empty output consistently
- `/discover` Step 3 breaks the existing discovery flow (e.g., fails to create `01_DISCOVERY.md`)
- `researching-code` Step 0 crashes or enters an infinite loop

### Steps
1. `git revert <commit-hash>` — reverts all 4 file changes in one commit
2. Push the revert commit
3. Verify: run `/discover` on a test project — confirm it works without the repo map step

### Verification
- `/discover` produces `01_DISCOVERY.md` with all standard sections (stack, quality tooling, etc.)
- `/research` falls back to the previous flat MCP/Grep approach
- No error messages or missing sections in the output

### Communication
- No external stakeholders — this is a developer tooling change
- If rolled back, note the reason in a git commit message for future reference

---

## 5. Post-Deployment Verification

**Immediate (within the same session):**
- [ ] Run `/repo-map` on this project — verify compact map output with correct format
- [ ] Run `/repo-map src/` on a TypeScript project — verify path-scoped mode works
- [ ] Run `/discover` on a fresh project — verify `01_DISCOVERY.md` includes `## Repository Map`
- [ ] Verify the repo map appears in the user-facing output from `/discover`

**Next session (24h):**
- [ ] Run `/research {issue}` on a project with `01_DISCOVERY.md` — verify Level 1→2 escalation (reads map first)
- [ ] Run `/research {issue}` on a project WITHOUT prior `/discover` — verify on-the-fly map generation
- [ ] Verify CLAUDE.md Quick Reference Commands section includes `/repo-map`

---

## 6. Communication Plan

### Commit Message

```
feat: add hierarchical repo context engine (/repo-map + /discover integration)

- New /repo-map command: generates compact structural overview (file tree +
  top-level symbols) in ≤2K tokens, supports 6 language patterns
- /discover now auto-generates repo map as Step 3, embedded in 01_DISCOVERY.md
- researching-code skill upgraded to Level 1→Level 2 escalation: reads repo
  map first, then does targeted search on candidate files only
- ARCHITECTURE.md updated with Context Engine (Hierarchical) section

Inspired by Aider's repo map approach. Uses Glob + Grep (no MCP required).
```

### CLAUDE.md Update

Add to the Quick Reference Commands → Workflow section:
```markdown
/repo-map [path]                     # Generate compact repo structural overview
```

And update the `/discover` description to note it now includes repo map generation.

### README Update

If the README lists available commands, add `/repo-map` to the list.
