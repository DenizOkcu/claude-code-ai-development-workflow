# SDLC Workflow Troubleshooting Guide

**Version:** 1.0.0
**For:** Users of the `/sdlc` workflow command
**Purpose:** Recover from common workflow issues without code execution

---

## Quick Diagnostics

### First Step: Check STATUS.md

```bash
cat docs/{your-issue-name}/STATUS.md
```

**What to look for:**
- Current phase (should match where you expect to be)
- Progress checkboxes (which phases are complete)
- Artifacts listed (which files exist)
- Iteration count (if in review-fix loop)

### Second Step: Check for Artifacts

```bash
ls -la docs/{your-issue-name}/
```

**Expected artifacts per phase:**
| Phase | Expected Artifacts |
|-------|-------------------|
| Research | CODE_RESEARCH.md, RESEARCH_SUMMARY.md, STATUS.md |
| Planning | IMPLEMENTATION_PLAN.md, PROJECT_SPEC.md, PLAN_SUMMARY.md, STATUS.md |
| Implementation | IMPLEMENTATION_SUMMARY.md, STATUS.md, code files |
| Review | CODE_REVIEW.md, STATUS.md |

---

## Common Issues by Phase

### Research Phase Issues

#### Issue: "No files found matching pattern"

**Symptom:**
```
Error: No files found matching pattern `src/**/*.ts`
```

**Diagnosis:**
- Pattern may be incorrect for your codebase
- Files may be in different directory
- File extension may be different

**Recovery:**
1. Check actual file structure: `find . -name "*.ts" | head -20`
2. Update glob pattern in research prompt
3. Re-run: `/sdlc {issue-name} --from-research`

**Example patterns:**
```bash
# TypeScript
src/**/*.ts

# Python
**/*.py

# JavaScript (multiple locations)
src/**/*.js lib/**/*.js

# All code files
**/*.{ts,js,py,go,java}
```

---

#### Issue: "Git commands unavailable"

**Symptom:**
```
Warning: Git commands unavailable - historical analysis skipped
```

**Diagnosis:**
- Not in a git repository
- Git not installed
- No commits yet

**Recovery:**
- This is **non-blocking** - research continues without git history
- Proceed with available files
- Document that git analysis was skipped

---

#### Issue: "Cannot create directory"

**Symptom:**
```
Error: Cannot create directory docs/{issue-name}/
```

**Diagnosis:**
- Parent directory `docs/` doesn't exist
- Permission issues
- Invalid issue name (path traversal attempt)

**Recovery:**
```bash
# Create parent directory
mkdir -p docs

# Verify issue name is valid (kebab-case)
# Good: add-oauth-login
# Bad: ../etc/passwd, AddOAuthLogin

# Try again
/sdlc {issue-name} --from-start
```

---

### Planning Phase Issues

#### Issue: "Missing CODE_RESEARCH.md"

**Symptom:**
```
Error: CODE_RESEARCH.md not found - proceeding without research context
```

**Diagnosis:**
- Research phase was skipped
- Research file was not created
- Wrong issue directory

**Recovery:**
```bash
# Option 1: Run research first
/sdlc {issue-name} --from-research

# Option 2: Check if file exists elsewhere
find docs/ -name "CODE_RESEARCH.md"

# Option 3: Proceed without research (not recommended)
/sdlc {issue-name} --from-plan
# Will proceed but with limited context
```

---

#### Issue: "Ambiguous requirements"

**Symptom:**
```
Planning cannot proceed - requirements are unclear
```

**Diagnosis:**
- Feature description too vague
- Conflicting requirements
- Missing acceptance criteria

**Recovery:**
1. Provide more specific feature description
2. Answer clarifying questions
3. Re-run planning with better input:

```bash
/sdlc {issue-name} "Add OAuth2 login with Google and GitHub providers, following existing passport.js pattern, with session management and logout support"
```

---

### Implementation Phase Issues

#### Issue: "Stopped after Phase 1 / MVP"

**Symptom:**
```
✓ Phase 1 complete
✓ Implementation complete!  ← WRONG
```

**Diagnosis:**
- Implementation stopped early
- Only MVP/features implemented
- Phases 2, 3, 4 not completed

**Recovery:**
```bash
# Check IMPLEMENTATION_PLAN.md for phase count
grep "Phase " docs/{issue-name}/IMPLEMENTATION_PLAN.md | wc -l

# Should show all phases (e.g., 4 phases)

# Resume implementation
/sdlc {issue-name} --from-implement

# Explicitly tell it to complete ALL phases:
"Continue from Phase 2. Implement ALL remaining phases from IMPLEMENTATION_PLAN.md. Do not stop until ALL phases are complete."
```

---

#### Issue: "Tests failing"

**Symptom:**
```
Tests: 10/15 passing
Error: Test failures detected
```

**Diagnosis:**
- Implementation has bugs
- Test expectations wrong
- Missing test setup

**Recovery:**
1. Read test output for specific failures
2. Fix implementation or tests as appropriate
3. Re-run tests: `npm test` (or equivalent)
4. Verify all tests pass before completing phase

**If tests are wrong:**
- Update test expectations
- Document why tests were updated
- Re-run implementation

**If implementation is wrong:**
- Fix the code
- Re-run tests
- Ensure no regressions

---

#### Issue: "Type check failed"

**Symptom:**
```
Type Checking: Failed (12 errors)
```

**Diagnosis:**
- TypeScript errors in implementation
- Missing type definitions
- Incorrect type annotations

**Recovery:**
1. Read type check output for specific errors
2. Fix type errors:
   - Add missing types
   - Correct type annotations
   - Fix type mismatches
3. Re-run type check: `npm run type-check` (or equivalent)
4. Verify no errors before completing

---

### Review Phase Issues

#### Issue: "No implementation changes found"

**Symptom:**
```
Error: No implementation changes found
```

**Diagnosis:**
- Implementation phase not completed
- No code files were created/modified
- Git shows no changes

**Recovery:**
```bash
# Check if implementation completed
cat docs/{issue-name}/IMPLEMENTATION_SUMMARY.md

# If missing, run implementation
/sdlc {issue-name} --from-implement

# Check git diff
git diff

# If no changes, implementation didn't create files
# Re-run implementation with explicit instruction to create code
```

---

#### Issue: "Automated checks unavailable"

**Symptom:**
```
Warning: Linting not configured
Warning: Type checking not available
```

**Diagnosis:**
- Project doesn't have these tools configured
- This is **non-blocking** for review

**Recovery:**
- Review proceeds with manual checks only
- Document which checks are unavailable
- Continue with manual code review

---

#### Issue: "Approval decision unclear"

**Symptom:**
```
Unable to determine approval status
```

**Diagnosis:**
- CODE_REVIEW.md doesn't have clear status
- Issues not categorized properly

**Recovery:**
1. Count issues in CODE_REVIEW.md:
   - Critical: Security, data loss, broken functionality
   - Important: Errors, poor handling, maintainability
   - Suggestions: Optional improvements

2. Determine status:
   ```
   If critical > 0 OR important > 3:
     → NEEDS_REVISION
   Else if suggestions > 0:
     → APPROVED_WITH_NOTES
   Else:
     → APPROVED
   ```

3. Update CODE_REVIEW.md with clear status

---

### Review-Fix Loop Issues

#### Issue: "Maximum iterations (3) reached"

**Symptom:**
```
Error: Maximum review-fix iterations (3) reached
Status: BLOCKED
```

**Diagnosis:**
- Review-fix loop ran 3 times
- Issues still remain
- Workflow blocked

**Recovery:**
```bash
# Check current state
cat docs/{issue-name}/STATUS.md

# Read CODE_REVIEW.md for remaining issues
cat docs/{issue-name}/CODE_REVIEW.md

# Options:
# 1. Manual intervention required
#    - Address remaining issues manually
#    - Update STATE.json current_phase to "complete"
#
# 2. Restart workflow with clearer requirements
#    /sdlc {issue-name} "Clearer feature description"
#
# 3. Accept current state (if issues are minor)
#    - Document known limitations
#    - Manually approve for deployment
```

---

#### Issue: "Fix made code worse"

**Symptom:**
```
Error: Fix caused regression
Tests were passing, now failing
```

**Diagnosis:**
- Fix introduced new bugs
- Tests now fail
- Implementation regressed

**Recovery:**
1. **Immediate:** Revert the fix
   ```bash
   git diff
   git checkout -- [files changed in fix]
   ```

2. **Verify:** Tests pass again
   ```bash
   npm test
   ```

3. **Alternative approach:** Fix differently
   - Read CODE_REVIEW.md issue again
   - Consider alternative solution
   - Apply new fix
   - Verify no regression

---

## General Issues

### Issue: "Wrong issue directory"

**Symptom:**
```
Working in wrong issue directory
```

**Recovery:**
```bash
# List all issue directories
ls docs/

# Identify correct directory
ls docs/{expected-name}/

# If wrong name used, restart with correct name
/sdlc {correct-issue-name} "Feature description"

# Or rename directory
mv docs/{wrong-name} docs/{correct-name}
```

---

### Issue: "Orchestrator stopped mid-workflow"

**Symptom:**
```
Workflow stopped unexpectedly
No completion message
```

**Diagnosis:**
- LLM context overflow
- Unclear instructions
- Error occurred

**Recovery:**
```bash
# Check current state
cat docs/{issue-name}/STATUS.md

# Resume from last phase
/sdlc {issue-name} --from-[last-phase]

# Available resume points:
# --from-research
# --from-plan
# --from-implement
# --from-review
```

---

### Issue: "Artifact format incorrect"

**Symptom:**
```
Artifact validation failed
Missing required sections
```

**Diagnosis:**
- Artifact doesn't follow expected format
- Required sections missing
- Content too brief

**Recovery:**
1. Check artifact against expected sections
2. Add missing sections
3. Re-run phase to regenerate

**Example - CODE_RESEARCH.md required sections:**
- Summary
- Integration Points
- Technical Context
- Risks
- Key Files
- Open Questions

---

### Issue: "Phase transition rejected"

**Symptom:**
```
InvalidTransitionError: Cannot transition from X to Y
```

**Diagnosis:**
- Attempting invalid phase transition
- Previous phase not complete
- State inconsistency

**Recovery:**
```bash
# Check current state
cat docs/{issue-name}/STATUS.md

# Verify previous phase is complete
# Look for [x] checkbox

# If previous phase incomplete, complete it first
/sdlc {issue-name} --from-[previous-phase]

# Valid transitions:
# start → research
# research → planning
# planning → implementation
# implementation → review
# review → complete OR fix
# fix → review (max 3 times)
```

---

## Prevention Best Practices

### 1. Use Clear Feature Descriptions

**Good:**
```bash
/sdlc add-oauth-login "Add OAuth2 authentication with Google and GitHub providers, following existing passport.js pattern in src/auth/, with session management and logout support"
```

**Bad:**
```bash
/sdlc feature "Add login"
```

### 2. Let Each Phase Complete Fully

- Don't interrupt phases
- Verify artifacts are created
- Check STATUS.md before proceeding

### 3. Read STATUS.md First

Always check STATUS.md before taking action:
```bash
cat docs/{issue-name}/STATUS.md
```

This tells you:
- Where you are in the workflow
- What's been completed
- What's next
- Any issues

### 4. Use Resume Points Wisely

If workflow stopped:
```bash
# Resume from where it stopped
/sdlc {issue-name} --from-[phase]
```

Don't restart from beginning unless necessary.

### 5. Verify Artifacts Exist

After each phase, verify:
```bash
ls -la docs/{issue-name}/
```

Expected files per phase should exist.

---

## Getting Help

### 1. Check Documentation

- `.claude/commands/README.md` - Workflow documentation
- `.claude/schemas/STATE.json.schema` - State reference
- `docs/SDL_WORKFLOW_COMPREHENSIVE_REVIEW.md` - System review

### 2. Check Examples

- `.claude/examples/` - Example artifacts
- `docs/example-fix-terminal-flicker/` - Full workflow example

### 3. Diagnostic Commands

```bash
# Check current phase
cat docs/{issue-name}/STATUS.md | grep "Phase:"

# List artifacts
ls -la docs/{issue-name}/

# Check for errors
grep -i "error" docs/{issue-name}/*.md

# Check state (if STATE.json exists)
cat docs/{issue-name}/STATE.json | jq '.current_phase'
```

### 4. Reset Workflow (Last Resort)

If completely stuck:
```bash
# Backup current work
mv docs/{issue-name} docs/{issue-name}.backup

# Restart from beginning
/sdlc {issue-name} "Feature description"

# Copy over any useful artifacts from backup
# cp docs/{issue-name}.backup/CODE_RESEARCH.md docs/{issue-name}/
```

---

## Quick Reference

### Phase Artifacts Checklist

| Phase | Check Artifacts Exist |
|-------|---------------------|
| Research | `ls docs/{issue}/CODE_RESEARCH.md RESEARCH_SUMMARY.md STATUS.md` |
| Planning | `ls docs/{issue}/IMPLEMENTATION_PLAN.md PROJECT_SPEC.md PLAN_SUMMARY.md STATUS.md` |
| Implementation | `ls docs/{issue}/IMPLEMENTATION_SUMMARY.md STATUS.md` + code files |
| Review | `ls docs/{issue}/CODE_REVIEW.md STATUS.md` |

### Resume Commands

```bash
/sdlc {name} --from-research    # Resume at research
/sdlc {name} --from-plan        # Resume at planning
/sdlc {name} --from-implement   # Resume at implementation
/sdlc {name} --from-review      # Resume at review
```

### Common Fixes

```bash
# Research: No files found
# → Update glob pattern, re-run

# Planning: Missing research
# → Run /sdlc {name} --from-research first

# Implementation: Stopped early
# → Run /sdlc {name} --from-implement, emphasize ALL phases

# Review: No changes
# → Run /sdlc {name} --from-implement first

# Fix: Max iterations reached
# → Manual intervention required
```

---

**Still stuck?** Check the comprehensive review document:
```bash
cat docs/SDL_WORKFLOW_COMPREHENSIVE_REVIEW.md
```
