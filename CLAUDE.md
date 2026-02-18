# CLAUDE.md

Autonomous SDLC workflow for Claude Code.

## Quick Start

```bash
/sdlc add-oauth-auth "Implement OAuth2 with Google"
cat docs/add-oauth-auth/STATUS.md
```

## Main Command

### `/sdlc <issue-name> [description] [flags]`

Execute complete SDLC: Research → Plan → Implement → Review

**Flags:**
- `--resume` - Continue from STATUS.md
- `--plan` - Start from Planning
- `--implement` - Start from Implementation
- `--review` - Start from Review

## Architecture

```
/sdlc → SDLC Orchestrator → Skills → Artifacts
```

**4 Phases:**
1. Research → RESEARCH.md
2. Planning → PLAN.md
3. Implementation → IMPLEMENTATION.md + code
4. Review → REVIEW.md

**5 Artifacts:**
- STATUS.md (progress)
- RESEARCH.md (findings)
- PLAN.md (plan)
- IMPLEMENTATION.md (what was built)
- REVIEW.md (approval)

## Issue Names

Kebab-case, 1-50 chars: `add-oauth-auth`, `fix-memory-leak`, `refactor-api`

## Key Files

| File | Purpose |
|------|---------|
| `.claude/commands/sdlc.md` | Main command |
| `.claude/agents/sdlc-orchestrator.md` | Workflow orchestration |
| `.claude/skills/*.md` | Phase capabilities |
| `.claude/templates/ARTIFACT_TEMPLATES.md` | Artifact templates |

## Principles

1. **Single command** - Full SDLC autonomously
2. **Minimal artifacts** - 5 files per issue
3. **Trust Claude Code** - Native context management
4. **Quality gates** - Validation at each phase
5. **Self-healing** - Auto fix loop (max 3)
