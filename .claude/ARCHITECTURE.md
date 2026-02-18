# Architecture Overview

Lean SDLC system aligned with Claude Code best practices.

## Visual Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      USER INTERFACE                          │
│                    /sdlc command                             │
│              (parse input, invoke agent)                     │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                   ORCHESTRATION LAYER                        │
│                  SDLC Orchestrator Agent                     │
│                                                              │
│  • Phase state machine: Research → Plan → Implement → Review│
│  • Gate enforcement                                          │
│  • Review-fix loop (max 3)                                   │
│  • State via STATUS.md                                       │
└─────────────────────────┬───────────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│researching-  │ │planning-     │ │implementing- │
│code          │ │solutions     │ │code          │
└──────────────┘ └──────────────┘ └──────────────┘
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
┌──────────────┐         ┌──────────────┐
│reviewing-    │         │fixing-review │
│code          │         │-issues       │
└──────────────┘         └──────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────────┐
│                      ARTIFACTS                               │
│                  docs/{issue-name}/                          │
│                                                              │
│  ├── STATUS.md           (progress tracker)                  │
│  ├── RESEARCH.md         (what we found)                     │
│  ├── PLAN.md             (what we'll build)                  │
│  ├── IMPLEMENTATION.md   (what we built)                     │
│  └── REVIEW.md           (is it ready?)                      │
└─────────────────────────────────────────────────────────────┘
```

## Component Responsibilities

### User Interface Layer

**Command (`/sdlc`):**
- Parse arguments
- Validate input
- Invoke agent
- No workflow logic

### Orchestration Layer

**SDLC Orchestrator Agent:**
- Execute phases sequentially
- Enforce gates between phases
- Manage review-fix loop
- Track state in STATUS.md
- Provide user notifications

### Capabilities Layer

**Skills (stateless procedures):**

| Skill | Mindset | Output |
|-------|---------|--------|
| researching-code | Understand minimum context | RESEARCH.md |
| planning-solutions | Define WHAT, not HOW | PLAN.md |
| implementing-code | Build working software | IMPLEMENTATION.md + code |
| reviewing-code | Is this deployable? | REVIEW.md |
| fixing-review-issues | Fix blocking issues only | Fixed code |

### Data Layer

**5 artifacts per issue:**

| File | Purpose | Size |
|------|---------|------|
| STATUS.md | Progress tracking | ~30 lines |
| RESEARCH.md | Research findings | ~50 lines |
| PLAN.md | Implementation plan | ~80 lines |
| IMPLEMENTATION.md | What was built | ~60 lines |
| REVIEW.md | Review findings | ~40 lines |

---

## Design Principles

### 1. Trust Claude Code

Claude Code handles:
- Context management (200K window)
- File reading on demand
- Session resumption

We don't need:
- Summary artifacts (removed)
- Pseudo-code state machines (removed)
- LLM parameter specs (removed)

### 2. Minimal Artifacts

**Before:** 8+ files per issue
**After:** 5 files per issue

Consolidated:
- CODE_RESEARCH.md + RESEARCH_SUMMARY.md → RESEARCH.md
- IMPLEMENTATION_PLAN.md + PROJECT_SPEC.md + PLAN_SUMMARY.md → PLAN.md
- IMPLEMENTATION_SUMMARY.md → IMPLEMENTATION.md
- CODE_REVIEW.md → REVIEW.md

### 3. Extended Thinking

Skills use extended thinking for complex reasoning:
- Research: "Think deeply about hidden dependencies"
- Planning: "Think harder about edge cases"
- Review: "Think a lot about security"

### 4. Lean Communication

Essential notifications only:
- Phase start
- Phase complete
- Errors/blockers
- Final completion

---

## Workflow State Machine

```
[START]
   │
   ▼
Research ──gate──▶ Plan ──gate──▶ Implement ──gate──▶ Review
                                                      │
                                          ┌───────────┴───────────┐
                                          │                       │
                                      APPROVED               NEEDS_FIX
                                          │                       │
                                          ▼                       ▼
                                      [COMPLETE]              Fix (max 3)
                                                                  │
                                                                  ▼
                                                               Review
```

### Gate Checks

| Gate | Check |
|------|-------|
| Research → Plan | 3 questions answered |
| Plan → Implement | Scope + phases + criteria defined |
| Implement → Review | All phases done + tests pass |
| Review → Complete | APPROVED verdict |

---

## Comparison: Before vs After

| Aspect | Before | After |
|--------|--------|-------|
| Artifacts | 8+ | 5 |
| Orchestrator | 860 lines | 250 lines |
| Skills | 400+ lines each | 100-200 lines each |
| LLM params | Specified | Trust Claude Code |
| Resume logic | Complex `--from-*` | Simple `--resume` |
| Communication | Extensive templates | Essential only |

---

## Quality Invariants

1. **No logic in command** - Only parsing and invocation
2. **Stateless skills** - Procedures, not state holders
3. **Agent orchestrates** - Only agent manages workflow
4. **Minimal artifacts** - 5 files, no duplication
5. **Trust the tool** - Claude Code handles context
6. **Extended thinking** - For complex reasoning
7. **Max 3 fix iterations** - Escalate if stuck
