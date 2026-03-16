# Deploy Plan: add-semantic-retrieval

**Strategy:** Big Bang (git commit + push to main)
**Risk:** Low — additive-only change, zero breaking changes, implicit feature flag

---

## Context

This is a Markdown/YAML toolkit — there is no server, no production environment, and no database. "Deployment" means:
1. Committing changes to the `main` branch of this repository
2. Users who copy/pull `.claude/` into their projects get the new capability
3. Existing users are unaffected until they run `/retrieval/setup`

---

## 1. Pre-Deployment Checklist

- [x] Code review approved — `06_CODE_REVIEW.md`: APPROVED, 0 blocking findings
- [x] Security audit passed — `07a_SECURITY_AUDIT.md`: PASSED, 0 critical/high/medium, 2 low (F-02 fixed)
- [x] No database migrations — purely additive file changes
- [x] No environment variables introduced into CI/CD — retrieval env vars are user-configured via setup wizard
- [x] Feature flag in place — implicit via `claude-context` presence in `settings.json`; off by default
- [x] Rollback plan documented — `git revert` or targeted file revert (see below)
- [ ] CLAUDE.md `## Learnings` updated — pending `/retro` phase
- [ ] Changelog entry drafted — see Section 6

**Commands to verify before committing:**
```bash
# Confirm all expected files are present
ls .claude/commands/retrieval.md .claude/commands/retrieval/setup.md

# Confirm no secrets introduced
git diff --stat
git diff

# Validate settings.json is valid JSON
python3 -c "import json; json.load(open('.claude/settings.json')); print('OK')"

# Confirm Milvus port binding fix is in place
grep "127.0.0.1:19530" .claude/commands/retrieval/setup.md
```

---

## 2. Rollout Strategy: Big Bang

**Rationale:** This is a static toolkit distributed via git. There are no staged rollouts — users adopt changes when they pull. The implicit feature flag (retrieval is off until `/retrieval/setup` is run) means zero impact on existing users regardless of when they update.

| Stage | Action | Criteria to Proceed |
|-------|--------|---------------------|
| 1. Commit | `git commit` all changes to `main` | All pre-deploy checks pass |
| 2. Push | `git push origin main` | Commit is clean |
| 3. Done | Users pull at their own pace | N/A — no forced update |

**Rollout percentage:** N/A — users self-select when to pull.

**Success criteria:**
- `git push` succeeds with no conflicts
- Repository is browsable on GitHub with new files visible
- README.md renders correctly (Semantic Code Retrieval section)

---

## 3. Database Migration Plan

**Not applicable.** No database involved in this change. Milvus is a user-local infrastructure component set up on-demand by each user via the setup wizard.

---

## 4. Rollback Playbook

### Triggers (when to rollback)
- A user reports that existing workflow commands (`/research`, `/implement`) are broken after pulling
- A security vulnerability is discovered in `@zilliz/claude-context-mcp` or one of its dependencies
- The session-start retrieval suggestion causes confusion or disrupts existing users

### Steps

**Option A: Targeted file revert (preferred — surgical)**
```bash
# Revert only the changed files, keeping planning artifacts
git revert HEAD --no-commit

# Or revert specific files:
git checkout HEAD~1 -- .claude/skills/researching-code/SKILL.md
git checkout HEAD~1 -- .claude/skills/implementing-code/SKILL.md
git checkout HEAD~1 -- .claude/ARCHITECTURE.md
git checkout HEAD~1 -- .claude/settings.json
git checkout HEAD~1 -- CLAUDE.md
git checkout HEAD~1 -- README.md
git rm .claude/commands/retrieval.md .claude/commands/retrieval/setup.md
git commit -m "revert: remove semantic retrieval (add-semantic-retrieval)"
git push origin main
```

**Option B: Full revert commit**
```bash
# Find the commit hash before this feature
git log --oneline -10

# Revert the feature commit
git revert <commit-hash>
git push origin main
```

### Verification
```bash
# Confirm retrieval commands are gone
ls .claude/commands/retrieval.md 2>/dev/null && echo "FAIL: file still exists" || echo "OK"

# Confirm skills are back to original
grep "Step 0: Semantic Retrieval" .claude/skills/researching-code/SKILL.md && echo "FAIL: step 0 still present" || echo "OK"

# Confirm settings.json is valid
python3 -c "import json; json.load(open('.claude/settings.json')); print('OK')"
```

### Impact of Rollback
- **Zero data loss** — no user data was stored by this change; Milvus/Ollama are local user infrastructure
- **Zero workflow disruption** — skills fall back to Glob/Grep automatically without the MCP
- **Users who already ran `/retrieval/setup`** will have `claude-context` in their local `settings.json`; they should remove it manually or it will fail silently (MCP won't load)

### Communication
Notify in commit message: `revert: remove semantic retrieval (add-semantic-retrieval) — see issue #N`

---

## 5. Post-Deployment Verification

After `git push`, verify:

```bash
# 1. README renders correctly on GitHub
# Check: https://github.com/{user}/{repo} — "Semantic Code Retrieval" section visible

# 2. New command files exist in repo
git show HEAD:.claude/commands/retrieval.md | head -5
git show HEAD:.claude/commands/retrieval/setup.md | head -5

# 3. Skills updated correctly
git show HEAD:.claude/skills/researching-code/SKILL.md | grep "Step 0"

# 4. settings.json valid
git show HEAD:.claude/settings.json | python3 -c "import json,sys; json.load(sys.stdin); print('OK')"
```

**Manual smoke test (for users with Ollama + Docker):**
1. Copy `.claude/` to a test project
2. Run `/retrieval/setup` → choose Option 1 (local)
3. Apply the generated `settings.json` block
4. Restart Claude Code
5. Run `/retrieval index` on the test project
6. Run `/retrieval search "authentication"` → expect ranked results
7. Run `/research` on a known issue → verify Step 0 fires and results appear

---

## 6. Communication Plan

### Commit Message
```
feat: add semantic code retrieval via @zilliz/claude-context-mcp

Introduces hybrid BM25 + vector search over AST-indexed codebases:
- /retrieval/setup — interactive wizard (Ollama+Milvus local, Zilliz Cloud)
- /retrieval — search, index, status, clear commands
- researching-code skill: Step 0 queries index before Glob/Grep
- implementing-code skill: optional pattern discovery via search_code
- Off by default — active only when claude-context is in settings.json

Security: Milvus bound to 127.0.0.1 (not 0.0.0.0) per audit finding F-02.

Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>
```

### README Changelog Entry
Add to README.md `## Changelog` section (if present) or include in GitHub release notes:

> **v{next} — Semantic Code Retrieval**
> - New: `/retrieval/setup` — configure hybrid semantic + keyword code search
> - New: `/retrieval` — search, index, and manage codebase index
> - Enhanced: `/research` now queries semantic index before Glob/Grep (when configured)
> - Enhanced: `/implement` now suggests related-file discovery (when configured)
> - Powered by `@zilliz/claude-context-mcp` — Tree-sitter AST, Merkle tree incremental indexing
> - Local-first: Ollama embeddings + Docker Milvus (no cloud required)
> - Off by default — zero impact until you run `/retrieval/setup`
