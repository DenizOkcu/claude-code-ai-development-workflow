# Custom Command Usage Guide

This document explains how to use the custom slash commands in this repository.

## Command Overview

All planning and implementation commands now accept the **issue name** as the first argument. This allows you to work on multiple features simultaneously and keep artifacts organized.

## Command Syntax

### 1. `/research-code {issue-name} [description]`

Research the codebase to understand architecture and integration points.

**Examples:**
```bash
/research-code openai-post-processing Add OpenAI chat completions for structuring
/research-code fix-auth-bug Investigate authentication failure scenarios
/research-code dashboard-view Research patterns for dashboard implementation
```

**Output:**
- Creates `.claude/planning/{issue-name}/CODE_RESEARCH.md`
- Creates `.claude/planning/{issue-name}/STATUS.md`

---

### 2. `/issue-planner {issue-name} [description]`

Create implementation plan and project specification based on research.

**Examples:**
```bash
/issue-planner openai-post-processing
/issue-planner fix-auth-bug Add error handling for failed authentication
/issue-planner dashboard-view Implement user dashboard with analytics
```

**Output:**
- Creates `.claude/planning/{issue-name}/IMPLEMENTATION_PLAN.md`
- Creates `.claude/planning/{issue-name}/PROJECT_SPEC.md`
- Updates `.claude/planning/{issue-name}/STATUS.md`

---

### 3. `/execute-plan {issue-name}`

Execute the implementation plan systematically.

**Examples:**
```bash
/execute-plan openai-post-processing
/execute-plan fix-auth-bug
/execute-plan dashboard-view
```

**Output:**
- Implements code according to IMPLEMENTATION_PLAN.md
- Updates `.claude/planning/{issue-name}/STATUS.md` with progress

---

### 4. `/review-code {issue-name}`

Perform comprehensive code review with automated checks.

**Examples:**
```bash
/review-code openai-post-processing
/review-code fix-auth-bug
/review-code dashboard-view
```

**Output:**
- Creates `.claude/planning/{issue-name}/CODE_REVIEW.md`
- Updates `.claude/planning/{issue-name}/STATUS.md`

---

## Workflow Example

Here's a complete workflow for implementing a new feature:

```bash
# Step 1: Research the codebase
/research-code openai-post-processing Add OpenAI chat API for post-processing transcriptions

# Step 2: Create implementation plan
/issue-planner openai-post-processing

# Step 3: Implement the feature
/execute-plan openai-post-processing

# Step 4: Review the implementation
/review-code openai-post-processing
```

## File Organization

All artifacts are organized by issue name:

```
.claude/planning/
├── openai-post-processing/
│   ├── CODE_RESEARCH.md
│   ├── IMPLEMENTATION_PLAN.md
│   ├── PROJECT_SPEC.md
│   ├── CODE_REVIEW.md
│   └── STATUS.md
├── fix-auth-bug/
│   ├── CODE_RESEARCH.md
│   ├── IMPLEMENTATION_PLAN.md
│   └── STATUS.md
└── dashboard-view/
    ├── CODE_RESEARCH.md
    └── STATUS.md
```

## Issue Name Format

Issue names should be:
- **kebab-case** (lowercase with hyphens)
- **Concise** (2-5 words)
- **Descriptive** of the feature or fix

**Good examples:**
- `openai-post-processing`
- `fix-auth-bug`
- `dashboard-view`
- `add-user-settings`

**Bad examples:**
- `OpenAIPostProcessing` (not kebab-case)
- `fix` (too vague)
- `add-a-new-feature-for-user-authentication-with-oauth` (too long)

## Benefits

1. **Multiple features**: Work on several features simultaneously without conflicts
2. **Organization**: All artifacts for a feature are in one place
3. **History**: Easy to track progress and decisions for each feature
4. **Reusability**: Commands reference the same issue name throughout the workflow
5. **Clarity**: Clear separation between different features/issues

## Fallback Behavior

If you don't provide an issue name:
- Commands will try to detect the most recent planning directory
- If multiple exist, you'll be asked to specify which one
- Some commands will generate an issue name from your description

**Recommendation:** Always provide the issue name explicitly for clarity and consistency.
