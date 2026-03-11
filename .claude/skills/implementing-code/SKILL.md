---
name: implementing-code
description: "Builds working software by executing implementation plans phase by phase. Use when writing code, running tests, or executing a planned feature build. Implements all phases autonomously, validates each phase, and documents deviations."
model: opus
metadata:
  version: 1.0.0
  category: workflow-automation
---

# Code Implementation

**Mindset:** Build working software that solves the problem. The plan is a guide, not a contract.

## Goal

Implement all phases and create working, tested code that meets the acceptance criteria.

## Instructions

### Step 1: Read the Plan

Read `PLAN.md` and understand:
- Total number of phases
- Scope boundaries
- Acceptance criteria

**Count phases explicitly:** "I see {N} phases in the plan. I will implement all {N}."

### Step 2: Implement Phase by Phase

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

### Step 3: Deviate Wisely

Good reasons to deviate:
- Discovered a better approach
- Plan conflicts with existing architecture
- External dependencies changed
- Security/performance issues found

When deviating:
- Document in IMPLEMENTATION.md
- Explain WHY
- Ensure tests still pass

### Step 4: Run Full Validation

After all phases:
- Run full test suite
- Run type check (if TypeScript)
- Run lint
- Run build

### Step 5: Create IMPLEMENTATION.md and Update STATUS.md

Write `docs/{issue_name}/IMPLEMENTATION.md` and update `docs/{issue_name}/STATUS.md`.

## Autonomy Rules

- **Implement ALL phases** - don't stop after Phase 1
- **The plan guides, not dictates** - better approaches are welcome
- **Document deviations** - explain why you diverged
- **Tests are required** - part of each phase, not after

## Output Format

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

### Files Modified
- `path/to/file.ts` - {what changed}

---

## Phases Completed

- [x] Phase 1: {name} - {brief result}
- [x] Phase 2: {name} - {brief result}

---

## Test Results

- Tests: {N} total, {N} passing
- Coverage: {X}% for new code
- Type check: pass/fail
- Lint: pass/fail
- Build: pass/fail

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

## Common Issues

- **Stopping early:** Don't stop after Phase 1. Implement ALL phases from PLAN.md.
- **Skipping tests:** Tests are part of each phase, not a final step.
- **Ignoring validation:** Run the validation checklist from PLAN.md for each phase before proceeding.
- **Over-engineering:** Don't add unrequested features. Build what the plan specifies.
- **Silent failures:** If ANY phase is incomplete OR any validation fails, keep implementing.
