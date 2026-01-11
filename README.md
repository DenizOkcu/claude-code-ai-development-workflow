# AI Development Workflow with Claude Code

A modern autonomous software development lifecycle (SDLC) system using the `/sdlc` unified command for complete feature development: Research → Planning → Implementation → Review.

## Quick Start

```bash
# Execute complete SDLC autonomously
/sdlc add-oauth-login "Implement OAuth2 authentication with Google"

# Check progress anytime
cat docs/add-oauth-login/STATUS.md
```

**The `/sdlc` command autonomously executes all phases:**
1. **Research** - Analyzes codebase architecture and patterns
2. **Planning** - Creates implementation plans and specifications
3. **Implementation** - Builds the feature phase-by-phase
4. **Review** - Runs automated checks and manual review
5. **Review-Fix Loop** (if needed) - Addresses issues automatically

---

## Architecture

### 2026 Claude Code Architecture

```
┌─────────────────────────────────────────────┐
│              USER INTERFACE                 │
│           ┌──────────────────┐              │
│           │  Unified Command │              │
│           │      /sdlc       │              │
│           │  (Recommended)   │              │
│           └────────┬─────────┘              │
│                    │                        │
└────────────────────┼────────────────────────┘
                     │
                     │ Invoke Agent
                     ▼
┌─────────────────────────────────────────────┐
│         SDLC Orchestrator Agent             │
│  ┌───────────────────────────────────────┐  │
│  │ • Phase state machine                 │  │
│  │ • Gate enforcement                    │  │
│  │ • Review-fix loop (max 3)             │  │
│  │ • State persistence (STATUS.md)       │  │
│  │ • Diagnostic output                   │  │
│  └───────────────────────────────────────┘  │
└───────────────────┬─────────────────────────┘
                    │
                    │ Invokes Skills (sequentially)
                    │
    ┌───────────────┼───────────────┐
    │               │               │
    ▼               ▼               ▼
┌──────────────┐ ┌──────────┐ ┌──────────────┐
│ code-research│ │solution- │ │ code-        │
│              │ │planning  │ │implementation│
└──────────────┘ └──────────┘ └──────────────┘
                                        │
        ┌───────────────┐               │
        ▼               ▼               ▼
┌──────────────┐ ┌──────────┐ ┌──────────────┐
│ code-review  │ │review-fix│ │   State      │
│              │ │          │ │ STATUS.md    │
└──────────────┘ └──────────┘ └──────────────┘
        │
        ▼
┌─────────────────────────────────────────┐
│         Artifacts (Persistent)          │
│  docs/{issue-name}/                     │
│  ├── STATUS.md                          │
│  ├── CODE_RESEARCH.md                   │
│  ├── IMPLEMENTATION_PLAN.md             │
│  ├── PROJECT_SPEC.md                    │
│  └── CODE_REVIEW.md                     │
└─────────────────────────────────────────┘
```

### Component Responsibilities

**User Interface Layer**
- **Unified Command (`/sdlc`):** Parse arguments, invoke SDLC Orchestrator Agent, no workflow logic

**Orchestration Layer**
- **SDLC Orchestrator Agent:** Execute full SDLC autonomously, invoke skills, enforce gates, manage state

**Capabilities Layer**
- **Skills (Stateless Procedures):**
  - `code-research`: Investigate codebase, create CODE_RESEARCH.md
  - `solution-planning`: Create IMPLEMENTATION_PLAN.md and PROJECT_SPEC.md
  - `code-implementation`: Execute plan phase-by-phase
  - `code-review`: QA checks, create CODE_REVIEW.md
  - `review-fix`: Address review issues

**Data Layer**
- **Artifacts:** Research, plans, specs, implementations, reviews
- **State:** STATUS.md (progress), STATE.json (machine-readable)

---

## Usage

### `/sdlc <issue-name> [description] [--from-<phase>]`

**Unified SDLC command** - Executes the entire software development lifecycle autonomously.

#### Syntax

```bash
/sdlc <issue-name> [feature-description] [--from-<phase>]
```

#### Arguments

- `issue-name` (required): Kebab-case identifier
  - Example: `add-oauth-auth`, `fix-memory-leak`, `refactor-api-layer`

- `feature-description` (optional when resuming): What to build
  - Example: "Implement OAuth2 with Google and GitHub"

- `--from-<phase>` (optional): Resume from specific phase
  - `--from-start`: Execute full SDLC (default)
  - `--from-plan`: Resume from planning phase
  - `--from-implement`: Resume from implementation phase
  - `--from-review`: Resume from review phase

#### Examples

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

#### What It Creates

All artifacts organized in `docs/{issue-name}/`:
- `STATUS.md` - Progress tracker (single source of truth)
- `CODE_RESEARCH.md` - Research findings
- `IMPLEMENTATION_PLAN.md` - Phased implementation strategy
- `PROJECT_SPEC.md` - Technical specification
- `CODE_REVIEW.md` - Review findings

#### How It Works

1. **Research Phase**: Investigates codebase architecture and patterns
2. **Planning Phase**: Creates implementation plans and specs
3. **Implementation Phase**: Executes code following the plan
4. **Review Phase**: Runs automated checks and manual review
5. **Review-Fix Loop** (if needed): Addresses issues (max 3 iterations)

Each phase must pass validation before progressing. The agent manages all state transitions automatically.

#### Check Progress

```bash
cat docs/{issue-name}/STATUS.md
```

STATUS.md shows:
- Current phase
- Progress indicators
- Artifacts created
- Next steps

#### Issue Name Format

**Rules:**
- kebab-case (lowercase-with-hyphens)
- Concise (2-5 words)
- Descriptive

**Examples:**
- ✅ `add-oauth-auth`, `fix-memory-leak`, `refactor-api-layer`
- ❌ `AddOAuthAuth` (not kebab-case), `fix` (too vague), `add-new-auth-system-with-oauth2` (too long)

#### Complete Workflow Example

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

## Benefits

- **Single command** - Execute entire SDLC with one invocation
- **Autonomous execution** - No manual command switching required
- **Multi-feature support** - Work on multiple issues simultaneously
- **Organized artifacts** - All docs for a feature in one directory
- **Progress tracking** - STATUS.md shows complete workflow state
- **Quality gates** - Automated validation at each phase
- **Self-healing** - Automatic review-fix loop for issues

---

## File Organization

All artifacts are organized by issue name in `docs/{issue-name}/`:

```
docs/
├── add-oauth-auth/         # Issue directory
│   ├── STATUS.md           # ⭐ Central progress tracker
│   ├── CODE_RESEARCH.md    # Research findings
│   ├── IMPLEMENTATION_PLAN.md
│   ├── PROJECT_SPEC.md
│   └── CODE_REVIEW.md
└── fix-memory-leak/
    ├── STATUS.md
    └── CODE_RESEARCH.md
```

**STATUS.md** = Single source of truth for workflow state, updated by all phases.

---

## STATUS.md - Your Progress Dashboard

**All phases update `STATUS.md`** to track workflow progress:

- ✓ Which phases are completed
- 🔄 What phase is currently in progress
- ⏳ What's next
- 📋 All artifacts created
- 🔑 Key findings and decisions
- 📊 Current status (approved/blocked/in progress)

**Example STATUS.md:**

```markdown
# Status: add-jwt-auth

**Risk:** Medium | **Updated:** 2025-11-08 2:45 PM

## Progress

- [x] Research - Completed
- [x] Planning - Completed
- [x] Implementation - Completed
- [x] Review - Completed ✓ APPROVED
- [ ] Deployment - Ready

## Phase: Review ✓ APPROVED

- **Critical Issues:** 0
- **Important Issues:** 0
- **Suggestions:** 3
- **Next:** Deploy or commit changes

## Artifacts

- CODE_RESEARCH.md
- IMPLEMENTATION_PLAN.md
- PROJECT_SPEC.md
- Implementation code (8 files)
- Tests (23 tests, all passing)
- CODE_REVIEW.md
```

**Benefits:**
- 📍 Always know where you are
- 📝 Single source of truth
- 🔄 Resume after interruptions
- 👥 Share progress with team
- 📊 Track all decisions

**Quick status check:**
```bash
cat docs/{issue-name}/STATUS.md
```

---

## Quality Invariants

1. **No logic in command**: Command only parses and invokes agent
2. **Stateless skills**: Skills contain procedures, not state
3. **Agent orchestrates**: Only agent manages workflow state
4. **Persistent artifacts**: All progress in `docs/`
5. **Deterministic gates**: Clear validation before progression
6. **Max iterations**: Review-fix limited to 3 attempts
7. **Clear separation**: UX, orchestration, capabilities, data

---

## Additional Documentation

- **Command syntax reference:** `.claude/commands/COMMAND_USAGE.md`
- **Architecture overview:** `.claude/ARCHITECTURE.md`
- **State management:** `.claude/STATE_MANAGEMENT.md`
- **Migration report:** `.claude/MIGRATION_REPORT.md`
- **Artifact templates:** `.claude/templates/ARTIFACT_TEMPLATES.md`
- **Example artifacts:** `.claude/examples/` (reference only)

---

## Next Steps

Try the workflow with a simple feature:

```bash
/sdlc add-health-check "Implement a health check endpoint"
```

This will:
1. Research your codebase for health check patterns
2. Create an implementation plan
3. Implement the health check endpoint
4. Run automated checks and review
5. Report completion with deployment guidance

**Check STATUS.md anytime to see where you are!**
