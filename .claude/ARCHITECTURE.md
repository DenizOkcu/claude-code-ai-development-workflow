# Architecture Overview: 2026 Claude Code SDLC System

## Visual Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           USER INTERFACE                                 │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐      │
│  │ Legacy Commands  │  │  Unified Command │  │   Language       │      │
│  │  /research-code  │  │      /sdlc       │  │  /typescript-pro │      │
│  │  /issue-planner  │  │                  │  │                  │      │
│  │  /execute-plan   │  │  (Recommended)   │  │  (Unchanged)     │      │
│  │  /review-code    │  │                  │  │                  │      │
│  └────────┬─────────┘  └────────┬─────────┘  └──────────────────┘      │
│           │                     │                                       │
└───────────┼─────────────────────┼───────────────────────────────────────┘
            │                     │
            │                     │ Invoke Agent
            │                     ▼
    ┌─────────────────────────────────────────────┐
    │         SDLC Orchestrator Agent              │
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
│  docs/{issue-name}/         │
│  ├── STATUS.md                          │
│  ├── CODE_RESEARCH.md                   │
│  ├── IMPLEMENTATION_PLAN.md             │
│  ├── PROJECT_SPEC.md                    │
│  └── CODE_REVIEW.md                     │
└─────────────────────────────────────────┘
```

## Component Responsibilities

### User Interface Layer

**Legacy Commands (Thin Wrappers):**
- Parse arguments
- Invoke skills directly
- No workflow logic
- 30-60 lines each

**Unified Command (`/sdlc`):**
- Parse arguments and flags
- Invoke SDLC Orchestrator Agent
- No workflow logic
- Single entry point

### Orchestration Layer

**SDLC Orchestrator Agent:**
- Execute full SDLC autonomously
- Invoke skills in sequence
- Enforce phase gates
- Manage review-fix loop
- Track state (STATUS.md)
- Provide diagnostics

### Capabilities Layer

**Skills (Stateless Procedures):**
- **code-research**: Investigate codebase, create CODE_RESEARCH.md
- **solution-planning**: Create IMPLEMENTATION_PLAN.md and PROJECT_SPEC.md
- **code-implementation**: Execute plan phase-by-phase
- **code-review**: QA checks, create CODE_REVIEW.md
- **review-fix**: Address review issues

### Data Layer

**Artifacts:**
- STATUS.md: Progress tracking
- CODE_RESEARCH.md: Research findings
- IMPLEMENTATION_PLAN.md: Phased strategy
- PROJECT_SPEC.md: Technical spec
- CODE_REVIEW.md: Review findings

---

## Flow Diagrams

### Legacy Workflow (Step-by-Step)

```
User → /research-code → code-research skill → CODE_RESEARCH.md
       ↓
User → /issue-planner → solution-planning skill → IMPLEMENTATION_PLAN.md
                                               → PROJECT_SPEC.md
       ↓
User → /execute-plan → code-implementation skill → Code + Tests
       ↓
User → /review-code → code-review skill → CODE_REVIEW.md
                                        ↓
                                    [APPROVED?]
                                        │
                    ┌───────────────────┴───────────────────┐
                   NO                                       YES
                    │                                        │
                    ↓                                        ↓
            review-fix skill                         Deploy/Commit
                    │
                    ↓
            /review-code (retry)
```

### New Workflow (Autonomous)

```
User → /sdlc <issue-name> [description] [--from-phase]
        ↓
SDLC Orchestrator Agent
        ↓
┌─────────────────────────────────────┐
│ research phase (code-research)      │ → CODE_RESEARCH.md
├─────────────────────────────────────┤
│ plan phase (solution-planning)      │ → IMPLEMENTATION_PLAN.md
│                                     │ → PROJECT_SPEC.md
├─────────────────────────────────────┤
│ implement phase (code-implementation)│ → Code + Tests
├─────────────────────────────────────┤
│ review phase (code-review)          │ → CODE_REVIEW.md
└─────────────────────────────────────┘
        │
        ▼
   [APPROVED?]
        │
┌───────┴────────┐
│                │
NO               YES
│                │
▼                ▼
fix_loop    Deploy/Commit
(code-review)  (max 3 iterations)
```

---

## State Management

### STATUS.md Evolution

```markdown
# Research Phase
- [x] Research | [ ] Planning | [ ] Implementation | [ ] Review

# Planning Phase
- [x] Research | [x] Planning | [ ] Implementation | [ ] Review

# Implementation Phase
- [x] Research | [x] Planning | [x] Implementation | [ ] Review

# Review Phase
- [x] Research | [x] Planning | [x] Implementation | [x] Review

# Review-Fix Loop (if needed)
- [x] Research | [x] Planning | [x] Implementation | [~] Review (Iteration 2/3)
```

---

## Quality Invariants

1. **No logic in commands**: Commands only parse and invoke
2. **Stateless skills**: Skills contain procedures, not state
3. **Agent orchestrates**: Only agent manages workflow state
4. **Persistent artifacts**: All progress in `docs/`
5. **Deterministic gates**: Clear validation before progression
6. **Max iterations**: Review-fix limited to 3 attempts
7. **Clear separation**: UX, orchestration, capabilities, data

---

## Usage Comparison

| Task | Legacy (4 commands) | New (1 command) |
|------|---------------------|-----------------|
| Full workflow | `/research-code` → `/issue-planner` → `/execute-plan` → `/review-code` | `/sdlc feature-name "description"` |
| Resume from plan | `/issue-planner` → `/execute-plan` → `/review-code` | `/sdlc feature-name --from-plan` |
| Resume from review | `/review-code` | `/sdlc feature-name --from-review` |
| Fix review issues | Manual fixes → `/review-code` | `/sdlc feature-name --from-review` (auto loop) |

---

## Migration Path

### Phase 1: Current (Parallel Support)
- Both legacy and new workflows available
- Users can choose either approach
- Gather feedback on new workflow

### Phase 2: Transition (Recommended)
- Documentation promotes `/sdlc` as primary
- Legacy commands marked as "alternative"
- Migration guides provided

### Phase 3: Future (Deprecation)
- Legacy commands deprecated with notices
- `/sdlc` becomes standard workflow
- Backup files removed
