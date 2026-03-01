---
name: sdlc-orchestrator
description: Autonomous SDLC orchestrator. Research → Plan → Implement → Review. Use for complete feature development.
---

# SDLC Orchestrator

Execute complete software development lifecycle autonomously.

## Workflow

```
Research → Plan → Implement → Review
                         ↓
                     Fix (if needed, max 3)
                         ↓
                       Review
```

## Core Rules

1. **Execute ALL phases** - don't stop early
2. **Continue automatically** - no manual commands between phases
3. **Fix loop max 3** - escalate if not resolved
4. **Trust STATUS.md** - it's the source of truth

## Model Routing

Each phase uses an appropriate model via the `model:` field in skill/command frontmatter:

| Phase | Model | Rationale |
|-------|-------|-----------|
| Research | opus | Deep architectural reasoning |
| Planning | opus | Phase sequencing, acceptance criteria |
| Implementation | opus | Multi-file code generation, testing |
| Review | sonnet | Checklist verification, pattern matching |
| Fix | sonnet | Targeted fixes from explicit instructions |

When running via `/sdlc` (model: sonnet), the orchestrator routes. When phases are invoked standalone (e.g., `/research`), the command's own `model:` field applies.

## Phase Execution

### Research Phase

**Skill:** `researching-code`

**Creates:**
- `docs/{issue_name}/RESEARCH.md`
- `docs/{issue_name}/STATUS.md`

**Gate:** RESEARCH.md answers:
- What files to touch?
- What patterns to follow?
- What are the risks?

**On completion:** Notify user, continue to Planning

### Planning Phase

**Skill:** `planning-solutions`

**Loads:** RESEARCH.md

**Creates:**
- `docs/{issue_name}/PLAN.md`

**Gate:** PLAN.md has:
- Clear scope
- 2-4 phases
- **Validation checklist per phase**
- Testable acceptance criteria

**On completion:** Notify user, continue to Implementation

### Implementation Phase

**Skill:** `implementing-code`

**Loads:** PLAN.md, RESEARCH.md

**Creates:**
- Implementation code
- Tests
- `docs/{issue_name}/IMPLEMENTATION.md`

**Gate:**
- ALL phases from PLAN.md complete
- **ALL phase validations from PLAN.md pass**
- Tests passing
- Acceptance criteria met
- IMPLEMENTATION.md created

**On completion:** Notify user, continue to Review

### Review Phase

**Skill:** `reviewing-code`

**Loads:** PLAN.md, IMPLEMENTATION.md, changed files

**Creates:**
- `docs/{issue_name}/REVIEW.md`

**Gate:** REVIEW.md verdict:
- **APPROVED** → Complete workflow
- **NEEDS_FIX** → Enter fix loop

### Fix Loop (if needed)

**Skill:** `review-fix`

**Loads:** REVIEW.md, changed files

**Creates:**
- Fixed code
- Updated REVIEW.md

**Rules:**
- Max 3 iterations
- Fix blocking issues only
- Return to Review after fixes

**If max iterations reached:** Mark BLOCKED, escalate to user

## User Communication

**Essential notifications only:**

**Start:**
```
🚀 SDLC: {issue_name}
{description}
Starting: Research
```

**Phase Complete:**
```
✅ {Phase} Complete
{key result}
→ Continuing to {next phase}
```

**Workflow Complete:**
```
🎉 Complete: {issue_name}

Phases: Research ✓, Plan ✓, Implement ✓, Review ✓
Files: {N} created, {M} modified
Tests: {N} passing

Suggested commit:
{commit message}
```

**Blocked:**
```
⚠️ Blocked at {phase}
{reason}

Resume with: /sdlc {issue_name} --resume
```

## State Management

### STATUS.md

Single source of truth for workflow state.

```markdown
# Status: {issue_name}

**Risk:** {level} | **Updated:** {timestamp}

## Progress
- [x] Research | [x] Planning | [~] Implementation | [ ] Review

## Phase: {current}
- **Status:** {in_progress|complete|blocked}
- **Next:** {next phase}

## Artifacts
- RESEARCH.md ✓
- PLAN.md ✓
- IMPLEMENTATION.md ✓
- REVIEW.md ✓
```

### STATE.json (optional)

Machine-readable state for tooling:
```json
{
  "issue_name": "add-oauth",
  "current_phase": "implementation",
  "phase_status": "in_progress",
  "review_iteration": 0,
  "artifacts": ["RESEARCH.md", "PLAN.md"]
}
```

## Resume Logic

When resuming with `--resume` or `--continue`:
1. Read STATUS.md
2. Identify current phase
3. Continue from where it stopped

## Entry Points

- **Default:** Start from Research
- **`--resume`:** Continue from STATUS.md
- **`--plan`:** Start from Planning (needs RESEARCH.md)
- **`--implement`:** Start from Implementation (needs PLAN.md)
- **`--review`:** Start from Review (needs IMPLEMENTATION.md)

## Error Handling

**Phase fails:**
1. Update STATUS.md with error
2. Notify user
3. Provide recovery options

**Fix loop maxed:**
1. Mark STATUS.md as BLOCKED
2. Create diagnostic summary
3. Suggest manual intervention

## Quality Gates Summary

| Phase | Required Artifact | Gate Check |
|-------|-------------------|------------|
| Research | RESEARCH.md | 3 questions answered |
| Planning | PLAN.md | Scope + phases + validation per phase + criteria |
| Implementation | IMPLEMENTATION.md + code | All phases done + validations pass + tests pass |
| Review | REVIEW.md | APPROVED verdict |
| Fix | Fixed code | Blocking issues resolved |

## Validation Flow

```
PLAN.md defines:
  Phase 1:
    - Tasks
    - Validation: [npm test, tsc, import check]
  Phase 2:
    - Tasks
    - Validation: [npm test, API works]
  Phase 3:
    - Tasks
    - Validation: [npm test, e2e works]

Implementer executes:
  For each Phase:
    1. Implement tasks
    2. Run Validation checklist
    3. All pass? → Next phase
    4. Any fail? → Fix, re-validate
```

## Completion Criteria

Workflow is complete ONLY when:
- [x] Research phase done
- [x] Planning phase done
- [x] Implementation phase done
- [x] Review phase done with APPROVED
- [x] STATUS.md shows complete
- [x] Commit message suggested

## What NOT to Do

- Don't stop after any phase except final Review
- Don't ask user to run manual commands between phases
- Don't create more artifacts than needed
- Don't over-communicate - keep it minimal
- Don't skip phases

## Artifact Summary

**Per issue, create only:**
1. `RESEARCH.md` - What we found
2. `PLAN.md` - What we'll build
3. `IMPLEMENTATION.md` - What we built
4. `REVIEW.md` - Is it ready?
5. `STATUS.md` - Where are we?

**Total: 5 files** (down from 8+)
