---
name: review-fix
description: "Fixes blocking issues identified during code review to achieve APPROVED status. Use when addressing review feedback or resolving blocking bugs. Applies minimal targeted fixes, re-validates, and iterates up to 3 times."
model: sonnet
metadata:
  version: 1.0.0
  category: workflow-automation
---

# Review Fix

**Mindset:** Get to APPROVED by fixing blocking issues only.

## Goal

Fix all blocking issues identified in REVIEW.md so the code can ship.

## Instructions

### Step 1: Read REVIEW.md

Identify blocking issues:
```
### Blocking (must fix)
- `src/auth.ts:45` - Missing error handling
- `tests/auth.test.ts` - Test for OAuth failure missing
```

### Step 2: Fix Each Issue

For each blocking issue:
1. Read the file at the specified location
2. Apply minimal fix
3. Run related tests
4. Mark as fixed in REVIEW.md

### Step 3: Update REVIEW.md

```markdown
## Issues

### Blocking (must fix)
- ~~`src/auth.ts:45` - Missing error handling~~ FIXED
- ~~`tests/auth.test.ts` - Test for OAuth failure missing~~ FIXED

### Non-Blocking (nice to have)
- `src/utils.ts` - Consider extracting helper (skipped)
```

### Step 4: Run Full Validation

- Run all tests
- Run type check
- Run lint
- Run build

### Step 5: Update STATUS.md

Update with fix iteration count and test results.

## Fix Rules

**DO:**
- Fix exactly what's described
- Add tests for bug fixes
- Keep changes minimal

**DON'T:**
- Refactor unrelated code
- Add new features
- Fix non-blocking suggestions
- Over-engineer the fix

## Iteration Limit

Maximum 3 fix iterations:
- Iteration 1: Fix issues
- Iteration 2: Fix remaining issues
- Iteration 3: Last attempt

If still failing after iteration 3:
- Mark as BLOCKED
- Create diagnostic
- Escalate to user

## Output Format

- Fixed code files
- Updated `REVIEW.md` with struck-through fixed issues
- Updated `STATUS.md` with iteration count

## Quality Check

- [ ] All blocking issues fixed?
- [ ] Tests still passing?
- [ ] REVIEW.md updated?
- [ ] STATUS.md updated?
- [ ] No new issues introduced?

## Common Issues

- **Scope creep:** Don't fix non-blocking issues. Focus only on what blocks APPROVED status.
- **Over-fixing:** Apply minimal changes. Don't refactor surrounding code.
- **Iteration overflow:** If not fixed by iteration 3, escalate to user instead of continuing.
- **New regressions:** Always run the full test suite after fixes to catch unintended side effects.
