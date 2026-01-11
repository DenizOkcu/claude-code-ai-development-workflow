# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **Claude Code configuration repository** that implements an autonomous Software Development Lifecycle (SDLC) system. It's not a traditional software application with dependencies or build processes - it's a workflow orchestration system using Claude Code commands, agents, and skills.

**Key Concept:** The `/sdlc` unified command executes complete feature development autonomously: Research → Planning → Implementation → Review.

## Architecture

### Four-Layer System

1. **User Interface Layer** (`.claude/commands/`)
   - `sdlc.md` - Unified SDLC command (single entry point)
   - Only parses arguments and invokes agent (NO workflow logic)

2. **Orchestration Layer** (`.claude/agents/`)
   - `sdlc-orchestrator.md` - Manages complete SDLC workflow
   - Phase state machine, gate enforcement, review-fix loops (max 3 iterations)
   - State persistence via STATUS.md and STATE.json

3. **Capabilities Layer** (`.claude/skills/`)
   - `code-research.md` - Investigate codebase architecture and patterns
   - `solution-planning.md` - Create implementation plans and specs
   - `code-implementation.md` - Execute code phase-by-phase
   - `code-review.md` - QA checks and review
   - `review-fix.md` - Address review issues

4. **Data Layer** (`docs/{issue-name}/`)
   - Persistent artifacts: STATUS.md, CODE_RESEARCH.md, IMPLEMENTATION_PLAN.md, PROJECT_SPEC.md, CODE_REVIEW.md
   - Summary artifacts for phase transitions: RESEARCH_SUMMARY.md, PLAN_SUMMARY.md, IMPLEMENTATION_SUMMARY.md

## Essential Commands

### Main Workflow Command

```bash
# Execute complete SDLC autonomously
/sdlc add-oauth-auth "Implement OAuth2 with Google and GitHub"

# Resume from specific phase
/sdlc add-oauth-auth --from-plan      # Resume from planning
/sdlc add-oauth-auth --from-implement  # Resume from implementation
/sdlc add-oauth-auth --from-review    # Resume from review

# Check progress
cat docs/add-oauth-auth/STATUS.md
```

### Issue Name Format

**Rules:**
- Kebab-case (lowercase-with-hyphens)
- 1-50 characters
- Cannot start/end with hyphen
- No path traversal sequences (`..`, `/`, `\`)

**Examples:**
- ✅ `add-oauth-auth`, `fix-memory-leak`, `refactor-api-layer`
- ❌ `AddOAuthAuth` (not kebab-case), `../etc/passwd` (path traversal)

## Quality Invariants

When modifying this codebase, maintain these core principles:

1. **No logic in command** - Commands only parse and invoke agents
2. **Stateless skills** - Skills contain procedures, not state
3. **Agent orchestrates** - Only agent manages workflow state
4. **Persistent artifacts** - All progress in `docs/`
5. **Deterministic gates** - Clear validation before progression
6. **Max iterations** - Review-fix limited to 3 attempts
7. **Clear separation** - UX, orchestration, capabilities, data layers

## State Management

### STATE.json (Machine-Readable Index)

Tracks workflow state, artifacts created, and phase transitions. Schema: `.claude/schemas/STATE.json.schema`

### STATUS.md (Human-Readable Progress)

Single source of truth for workflow progress. Shows:
- Which phases are completed
- Current phase status
- Artifacts created
- Next steps
- Review iteration count

### Artifact Strategy

**Critical:** Artifacts ARE the communication mechanism between phases. They must be comprehensive, not minimized.

- **Full Artifacts:** Complete research findings, plans, specs, reviews (for reference)
- **Summary Artifacts:** Comprehensive summaries for next phase input (150-200 lines)
- **Phase Transitions:** Clear ephemeral context (chat), preserve files, load on-demand

## Key File Locations

### Core Configuration
- `.claude/commands/sdlc.md` - Main command (UX interface)
- `.claude/agents/sdlc-orchestrator.md` - Workflow orchestration
- `.claude/skills/` - Five atomic skills (stateless procedures)
- `.claude/settings.local.json` - Permissions (WebSearch, Bash)

### Documentation
- `README.md` - Complete usage guide (336 lines)
- `.claude/ARCHITECTURE.md` - Architecture overview (226 lines)
- `.claude/STATE_MANAGEMENT.md` - State persistence strategy
- `.claude/TROUBLESHOOTING.md` - Problem resolution guide
- `.claude/USER_COMMUNICATION.md` - Communication patterns

### Templates and Examples
- `.claude/templates/ARTIFACT_TEMPLATES.md` - Standard templates
- `.claude/examples/` - Reference artifacts

## Workflow Mechanics

### Phase State Machine

```
[START] → research → plan → implement → review → [COMPLETE]
                                      ↓
                                  fix_loop (max 3)
                                      ↓
                                   review
```

### Entry Point Validation

Before resuming with `--from-*`, required artifacts must exist:

- `--from-plan`: Requires CODE_RESEARCH.md, RESEARCH_SUMMARY.md
- `--from-implement`: Requires all research + planning artifacts
- `--from-review`: Requires all research + planning + implementation artifacts

**Cannot skip phases** - The orchestrator validates artifacts before allowing resume.

### Review-Fix Loop

- Max 3 iterations to fix issues
- Each iteration loads minimal context (issues + changed files only)
- After 3 failed iterations, workflow blocks with diagnostic

### Context Management

**Cleared at phase transition:**
- Chat history from previous phases
- Agent conversation memory
- Temporary working context

**Preserved (files on disk):**
- All artifact files (comprehensive)
- STATE.json (index)
- STATUS.md (human progress tracker)

**Loading strategy:**
1. STATE.json (index, always)
2. Required summary artifact (comprehensive, always)
3. Full artifacts (on-demand, as needed)

## Agent Behavior

### Critical Workflow Rule

The orchestrator MUST continue through ALL phases sequentially. Do NOT terminate until:
1. Research phase is complete
2. Planning phase is complete
3. Implementation phase is complete
4. **Review phase is complete** (REQUIRED, not optional)

The workflow is ONLY complete when STATE.json shows `current_phase: "complete"` after review.

### User Notification Requirements

**ALWAYS notify:**
- Workflow start with issue details and estimated duration
- Phase transitions with duration and artifacts created
- Gate validation results (passed/failed)
- Review-fix iteration count and issues found
- Workflow completion with summary and next steps

**NEVER:**
- Proceed without informing user
- Hide errors or failures
- Complete silently without summary
- Terminate before all phases complete

### Skill Coordination

When running autonomously, **IGNORE** any "Next:" commands in skill outputs. Skills may suggest manual commands, but the orchestrator must automatically continue to the next phase.

## Testing the Workflow

```bash
# Try a simple feature
/sdlc add-health-check "Implement a health check endpoint"

# Monitor progress
cat docs/add-health-check/STATUS.md

# Review artifacts
ls docs/add-health-check/
```

## Important Constraints

1. **No traditional build system** - This is a configuration repository, not an application
2. **No package.json or dependencies** - Uses Claude Code's native capabilities
3. **No npm/pip/build commands** - Workflow orchestration via agents and skills only
4. **No production deployment** - Creates artifacts and documentation, not deployable code
5. **Issue directory organization** - All artifacts organized by issue name in `docs/`

## Modifying the System

When editing agents, skills, or commands:

1. **Maintain separation** - Don't add logic to commands, don't add state to skills
2. **Preserve determinism** - Same inputs should produce same outputs
3. **Update gate validation** - If adding phases, update gate checklists
4. **Test autonomous flow** - Ensure orchestrator continues through all phases
5. **Document changes** - Update relevant documentation files
6. **Validate artifacts** - Ensure both full and summary artifacts are created

## Common Patterns

### Adding a New Phase

1. Create skill in `.claude/skills/`
2. Add state transitions to orchestrator
3. Update gate validation checklists
4. Add artifact templates to templates/
5. Update STATE.json schema if needed

### Modifying Artifact Structure

1. Update templates in `.claude/templates/`
2. Update skill instructions to create new artifacts
3. Update gate validation to check new artifacts
4. Update STATE.json schema
5. Update summary artifact loading strategy

### Debugging Workflow Issues

1. Check STATUS.md for current phase and status
2. Check STATE.json for artifact tracking
3. Review relevant phase artifacts in `docs/{issue-name}/`
4. Check `.claude/TROUBLESHOOTING.md` for common issues
5. Review gate validation checklists in orchestrator

## Security Considerations

### Input Validation (Command Layer)

- Issue names: kebab-case, max 50 chars, no path traversal
- Feature descriptions: max 1000 chars, HTML stripped, command injection removed
- Shell metacharacters escaped: `;`, `|`, `&`, `$`, `(`, `)`, `<`, `>`

### Path Traversal Prevention

- Reject issue names containing `..`
- Reject issue names starting with `/` or `\\`
- Reject issue names containing null bytes
- Resolve paths and verify within expected directory

## Related Files

- `.claude/commands/COMMAND_USAGE.md` - Complete command reference
- `.claude/MIGRATION_REPORT.md` - Architecture migration history
- `.claude/examples/` - Reference artifacts for each phase
