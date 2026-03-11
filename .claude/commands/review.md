---
model: sonnet
---

## Phase 6: Code Review & Quality Assurance

You are entering the **Review** phase for issue: `$ARGUMENTS`

### Pre-Conditions
- Read `00_STATUS.md` — confirm Implementation is complete
- Read `04_IMPLEMENTATION_PLAN.md` for what was planned
- Read `03_PROJECT_SPEC.md` for requirements
- Read `03_ARCHITECTURE.md` and 03_ADRs for design decisions

### Instructions

Perform a comprehensive code review simulating a senior engineer. Create `.claude/planning/$ARGUMENTS/06_CODE_REVIEW.md`:

#### 1. Automated Checks

Run and report results for all available tools:
```bash
# Run in order, report all results
npm run lint          # or equivalent
npm run typecheck     # TypeScript/type checking
npm run test          # Full test suite
npm run build         # Production build
```

Report: pass/fail, error count, warning count for each.

#### 2. Manual Code Review

Review every file changed/created for this issue. For each finding, categorize:

| Severity | Meaning | Action Required |
|----------|---------|-----------------|
| 🔴 Critical | Bug, security issue, data loss risk | Must fix before merge |
| 🟡 Important | Design flaw, missing test, poor pattern | Should fix before merge |
| 🔵 Suggestion | Style, naming, minor improvement | Nice to have |
| 💡 Praise | Excellent code, good pattern | Recognition |

**Review Checklist:**
- [ ] Code matches the 03_ARCHITECTURE.md design
- [ ] All 03_PROJECT_SPEC.md requirements are met
- [ ] Error handling is comprehensive (no silent failures)
- [ ] Edge cases are handled and tested
- [ ] No hardcoded values that should be configurable
- [ ] No sensitive data in code (secrets, keys, PII)
- [ ] Naming is clear and consistent with project conventions
- [ ] Functions are focused (single responsibility)
- [ ] No unnecessary complexity or over-engineering
- [ ] Tests are meaningful (not just coverage padding)
- [ ] API changes are backwards compatible (or migration exists)
- [ ] Documentation is updated for public APIs
- [ ] Performance implications are acceptable
- [ ] Accessibility requirements are met (if UI)

#### 3. Test Quality Review
- Are critical paths tested?
- Are edge cases covered?
- Are error paths tested?
- Do tests test behavior (not implementation details)?
- Are mocks/stubs appropriate (not over-mocking)?

#### 4. Summary & Verdict

```markdown
## Review Summary

| Category | Count |
|----------|-------|
| 🔴 Critical | N |
| 🟡 Important | N |
| 🔵 Suggestions | N |
| 💡 Praise | N |

## Verdict: ✓ APPROVED / ⚠ NEEDS REVISION / ✗ REJECTED

### Required Changes (if any)
1. ...
2. ...

### Recommendations (non-blocking)
1. ...
```

### Post-Actions
- Update `00_STATUS.md` with review outcome
- If APPROVED: suggest `/security $ARGUMENTS`
- If NEEDS REVISION: list specific changes needed, then re-run `/review $ARGUMENTS` after fixes

### Quality Gates
- All automated checks were run and results documented
- Every file was reviewed against the checklist
- Findings are specific (file, line, description) not vague
- Verdict is justified by findings
