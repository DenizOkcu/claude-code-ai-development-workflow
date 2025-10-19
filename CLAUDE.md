# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains a structured workflow system for software development using Claude slash commands. It implements a complete development lifecycle: Research → Planning → Implementation → Review.

## Architecture

### Slash Command System

The repository uses `.claude/commands/` to define custom slash commands that form a development workflow:

**Core Workflow Commands:**
- `/research-code` - Deep codebase analysis and architecture research
- `/issue-planner` - Transform requirements into implementation plans
- `/execute-plan` - Systematic implementation following documented plans
- `/review-code` - Code review, quality checks, and approval

**Language-Specific Commands:**
- `/typescript-pro` (in `language/` subdirectory) - TypeScript expert guidance

### File Organization Structure

When you execute workflow commands, they automatically create planning artifacts in `.claude/planning/{issue-name}/` directories:

```
.claude/
├── commands/           # Slash command definitions
│   ├── research-code.md
│   ├── issue-planner.md
│   ├── execute-plan.md
│   ├── review-code.md
│   ├── language/
│   │   └── typescript-pro.md
│   └── README.md       # Comprehensive workflow documentation
└── planning/
    └── {issue-name}/   # Created automatically per feature/issue
        ├── STATUS.md            # Workflow progress tracker
        ├── CODE_RESEARCH.md     # Research findings
        ├── IMPLEMENTATION_PLAN.md
        ├── PROJECT_SPEC.md
        └── CODE_REVIEW.md
```

**Note:** The commands create these directories automatically. You don't need to create them manually.

### Issue Name Convention

Issue names are generated from feature descriptions using kebab-case:
- "Fix terminal flickering" → `fix-terminal-flickering`
- "Add user authentication" → `add-user-authentication`
- Keep concise (2-5 words)

### STATUS.md - The Central Progress Tracker

Every issue directory contains a `STATUS.md` file that:
- Tracks workflow progress through all phases
- Shows which artifacts have been created
- Documents key findings and decisions from each phase
- Indicates next recommended command
- Updates automatically as commands execute

## Development Workflow

### Standard Process

```bash
# 1. Research (optional for simple changes)
/research-code "feature description"
# Creates: .claude/planning/{issue-name}/CODE_RESEARCH.md + STATUS.md

# 2. Plan
/issue-planner "feature description"
# Creates: IMPLEMENTATION_PLAN.md + PROJECT_SPEC.md
# Updates: STATUS.md

# 3. Implement
/execute-plan
# Implements code following the plan
# Updates: STATUS.md with progress

# 4. Review
/review-code
# Creates: CODE_REVIEW.md
# Updates: STATUS.md with approval status
```

### When to Use Each Command

**Skip Research for:**
- Simple changes or bug fixes
- Very familiar code areas
- Trivial updates

**Always Research for:**
- New features in unfamiliar areas
- Complex architectural changes
- Integration with existing systems

## Workflow Command Behavior

### Research Phase
- Investigates codebase architecture and patterns
- Identifies integration points and dependencies
- Assesses risks and technical debt
- Creates `CODE_RESEARCH.md` with findings
- Initializes `STATUS.md` with workflow tracking

### Planning Phase
- Reads `CODE_RESEARCH.md` (if exists) for context
- Reads `STATUS.md` for current state
- Creates detailed `IMPLEMENTATION_PLAN.md` with phases
- Creates complete `PROJECT_SPEC.md` with technical design
- Updates `STATUS.md` to show planning completion

### Implementation Phase
- Reads all planning artifacts: `IMPLEMENTATION_PLAN.md`, `PROJECT_SPEC.md`, `STATUS.md`
- Creates comprehensive TodoWrite list from implementation phases
- Systematically implements code phase by phase
- Marks todos as in_progress → completed in real-time
- Updates `STATUS.md` showing current phase and progress
- Updates `STATUS.md` with final implementation summary

### Review Phase
- Reads all artifacts for context: `STATUS.md`, plans, research docs
- Runs automated checks: linting, type checking, tests, build
- Performs manual code review for quality, security, performance
- Creates `CODE_REVIEW.md` with detailed findings
- Updates `STATUS.md` with approval status (APPROVED / NEEDS REVISION)

## Key Principles

### Artifact Organization
- **Always** use `.claude/planning/{issue-name}/` structure
- Each feature/issue gets its own directory
- All related artifacts stay together
- `STATUS.md` is the single source of truth

### STATUS.md Management
- Commands automatically read and update `STATUS.md`
- Shows complete workflow state at all times
- Check `STATUS.md` first to understand project status
- Use to resume work after interruptions

### Progressive Workflow
- Each phase builds on previous artifacts
- Commands read context from prior phase documents
- Can iterate: run planning again, re-implement, re-review
- `STATUS.md` tracks all iterations

## Important: Ignore Example Directory

**The `.claude/planning/fix-terminal-flicker/` directory should be IGNORED.** It's an example for repository readers and should be deleted before using this workflow system in your own repository.

When you run the workflow commands, they will create new planning directories automatically for your actual features/issues. Start fresh - don't try to read or use the example artifacts.

## Relationship Between Commands and Artifacts

| Command | Reads | Creates/Updates |
|---------|-------|-----------------|
| `/research-code` | Codebase files | `CODE_RESEARCH.md`, `STATUS.md` (new) |
| `/issue-planner` | `CODE_RESEARCH.md`, `STATUS.md` | `IMPLEMENTATION_PLAN.md`, `PROJECT_SPEC.md`, updates `STATUS.md` |
| `/execute-plan` | All planning docs, `STATUS.md` | Implementation code, tests, updates `STATUS.md` |
| `/review-code` | All docs, implementation, `STATUS.md` | `CODE_REVIEW.md`, updates `STATUS.md` |

## Best Practices

### Using STATUS.md Effectively
- Always check `STATUS.md` before starting work
- Use it to understand where you are in the workflow
- Share it to communicate progress
- Rely on it to resume after interruptions

### Todo List Management
- `/execute-plan` creates comprehensive todo lists via TodoWrite
- Exactly ONE task should be in_progress at a time
- Mark tasks completed immediately after finishing
- Don't batch completions

### Iterative Development
- Can re-run any command to update/refine
- Re-plan if requirements change
- Re-implement if approach changes
- Re-review after fixing issues
- `STATUS.md` tracks all iterations

## Command Documentation

Complete workflow documentation with detailed examples is in `.claude/commands/README.md`.
