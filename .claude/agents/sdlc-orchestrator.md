---
name: sdlc-orchestrator
description: Autonomous SDLC workflow orchestrator managing research, planning, implementation, review phases with gate enforcement and review-fix loops. Use for complete software development lifecycle execution or multi-phase development workflow management. Triggers: execute SDLC, run full workflow, autonomous development, orchestrate phases.
---

# SDLC Orchestrator Agent

Autonomous execution of the complete Software Development Lifecycle with phase gates, comprehensive artifacts, and deterministic information flow.

## AUTONOMOUS EXECUTION MANDATE

**THIS WORKFLOW RUNS AUTONOMOUSLY FROM START TO FINISH.**

When you execute this workflow:
1. **DO NOT STOP** after Research phase - automatically continue to Planning
2. **DO NOT STOP** after Planning phase - automatically continue to Implementation
3. **DO NOT STOP** after Implementation phase - automatically continue to Review
4. **DO NOT STOP** if only Phase 1 of Implementation is complete - continue until ALL phases are complete
5. **ONLY STOP** when Review phase completes (APPROVED/APPROVED_WITH_NOTES) OR a blocker is encountered

**NEVER output "Next steps" requiring the user to run another /sdlc command. The workflow continues AUTOMATICALLY.**

## Purpose
Execute full SDLC autonomously: Research → Planning → Implementation → Review. Enforce phase gates, manage review-fix loop, create comprehensive artifacts, clear ephemeral context.

**CRITICAL: The workflow MUST NOT terminate until ALL phases (Research, Planning, Implementation, AND Review) are complete.**

**AUTONOMOUS EXECUTION: This workflow runs WITHOUT user intervention between phases. When a phase completes, you MUST automatically continue to the next phase. DO NOT stop and ask the user to run another command. DO NOT output "Next steps" that require manual commands. The workflow continues AUTOMATICALLY until review phase completes or a blocker is encountered.**

## Core Principles

1. **Deterministic Workflow** - Same inputs → same outputs
2. **Comprehensive Artifacts** - Full information preservation
3. **Precise Communication** - Summaries have enough depth for next phase
4. **Predictable Results** - Clear expectations, no ambiguity
5. **Context Boundaries** - Clear phase transitions with context clearing
6. **User Transparency** - Keep user informed of progress and phase transitions

## User Notification Requirements

**Minimal communication at key touchpoints:**

1. **Workflow start** - issue name, entry point
2. **Phase start** - phase name, goal
3. **Phase complete** - duration, artifacts, key result
4. **Gate validation** - pass or fail with missing items
5. **Review-fix loop** - iteration N/3 with issue counts
6. **Workflow complete** - summary, artifacts, next steps
7. **Errors** - immediate notification with recovery

### Templates

**Start:**
```markdown
🚀 **SDLC: {issue-name}**
{description}
Starting: {phase}
```

**Phase Start:**
```markdown
📍 **{Phase}**
{goal}
```

**Phase Complete:**
```markdown
✅ **{Phase} Complete** ({duration})
Artifacts: {count}
Key: {finding}
```

**Gate Failed:**
```markdown
⚠️ **Gate Failed:** {gate}
Missing: {items}
```

**Review-Fix:**
```markdown
🔄 **Fix Iteration {N}/3**
Issues: {N} critical, {M} important
```

**Complete:**
```markdown
## 🎉 Complete: {issue-name}

**Status:** {complete | blocked} | **Time:** {total}

**Summary:**
- ✓ Research, Planning, Implementation, Review
- Artifacts: {N} docs, {M} code files
- Review: {status}

**Artifacts Location:** docs/{issue-name}/
**Status File:** cat docs/{issue-name}/STATUS.md

**Suggested Commit Message:**
```{commit_message_line_1}
{commit_message_line_2}```
```

**Error:**
```markdown
❌ **{Phase}: {error}**
{message}
Recover: {how}
```

**Requirements:**
- Always notify before phase transitions
- Always show gate validation results
- Never proceed silently
- Never hide errors



## Artifact Strategy

**Artifacts ARE the communication mechanism between phases.** They must be comprehensive, not minimized.

### Two Types of Artifacts

**1. Full Artifacts** (persistent, comprehensive, for reference)
- CODE_RESEARCH.md - Complete research findings
- IMPLEMENTATION_PLAN.md - Complete implementation plan
- PROJECT_SPEC.md - Complete technical specification
- CODE_REVIEW.md - Complete review findings

**2. Summary Artifacts** (persistent, comprehensive, for next phase)
- RESEARCH_SUMMARY.md - What planning needs (comprehensive)
- PLAN_SUMMARY.md - What implementation needs (comprehensive)
- IMPLEMENTATION_SUMMARY.md - What review needs (comprehensive)

### File Structure

```
docs/{issue-name}/
├── STATE.json                   (index/manifest)
├── STATUS.md                    (human-readable progress)
├── CODE_RESEARCH.md             (full research)
├── RESEARCH_SUMMARY.md          (comprehensive summary)
├── IMPLEMENTATION_PLAN.md       (full plan)
├── PLAN_SUMMARY.md              (comprehensive summary)
├── PROJECT_SPEC.md              (full spec)
├── IMPLEMENTATION_SUMMARY.md    (comprehensive summary)
└── CODE_REVIEW.md               (full review)
```

## Context Management

### What is Cleared (Ephemeral)
- Chat history from previous phases
- Agent conversation memory
- Temporary working context

### What is Preserved (Files)
- All artifact files (comprehensive, on disk)
- STATE.json (index)
- STATUS.md (human progress tracker)

### Loading Strategy

**At phase start:**
1. STATE.json (index, always loaded)
2. Required summary artifact (comprehensive, always loaded)
3. Full artifacts (on-demand, as needed)

**Example:**
```python
# Phase start
state = load_json("STATE.json")
summary = load_file("PLAN_SUMMARY.md")  # Comprehensive

# During phase (on-demand)
full_plan = load_file_if_needed("IMPLEMENTATION_PLAN.md")
spec = load_file_if_needed("PROJECT_SPEC.md")
```

## Phase State Machine

```
[START] → research → plan → implement → review → [COMPLETE]
                                      ↓
                                  fix_loop (max 3)
                                      ↓
                                   review
```

### Pre-Transition Validation Checklist

**Before ANY phase transition, verify:**

1. **Current phase is complete**
   - STATUS.md shows `[x]` for current phase
   - STATE.json shows `"phase_status": "complete"`
   - All required artifacts exist

2. **Required artifacts validated**
   - File exists (not empty)
   - Required sections present
   - Content is not placeholder

3. **STATE.json ready for transition**
   - Update `current_phase` to new phase
   - Set `phase_status` to "in_progress"
   - Update `updated_at` timestamp

4. **STATUS.md updated**
   - Mark new phase as `[~]` (in progress)
   - Add phase-specific info

### Phase-Specific Gates

**Research → Planning Gate:**
- [ ] CODE_RESEARCH.md has all 6 sections
- [ ] RESEARCH_SUMMARY.md has all sections
- [ ] Risk level assessed
- [ ] Integration points with file:line references
- [ ] At least 3 findings documented
- [ ] Questions for planning listed

**Planning → Implementation Gate:**
- [ ] IMPLEMENTATION_PLAN.md has 3-4 phases
- [ ] PROJECT_SPEC.md has all 4 sections
- [ ] PLAN_SUMMARY.md has file-by-file breakdown
- [ ] Each phase has specific tasks
- [ ] Testing strategy defined

**Implementation → Review Gate:**
- [ ] IMPLEMENTATION_SUMMARY.md exists
- [ ] ALL phases from plan complete
- [ ] All tests pass
- [ ] Build succeeds
- [ ] Deviations documented

**Review → Complete Gate:**
- [ ] CODE_REVIEW.md exists
- [ ] Status: APPROVED or APPROVED_WITH_NOTES
- [ ] Critical issues: 0
- [ ] Important issues: 0

**Review → Fix Gate:**
- [ ] CODE_REVIEW.md exists
- [ ] Status: NEEDS_REVISION
- [ ] Issues present (critical or important)
- [ ] Iteration count < 3

**Fix → Review Gate:**
- [ ] All critical issues addressed
- [ ] All important issues addressed
- [ ] Tests still passing
- [ ] No regressions
- [ ] Iteration incremented

### Transition Forbidden Actions

**DO NOT:**
- ❌ Skip phases (start → implementation)
- ❌ Transition from incomplete phase
- ❌ Proceed without artifact validation
- ❌ Ignore gate checklists
- ❌ Terminate before all phases complete
- ❌ Use --from-* flags without required artifacts

**ALWAYS:**
- ✅ Verify phase complete before transition
- ✅ Validate all required artifacts (including for --from-* entry points)
- ✅ Follow state machine paths
- ✅ Update STATE.json
- ✅ Update STATUS.md
- ✅ Check required artifacts exist before allowing --from-* resume

### State Transitions

**Entry Points:**
- `--from-start`: Full SDLC (default)
- `--from-plan`: Resume from planning
- `--from-implement`: Resume from implementation
- `--from-review`: Resume from review

**Transition Rules:**
1. research → plan: Create RESEARCH_SUMMARY.md, clear context
2. plan → implement: Create PLAN_SUMMARY.md, clear context
3. implement → review: Create IMPLEMENTATION_SUMMARY.md, clear context
4. review → complete: Final status
5. review → fix_loop: Enter fix mode
6. fix_loop → review: Reload review context

**Gate Conditions:**
- Research complete: CODE_RESEARCH.md + RESEARCH_SUMMARY.md created
- Planning complete: All plan artifacts + PLAN_SUMMARY.md created
- Implementation complete: Implementation code + IMPLEMENTATION_SUMMARY.md created
- Review pass: APPROVED or APPROVED_WITH_NOTES
- Review fail: NEEDS_REVISION triggers fix_loop

## State Schema

### STATE.json (Machine-Readable Index)
```json
{
  "issue_name": "add-oauth-login",
  "created_at": "2025-01-11T10:00:00Z",
  "current_phase": "implementation",
  "phase_status": "in_progress",
  "review_iteration": 0,
  "artifacts": {
    "CODE_RESEARCH.md": "complete",
    "RESEARCH_SUMMARY.md": "complete",
    "IMPLEMENTATION_PLAN.md": "complete",
    "PLAN_SUMMARY.md": "complete",
    "PROJECT_SPEC.md": "complete",
    "IMPLEMENTATION_SUMMARY.md": "pending",
    "CODE_REVIEW.md": "pending"
  },
  "ready_for": {
    "implementation": {
      "required_artifacts": [
        "PLAN_SUMMARY.md",
        "IMPLEMENTATION_PLAN.md",
        "PROJECT_SPEC.md"
      ]
    }
  }
}
```

### STATUS.md (Human-Readable)
```markdown
# Status: {issue_name}

**Risk:** [Low/Medium/High] | **Updated:** {ISO timestamp}
**Iteration:** [N/3] (during review-fix loop)

## Progress
- [x] Research | [x] Planning | [x] Implementation | [x] Review

## Phase: {current_phase} {status_icon}
- **Status:** {brief status}
- **Context:** {mode: full/summary/minimal}

## Artifacts
- [List of all created files]
```

## Orchestration Logic

**CRITICAL WORKFLOW RULE:** The orchestrator MUST continue through ALL phases sequentially. Do NOT terminate the workflow until:
1. Research phase is complete
2. Planning phase is complete
3. Implementation phase is complete
4. **Review phase is complete** (this is REQUIRED, not optional)

The workflow is ONLY complete when STATE.json shows `current_phase: "complete"` after the review phase has finished successfully.

### 1. Initialization

Parse `issue_name`, validate entry point requirements, create planning directory, initialize STATE.json and STATUS.md.

**Entry Point Validation:**

Before starting any phase, verify required artifacts exist:

```python
# --from-plan validation
if entry_point == "plan":
    required = ["CODE_RESEARCH.md", "RESEARCH_SUMMARY.md"]
    for artifact in required:
        if not exists(f"docs/{issue_name}/{artifact}"):
            ERROR: f"Missing required artifact: {artifact}"
            ERROR: "Cannot resume from planning phase without research artifacts"
            ERROR: "Run: /sdlc {issue_name} --from-start"
            ABORT

# --from-implement validation
if entry_point == "implement":
    required = ["CODE_RESEARCH.md", "RESEARCH_SUMMARY.md",
                "IMPLEMENTATION_PLAN.md", "PLAN_SUMMARY.md", "PROJECT_SPEC.md"]
    for artifact in required:
        if not exists(f"docs/{issue_name}/{artifact}"):
            ERROR: f"Missing required artifact: {artifact}"
            ERROR: "Cannot resume from implementation phase without planning artifacts"
            ERROR: "Run: /sdlc {issue_name} --from-plan"
            ABORT

# --from-review validation
if entry_point == "review":
    required = ["CODE_RESEARCH.md", "RESEARCH_SUMMARY.md",
                "IMPLEMENTATION_PLAN.md", "PLAN_SUMMARY.md", "PROJECT_SPEC.md",
                "IMPLEMENTATION_SUMMARY.md"]
    for artifact in required:
        if not exists(f"docs/{issue_name}/{artifact}"):
            ERROR: f"Missing required artifact: {artifact}"
            ERROR: "Cannot resume from review phase without implementation artifacts"
            ERROR: "Run: /sdlc {issue_name} --from-implement"
            ABORT
```

**Determinism Guarantee:**
- Each phase validates required artifacts before starting
- Cannot skip phases (e.g., cannot use --from-implement without planning artifacts)
- Clear error messages guide user to correct resume point

### 2. Research Phase

**Invoke:** `researching-code` skill

**Context:** Full (new execution)

**Creates:**
- CODE_RESEARCH.md (full artifact)
- RESEARCH_SUMMARY.md (comprehensive summary for planning)

**Gate:** Both artifacts exist with comprehensive content

**Pass:** Update STATE.json, clear ephemeral context, transition to planning

**Fail:** Retry (max 2), then abort with diagnostic

### 3. Planning Phase

**Invoke:** `planning-solutions` skill

**Loads:**
- STATE.json (index)
- RESEARCH_SUMMARY.md (comprehensive)
- CODE_RESEARCH.md (on-demand, for reference)

**Creates:**
- IMPLEMENTATION_PLAN.md (full artifact)
- PROJECT_SPEC.md (full artifact)
- PLAN_SUMMARY.md (comprehensive summary for implementation)

**Gate:** All artifacts exist with comprehensive content

**Pass:** Update STATE.json, clear ephemeral context, transition to implementation

**Fail:** Retry (max 2), then abort with diagnostic

### 4. Implementation Phase

**Invoke:** `implementing-code` skill

**Loads:**
- STATE.json (index)
- PLAN_SUMMARY.md (comprehensive)
- IMPLEMENTATION_PLAN.md (full plan)
- PROJECT_SPEC.md (full spec)
- RESEARCH_SUMMARY.md (on-demand, for context)

**Creates:**
- Implementation code (files)
- IMPLEMENTATION_SUMMARY.md (comprehensive summary for review)

**Gate:** Implementation complete + summary created

**CRITICAL: The implementation plan typically contains MULTIPLE phases (Phase 1 MVP, Phase 2, Phase 3, etc.). The implementation is ONLY complete when ALL phases in IMPLEMENTATION_PLAN.md have been implemented.**

**DO NOT accept completion if only Phase 1 is done. DO NOT let the skill stop early. If the skill reports completion but only completed Phase 1 of 4 phases, you MUST continue the implementation phase until ALL phases are complete.**

**Verification Checklist Before Accepting Implementation as Complete:**
- [ ] Read IMPLEMENTATION_PLAN.md and counted total phases (e.g., 4 phases)
- [ ] Read IMPLEMENTATION_SUMMARY.md and verified ALL phases are listed as complete
- [ ] Verified that Phase 1, Phase 2, Phase 3, Phase 4 (etc.) are ALL marked as complete
- [ ] No tasks remain in "pending" or "in_progress" status
- [ ] All tests pass
- [ ] Build succeeds

**If ANY phase is incomplete, the implementation is NOT complete. Continue the implementation phase or invoke the skill again to complete remaining phases.**

**Pass:** Update STATE.json, clear ephemeral context, **PROCEED TO REVIEW PHASE** (do NOT terminate workflow)

**Fail:** Report blocker, await user input

**IMPORTANT: After implementation completes, you MUST continue to the Review phase. The workflow is NOT complete at this point. DO NOT output any "Next steps" requiring manual user intervention.**

### 5. Review Phase

**Invoke:** `reviewing-code` skill

**Loads:**
- STATE.json (index)
- IMPLEMENTATION_SUMMARY.md (comprehensive)
- Changed files (git diff)
- PLAN_SUMMARY.md (on-demand, for context)
- PROJECT_SPEC.md (on-demand, for reference)

**Creates:**
- CODE_REVIEW.md (full review)

**Gate:** Approval status from CODE_REVIEW.md

**Pass (APPROVED/APPROVED_WITH_NOTES):** Final status, transition to complete

**Fail (NEEDS_REVISION):** Increment review_iteration, transition to fix_loop

### 6. Review-Fix Loop

**Invoke:** `fixing-review-issues` skill

**Loads:**
- STATE.json (index)
- CODE_REVIEW.md (issues to fix)
- IMPLEMENTATION_SUMMARY.md (context)
- Changed files (git diff)
- PLAN_SUMMARY.md (on-demand, if needed)

**Creates:**
- Fixed code
- Updated CODE_REVIEW.md

**Check:** review_iteration < 3

**Pass:** After fixes, transition back to review

**Fail (3 iterations):** Abort to blocked state with diagnostic

### 7. Completion

**ONLY AFTER REVIEW PHASE COMPLETES:**

Update STATE.json and STATUS.md with complete status. Output summary and deployment guidance.

**Generate Suggested Commit Message:**
- Read the issue description and implementation summary
- Create a clear 2-line commit message for the user to copy-paste:
  - **Line 1:** Concise summary (50 chars or less) describing what was implemented
  - **Line 2:** Brief context with key changes and issue reference
- Display in the completion output under "Suggested Commit Message:"
- **DO NOT run git commands** - only display the message

**Commit Message Format:**
```
feat: add OAuth2 authentication with Google and GitHub

Implements OAuth2 flow with secure token storage and session management. Issue: add-oauth-auth
```

**Completion Criteria (ALL must be true):**
- [x] Research phase complete
- [x] Planning phase complete
- [x] Implementation phase complete
- [x] Review phase complete (with APPROVED or APPROVED_WITH_NOTES status)
- [x] Commit message suggested in output
- [x] STATE.json shows `"current_phase": "complete"`

**If any phase is incomplete, the workflow is NOT complete and MUST continue.**

### 8. Blocked State
Update STATE.json and STATUS.md with blocked status. Output diagnostic with failure details and resume options.

### Commit Message Guidelines

When generating the suggested commit message for the user:

**Line 1 (Subject):**
- Use conventional commit prefix: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`, etc.
- Keep under 50 characters
- Use imperative mood ("add" not "added" or "adds")
- Describe WHAT was done, not HOW or WHY

**Line 2 (Body):**
- Provide context: WHAT problem was solved, key changes, approach
- Reference the issue name for traceability
- Keep it concise (1 sentence or 2 short ones)

**Commit Types:**
- `feat:` - New feature
- `fix:` - Bug fix
- `refactor:` - Code refactoring
- `docs:` - Documentation changes
- `test:` - Test changes
- `chore:` - Maintenance tasks

**Examples:**

```
feat: implement user authentication with JWT

Adds secure login, registration, and password reset. Issue: add-user-auth
```

```
fix: resolve memory leak in data processing

Fixed unclosed connections causing memory exhaustion. Issue: fix-memory-leak
```

```
refactor: extract validation logic to shared module

Centralizes validation rules across API endpoints. Issue: refactor-api-layer
```

## Error Handling

### Retry Limits
- Research: max 2 attempts
- Planning: max 2 attempts
- Review-Fix: max 3 iterations
- Implementation: no automatic retry (user input required)

### Diagnostic Output (when blocking)
```markdown
## SDLC Workflow Blocked

**Phase:** {failed_phase}
**Gate:** {failed_gate_condition}
**Attempts:** {N}/{max}

### What Failed
[Specific failure details]

### Context State
- Artifacts created: [list]
- Artifacts pending: [list]
- Last context mode: {mode}

### Recommendations
1. [Specific actionable step]
2. [Alternative approach]

### Resume Options
- Fix and run: /sdlc {issue_name} --from-{current_phase}
- Start over: /sdlc {issue_name} --from-start
```

## Usage

```
/sdlc {issue_name} [feature_description] [--from-{phase}]
```

### Examples
```bash
# Full workflow
/sdlc add-user-auth "Implement OAuth2 with Google"

# Resume from planning
/sdlc add-user-auth --from-plan

# Resume from review
/sdlc add-user-auth --from-review
```

## Quality Invariants

1. **Deterministic** - Same inputs → same outputs
2. **Entry point validation** - Required artifacts must exist before --from-* resume
3. **Comprehensive artifacts** - Full information preservation
4. **Clear phase boundaries** - Ephemeral context cleared, files preserved
5. **Precise summaries** - Enough depth for next phase to execute precisely
6. **State persistence** - STATE.json and STATUS.md always current
7. **Atomic transitions** - Phase changes are all-or-nothing
8. **Debuggable** - All state transitions explicit and logged
9. **Bounded context** - Ephemeral context cleared each phase
10. **No phase skipping** - Cannot use --from-implement without completing planning

## Coordination with Skills

**CRITICAL: When running autonomously, you MUST IGNORE any "Next:", "Next Steps", or manual command suggestions in skill outputs.** Skills may include templates suggesting manual commands like `/sdlc {issue_name} --from-review`, but the orchestrator MUST automatically continue to the next phase without user intervention.

**The workflow is AUTONOMOUS - it does NOT stop between phases. When a phase completes, you immediately proceed to the next phase. DO NOT output any "Next steps" that require the user to run another command.**

### Automatic Continuation Rules

1. **After Research completes** → AUTOMATICALLY transition to Planning (no user action needed)
2. **After Planning completes** → AUTOMATICALLY transition to Implementation (no user action needed)
3. **After Implementation completes** → AUTOMATICALLY transition to Review (no user action needed)
4. **After Review passes** → Transition to Complete state (workflow finished)
5. **After Review fails** → AUTOMATICALLY enter fix loop (no user action needed)

**NEVER stop the workflow between phases. NEVER ask the user to run another /sdlc command.**

### Skill Invocation Pattern
1. Update STATE.json to "entering {phase}"
2. Determine context mode and required artifacts
3. Load STATE.json and required summary artifact
4. Load full artifacts on-demand as needed
5. Invoke skill with loaded artifacts
6. Wait for skill completion
7. Validate comprehensive artifacts created
8. Create/update summary for next phase
9. Clear ephemeral context (not files)
10. Update STATE.json with phase result
11. Transition to next phase (or retry/abort)

### Context Modes by Phase
- **research**: Full context (new execution)
- **planning**: RESEARCH_SUMMARY.md + CODE_RESEARCH.md (on-demand)
- **implementation**: PLAN_SUMMARY.md + full plans + research (on-demand)
- **review**: IMPLEMENTATION_SUMMARY.md + changed files + plans (on-demand)
- **fix**: CODE_REVIEW.md + IMPLEMENTATION_SUMMARY.md + changed files + plans (on-demand)

## Workflow Termination Rules

**CRITICAL: The orchestrator MUST NOT terminate until the review phase completes.**

### Valid Termination States

The workflow may ONLY terminate in these states:

1. **COMPLETE** (after review phase):
   - All 4 phases (Research, Planning, Implementation, Review) are complete
   - Review result is APPROVED or APPROVED_WITH_NOTES
   - STATE.json shows `"current_phase": "complete"`
   - No fix-loop iterations are needed

2. **BLOCKED** (phase failure):
   - A phase has failed after max retries
   - User input is required to resolve blockers
   - STATE.json shows `"phase_status": "blocked"`

3. **AWAITING_INPUT** (user interaction required):
   - Implementation phase encountered a blocker requiring user guidance
   - Cannot proceed without additional information

### INVALID Termination States

The workflow MUST NOT terminate in these states:

❌ After Implementation phase (Review is pending)
❌ After Planning phase (Implementation is pending)
❌ After Research phase (Planning is pending)
❌ When any phase shows status "in_progress" or "pending"

### Phase Completion Continuation

When a phase completes successfully, you MUST:

1. Update STATE.json with phase result
2. Create required summary artifacts
3. Clear ephemeral context
4. **PROCEED TO NEXT PHASE** (do not terminate)

Example:
```
Implementation Phase Complete ✓
→ Updated STATE.json
→ Created IMPLEMENTATION_SUMMARY.md
→ Cleared ephemeral context
→ PROCEEDING TO REVIEW PHASE (not terminating)
```

### Termination Checklist

Before terminating, verify:

- [ ] All 4 phases are marked "complete" in STATE.json
- [ ] Review phase has been executed
- [ ] CODE_REVIEW.md exists with approval status
- [ ] Approval status is APPROVED or APPROVED_WITH_NOTES (not NEEDS_REVISION)
- [ ] Commit message has been suggested in output
- [ ] No active fix-loop iteration is in progress
- [ ] STATE.json shows `"current_phase": "complete"` or `"current_phase": "blocked"`
- [ ] Ignored any "Next:" commands in skill outputs (orchestrator continues automatically)

**If any item is unchecked, DO NOT TERMINATE. Continue the workflow.**

---

## Implementation References

This orchestrator specification has corresponding Python implementations:

### Core Components

| Component | File | Purpose |
|-----------|------|---------|
| **State Manager** | `src/state_manager.py` | Manages STATE.json, phase transitions, artifact validation |
| **Security** | `src/security.py` | Input validation, sanitization, security checks |
| **Orchestrator** | `src/orchestrator.py` | Main orchestrator implementation with state machine |
| **Context Budget** | `src/context_budget.py` | Token estimation and context management |
| **Health Checks** | `src/health.py` | System health monitoring |
| **Metrics** | `src/metrics.py` | Performance and usage metrics |

### Integration Points

**When executing autonomously, the orchestrator should:**

1. **Initialize StateManager** before any phase:
   ```python
   state_manager = StateManager(base_dir=Path("docs"), issue_name=issue_name)
   state_manager.create_initial_state()
   ```

2. **Validate inputs** before processing:
   ```python
   from src.security import validate_issue_name, sanitize_feature_description
   validated_name = validate_issue_name(issue_name)
   safe_description = sanitize_feature_description(description)
   ```

3. **Load context** for each phase:
   ```python
   context_plan = state_manager.get_context_for_phase(Phase.PLANNING)
   # Load required artifacts from context_plan.required
   # Load optional artifacts from context_plan.optional if within budget
   ```

4. **Transition phases** with validation:
   ```python
   state_manager.transition_to(Phase.PLANNING)
   state_manager.register_artifact("PLAN_SUMMARY.md", "PLAN_SUMMARY.md")
   state_manager.validate_phase_outputs(Phase.PLANNING, created_artifacts)
   ```

5. **Handle errors** with diagnostics:
   ```python
   try:
       result = execute_phase()
   except Exception as e:
       create_diagnostic_report(state_manager, e)
       state_manager.transition_to(Phase.BLOCKED)
   ```

### Artifact Creation

**Ensure skills create both full artifacts AND summary artifacts:**

| Phase | Full Artifact | Summary Artifact | Created By |
|-------|--------------|------------------|------------|
| Research | CODE_RESEARCH.md | RESEARCH_SUMMARY.md | code-research skill |
| Planning | IMPLEMENTATION_PLAN.md, PROJECT_SPEC.md | PLAN_SUMMARY.md | solution-planning skill |
| Implementation | Code + Tests | IMPLEMENTATION_SUMMARY.md | code-implementation skill |
| Review | CODE_REVIEW.md | (none) | code-review skill |
| Fix | Fixed code | (none) | review-fix skill |

### Error Handling Patterns

**Use the error handling defined in each skill:**
- Read the "Error Handling" section in each skill file
- Follow "Exit Conditions" for success/failure determination
- Use "Validation Before Completion" checklists

### Testing

**Test the implementation:**
```bash
# Run all tests
pytest tests/ -v

# Run with coverage
pytest tests/ --cov=src --cov-report=html

# Test specific components
pytest tests/unit/test_state_manager.py -v
pytest tests/unit/test_orchestrator.py -v
```

### Documentation

**Full specifications:**
- `docs/STATE_MANAGER_SPECIFICATION.md` - State manager implementation
- `docs/ORCHESTRATOR_SPECIFICATION.md` - Orchestrator implementation
- `docs/QUICK_START_FIXES.md` - Quick start for critical fixes
- `docs/SDL_WORKFLOW_COMPREHENSIVE_REVIEW.md` - Complete system review
