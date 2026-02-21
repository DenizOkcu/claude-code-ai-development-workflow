## Phase 4: Implementation Planning

You are entering the **Planning** phase for issue: `$ARGUMENTS`

### Pre-Conditions
- Read `STATUS.md` — confirm Design is complete
- Read `ARCHITECTURE.md`, `PROJECT_SPEC.md`, and all ADRs
- Read `CODE_RESEARCH.md` for codebase context

### Instructions

Transform the design into an actionable, phase-by-phase implementation plan. Create `.claude/planning/$ARGUMENTS/IMPLEMENTATION_PLAN.md`:

#### Structure

```markdown
# Implementation Plan: {issue-name}

## Overview
- Total Phases: N
- Estimated Effort: {T-shirt size}
- Dependencies: {list any blockers}
- Feature Flag: {name if applicable}

## Phase 1: {Phase Title}
### Objective
What this phase accomplishes.

### Tasks
- [ ] Task 1.1: {description} — {file(s) affected}
- [ ] Task 1.2: {description} — {file(s) affected}

### Tests
- [ ] Unit: {what to test}
- [ ] Integration: {what to test}

### Acceptance Criteria
- Criteria that must pass before moving to Phase 2

### Rollback
- How to undo this phase if needed

---

## Phase 2: {Phase Title}
(same structure)
...
```

#### Planning Principles

1. **Small phases**: Each phase should be completable in a single session and produce a working (if incomplete) system
2. **Test-first**: Every phase includes its test tasks alongside implementation tasks
3. **Incremental value**: Each phase delivers measurable progress
4. **Safe rollback**: Each phase can be independently reverted
5. **Dependency ordering**: Phases are ordered to minimize blocked work
6. **Interface-first**: Define contracts before implementations

#### Test Strategy Section

Include at the end of the plan:

```markdown
## Test Strategy

### Unit Tests
- Coverage target: {percentage}
- Key areas: {list}
- Mocking strategy: {approach}

### Integration Tests
- API contract tests
- Database integration
- External service mocks

### E2E Tests (if applicable)
- Critical user flows to cover
- Browser/environment matrix

### Performance Tests (if applicable)
- Baseline metrics to capture
- Load test scenarios
```

### Post-Actions
- Update `STATUS.md`: mark Planning as completed
- Add phase count and key milestones to STATUS.md
- Suggest next command: `/implement $ARGUMENTS`

### Quality Gates
- Every task specifies affected files
- Every phase has acceptance criteria
- Every phase has a rollback strategy
- Test tasks are present in every phase (not deferred to the end)
- No phase has more than 8 tasks (break it up if larger)
