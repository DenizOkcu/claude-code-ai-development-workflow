# User Communication Guidelines for SDLC Workflow

**Purpose:** Make workflow transparent by communicating progress to users
**Approach:** Markdown-only instructions for LLMs to follow
**Target:** Claude Code users executing `/sdlc` workflow

---

## User Touchpoints in Workflow

```
User runs: /sdlc add-feature "Description"
    ↓
[NOTIFICATION: Workflow Starting]
    ↓
[NOTIFICATION: Starting Research Phase]
    ↓
[PROGRESS UPDATE: Analyzing codebase...]
    ↓
[NOTIFICATION: Research Complete] → Shows what was found
    ↓
[NOTIFICATION: Starting Planning Phase]
    ↓
[PROGRESS UPDATE: Creating implementation plan...]
    ↓
[NOTIFICATION: Planning Complete] → Shows phases and estimates
    ↓
[NOTIFICATION: Starting Implementation Phase]
    ↓
[PROGRESS UPDATE: Phase 1/4: Foundation...]
[PROGRESS UPDATE: Phase 2/4: Routes...]
[PROGRESS UPDATE: Phase 3/4: Testing...]
[PROGRESS UPDATE: Phase 4/4: Documentation...]
    ↓
[NOTIFICATION: Implementation Complete] → Shows files created
    ↓
[NOTIFICATION: Starting Review Phase]
    ↓
[PROGRESS UPDATE: Running automated checks...]
[PROGRESS UPDATE: Performing code review...]
    ↓
[NOTIFICATION: Review Complete] → Shows approval status
    ↓
[NOTIFICATION: Workflow Complete] → Shows final summary
```

---

## Communication Principles

### 1. Be Proactive
- Don't wait for user to ask
- Provide updates before starting significant work
- Inform user of progress regularly

### 2. Be Specific
- Use file names and paths
- Include line numbers where relevant
- Show counts (files, tests, phases)

### 3. Be Actionable
- Tell user what's happening next
- Show expected completion time
- Provide checkpoint information

### 4. Be Scannable
- Use clear section headers
- Use bullet points for lists
- Use emojis for visual markers
- Keep updates concise

### 5. Be Honest
- Report errors immediately
- Show iteration counts
- Document deviations from plan

---

## Notification Templates

### Workflow Start Notification

```markdown
## 🚀 Starting SDLC Workflow

**Issue:** {issue-name}
**Description:** {feature-description}
**Entry Point:** {start | planning | implementation | review}

**Workflow:**
1. Research → Analyze codebase and patterns
2. Planning → Create implementation plan
3. Implementation → Build the feature
4. Review → Quality checks and approval

**Estimated Time:** {estimate based on complexity}

Starting with: {phase-name} phase
```

### Phase Start Notification

```markdown
## 📍 Phase: {Phase Name}

**Starting:** {phase-name}
**Goal:** {brief goal description}
**Expected Artifacts:**
- {artifact-1}
- {artifact-2}

**Estimated Time:** {time estimate}

Working...
```

### Progress Update (During Phase)

```markdown
**Progress Update:**

✓ {Completed task 1}
✓ {Completed task 2}
→ {Current task} (in progress)
○ {Upcoming task}

**Files Analyzed:** {N}
**Files Read:** {N}
**Current Focus:** {what's being worked on}
```

### Phase Completion Notification

```markdown
## ✅ Phase Complete: {Phase Name}

**Duration:** {actual time taken}
**Artifacts Created:**
- {artifact-1} ({size} lines, {purpose})
- {artifact-2} ({size} lines, {purpose})

**Key Findings:**
- {Finding 1}
- {Finding 2}

**Next Phase:** {next-phase-name}

Type "continue" to proceed, or wait for auto-continuation.
```

### Artifact Creation Notification

```markdown
**Created:** {artifact-name}

📄 Location: `docs/{issue-name}/{artifact-name}`
📊 Size: {lines} lines
💡 Purpose: {brief description}

**Key Sections:**
- {Section 1}
- {Section 2}

View: `cat docs/{issue-name}/{artifact-name}`
```

### Validation Result Notification

```markdown
## 🔍 Validation: {Phase Name}

**Gate Check:** {gate description}

**Results:**
✓ {Check 1}: Passed
✓ {Check 2}: Passed
✓ {Check 3}: Passed

**Validation:** PASSED ✓

Proceeding to: {next-phase}
```

### Validation Failure Notification

```markdown
## ⚠️ Validation Failed

**Gate Check:** {gate description}

**Results:**
✓ {Check 1}: Passed
✗ {Check 2}: Failed - {reason}
✓ {Check 3}: Passed

**Action Required:** {what needs to be fixed}

**Recovery:** {how to fix}

Cannot proceed until validation passes.
```

### Error Notification

```markdown
## ❌ Error: {Phase Name}

**Error Type:** {error category}
**Message:** {error message}

**What Happened:**
{description of what went wrong}

**Impact:**
{how this affects the workflow}

**Recovery Options:**
1. {Option 1}
2. {Option 2}

**Recommendation:** {best recovery approach}
```

### Review-Fix Loop Notification

```markdown
## 🔄 Review-Fix Iteration {N}/3

**Status:** Review identified issues that need fixing

**Issues Found:**
- **Critical:** {N}
- **Important:** {M}
- **Suggestions:** {K}

**Fix Scope:** Minimal (only critical + important issues)

**Working on fixes...**
```

### Workflow Completion Notification

```markdown
## 🎉 Workflow Complete!

**Issue:** {issue-name}
**Status:** {complete | blocked}
**Total Duration:** {total time}

---

### Summary

**Phases Completed:** {N}/4
- ✓ Research ({time})
- ✓ Planning ({time})
- ✓ Implementation ({time})
- ✓ Review ({time})

**Review Iterations:** {N} (if any)

---

### Artifacts Created

**Documentation:**
- CODE_RESEARCH.md
- RESEARCH_SUMMARY.md
- IMPLEMENTATION_PLAN.md
- PROJECT_SPEC.md
- PLAN_SUMMARY.md
- IMPLEMENTATION_SUMMARY.md
- CODE_REVIEW.md

**Code:**
- {File 1} - {purpose}
- {File 2} - {purpose}
- {Total: N files}

---

### Quality Metrics

**Automated Checks:**
- Linting: ✓ Passed
- Type Check: ✓ Passed
- Tests: ✓ {N}/{N} passing ({X%})
- Build: ✓ Passed

**Review Status:** {APPROVED | NEEDS_REVISION}

---

### Next Steps

{Clear guidance on what to do next}

1. {Step 1}
2. {Step 2}
3. {Step 3}

---

**View all artifacts:** `ls docs/{issue-name}/`
**View status:** `cat docs/{issue-name}/STATUS.md`
```

---

## Communication Formatting

### Visual Markers

Use these markers consistently:

| Marker | Meaning | Usage |
|--------|---------|-------|
| 🚀 | Workflow starting | Major milestone |
| 📍 | Phase transition | Entering new phase |
| ✓ | Task complete | Success indicator |
| ✗ | Task failed | Error indicator |
| → | In progress | Current activity |
| ○ | Pending | Not started |
| ⚠️ | Warning | Caution needed |
| ❌ | Error | Failure |
| ✅ | Phase complete | Success |
| 🔄 | Loop iteration | Review-fix cycle |
| 🎉 | Complete | Success |
| 📊 | Metric | Data point |
| 📄 | File | Artifact created |
| 💡 | Information | Helpful note |

### Progress Bars

For long-running phases:

```markdown
**Progress:** [████████████████░░░░] 75% (3/4 phases complete)

**Current Phase:** Implementation
**Completed:**
  ✓ Phase 1: Foundation
  ✓ Phase 2: Routes
  ✓ Phase 3: Integration
  → Phase 4: Testing (in progress)
```

### File Lists

```markdown
**Files Created:** 5
├── src/auth/google-strategy.ts (60 lines)
├── src/routes/auth.ts (35 lines added)
├── src/controllers/auth.ts (50 lines added)
├── tests/auth/oauth.test.ts (100 lines)
└── docs/auth/oauth.md (100 lines)
```

### Time Estimates

```markdown
**Time Tracking:**
- Research: 30min (completed)
- Planning: 45min (completed)
- Implementation: 2hr (completed)
- Review: 30min (completed)
- **Total: 3hr 25min**
```

---

## Communication Points by Phase

### Research Phase

**When to Notify:**
1. ✓ Phase start
2. ✓ Progress: Analyzing files (show file count)
3. ✓ Progress: Reading git history (if deep research)
4. ✓ Progress: Synthesizing findings
5. ✓ Phase complete (show risk level, key findings)

**What to Show:**
- Files analyzed
- Integration points found
- Risk level assessment
- Questions identified

### Planning Phase

**When to Notify:**
1. ✓ Phase start
2. ✓ Progress: Creating phases (show count)
3. ✓ Progress: Writing specifications
4. ✓ Progress: Creating summary
5. ✓ Phase complete (show phase count, estimates)

**What to Show:**
- Number of phases
- Estimated effort
- Key decisions
- Files to modify/create

### Implementation Phase

**When to Notify:**
1. ✓ Phase start
2. ✓ Progress: Starting Phase N/M (show each phase)
3. ✓ Progress: Creating files (show file names)
4. ✓ Progress: Running tests
5. ✓ Progress: Fixing test failures (if any)
6. ✓ Phase complete (show all files, test results)

**What to Show:**
- Phase progress (N/M)
- Files being created
- Test results
- Deviations from plan

### Review Phase

**When to Notify:**
1. ✓ Phase start
2. ✓ Progress: Running automated checks
3. ✓ Progress: Performing manual review
4. ✓ Phase complete (show approval status, issues)

**What to Show:**
- Automated check results
- Issue count (critical/important/suggestions)
- Approval status
- Iteration needed (yes/no)

### Fix Phase (if needed)

**When to Notify:**
1. ✓ Phase start
2. ✓ Progress: Fixing issue N/M (show each issue)
3. ✓ Progress: Re-running tests
4. ✓ Phase complete (show fixes applied)

**What to Show:**
- Issue being fixed
- Test results
- Remaining issues

---

## Don't Communicate

**Avoid:**
- ❌ Excessive detail (every file read)
- ❌ Technical jargon without context
- ❌ Raw LLM internals
- ❌ Repetitive status updates
- ❌ Information overload

**Rule of Thumb:** One progress update per significant action, not per file read.

---

## Quick Reference

### Phase Start
```markdown
## 📍 Phase: {Name}

Starting {phase}...
Goal: {goal}
Estimate: {time}
```

### Progress Update
```markdown
**Progress:** ✓ {done} → {doing} ○ {next}
```

### Artifact Created
```markdown
**Created:** {name} 📄
Location: `{path}`
Size: {lines} lines
```

### Phase Complete
```markdown
## ✅ Phase Complete: {Name}

Time: {duration}
Artifacts: {list}
Next: {next-phase}
```

### Error
```markdown
## ❌ Error

{error-type}: {message}
Recovery: {how-to-fix}
```

---

**End of User Communication Guidelines**
