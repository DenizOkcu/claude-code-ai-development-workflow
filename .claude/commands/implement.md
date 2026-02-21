## Phase 5: Implementation

You are entering the **Implementation** phase for issue: `$ARGUMENTS`

### Pre-Conditions
- Read `STATUS.md` — confirm Planning is complete
- Read `IMPLEMENTATION_PLAN.md` for phased task list
- Read `PROJECT_SPEC.md` for interface contracts and requirements
- Read `ARCHITECTURE.md` for design decisions
- Read `CLAUDE.md` (project root) for coding conventions

### Instructions

Systematically implement the feature phase by phase, following the IMPLEMENTATION_PLAN.md exactly.

#### Execution Protocol

For **each phase** in the plan:

1. **Announce**: State which phase you are starting and its objective
2. **Create a todo list**: Convert the phase's tasks into a trackable checklist
3. **Implement**: Write code following project conventions from CLAUDE.md
4. **Test**: Write and run tests for every task in the phase
5. **Verify**: Run linting, type checks, and the full test suite
6. **Checkpoint**: Update `STATUS.md` with current phase progress

#### Coding Standards

- **Follow existing patterns**: Match the codebase's style, naming, and structure
- **Write tests alongside code**: Never implement a function without its test
- **Handle errors explicitly**: No silent failures, no bare `catch {}`
- **Type everything**: Full TypeScript types, no `any` unless absolutely necessary
- **Document public APIs**: JSDoc/TSDoc for exported functions and types
- **Keep commits logical**: Each phase = one logical commit

#### Test-Driven Implementation

For each task:
1. Write the test first (red)
2. Write the minimum code to pass (green)
3. Refactor while tests stay green (refactor)
4. Verify no regressions in existing tests

#### Progress Tracking

Update `STATUS.md` during execution:
```markdown
- [~] Implementation - In Progress (Phase {N}/{Total})
  - Phase 1: ✓ Complete
  - Phase 2: ✓ Complete
  - Phase 3: 🔄 In Progress (Task 3/5)
  - Phase 4: ⏳ Not started
```

Upon completion:
```markdown
- [x] Implementation - Completed ({N} files, {M} tests)
```

### Post-Actions
- Update `STATUS.md`: mark Implementation as completed with file/test counts
- List all files created or modified
- Run full test suite and report results
- Suggest next command: `/review $ARGUMENTS`

### Quality Gates
- All phases in IMPLEMENTATION_PLAN.md are completed
- All tests pass (unit, integration, and E2E if specified)
- Linting passes with zero errors
- Type checking passes with zero errors
- No `TODO` or `FIXME` comments left without tracking
- All acceptance criteria from each phase are met
