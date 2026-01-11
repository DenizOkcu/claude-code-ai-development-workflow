---
name: fixing-review-issues
description: "Address code review issues while preserving approved functionality. Use when fixing critical bugs, resolving quality issues, or addressing review feedback. Triggers: fix issues, address review, resolve bugs, fix critical, review feedback."
model: sonnet
temperature: 0
---

# Review-Fix

Fix issues identified during code review with **minimal context loading**.

## LLM Parameters

**For consistent, deterministic fixes:**
- **Temperature:** 0 (deterministic, same inputs → same outputs)
- **Max Tokens:** 4000 (minimal context, focused fixes)
- **Top-P:** 1.0 (use full distribution)
- **Frequency Penalty:** 0 (no penalty for repetition)
- **Presence Penalty:** 0 (no penalty for new topics)

**Seed Strategy:** Use `issue_name + iteration` as seed

**Why These Parameters:**
- Temperature 0 ensures reproducible fixes
- Lower token limit enforces minimal context
- No penalties preserve complete fix information
- Iteration-based seed provides variation per fix round

## Few-Shot Example

### Example Input
```yaml
issue_name: "add-oauth-login"
CODE_REVIEW.md: [contains 3 important issues]
git diff: [shows changed files]
iteration: 1
```

### Example Context Loading (Good ✅)
```
[Loading minimal context]
✓ Read CODE_REVIEW.md (3 issues identified)
✓ Read git diff (shows 5 files changed)
✓ Read STATUS.md (check iteration count)

Context loaded: ~2500 tokens ✓
Skipped: CODE_RESEARCH.md, IMPLEMENTATION_PLAN.md, PROJECT_SPEC.md
```

### Example Context Loading (Bad ❌)
```
[Loading full context]
✓ Read CODE_REVIEW.md
✓ Read CODE_RESEARCH.md
✓ Read IMPLEMENTATION_PLAN.md
✓ Read PROJECT_SPEC.md
✓ Read git diff

Context loaded: ~15000 tokens ✗
(Too much context for simple fixes)
```

### Example Fix Process (Good ✅)
```
[From CODE_REVIEW.md]
Important Issues:
1. `src/auth/google-strategy.ts:45` - Missing error handling for OAuth failure
2. `src/middleware/session.ts:135` - Function lacks JSDoc
3. `tests/auth/oauth.test.ts:78` - Missing test for OAuth failure scenario

[Fixing Issue 1 - Important]
✓ Reading src/auth/google-strategy.ts:40-50
✓ Added try-catch for OAuth callback
✓ Added error logging
✓ Ran tests - all still passing
✓ Issue 1 fixed

[Fixing Issue 2 - Important]
✓ Reading src/middleware/session.ts:130-140
✓ Added JSDoc comment
✓ Verified types are correct
✓ Issue 2 fixed

[Fixing Issue 3 - Important]
✓ Reading tests/auth/oauth.test.ts:75-85
✓ Added test case for OAuth failure
✓ Ran tests - new test passes
✓ Issue 3 fixed

[Update CODE_REVIEW.md]
✓ Strikethrough all 3 fixed issues
✓ All critical and important issues addressed

[Update STATUS.md]
✓ Marked fix iteration 1/3 complete
✓ Ready for review phase
```

### Example Fix Process (Bad ❌)
```
[From CODE_REVIEW.md]
Important Issues:
1. Missing error handling
2. Missing JSDoc

[Fixing Issue 1]
✓ Read src/auth/google-strategy.ts
✓ Added error handling somewhere in file

[Fixing Issue 2]
✓ Added JSDoc to auth file

✓ All fixed! ✓
```

**Why Bad:**
- ❌ Didn't read specific line ranges
- ❌ Fix locations imprecise
- ❌ No verification tests ran
- ❌ No regression check
- ❌ CODE_REVIEW.md not updated properly
- ❌ Can't verify fixes are correct

### Example CODE_REVIEW.md Update (Good ✅)
```markdown
### Issues

**⚠ IMPORTANT:**
- ~~`src/auth/google-strategy.ts:45` - Missing error handling for OAuth failure~~ ✓ FIXED
- ~~`src/middleware/session.ts:135` - Function lacks JSDoc~~ ✓ FIXED
- ~~`tests/auth/oauth.test.ts:78` - Missing test for OAuth failure scenario~~ ✓ FIXED

**💡 SUGGESTIONS (optional):**
- `src/auth/google-strategy.ts:55` - Consider adding retry logic for OAuth calls

All critical and important issues addressed.
See recommendations for optional improvements.
```

### Example CODE_REVIEW.md Update (Bad ❌)
```markdown
### Issues

Fixed the issues.

All good now.
```

**Why Bad:**
- ❌ Doesn't show what was fixed
- ❌ No strikethrough formatting
- ❌ Can't track fix completion
- ❌ Lost original issue details

## Context Management

**Minimal Context Mode:**
- Only load CODE_REVIEW.md (issues to fix)
- Only load changed files (git diff)
- DO NOT load full research, plan, or implementation artifacts
- Each fix iteration ~3K tokens (not 15K+)

## Inputs
- `issue_name`: Kebab-case identifier
- `CODE_REVIEW.md` - Issues to fix (Critical and Important)
- Changed files (via git diff)

## Outputs
- Fixed implementation code
- Updated `CODE_REVIEW.md` (strikethrough fixed issues)
- Updated `STATUS.md` with fix completion

## Procedure

### 1. Load Minimal Context

**Read ONLY:**
- `CODE_REVIEW.md` - extract issues list
- Changed files via `git diff` - see what changed
- STATUS.md - check current iteration count

**DO NOT read:**
- CODE_RESEARCH.md (not needed for fixes)
- IMPLEMENTATION_PLAN.md (not needed for fixes)
- PROJECT_SPEC.md (not needed for fixes)

### 2. Review Issue List
Read `CODE_REVIEW.md`. Categorize:
- **❌ CRITICAL**: Must fix (security, data loss, broken functionality)
- **⚠ IMPORTANT**: Should fix (errors, poor handling, maintainability)
- **💡 SUGGESTIONS**: Optional improvements

### 3. Create Fix Todo List
Use TodoWrite with tasks. **Exactly ONE task in_progress at a time.**

### 4. Fix Implementation

**For each issue:**
1. Mark task as `in_progress`
2. Read code at file:line reference (from git diff)
3. Apply minimal fix addressing the problem
4. Don't refactor beyond issue scope
5. Maintain existing functionality
6. Verify fix addresses the issue
7. Check for unintended side effects
8. Run relevant tests
9. Mark task as `completed`

**Fix principles:**
- Minimal changes (fix only the identified issue)
- Preserve working code (don't change what isn't broken)
- Add tests if issue was lack of coverage
- Document only if logic is complex
- Re-verify after each fix

### 5. Test Verification
After all fixes:
- Run full test suite
- Verify all tests pass
- Check for regressions
- Validate fixes against original issues

### 6. Update CODE_REVIEW.md
Strike through fixed issues:

```markdown
### Issues

**❌ CRITICAL:**
- ~~`file.ts:line` - Issue~~ ✓ FIXED

**⚠ IMPORTANT:**
- ~~`file.ts:line` - Issue~~ ✓ FIXED
```

Or:
```markdown
### Issues
All critical and important issues addressed.
See recommendations for optional improvements.
```

### 7. Update STATUS.md

```markdown
## Progress
- [x] Research | [x] Planning | [x] Implementation | [~] Review (Iteration N/3)

## Phase: Review Fix 🔄
- **Fixed:** [N critical, M important]
- **Context:** Minimal mode (issues + changed files only)
- **Next:** `/sdlc {issue_name} --phase review`
```

### 8. Prepare for Next Review

**Before returning to review phase:**
- Clear fix context (free up tokens)
- Only keep CODE_REVIEW.md with fixed status
- Next review loads minimal context again

## Error Handling

### Known Error Scenarios

1. **Missing CODE_REVIEW.md**
   - Check for review artifact
   - Error: "CODE_REVIEW.md not found - cannot fix issues"
   - Recovery: Ask user to run review phase first

2. **Cannot Read File to Fix**
   - Check file permissions
   - Verify file path from review
   - Error: "Cannot read file: {path}"
   - Recovery: Document issue, skip fix

3. **Fix Makes Code Worse**
   - Fix introduces new errors
   - Tests fail after fix
   - Error: "Fix caused regression"
   - Recovery: Revert fix, try alternative approach

4. **Git Commands Fail**
   - Cannot get git diff
   - Fallback: Use file modification times
   - Warning: "Git diff unavailable - using alternative detection"

5. **Max Iterations Reached**
   - Review iteration 3 completed with issues remaining
   - Error: "Maximum review-fix iterations (3) reached"
   - Recovery: Escalate to user, create diagnostic summary

### Error Recovery

- **Before fixing**: Verify CODE_REVIEW.md and IMPLEMENTATION_SUMMARY.md exist
- **During fixing**: If a file cannot be modified, document issue, continue with other fixes
- **Fix causes regression**: Revert immediately, try alternative approach
- **Missing context**: Load PLAN_SUMMARY.md or PROJECT_SPEC.md if needed for understanding

### Exit Conditions

**Success:**
- All critical issues addressed
- All important issues addressed
- Tests still passing
- No regressions
- CODE_REVIEW.md updated
- STATUS.md updated

**Failure (Retryable):**
- File access issues (fix permissions, retry)
- Fix caused regression (revert, retry)

**Failure (Non-retryable):**
- Maximum iterations reached (3)
- Cannot fix issues without architectural decision
- User clarification required

### Validation Before Completion

Before marking fix complete, verify:
- [ ] CODE_REVIEW.md read and issues extracted
- [ ] All critical issues addressed
- [ ] All important issues addressed
- [ ] Tests still passing (or improved)
- [ ] No new regressions
- [ ] CODE_REVIEW.md updated (strikethrough fixed issues)
- [ ] STATUS.md updated
- [ ] Git shows only intended changes

## Quality Checks
- All critical issues addressed
- All important issues addressed
- Tests still passing
- No regressions
- Fixes minimal and targeted
- CODE_REVIEW.md updated
- STATUS.md updated
- Context was minimal (verified no unnecessary files loaded)
- Error scenarios handled

## Iteration Limits
- Maximum 3 review-fix iterations
- Each iteration uses minimal context (~3K tokens)
- After 3rd iteration, escalate with diagnostic summary
