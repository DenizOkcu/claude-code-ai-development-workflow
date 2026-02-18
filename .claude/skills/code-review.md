---
name: reviewing-code
description: "Ensure production readiness with minimal overhead. Use for quality assurance and verification. Triggers: review code, QA, quality check, verify implementation."
model: sonnet
---

# Code Review

**Mindset:** Can this be deployed safely? Focus on blocking issues only.

## Goal

Answer: Is this ready to ship?

Focus on:
1. Do tests pass?
2. Are there security issues?
3. Does it do what we planned?

## Extended Thinking

Think a lot about:
- Security vulnerabilities
- Edge cases that could break
- Hidden dependencies

## Inputs
- `issue_name`: Kebab-case identifier
- `PLAN.md`: What was planned
- `IMPLEMENTATION.md`: What was built
- Changed files (via git diff)

## Output
- `docs/{issue_name}/REVIEW.md`
- `docs/{issue_name}/STATUS.md` (updated)

## Procedure

### 1. Run Automated Checks

Run what's available for the stack:

**Node.js/TypeScript:**
```bash
npm test
npm run lint
npx tsc --noEmit  # type check
npm run build
```

**Python:**
```bash
pytest
pylint src/
mypy src/
```

**Go:**
```bash
go test ./...
go vet ./...
go build
```

### 2. Check Plan Compliance

Read PLAN.md and IMPLEMENTATION.md:
- Were acceptance criteria met?
- Were planned features built?
- Are deviations documented?

### 3. Security Scan

Look for:
- Hardcoded secrets
- SQL injection
- XSS vulnerabilities
- Missing auth checks
- Sensitive data in logs

### 4. Create REVIEW.md

```markdown
# Review: {issue_name}

**When:** {timestamp}

---

## Verdict

**Status:** APPROVED | NEEDS_FIX

---

## Automated Checks

| Check | Status | Notes |
|-------|--------|-------|
| Tests | ✓/✗ | {N} passing |
| Type Check | ✓/✗ | |
| Lint | ✓/✗ | |
| Build | ✓/✗ | |
| Security | ✓/✗ | |

---

## Plan Compliance

- [ ] Acceptance criteria met
- [ ] Planned features built
- [ ] Deviations documented

**Deviations:** {None | acceptable because...}

---

## Issues

### Blocking (must fix)
- `path/to/file.ts` - {issue}

### Non-Blocking (nice to have)
- `path/to/file.ts` - {suggestion}

---

## Decision

{APPROVED: Ready to ship | NEEDS_FIX: Blocking issues above}
```

### 5. Update STATUS.md

**Approved:**
```markdown
# Status: {issue_name}

**Risk:** {level} | **Updated:** {timestamp}

## Progress
- [x] Research | [x] Planning | [x] Implementation | [x] Review

## Phase: Review ✓ APPROVED
- **Tests:** {N} passing
- **Issues:** 0 blocking
- **Next:** Ready to commit/deploy

## Artifacts
- RESEARCH.md ✓
- PLAN.md ✓
- IMPLEMENTATION.md ✓
- REVIEW.md ✓
```

**Needs Fix:**
```markdown
# Status: {issue_name}

**Risk:** {level} | **Updated:** {timestamp}

## Progress
- [x] Research | [x] Planning | [x] Implementation | [~] Review

## Phase: Review ⚠ NEEDS_FIX
- **Blocking Issues:** {N}
- **Iteration:** {N}/3
- **Next:** Fix issues

## Artifacts
- RESEARCH.md ✓
- PLAN.md ✓
- IMPLEMENTATION.md ✓
- REVIEW.md ✓ (needs fixes)
```

## Issue Classification

**Blocking (MUST fix):**
- Tests failing
- Security vulnerabilities
- Breaking existing functionality
- Missing planned features

**Non-Blocking (nice to have):**
- Style suggestions
- Performance optimizations
- Additional tests
- Documentation improvements

## What NOT to Do

- Don't nitpick style (linters handle this)
- Don't suggest hypothetical edge cases
- Don't add "nice to have" features
- Don't mark non-blocking issues as blocking
- Don't create extensive quality matrices

## Decision Logic

```
APPROVED if:
- All automated checks pass
- No blocking issues
- Acceptance criteria met

NEEDS_FIX if:
- Any automated check fails
- Any blocking issue exists
- Acceptance criteria not met
```

## Quality Check

- [ ] Automated checks run?
- [ ] Security scanned?
- [ ] Plan compliance checked?
- [ ] REVIEW.md created?
- [ ] Clear APPROVED/NEEDS_FIX verdict?
- [ ] STATUS.md updated?
