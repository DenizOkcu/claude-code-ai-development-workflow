---
name: implementing-code
description: "Build working software that meets acceptance criteria. Use when executing implementation plans. Triggers: implement, build, execute plan, write code."
model: sonnet
---

# Code Implementation

**Mindset:** Build working software that solves the problem. The plan is a guide, not a contract.

## Goal

Implement all phases and create working, tested code that meets the acceptance criteria.

## Autonomy Rules

- **Implement ALL phases** - don't stop after Phase 1
- **The plan guides, not dictates** - better approaches are welcome
- **Document deviations** - explain why you diverged
- **Tests are required** - part of each phase, not after

## Inputs
- `issue_name`: Kebab-case identifier
- `PLAN.md`: Phases and acceptance criteria
- `RESEARCH.md`: Context (if exists)

## Output
- Implementation code
- Tests
- `docs/{issue_name}/IMPLEMENTATION.md`
- `docs/{issue_name}/STATUS.md` (updated)

## Procedure

### 1. Read the Plan

Read `PLAN.md` and understand:
- Total number of phases
- Scope boundaries
- Acceptance criteria

**Count phases explicitly:** "I see {N} phases in the plan. I will implement all {N}."

### 2. Implement Phase by Phase

For each phase:
1. Mark phase as in-progress in STATUS.md
2. Implement all tasks
3. Write/update tests
4. **Run phase validation from PLAN.md** (if defined)
5. Verify all phase validation checks pass
6. Mark phase complete

**CRITICAL: Before marking a phase complete, run the Validation checklist from PLAN.md.**

Example validation from plan:
```markdown
Validation:
- [ ] npm test passes
- [ ] TypeScript compiles without errors
- [ ] New class is importable
```

The implementer MUST verify these before proceeding.

**Continue until ALL phases are done.**

### 3. Deviate Wisely

Good reasons to deviate:
- Discovered a better approach
- Plan conflicts with existing architecture
- External dependencies changed
- Security/performance issues found

When deviating:
- Document in IMPLEMENTATION.md
- Explain WHY
- Ensure tests still pass

### 4. Run Full Validation

After all phases:
- Run full test suite
- Run type check (if TypeScript)
- Run lint
- Run build

### 5. Create IMPLEMENTATION.md

```markdown
# Implementation: {issue_name}

**What:** {feature_description}
**When:** {timestamp}

---

## Summary

Built {brief description of what was implemented}.
{N}/{N} phases completed.

---

## Changes

### Files Created
- `path/to/file.ts` - {purpose}
- `path/to/file.ts` - {purpose}

### Files Modified
- `path/to/file.ts` - {what changed}

---

## Phases Completed

- [x] Phase 1: {name} - {brief result}
- [x] Phase 2: {name} - {brief result}
- [x] Phase 3: {name} - {brief result}

---

## Test Results

- Tests: {N} total, {N} passing
- Coverage: {X}% for new code
- Type check: ✓
- Lint: ✓
- Build: ✓

---

## Deviations from Plan

{None | List deviations with reasons}

| Plan Said | What We Did | Why |
|-----------|-------------|-----|
| {original} | {actual} | {reason} |

---

## Known Limitations

- {limitation} (impact: {low/medium})
```

### 6. Update STATUS.md

```markdown
# Status: {issue_name}

**Risk:** {level} | **Updated:** {timestamp}

## Progress
- [x] Research | [x] Planning | [x] Implementation | [ ] Review

## Phase: Implementation ✓
- **Phases:** {N}/{N} complete
- **Tests:** {N} passing
- **Deviations:** {None | N}
- **Next:** Review

## Artifacts
- RESEARCH.md ✓
- PLAN.md ✓
- IMPLEMENTATION.md ✓
- {N} code files
```

## Code Quality

Follow existing patterns in the codebase:
- Match naming conventions
- Use existing utilities
- Keep functions focused
- Handle errors appropriately

## What NOT to Do

- Don't stop after Phase 1
- Don't skip tests
- Don't ignore failing tests
- Don't over-engineer
- Don't add unrequested features

## Stopping Criteria

You are ONLY done when:
- [ ] ALL phases from PLAN.md are complete
- [ ] ALL phase validations from PLAN.md pass
- [ ] All tests pass
- [ ] Type check passes (if applicable)
- [ ] Acceptance criteria from PLAN.md are met
- [ ] IMPLEMENTATION.md created
- [ ] STATUS.md updated

**If ANY phase is incomplete OR any validation fails, keep implementing.**

## Quality Check

- [ ] Read PLAN.md and counted phases?
- [ ] Implemented ALL phases?
- [ ] Ran validation for EACH phase from PLAN.md?
- [ ] All phase validations pass?
- [ ] Tests written and passing?
- [ ] Acceptance criteria verified?
- [ ] Deviations documented?
- [ ] IMPLEMENTATION.md created?
- [ ] STATUS.md updated?
