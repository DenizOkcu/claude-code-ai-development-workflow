---
name: reviewing-code
description: "Comprehensive code review with automated checks and manual quality assessment. Use for quality assurance, compliance verification, or implementation validation. Triggers: review code, QA, quality check, verify implementation, code inspection, compliance."
model: sonnet
temperature: 0
---

# Code Review

Run automated checks and manual review to ensure production readiness.

## LLM Parameters

**For consistent, deterministic review:**
- **Temperature:** 0 (deterministic, same inputs → same outputs)
- **Max Tokens:** 8000 (comprehensive review coverage)
- **Top-P:** 1.0 (use full distribution)
- **Frequency Penalty:** 0 (no penalty for repetition)
- **Presence Penalty:** 0 (no penalty for new topics)

**Seed Strategy:** Use `issue_name` as seed for consistency

**Why These Parameters:**
- Temperature 0 ensures reproducible review results
- Higher token limit allows thorough analysis
- No penalties enable complete issue identification
- Consistent reviews enable reliable quality gates

## Few-Shot Example

### Example Input
```yaml
issue_name: "add-oauth-login"
IMPLEMENTATION_SUMMARY.md: [exists]
git diff: [shows changes]
```

### Example Output: CODE_REVIEW.md (Good ✅)
```markdown
### Summary
- **Status:** APPROVED_WITH_NOTES
- **Risk:** Low

### Automated Checks
```
✓ Linting: Passed (0 errors, 0 warnings)
✓ Type Checking: Passed (0 errors)
✓ Tests: 15/15 passing (100%)
✓ Build: Passed
⚠ Security: 1 informational (no vulnerabilities)
```

### Code Quality
| Area           | Rating     | Notes                        |
| -------------- | ---------- | ---------------------------- |
| Structure      | Excellent  | Clean, follows MVC pattern  |
| Naming         | Excellent  | Clear, consistent           |
| Error Handling | Good       | Comprehensive, could use logging |
| Type Safety    | Excellent  | Strict TypeScript, proper types |
| Testing        | Good       | Good coverage, add edge case tests |

### Issues

**💡 SUGGESTIONS (optional):**
- `src/auth/google-strategy.ts:45` - Consider adding retry logic for OAuth calls
- `src/middleware/session.ts:135` - Add JSDoc for provider tracking function
- `tests/auth/oauth.test.ts:78` - Add test for OAuth failure scenario

### Compliance
- ✓ All planned features implemented
- ✓ Design followed
- Deviations: None

### Recommendations
1. Add OAuth failure test (edge case)
2. Consider logging OAuth callbacks (debugging)
3. Document OAuth setup in README (user onboarding)
```

### Example Output: CODE_REVIEW.md (Bad ❌)
```markdown
# Code Review

Reviewed the OAuth implementation.

Looks good! Tests pass.

Approved.
```

**Why Bad:**
- ❌ Missing automated checks section
- ❌ No structured quality assessment
- ❌ No specific file:line references
- ❌ No compliance verification
- ❌ Missing approval status format
- ❌ No recommendations
- ❌ Not actionable for fixes

### Example Approval Determination (Good ✅)
```
[Analyzing CODE_REVIEW.md]

Automated Checks:
✓ Linting: Passed
✓ Type Checking: Passed
✓ Tests: 15/15 Passed
✓ Build: Passed
✓ Security: No vulnerabilities

Issues:
- Critical: 0
- Important: 0
- Suggestions: 3

→ Decision: APPROVED_WITH_NOTES (suggestions are optional)
→ Update STATUS.md: Phase complete, ready for deployment
```

### Example Approval Determination (Bad ❌)
```
[Skimming review]

Looks mostly good. Some suggestions.

→ Decision: APPROVED (missed that suggestions should be noted)
```

**Why Bad:**
- ❌ Didn't count issues properly
- ❌ Wrong approval status (should be APPROVED_WITH_NOTES)
- ❌ Didn't verify automated checks all passed
- ❌ Inconsistent decision-making

## Inputs
- `issue_name`: Kebab-case identifier
- All planning artifacts (context)
- Implementation code changes

## Outputs
- `docs/{issue_name}/CODE_REVIEW.md` (focused review report)
- Updated `STATUS.md` with approval status

## Procedure

### 1. Context Gathering
Read all artifacts. Identify changed files (git diff). Update STATUS.md:
```markdown
## Phase: Review 🔄
- **Status:** Running automated checks and manual review
```

### 2. Automated Checks
Run checks appropriate to tech stack:

**Node.js/TypeScript:**
```bash
npm run lint
npm run type-check  # or npx tsc --noEmit
npm test
npm run build
npm audit
```

**Python:**
```bash
pylint src/
mypy src/
pytest
black --check src/
```

**Go:**
```bash
go vet ./...
golint ./...
go test ./...
go build
```

**Rust:**
```bash
cargo clippy
cargo test
cargo build
```

Document results for each check. Adapt these commands to the project.

### 3. Manual Code Review

**Code Quality:**
- Functions small and focused
- Clear, descriptive naming
- No magic numbers/strings
- No debug code or duplication

**Type Safety (if TypeScript):**
- Proper type annotations for APIs
- Minimal `any` usage
- Proper null/undefined handling

**Logic & Correctness:**
- Edge cases handled
- Input validation present
- Comprehensive error handling
- Proper async operations
- Resource cleanup

**Testing:**
- Unit tests for new functions
- Integration tests for interactions
- Edge cases covered
- Error scenarios tested

**Security:**
- No hardcoded secrets
- SQL injection prevention
- XSS prevention
- Input sanitization
- Auth/authz checks

**Performance:**
- No obvious bottlenecks
- Efficient queries
- Appropriate caching
- Pagination for large lists

**Maintainability:**
- Follows project conventions
- Documentation for complex logic
- API documentation (JSDoc/TSDoc)
- No unnecessary dependencies

**Accessibility (if UI):**
- Semantic HTML elements
- ARIA labels where needed
- Keyboard navigation works
- Color contrast sufficient
- Screen reader compatibility

**Integration Verification:**
- Follows existing architectural patterns
- Doesn't break existing functionality
- API contracts maintained
- Database migrations safe
- Environment variables documented
- Configuration changes documented

### 4. Plan Compliance
Compare implementation against plans:
- All planned features implemented
- Technical design followed
- Success criteria met
- Document deviations

### 5. Create CODE_REVIEW.md

```markdown
### Summary
- **Status:** APPROVED / APPROVED WITH NOTES / NEEDS REVISION
- **Risk:** Low/Medium/High

### Automated Checks
```
✓/✗ Linting: [status]
✓/✗ Type Checking: [status]
✓/✗ Tests: [N/M passing, X% coverage]
✓/✗ Build: [status]
✓/⚠ Security: [vulnerabilities if any]
```

### Code Quality
| Area           | Rating     | Notes        |
| -------------- | ---------- | ------------ |
| Structure      | Good       | [brief note] |
| Naming         | Excellent  | -            |
| Error Handling | Needs Work | [brief note] |
| Type Safety    | Good       | -            |
| Testing        | Good       | [brief note] |

### Issues

**❌ CRITICAL** (must fix before deploy):
- `file.ts:line` - Issue + fix

**⚠ IMPORTANT** (should fix):
- `file.ts:line` - Issue + fix

**💡 SUGGESTIONS** (optional):
- `file.ts:line` - Improvement

### Compliance
- ✓/✗ All planned features implemented
- ✓/✗ Design followed
- Deviations: [list or "None"]

### Recommendations
1. [Top priority action]
2. [Second priority]
3. [Third priority]
```

### 6. Determine Approval

**APPROVED:**
- All automated checks passing
- No critical issues
- No important issues
- Minor suggestions OK

**APPROVED WITH NOTES:**
- All automated checks passing
- No critical issues
- Minor important issues or suggestions
- Safe to deploy with follow-up tasks

**NEEDS REVISION:**
- Any automated check failures, OR
- Critical issues present, OR
- Multiple important issues

### 7. Update STATUS.md

**Approved:**
```markdown
## Phase: Review ✓ APPROVED
- **Critical:** 0
- **Important:** 0
- **Suggestions:** [N]
- **Status:** Review complete, workflow finished
```

**Needs Revision:**
```markdown
## Phase: Review ⚠ NEEDS REVISION
- **Critical:** [N]
- **Important:** [M]
- **Status:** Issues found, entering fix loop
```

## Error Handling

### Known Error Scenarios

1. **No Implementation Found**
   - Check for IMPLEMENTATION_SUMMARY.md
   - Check git diff for changes
   - Error: "No implementation changes found"
   - Recovery: Verify implementation phase completed

2. **Automated Checks Unavailable**
   - Linting not configured: Document, continue with manual review
   - Type checking not available: Skip, note limitation
   - Tests not configured: Document, continue without tests
   - Warning: "Check not available: {check_name}"

3. **Check Failures**
   - Linting failures: Report as critical or important based on severity
   - Type errors: Report as critical
   - Test failures: Report as critical
   - Build failures: Report as critical
   - Security vulnerabilities: Report as critical

4. **Cannot Read Files**
   - Check file permissions
   - Verify file paths
   - Error: "Cannot read file: {path}"
   - Recovery: Skip file, document limitation

5. **Missing Planning Documents**
   - Check for PLAN_SUMMARY.md, PROJECT_SPEC.md
   - Warning: "Planning documents not found - reviewing without plan context"
   - Recovery: Continue with general best practices

### Error Recovery

- **Before review**: Verify IMPLEMENTATION_SUMMARY.md exists
- **During automated checks**: If check fails, document result, continue
- **Manual review**: If files cannot be read, note limitation, review accessible files
- **Missing context**: Review based on general best practices if plan not available

### Exit Conditions

**Success:**
- All automated checks run (or documented as unavailable)
- Manual review completed
- CODE_REVIEW.md created with all sections
- Approval status determined
- STATUS.md updated

**Failure (Retryable):**
- Temporary file access issues
- Intermittent tool failures

**Failure (Non-retryable):**
- No implementation to review
- Repository access issues
- Critical errors preventing review

### Validation Before Completion

Before marking review complete, verify:
- [ ] IMPLEMENTATION_SUMMARY.md read (or git diff analyzed)
- [ ] All available automated checks run
- [ ] Manual review completed for all accessible files
- [ ] CODE_REVIEW.md has Summary, Automated Checks, Code Quality sections
- [ ] Issues prioritized (Critical/Important/Suggestions)
- [ ] File references include line numbers
- [ ] Approval status clearly stated (APPROVED/APPROVED_WITH_NOTES/NEEDS_REVISION)
- [ ] STATUS.md updated with approval status

## Review Guidelines

### Be Thorough but Practical
- Focus on issues that matter
- Don't nitpick style if linter handles it
- Consider context and constraints
- Balance perfection with shipping

### Be Constructive
- Explain why something is an issue
- Suggest concrete solutions
- Acknowledge good work
- Use collaborative language

### Prioritize Issues
- **Critical**: Blocks deployment, security risks, data loss
- **Important**: Significant bugs, poor error handling, maintainability
- **Suggestions**: Improvements, optimizations, style

### Use Specific References
- Include file paths and line numbers
- Show code snippets for issues
- Link to relevant documentation
- Reference best practices

## Special Considerations

### If No Automated Tools
- Note lack of automated checks
- Perform thorough manual review
- Recommend setting up linting/testing

### If Tests Are Missing
- Note lack of tests
- Recommend writing tests before deployment
- Suggest test cases to add

### If Plan Documents Don't Exist
- Review code on its own merits
- Use general best practices
- Recommend creating documentation

## Important Reminders

- Read STATUS.md first - understand workflow context
- Run ALL automated checks - don't skip any
- Read the actual code - don't just run tools
- Check git diff - see what actually changed
- Test the feature manually - if possible
- Be objective - good or bad, report it
- Document everything - all findings in CODE_REVIEW.md
- Update STATUS.md - reflect review completion
- Give credit - note what was done well

## Quality Checks
- All automated checks run (or documented as unavailable)
- Manual review completed
- CODE_REVIEW.md created with all sections
- Issues prioritized (Critical/Important/Suggestions)
- File references with line numbers
- Integration verified
- Accessibility checked (if UI)
- STATUS.md updated
- Error scenarios handled
