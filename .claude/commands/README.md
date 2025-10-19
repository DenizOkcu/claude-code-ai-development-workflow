# Development Workflow Commands

Comprehensive slash commands for a complete software development lifecycle - from research to implementation.

## Quick Start

```bash
/research-code "your feature description"  # Creates CODE_RESEARCH.md + STATUS.md
/issue-planner "your feature description"  # Creates plans + updates STATUS.md
/execute-plan                             # Implements + updates STATUS.md
/review-code                              # Reviews + updates STATUS.md
```

**All commands automatically update `STATUS.md`** - your centralized progress tracker that shows exactly where you are in the workflow.

## ⚠️ Important: Delete Example Before Using

**If you're using this workflow in your own repository, delete the `.claude/planning/fix-terminal-flicker/` directory first.**

This directory contains example artifacts to help you understand what the workflow produces. It should NOT be used as part of your actual workflow. The commands will create new planning directories automatically for your real features/issues.

## Command Organization

Commands are organized into categories:

- **Workflow Commands** - Core development lifecycle commands (research, planning, execution, review)
- **Language Commands** (`language/`) - Language-specific expert commands (TypeScript, etc.)

## Workflow Commands

### 1. `/research-code` - Codebase Research

**Purpose:** Deep analysis of your codebase before planning

**What it does:**

- Investigates architecture, patterns, and conventions
- Identifies integration points and dependencies
- Finds similar existing implementations
- Assesses risks and technical debt
- Creates recommendations
- **Creates STATUS.md** to track workflow progress

**Output:**

- `CODE_RESEARCH.md` - Detailed research findings
- `STATUS.md` - **Progress tracker (created here, updated by all other commands)**

**Example:**

```bash
/research-code Add rate limiting middleware with Redis
```

**When to use:**

- Before implementing any new feature
- When unfamiliar with the codebase area
- For complex features requiring architectural understanding

**STATUS.md after this command:**

```
- [x] Research - Completed
- [ ] Planning - Not started
```

---

### 2. `/issue-planner` - Implementation Planning

**Purpose:** Transform requirements into actionable plans

**What it does:**

- Reads CODE_RESEARCH.md (if exists) and STATUS.md
- Creates detailed implementation plan with phases
- Designs technical architecture and specifications
- Plans testing strategy
- Documents requirements and design decisions
- **Updates STATUS.md** with planning completion

**Output:**

- `IMPLEMENTATION_PLAN.md` - Phased implementation strategy
- `PROJECT_SPEC.md` - Complete technical specification
- Updates `STATUS.md`

**Example:**

```bash
/issue-planner Add OAuth2 authentication with Google and GitHub providers
```

**When to use:**

- After research phase
- When starting any non-trivial feature
- When you need structured planning

**STATUS.md after this command:**

```
- [x] Research - Completed
- [x] Planning - Completed
- [ ] Implementation - Not started
```

---

### 3. `/execute-plan` - Implementation

**Purpose:** Systematically implement the planned feature

**What it does:**

- Reads IMPLEMENTATION_PLAN.md, PROJECT_SPEC.md, and STATUS.md
- Creates comprehensive todo list
- **Updates STATUS.md** to "In Progress" with current phase
- Implements code phase by phase
- Writes tests alongside implementation
- Tracks progress with todo list
- Validates each phase before proceeding
- **Updates STATUS.md** when complete with implementation summary

**Output:**

- Implementation code
- Tests
- Progress tracking
- Updates `STATUS.md`

**Example:**

```bash
/execute-plan
```

**When to use:**

- After planning phase
- When IMPLEMENTATION_PLAN.md exists
- For systematic feature implementation

**STATUS.md during execution:**

```
- [x] Research - Completed
- [x] Planning - Completed
- [~] Implementation - In Progress (Phase 2/4)
- [ ] Review - Not started
```

**STATUS.md after completion:**

```
- [x] Research - Completed
- [x] Planning - Completed
- [x] Implementation - Completed (8 files, 23 tests)
- [ ] Review - Not started
```

---

### 4. `/review-code` - Quality Assurance

**Purpose:** Comprehensive code review and quality checks

**What it does:**

- Reads STATUS.md, plans, and research docs for context
- **Updates STATUS.md** to "Review In Progress"
- Runs all automated checks (linting, types, tests, build)
- Performs manual code review
- Security analysis
- Performance assessment
- Compliance verification against plan
- Creates detailed review report
- **Updates STATUS.md** with approval status

**Output:**

- `CODE_REVIEW.md` - Detailed review report
- Updates `STATUS.md` with APPROVED/NEEDS REVISION status

**Example:**

```bash
/review-code
```

**When to use:**

- After implementation
- Before committing/deploying
- After fixing issues (re-run for verification)

**STATUS.md after this command (if approved):**

```
- [x] Research - Completed
- [x] Planning - Completed
- [x] Implementation - Completed
- [x] Review - Completed ✓ APPROVED
- [ ] Deployment - Ready
```

**STATUS.md if needs work:**

```
- [x] Research - Completed
- [x] Planning - Completed
- [x] Implementation - Completed
- [x] Review - ⚠ NEEDS REVISION (2 critical, 3 important issues)
```

---

## Language Commands

Language-specific expert commands for advanced development patterns.

### `/typescript-pro` - TypeScript Expert

**Location:** `language/typescript-pro.md`

**Purpose:** Advanced TypeScript development and architecture

**What it does:**

- Provides TypeScript expert guidance
- Implements advanced type systems
- Optimizes TypeScript configuration
- Designs generic and utility types
- Ensures strict type safety

**Example:**

```bash
/typescript-pro Design a type-safe event system with typed handlers
```

**When to use:**

- Complex TypeScript challenges
- Type system design
- TypeScript architecture decisions
- Advanced typing patterns

---

## Complete Workflow Example

### Scenario: Add user authentication feature

```bash
# Step 1: Research
/research-code Add JWT-based authentication with refresh tokens
# Output:
#   - CODE_RESEARCH.md (detailed findings)
#   - STATUS.md (created with initial progress)
# Findings:
#   - Found existing auth patterns in src/auth/
#   - Identified integration points
#   - Noted security considerations
# STATUS.md now shows: [x] Research | [ ] Planning | [ ] Implementation | [ ] Review

# Step 2: Plan
/issue-planner Add JWT-based authentication with refresh tokens
# Output:
#   - IMPLEMENTATION_PLAN.md (4 phases: Setup, Core Auth, Refresh Logic, Testing)
#   - PROJECT_SPEC.md (technical design with interfaces, security requirements)
#   - STATUS.md updated
# STATUS.md now shows: [x] Research | [x] Planning | [ ] Implementation | [ ] Review

# Step 3: Review plan (optional)
# Read the generated plans in STATUS.md, CODE_RESEARCH.md, IMPLEMENTATION_PLAN.md
# Check STATUS.md to see what's been completed and what's next

# Step 4: Implement
/execute-plan
# During execution:
#   - STATUS.md shows: [x] Research | [x] Planning | [~] Implementation (Phase 2/4) | [ ] Review
# After completion:
#   - Complete implementation with tests
#   - STATUS.md updated with: 8 files modified, 23 tests written, all passing
# STATUS.md now shows: [x] Research | [x] Planning | [x] Implementation | [ ] Review

# Step 5: Quality Check
/review-code
# Output:
#   - CODE_REVIEW.md (comprehensive review report)
#   - STATUS.md updated with approval status
# STATUS.md now shows: [x] Research | [x] Planning | [x] Implementation | [x] Review ✓ APPROVED

# Step 6: Check STATUS.md
cat STATUS.md
# See complete summary of all phases, artifacts, and approval status

# Step 7: Fix any issues (if needed, if review found issues)
# Address issues found in review
/review-code  # Re-run to verify fixes
# STATUS.md will update to show re-review results

# Step 8: Commit and deploy
git add .
git commit -m "feat: add JWT authentication with refresh tokens"
git push
```

### What STATUS.md Looks Like Throughout

**After Research:**

```markdown
# Development Status

**Feature:** Add JWT-based authentication with refresh tokens
**Started:** 2025-10-11 10:30 AM

## Workflow Progress

- [x] Research - Completed
- [ ] Planning - Not started
- [ ] Implementation - Not started
- [ ] Review - Not started
```

**After Planning:**

```markdown
## Workflow Progress

- [x] Research - Completed
- [x] Planning - Completed
- [ ] Implementation - Not started
- [ ] Review - Not started
```

**During Implementation:**

```markdown
## Workflow Progress

- [x] Research - Completed
- [x] Planning - Completed
- [~] Implementation - In Progress (Phase 2/4)
- [ ] Review - Not started
```

**After Review (Approved):**

```markdown
## Workflow Progress

- [x] Research - Completed
- [x] Planning - Completed
- [x] Implementation - Completed
- [x] Review - Completed ✓ APPROVED
- [ ] Deployment - Ready

## Artifacts Created

- ✓ CODE_RESEARCH.md
- ✓ IMPLEMENTATION_PLAN.md
- ✓ PROJECT_SPEC.md
- ✓ Implementation code (8 files)
- ✓ Tests (23 tests, all passing)
- ✓ CODE_REVIEW.md
```

## Artifacts Created

Each command creates specific documentation:

| Command          | Artifact                 | Contains                                                        |
| ---------------- | ------------------------ | --------------------------------------------------------------- |
| `/research-code` | `CODE_RESEARCH.md`       | Architecture analysis, patterns, integration points, risks      |
| `/research-code` | `STATUS.md`              | **Workflow progress tracker (created/updated by all commands)** |
| `/issue-planner` | `IMPLEMENTATION_PLAN.md` | Phased implementation strategy, tasks, timeline                 |
| `/issue-planner` | `PROJECT_SPEC.md`        | Technical requirements, design, architecture                    |
| `/execute-plan`  | -                        | Implementation code and tests                                   |
| `/review-code`   | `CODE_REVIEW.md`         | Review findings, issues, approval status                        |

### STATUS.md - Your Progress Dashboard

**All commands update `STATUS.md`** to track your progress through the workflow. This file shows:

- ✓ Which phases are completed
- 🔄 What phase is currently in progress
- ⏳ What's next
- 📋 All artifacts created
- 🔑 Key findings and decisions from each phase
- 📊 Current status (approved, blocked, in progress, etc.)

**Example STATUS.md:**

```markdown
# Development Status

**Feature:** Add JWT authentication with refresh tokens
**Started:** 2025-10-11 10:30 AM
**Last Updated:** 2025-10-11 2:45 PM

## Workflow Progress

- [x] **Research** - Completed
- [x] **Planning** - Completed
- [x] **Implementation** - Completed
- [x] **Review** - Completed ✓ APPROVED
- [ ] **Deployment** - Not started

## Phase Details

### 1. Research (✓ Completed)

- **Risk Level:** Medium
- **Key Findings:** Found existing auth patterns in src/auth/

### 2. Planning (✓ Completed)

- **Phases Planned:** 4
- **Estimated Complexity:** Medium

### 3. Implementation (✓ Completed)

- **Files Modified:** 8
- **Tests Written:** 23
- **Test Status:** All passing

### 4. Review (✓ Completed - APPROVED)

- **Status:** APPROVED
- **Critical Issues:** 0
- **Next Command:** /deploy-release

## Artifacts Created

- ✓ CODE_RESEARCH.md
- ✓ IMPLEMENTATION_PLAN.md
- ✓ PROJECT_SPEC.md
- ✓ Implementation code
- ✓ Tests
- ✓ CODE_REVIEW.md
```

**Benefits:**

- 📍 Always know where you are in the workflow
- 📝 Single source of truth for project status
- 🔄 Easy to resume work after interruptions
- 👥 Share progress with team members
- 📊 Track decisions and key information

## Using STATUS.md to Track Progress

### Quick Status Check

```bash
# At any time, check your progress
cat STATUS.md

# Or open it in your editor
code STATUS.md
```

### Resume Work After Interruption

```bash
# Been away for a few days? Check STATUS.md first
cat STATUS.md

# It will show:
# - What phase you're in
# - What's been completed
# - What's next
# - All artifacts created
# - Key decisions made

# Then continue with the recommended next command
```

### Share Progress with Team

```bash
# Share STATUS.md to show complete project status
# It contains:
# - Timeline (started, last updated)
# - All phases with completion status
# - Key findings from each phase
# - Current blockers (if any)
# - List of all artifacts
```

### Track Multiple Features

```bash
# For different features, use different directories
mkdir feature-auth && cd feature-auth
/research-code "Add authentication"
# STATUS.md created in feature-auth/

mkdir feature-payments && cd feature-payments
/research-code "Add payment processing"
# STATUS.md created in feature-payments/
```

## Tips & Best Practices

### Always Check STATUS.md First

- Before starting work: `cat STATUS.md`
- Shows exactly where you are
- Tells you what command to run next
- Provides context on all previous decisions

### When to Skip Research

- Trivial changes (typo fixes, simple updates)
- Very familiar code areas
- Changes with clear implementation path

### When Research is Critical

- New features in unfamiliar areas
- Complex architectural changes
- Integration with existing systems
- Major refactoring

### Iterative Workflow

You can iterate at any stage. STATUS.md tracks everything:

```bash
# If planning needs adjustment:
/issue-planner   # Update plans (STATUS.md shows re-planning)
/execute-plan    # Re-implement

# If code needs fixes:
# Fix the code
/review-code     # Re-verify (STATUS.md shows re-review attempt)

# STATUS.md keeps history of iterations
```

### Using with Git

```bash
# Before starting
git checkout -b feature/your-feature-name

# After implementation and review passes
git add .
git commit -m "feat: your feature description"
git push origin feature/your-feature-name
```

### Combining Commands

For TypeScript-heavy features:

```bash
/research-code "Add type-safe event bus"
/typescript-pro "Design event bus type system"  # Get expert TypeScript guidance
/issue-planner "Add type-safe event bus"        # Plan with TypeScript focus
/execute-plan                                   # Implement
/review-code                                    # Verify
```

## Command Selection Guide

**Simple Change?**

- Just code it directly, no commands needed

**Medium Complexity?**

```bash
/issue-planner   # Plan it
/execute-plan    # Build it
/review-code     # Check it
```

**Complex/Unfamiliar?**

```bash
/research-code   # Understand it
/issue-planner   # Plan it
/execute-plan    # Build it
/review-code     # Check it
```

**TypeScript Challenge?**

```bash
/typescript-pro  # Get expert guidance
```

## Workflow Benefits

✅ **Structured** - Clear phases prevent getting lost
✅ **Documented** - All decisions and findings preserved in STATUS.md
✅ **Quality** - Built-in review and testing
✅ **Traceable** - STATUS.md + todo lists track complete progress
✅ **Informed** - Research ensures good integration
✅ **Maintainable** - Following existing patterns
✅ **Resumable** - STATUS.md lets you resume after any interruption
✅ **Transparent** - STATUS.md shows complete workflow state
✅ **Collaborative** - Share STATUS.md to communicate progress

## The Power of STATUS.md

**Single Source of Truth:**

- 📍 Current phase and progress
- 📋 All artifacts created
- 🔑 Key decisions and findings
- ⚠️ Blockers and issues
- ⏭️ Next recommended step
- 📊 Complete timeline

**Always Up-to-Date:**

- Every command reads and updates it
- Real-time reflection of workflow state
- No manual tracking needed

**Resume Anytime:**

- Come back after days/weeks
- Read STATUS.md to understand context
- Pick up exactly where you left off

## Next Steps

Start with a simple feature to try the workflow:

```bash
/research-code "Add health check endpoint"
```

Each command will:

1. Update STATUS.md with its progress
2. Guide you to the next step
3. Maintain complete workflow transparency

**Check STATUS.md at any time** to see where you are!
