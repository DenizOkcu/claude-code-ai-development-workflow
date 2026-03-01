# Command Usage Reference

Quick reference for the SDLC workflow.

## `/sdlc <issue-name> [description] [flags]`

Execute complete SDLC: Research → Plan → Implement → Review

### Syntax

```bash
/sdlc <issue-name> [description] [--resume | --plan | --implement | --review]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `issue-name` | Yes | Kebab-case identifier (1-50 chars) |
| `description` | No* | What to build (max 1000 chars) |
| `--resume` | No | Continue from STATUS.md |
| `--plan` | No | Start from Planning (needs RESEARCH.md) |
| `--implement` | No | Start from Implementation (needs PLAN.md) |
| `--review` | No | Start from Review (needs IMPLEMENTATION.md) |

*Required for new workflows

### Examples

```bash
# Full workflow
/sdlc add-oauth-auth Implement OAuth2 with Google

# Resume after interruption
/sdlc add-oauth-auth --resume

# Start from specific phase
/sdlc add-oauth-auth --plan
```

### What It Creates

```
docs/{issue-name}/
├── STATUS.md           # Progress tracker
├── RESEARCH.md         # What we found
├── PLAN.md             # What we'll build
├── IMPLEMENTATION.md   # What we built
└── REVIEW.md           # Is it ready?
```

### Issue Name Format

- Kebab-case: `add-oauth-auth`
- 1-50 characters
- No path traversal

**Good:**
- `add-oauth-auth`
- `fix-memory-leak`
- `refactor-api-layer`

**Bad:**
- `AddOAuthAuth` (not kebab-case)
- `fix` (too vague)
- `../etc/passwd` (path traversal)

### Check Progress

```bash
cat docs/{issue-name}/STATUS.md
```

---

## Workflow Phases

| Phase | Creates | Gate |
|-------|---------|------|
| Research | RESEARCH.md | 3 questions answered |
| Planning | PLAN.md | Scope + phases + criteria |
| Implementation | IMPLEMENTATION.md + code | All phases done + tests pass |
| Review | REVIEW.md | APPROVED verdict |
| Fix | Fixed code | Blocking issues resolved |

---

## Architecture

```
/sdlc command
    ↓
SDLC Orchestrator Agent
    ↓
Skills (sequential):
  • researching-code
  • planning-solutions
  • implementing-code
  • reviewing-code
  • review-fix (if needed)
    ↓
5 artifacts + code
```

---

## Benefits

- **Single command** - Full SDLC in one invocation
- **Autonomous** - No manual commands between phases
- **Organized** - All artifacts in one directory
- **Tracked** - STATUS.md shows progress
- **Quality gates** - Validation at each phase
- **Self-healing** - Auto fix loop (max 3)
