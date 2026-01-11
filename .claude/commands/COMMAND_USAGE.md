# Command Usage Reference

Quick reference for the SDLC workflow command.

## `/sdlc <issue-name> [description] [--from-<phase>]`

**Unified SDLC command** - Executes the entire software development lifecycle autonomously.

### Syntax

```bash
/sdlc <issue-name> [feature-description] [--from-<phase>]
```

### Arguments

- `issue-name` (required): Kebab-case identifier
  - Example: `add-oauth-auth`, `fix-memory-leak`, `refactor-api-layer`

- `feature-description` (optional): What to build
  - Example: "Implement OAuth2 with Google and GitHub"

- `--from-<phase>` (optional): Resume from specific phase
  - `--from-start`: Execute full SDLC (default)
  - `--from-plan`: Resume from planning phase
  - `--from-implement`: Resume from implementation phase
  - `--from-review`: Resume from review phase

### Examples

```bash
# Full workflow from start
/sdlc add-oauth-auth Implement OAuth2 with Google and GitHub

# Resume from planning (research already done)
/sdlc add-oauth-auth --from-plan

# Resume from implementation (plans exist)
/sdlc add-oauth-auth --from-implement

# Resume from review (implementation done)
/sdlc add-oauth-auth --from-review
```

### What It Creates

All artifacts organized in `docs/{issue-name}/`:
- `STATUS.md` - Progress tracker (single source of truth)
- `CODE_RESEARCH.md` - Research findings
- `IMPLEMENTATION_PLAN.md` - Phased implementation strategy
- `PROJECT_SPEC.md` - Technical specification
- `CODE_REVIEW.md` - Review findings

### How It Works

1. **Research Phase**: Investigates codebase architecture and patterns
2. **Planning Phase**: Creates implementation plans and specs
3. **Implementation Phase**: Executes code following the plan
4. **Review Phase**: Runs automated checks and manual review
5. **Review-Fix Loop** (if needed): Addresses issues (max 3 iterations)

Each phase must pass validation before progressing. The agent manages all state transitions automatically.

### Check Progress

```bash
cat docs/{issue-name}/STATUS.md
```

STATUS.md shows:
- Current phase
- Progress indicators
- Artifacts created
- Next steps

### Issue Name Format

**Rules:**
- kebab-case (lowercase-with-hyphens)
- Concise (2-5 words)
- Descriptive

**Examples:**
- ✅ `add-oauth-auth`, `fix-memory-leak`, `refactor-api-layer`
- ❌ `AddOAuthAuth` (not kebab-case), `fix` (too vague), `add-new-auth-system-with-oauth2` (too long)

### Complete Workflow Example

```bash
# Start to finish - one command
/sdlc add-oauth-auth Implement OAuth2 with Google and GitHub

# The agent will:
# 1. Research authentication patterns in codebase
# 2. Create implementation plan and technical spec
# 3. Implement the feature phase by phase
# 4. Run automated checks and manual review
# 5. Fix any issues (up to 3 iterations if needed)
# 6. Report completion with deployment guidance

# Check progress anytime
cat docs/add-oauth-auth/STATUS.md
```

---

## Architecture

The `/sdlc` command orchestrates the complete workflow:

```
/sdlc command
    ↓
SDLC Orchestrator Agent
    ↓
Sequential skill execution:
  • code-research
  • solution-planning
  • code-implementation
  • code-review
  • review-fix (if needed)
    ↓
Complete artifacts + approval
```

**Key features:**
- Autonomous execution (no manual command switching)
- Phase gate enforcement (validation before progression)
- Automatic review-fix loop (max 3 iterations)
- Persistent state tracking (STATUS.md)
- Clear progress feedback

---

## Benefits

- **Single command** - Execute entire SDLC with one invocation
- **Multi-feature support** - Work on multiple issues simultaneously
- **Organized artifacts** - All docs for a feature in one directory
- **Progress tracking** - STATUS.md shows complete workflow state
- **Quality gates** - Automated validation at each phase
- **Self-healing** - Automatic review-fix loop for issues
