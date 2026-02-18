# AI Development Workflow with Claude Code

Autonomous SDLC system: Research → Plan → Implement → Review

## Quick Start

```bash
# Execute complete SDLC
/sdlc add-oauth-auth "Implement OAuth2 with Google"

# Check progress
cat docs/add-oauth-auth/STATUS.md
```

---

## Architecture

```
┌─────────────────────────────────────────┐
│              /sdlc command              │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│          SDLC Orchestrator              │
│  • Phase state machine                  │
│  • Gate enforcement                     │
│  • Review-fix loop (max 3)              │
└─────────────────┬───────────────────────┘
                  │
    ┌─────────────┼─────────────┐
    ▼             ▼             ▼
┌────────┐   ┌────────┐   ┌────────┐
│Research│   │ Plan   │   │Implement│
└────────┘   └────────┘   └────────┘
                              │
    ┌─────────────┐           │
    ▼             ▼           ▼
┌────────┐   ┌────────┐   ┌────────┐
│ Review │   │  Fix   │   │ STATUS │
└────────┘   └────────┘   └────────┘
```

---

## Usage

### Syntax

```bash
/sdlc <issue-name> [description] [--resume | --plan | --implement | --review]
```

### Examples

```bash
# Full workflow
/sdlc add-oauth-auth Implement OAuth2 with Google

# Resume after interruption
/sdlc add-oauth-auth --resume

# Start from specific phase
/sdlc add-oauth-auth --plan
```

### Arguments

| Argument | Description |
|----------|-------------|
| `issue-name` | Kebab-case identifier (required) |
| `description` | What to build (required for new) |
| `--resume` | Continue from STATUS.md |
| `--plan` | Start from Planning |
| `--implement` | Start from Implementation |
| `--review` | Start from Review |

---

## Artifacts

5 files per issue in `docs/{issue-name}/`:

| File | Purpose | Phase |
|------|---------|-------|
| STATUS.md | Progress tracker | All |
| RESEARCH.md | What we found | Research |
| PLAN.md | What we'll build | Planning |
| IMPLEMENTATION.md | What we built | Implementation |
| REVIEW.md | Is it ready? | Review |

---

## Workflow

```
Research → Plan → Implement → Review
                        ↓
                    Fix (max 3)
                        ↓
                      Review
```

### Phase Gates

| Phase | Gate |
|-------|------|
| Research | 3 questions answered |
| Planning | Scope + phases + **validation per phase** + criteria |
| Implementation | All phases done + **validations pass** + tests pass |
| Review | APPROVED verdict |

### Validation per Phase

Each phase in the plan includes validation checkpoints:

```markdown
### Phase 1: Foundation
Tasks:
- [ ] Create OAuth class
- [ ] Register with passport

Validation:
- [ ] npm test passes
- [ ] TypeScript compiles
- [ ] New class is importable
```

The implementer must pass all validations before proceeding to the next phase.

---

## Issue Names

Kebab-case, 1-50 chars:

- ✅ `add-oauth-auth`
- ✅ `fix-memory-leak`
- ✅ `refactor-api-layer`
- ❌ `AddOAuthAuth` (not kebab-case)
- ❌ `fix` (too vague)

---

## Check Progress

```bash
cat docs/{issue-name}/STATUS.md
```

Shows:
- Current phase
- Progress indicators
- Artifacts created
- Next steps

---

## Key Principles

1. **Single command** - Full SDLC in one invocation
2. **Autonomous** - No manual commands between phases
3. **Minimal artifacts** - 5 files per issue
4. **Phase validation** - Each phase has explicit validation checkpoints
5. **Quality gates** - Validation at each phase boundary
6. **Self-healing** - Auto fix loop (max 3 iterations)

---

## Documentation

- **Command reference:** `.claude/commands/COMMAND_USAGE.md`
- **Architecture:** `.claude/ARCHITECTURE.md`
- **Templates:** `.claude/templates/ARTIFACT_TEMPLATES.md`
