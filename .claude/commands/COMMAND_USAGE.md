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
| Security (7a) | SECURITY_AUDIT.md | OWASP/STRIDE evaluated |
| Pentest (7b) | PENTEST_REPORT.md | Shannon run complete |
| AI Audit (7c) | AI_THREAT_MODEL.md | LLM threats documented |
| Harden (8) | HARDEN_PLAN.md + patches | P0 fixes implemented |

---

## Security Commands (DevSecOps)

### `/security/pentest {issue}`

Phase 7b — Dynamic pentest via Shannon (autonomous AI pentester).

```bash
/security/pentest add-jwt-rbac
```

**Prerequisites:** `/security` completed, staging running, Docker available, Shannon cloned.
**Output:** `PENTEST_REPORT.md` with proven exploits only.

> Never run against production. Staging or localhost only.

### `/security/redteam-ai {issue}`

Phase 7c — AI/LLM threat modeling (only if LLMs in stack).

```bash
/security/redteam-ai add-chat-assistant
```

**Skip if** no LLM/AI components. **Output:** `AI_THREAT_MODEL.md`.

### `/security/harden {issue}`

Phase 8 — Aggregate findings, prioritize, and implement fixes.

```bash
/security/harden add-jwt-rbac
```

**Priority:** P0 (fix now) → P1 (this sprint) → P2 (next sprint) → P3 (backlog).
**Output:** `HARDEN_PLAN.md` + P0 patches applied + GitHub issues for P1/P2.

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
Security Layer:
  • /security (7a: static)
  • /security/pentest (7b: dynamic, optional)
  • /security/redteam-ai (7c: AI audit, optional)
  • /security/harden (8: fix loop)
    ↓
5 core artifacts + security artifacts + code
```

---

## Benefits

- **Single command** - Full SDLC in one invocation
- **Autonomous** - No manual commands between phases
- **Organized** - All artifacts in one directory
- **Tracked** - STATUS.md shows progress
- **Quality gates** - Validation at each phase
- **Self-healing** - Auto fix loop (max 3)
- **DevSecOps** - Integrated security testing with proven exploits only
