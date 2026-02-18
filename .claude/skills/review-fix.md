---
name: fixing-review-issues
description: "Fix blocking issues to get APPROVED. Use when addressing review feedback. Triggers: fix issues, address review, resolve bugs."
model: sonnet
---

# Review Fix

**Mindset:** Get to APPROVED by fixing blocking issues only.

## Goal

Fix all blocking issues identified in REVIEW.md so the code can ship.

## Rules

- **Fix blocking issues only** - non-blocking are suggestions
- **Minimal changes** - don't refactor unrelated code
- **Max 3 iterations** - if not fixed by then, escalate

## Inputs
- `issue_name`: Kebab-case identifier
- `REVIEW.md`: Issues to fix
- Changed files

## Output
- Fixed code
- Updated `REVIEW.md`
- Updated `STATUS.md`

## Procedure

### 1. Read REVIEW.md

Identify blocking issues:
```
### Blocking (must fix)
- `src/auth.ts:45` - Missing error handling
- `tests/auth.test.ts` - Test for OAuth failure missing
```

### 2. Fix Each Issue

For each blocking issue:
1. Read the file at the specified location
2. Apply minimal fix
3. Run related tests
4. Mark as fixed in REVIEW.md

### 3. Update REVIEW.md

```markdown
## Issues

### Blocking (must fix)
- ~~`src/auth.ts:45` - Missing error handling~~ ✓ FIXED
- ~~`tests/auth.test.ts` - Test for OAuth failure missing~~ ✓ FIXED

### Non-Blocking (nice to have)
- `src/utils.ts` - Consider extracting helper (skipped)
```

### 4. Run Full Validation

- Run all tests
- Run type check
- Run lint
- Run build

### 5. Update STATUS.md

```markdown
## Phase: Review Fix
- **Fixed:** {N} blocking issues
- **Iteration:** {N}/3
- **Tests:** {N} passing
- **Next:** Re-review
```

## Fix Principles

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

## Quality Check

- [ ] All blocking issues fixed?
- [ ] Tests still passing?
- [ ] REVIEW.md updated?
- [ ] STATUS.md updated?
- [ ] No new issues introduced?
