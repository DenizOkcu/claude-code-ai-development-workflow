# Integration Plan: Restoring Missing Technical Knowledge

**Generated:** 2026-01-14
**Based on:** knowledge-gap-analysis.md
**Purpose:** Step-by-step instructions to enhance skills with missing technical knowledge

---

## Overview

This integration plan restores explicit technical guidance that was lost or less emphasized during the migration from slash-commands to skills-based architecture. The changes are organized by priority and skill file.

**Estimated Total Effort:** 2-3 hours
**Risk Level:** Low (additions only, no breaking changes)

---

## Phase 1: Critical Fixes (High Impact, Low Effort)

### 1.1 Add "Output Style" Section to solution-planning.md

**File:** `.claude/skills/solution-planning.md`
**Location:** After line 457 (after "Important Approach Guidelines" section)
**Priority:** HIGH

**Insert this new section:**

```markdown
## Output Style

Follow these explicit formatting rules for all planning artifacts:

### Documentation Format
- **Bullet points first** - Use paragraphs only when absolutely necessary
- **File refs with line numbers** - Always include: `src/utils/helper.ts:42`
- **Snippets only for novel/complex patterns** - Skip obvious examples
- **Tables for structured data** - Use for phases, estimates, components
- **Specific actions** - "Add useTimer hook" not "implement functionality"

### Good vs Bad Examples

**Good ✅:**
- `src/auth/oauth.ts:45` - Add error handling for OAuth failures
- `src/components/User.ts:100-120` - Implement user profile display
- Create `hooks/useTimer.ts:1-50` - Add timer hook for auto-refresh

**Bad ❌:**
- Implement OAuth functionality
- Add user features
- Create timer functionality

### Rationale

These formatting rules ensure:
- **Actionable plans** - Specific file locations and actions
- **Quick navigation** - Line numbers for direct access
- **Reduced verbosity** - Bullet points > paragraphs
- **Structured data** - Tables for easy scanning
```

**Why This Matters:**
- The old `issue-planner.md` had this explicit guidance
- Without it, plans may be wordy and less actionable
- Critical for maintaining consistent, high-quality planning artifacts

---

### 1.2 Enhance "Investigation Strategy" in code-research.md

**File:** `.claude/skills/code-research.md`
**Location:** Replace lines 196-206 (step 2 "Investigation Strategy")
**Priority:** HIGH

**Replace current content:**

```markdown
### 2. Investigation Strategy

**Quick depth** (simple changes):
- 5-10 files, focus on integration points and conventions

**Standard depth** (medium complexity):
- 15-30 files, full architecture mapping and dependency analysis

**Deep depth** (complex/major):
- 30+ files, include git history, performance, and security analysis
```

**With enhanced content:**

```markdown
### 2. Investigation Strategy

**Research Depth Levels:**

**Quick depth** (simple changes):
- 5-10 files, focus on integration points and conventions

**Standard depth** (medium complexity):
- 15-30 files, full architecture mapping and dependency analysis

**Deep depth** (complex/major):
- 30+ files, include git history, performance, and security analysis

**Search Methodology:**

**Start broad, then narrow:**
1. Search for relevant keywords in the entire codebase
2. Identify key files and directories
3. Deep dive into related components

**Use multiple search methods:**
- Grep for function/class names
- Glob for file patterns
- Read key files completely
- Trace import/export chains

**Follow the data flow:**
- Track how data enters the system
- Follow transformations
- See where data is stored/retrieved
- Understand side effects
```

**Why This Matters:**
- Restores the explicit "Start broad, then narrow" strategy
- Adds the "Use multiple search methods" guidance
- Makes "Follow the data flow" more prominent
- Critical for thorough, systematic codebase research

---

## Phase 2: Moderate Enhancements (Medium Impact, Low Effort)

### 2.1 Restore TypeScript Detail in code-implementation.md

**File:** `.claude/skills/code-implementation.md`
**Location:** Replace lines 366-388 (section 6, "TypeScript-Specific" subsections)
**Priority:** MEDIUM

**Replace current content:**

```markdown
**TypeScript-Specific (if applicable):**

*Type Safety:*
- Enable strict mode compliance
- Use explicit types for public APIs
- Leverage type inference for internal variables
- Create custom types/interfaces as specified in PROJECT_SPEC.md
- Use generics for reusable components
- Implement proper type guards for runtime checks

*Patterns:*
- Use interfaces for contracts, types for unions/intersections
- Implement factory patterns with proper typing
- Use dependency injection where appropriate
- Apply SOLID principles
- Leverage utility types (Partial, Pick, Omit, etc.)

*Error Handling:*
- Create typed error classes
- Use Result/Either types for operations that can fail
- Implement proper error boundaries
- Add descriptive error messages
```

**Wait - the current content is already quite similar. Let me check the old command for any missing bullets...**

Actually, looking at the current skill, it already has good TypeScript coverage. The old command had these additional bullets that are worth adding:

**Add to "Type Safety" subsection:**
- "Avoid `any` type (use `unknown` for truly dynamic types)"

**Add to "Patterns" subsection:**
- "Use branded types for type safety at runtime"

**Add to "Error Handling" subsection:**
- "Use discriminated unions for error types"

So the enhanced version should be:

```markdown
**TypeScript-Specific (if applicable):**

*Type Safety:*
- Enable strict mode compliance
- Use explicit types for public APIs
- Leverage type inference for internal variables
- Create custom types/interfaces as specified in PROJECT_SPEC.md
- Use generics for reusable components
- Implement proper type guards for runtime checks
- Avoid `any` type (use `unknown` for truly dynamic types)

*Patterns:*
- Use interfaces for contracts, types for unions/intersections
- Implement factory patterns with proper typing
- Use dependency injection where appropriate
- Apply SOLID principles
- Leverage utility types (Partial, Pick, Omit, etc.)
- Use branded types for type safety at runtime

*Error Handling:*
- Create typed error classes
- Use Result/Either types for operations that can fail
- Implement proper error boundaries
- Add descriptive error messages
- Use discriminated unions for error types
```

**Why This Matters:**
- Adds 3 missing TypeScript best practices
- Brings TypeScript guidance to parity with old command
- Important for high-quality TypeScript implementations

---

### 2.2 Consolidate "Communication Style" Section Across All Skills

**Files to modify:**
1. `.claude/skills/code-research.md`
2. `.claude/skills/solution-planning.md`
3. `.claude/skills/code-implementation.md`
4. `.claude/skills/code-review.md`
5. `.claude/skills/review-fix.md`

**Priority:** MEDIUM
**Note:** Some skills already have communication guidance. This adds consistency.

#### For code-research.md:

**Location:** After "Quality Checks" section (around line 482)

**Add:**

```markdown
## Communication Style

### Artifact Format
- **Bullet points over paragraphs** - Maximize information density
- **File references with line numbers** - `src/utils/helper.ts:42`
- **Specific findings** - "Found passport.js OAuth pattern" not "Auth research done"
- **Flag uncertainties** - Be explicit about unknowns

### Progress Updates
- Clear phase identification
- Investigation milestones
- Key findings summary
- Next steps
```

#### For solution-planning.md:

**Location:** After "Output Style" section (added in 1.1 above)

**Add:**

```markdown
## Communication Style

### Presentation Format
- **Bullet points over paragraphs** - Concise, scannable plans
- **File references with line numbers** - Enable quick navigation
- **Concrete proposals** - Specific files, functions, patterns
- **Flag uncertainties** - List open questions explicitly

### Presentation Style
- Present brief summary (3-5 bullets max)
- Number of phases and estimated effort
- Key architectural decisions
- Next step guidance
```

#### For code-implementation.md:

**Location:** Already has section 9 "Communication Style" - enhance it

**After line 453, add to "Progress Updates" subsection:**

```markdown
**Format Rules:**
- Bullet points for progress
- File:line references for all code changes
- Phase identification: "Phase N/M: [Phase Name]"
```

#### For code-review.md:

**Location:** After "Quality Checks" section (around line 467)

**Add:**

```markdown
## Communication Style

### Review Format
- **Bullet points over paragraphs** - Scannable review findings
- **File references with line numbers** - Specific issue locations
- **Constructive feedback** - Explain why + suggest fix
- **Acknowledge good work** - Note what was done well

### Approval Status
- Clear status (APPROVED/APPROVED_WITH_NOTES/NEEDS_REVISION)
- Issue counts (Critical/Important/Suggestions)
- Next steps (deploy/fix/escalate)
```

#### For review-fix.md:

**Location:** After "Quality Checks" section (around line 332)

**Add:**

```markdown
## Communication Style

### Fix Format
- **Minimal updates** - Only what changed
- **Strikethrough fixed issues** - Clear what was addressed
- **File references with line numbers** - Where fixes were applied
- **Test results** - Verification status

### Iteration Status
- Current iteration (N/3)
- Issues fixed this iteration
- Remaining issues (if any)
- Next steps
```

**Why This Matters:**
- Ensures consistent communication across all skills
- Makes output more predictable and actionable
- Improves overall SDLC workflow coherence

---

## Phase 3: Validation (Verify Effectiveness)

### 3.1 Test Enhanced Skills

**Create test issues** to verify enhancements:

```bash
# Test 1: Simple feature (Quick research depth)
/sdlc add-health-check "Add a health check endpoint"

# Test 2: Medium feature (Standard research depth)
/sdlc add-pagination "Add pagination to user list API"

# Test 3: Complex feature (Deep research depth)
/sdlc add-websockets "Add WebSocket support for real-time updates"
```

**Verify:**
1. Research uses "Start broad, then narrow" methodology
2. Planning artifacts follow "Output Style" rules
3. Plans use bullet points, not paragraphs
4. File references include line numbers
5. Plans specify concrete actions

### 3.2 Compare Output Quality

**Before Integration:** (use existing artifacts)
- Review `docs/*/RESEARCH_SUMMARY.md` files
- Review `docs/*/PLAN_SUMMARY.md` files
- Check for verbosity, missing line numbers, vague actions

**After Integration:** (run new test issues)
- Compare new artifacts with old ones
- Verify improved consistency and actionability
- Check for restored formatting rules

### 3.3 Update Documentation

**Files to update:**
1. `.claude/ARCHITECTURE.md` - Add note about enhanced skills
2. `.claude/TROUBLESHOOTING.md` - Add guidance if output is inconsistent
3. `README.md` - Update "Skills" section if needed

---

## Implementation Checklist

### Pre-Implementation
- [ ] Review knowledge-gap-analysis.md
- [ ] Backup current skills directory
- [ ] Create branch for integration work

### Phase 1: Critical Fixes
- [ ] 1.1 Add "Output Style" to solution-planning.md
- [ ] 1.2 Enhance "Investigation Strategy" in code-research.md
- [ ] Test with simple feature
- [ ] Verify output improvements

### Phase 2: Moderate Enhancements
- [ ] 2.1 Restore TypeScript detail in code-implementation.md
- [ ] 2.2 Add "Communication Style" to code-research.md
- [ ] 2.2 Add "Communication Style" to solution-planning.md
- [ ] 2.2 Enhance "Communication Style" in code-implementation.md
- [ ] 2.2 Add "Communication Style" to code-review.md
- [ ] 2.2 Add "Communication Style" to review-fix.md
- [ ] Test with medium and complex features
- [ ] Verify consistency across skills

### Phase 3: Validation
- [ ] 3.1 Run test issues
- [ ] 3.2 Compare before/after output quality
- [ ] 3.3 Update documentation
- [ ] Create summary of changes

### Post-Implementation
- [ ] Update knowledge-gap-analysis.md with "RESOLVED" status
- [ ] Delete integration-plan.md (or archive it)
- [ ] Commit changes with descriptive message
- [ ] Test full SDLC workflow end-to-end

---

## Testing Strategy

### Unit Testing (Per Skill)

**code-research.md:**
- Create test issue requiring Quick depth
- Verify "Start broad, then narrow" methodology in log
- Check CODE_RESEARCH.md for bullet points > paragraphs
- Verify file:line references present

**solution-planning.md:**
- Create test issue requiring planning
- Check IMPLEMENTATION_PLAN.md for:
  - Bullet points (not paragraphs)
  - File:line references
  - Specific actions (not vague "implement X")
  - Tables for phases/estimates
- Check PROJECT_SPEC.md for:
  - Code snippets only for novel patterns
  - Structured data in tables

**code-implementation.md:**
- Create test issue requiring TypeScript implementation
- Verify TypeScript best practices applied:
  - No `any` types
  - Discriminated unions for errors
  - Branded types where appropriate
- Check all phases completed (not just MVP)

**code-review.md:**
- Run review after implementation
- Verify CODE_REVIEW.md has:
  - Structured format
  - File:line references for issues
  - Clear approval status
- Check communication style consistency

**review-fix.md:**
- Create review with issues
- Run fix iteration
- Verify minimal context loading
- Check strikethrough format in updated CODE_REVIEW.md

### Integration Testing (Full Workflow)

**Test Scenario 1: Simple Feature**
```bash
/sdlc add-health-check "Add a health check endpoint that returns uptime"
```
Expected: All phases complete, output follows formatting rules

**Test Scenario 2: Medium Feature**
```bash
/sdlc add-pagination "Add pagination to user list API with cursor-based approach"
```
Expected: All phases complete, TypeScript best practices applied

**Test Scenario 3: Complex Feature**
```bash
/sdlc add-caching "Add Redis caching layer with TTL and cache invalidation"
```
Expected: Deep research, comprehensive planning, all phases complete

### Quality Metrics

**Before Integration:**
- Average artifact line count (baseline)
- Frequency of file:line references (baseline)
- Paragraph vs bullet point ratio (baseline)

**After Integration:**
- Reduced verbosity (target: 20% fewer lines)
- Increased file:line references (target: 100% coverage)
- Increased bullet point usage (target: 80%+ bullets)

---

## Rollback Plan

If issues arise after integration:

1. **Identify the problematic change:**
   - Check which skill is causing issues
   - Review the specific addition

2. **Partial rollback:**
   - Revert only the problematic skill file
   - Keep other enhancements

3. **Full rollback:**
   - Restore from backup: `cp -r .claude/skills.backup .claude/skills`
   - Document what didn't work
   - Adjust integration plan

4. **Iterate:**
   - Make smaller, incremental changes
   - Test each change individually
   - Re-integrate validated changes

---

## Success Criteria

Integration is successful when:

1. **All Phase 1 changes applied:**
   - [ ] solution-planning.md has "Output Style" section
   - [ ] code-research.md has enhanced "Investigation Strategy"

2. **All Phase 2 changes applied:**
   - [ ] code-implementation.md has restored TypeScript detail
   - [ ] All 5 skills have "Communication Style" sections

3. **Testing validates improvements:**
   - [ ] Test issues produce formatted output
   - [ ] Output quality metrics improved
   - [ ] No regressions in SDLC workflow

4. **Documentation updated:**
   - [ ] knowledge-gap-analysis.md marked resolved
   - [ ] Relevant docs updated with changes

---

## Next Steps

1. **Review this plan** and ensure all changes make sense
2. **Create backup** of current skills directory
3. **Begin Phase 1** (Critical Fixes)
4. **Test after each change** (don't batch)
5. **Proceed to Phase 2** after Phase 1 validation
6. **Complete Phase 3** validation
7. **Commit changes** with clear description

---

## Questions to Resolve

Before implementing:

1. Should we add "Output Style" to ALL skills or just solution-planning.md?
   - **Recommendation:** All skills, with skill-specific examples

2. Should the "Communication Style" sections be identical or customized?
   - **Recommendation:** Customized per skill (research ≠ planning ≠ implementation)

3. Should we add more TypeScript examples or keep guidelines concise?
   - **Recommendation:** Keep concise, add external reference to TypeScript docs

4. Should we add automated tests for output format validation?
   - **Recommendation:** Manual testing first, consider automated tests later

---

**End of Integration Plan**
