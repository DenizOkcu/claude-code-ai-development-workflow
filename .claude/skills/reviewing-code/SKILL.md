---
name: reviewing-code
description: "Reviews code for production readiness with focus on blocking issues. Use when performing quality assurance, verifying implementations, or checking security. Runs automated checks, validates plan compliance, and produces APPROVED or NEEDS_FIX verdict."
model: sonnet
metadata:
  version: 1.0.0
  category: workflow-automation
---

# Code Review

**Mindset:** Can this be deployed safely? Focus on blocking issues only.

## Goal

Answer: Is this ready to ship?

Focus on:
1. Do tests pass?
2. Are there security issues?
3. Does it do what we planned?

## Instructions

### Step 1: Think About Risks

Think a lot about:
- Security vulnerabilities
- Edge cases that could break
- Hidden dependencies

### Step 2: Run Automated Checks

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

### Step 3: Check Plan Compliance

Read PLAN.md and IMPLEMENTATION.md:
- Were acceptance criteria met?
- Were planned features built?
- Are deviations documented?

### Step 4: Security Scan

Look for:
- Hardcoded secrets
- SQL injection
- XSS vulnerabilities
- Missing auth checks
- Sensitive data in logs

### Step 5: Create REVIEW.md and Update STATUS.md

Write `docs/{issue_name}/REVIEW.md` and update `docs/{issue_name}/STATUS.md`.

## Output Format

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
| Tests | pass/fail | {N} passing |
| Type Check | pass/fail | |
| Lint | pass/fail | |
| Build | pass/fail | |
| Security | pass/fail | |

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

## Common Issues

- **Nitpicking style:** Don't nitpick style (linters handle this).
- **Hypothetical edge cases:** Don't suggest hypothetical edge cases that are unlikely to occur.
- **Feature creep:** Don't add "nice to have" features during review.
- **Misclassification:** Don't mark non-blocking issues as blocking.
