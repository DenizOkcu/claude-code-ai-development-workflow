---
name: implementing-code
description: "Systematic implementation following documented phases with test-driven development. Use when executing implementation plans or building features from specifications. Triggers: implement, build, execute plan, write code, follow phases, test-driven development."
model: sonnet
temperature: 0
---

# Code Implementation

Execute implementation from plans and specifications systematically.

**CRITICAL: You MUST implement ALL phases defined in IMPLEMENTATION_PLAN.md.** Do NOT stop after Phase 1, MVP, or any intermediate phase. The implementation is ONLY complete when EVERY phase in the plan has been implemented.

## LLM Parameters

**For consistent, deterministic code generation:**
- **Temperature:** 0 (deterministic, same inputs → same outputs)
- **Max Tokens:** 8000 (allows full implementation across all phases)
- **Top-P:** 1.0 (use full distribution)
- **Frequency Penalty:** 0 (no penalty for repetition)
- **Presence Penalty:** 0 (no penalty for new topics)

**Seed Strategy:** Use `issue_name` as seed for consistency

**Why These Parameters:**
- Temperature 0 ensures reproducible implementation
- Higher token limit supports complete multi-phase implementation
- No penalties allow complete code and test generation
- Deterministic output aids debugging and review

## User Communication

### Phase Start Notification

Before starting implementation, inform the user:

```markdown
## 📍 Phase: Implementation

**Starting:** Feature implementation
**Phases to Complete:** {N}
**Estimated Time:** {total estimate}

Reading implementation plan...
```

### Phase-Specific Notifications

**At start of EACH phase:**

```markdown
**Progress Update:** [Phase {N}/{M}: {phase-name}]

→ Creating {file-name}
→ Adding {feature}
→ Updating {component}
```

**When completing a phase:**

```markdown
✓ Phase {N}/{M}: {phase-name} - Complete

Files created/modified:
- {file-1} ({lines} lines)
- {file-2} ({lines} lines)
```

### File Creation Notifications

When creating files, notify:

```markdown
**Created:** {file-name} 📄

📄 Location: `{path}`
📊 Size: {lines} lines
💡 Purpose: {description}
```

### Testing Progress

During test execution:

```markdown
**Running Tests:** {test-framework}

✓ {N} tests passing
→ Running test suite...
```

**After tests complete:**

```markdown
**Test Results:** ✓ {N}/{N} passing ({X%} coverage)

All tests passed successfully.
```

### Phase Completion Notification

When ALL phases are complete:

```markdown
## ✅ Phase Complete: Implementation

**Duration:** {actual time}
**Phases Completed:** {N}/{N}
- ✓ Phase 1: {name}
- ✓ Phase 2: {name}
- ✓ Phase 3: {name}
- ✓ Phase 4: {name}

**Files Created:** {count}
- {file-1}
- {file-2}

**Tests Created:** {count} ({N} tests, {X%} passing)

**Deviations from Plan:** {None | description}

**Next Phase:** Review

Proceeding automatically...
```

### Error Notifications

If tests fail or errors occur:

```markdown
## ⚠️ Implementation Issue

**Issue:** {error-type}

{error-details}

**Fixing:** {what's being done to fix}

Retrying tests...
```

## Few-Shot Example

### Example Input
```yaml
issue_name: "add-oauth-login"
IMPLEMENTATION_PLAN.md: [4 phases defined]
PROJECT_SPEC.md: [technical spec exists]
```

### Example Behavior: Phase Completion (Good ✅)
```
✓ Read IMPLEMENTATION_PLAN.md - Counted 4 phases
✓ Created todos for ALL 4 phases (12 tasks total)

[Starting Phase 1: Foundation]
✓ Task 1.1: Create src/auth/google-strategy.ts (completed)
✓ Task 1.2: Update src/auth/passport.ts (completed)
✓ Task 1.3: Add dependency to package.json (completed)

[Starting Phase 2: Routes & Controllers]
✓ Task 2.1: Add routes to src/routes/auth.ts (completed)
✓ Task 2.2: Create callback handler (completed)
✓ Task 2.3: Update session middleware (completed)

[Starting Phase 3: Integration & Testing]
✓ Task 3.1: Add OAuth types (completed)
✓ Task 3.2: Write OAuth tests (completed)
✓ Task 3.3: Create OAuth mocks (completed)

[Starting Phase 4: Polish & Documentation]
✓ Task 4.1: Write OAuth docs (completed)
✓ Task 4.2: Update .env.example (completed)
✓ Task 4.3: Verify all tests pass (completed)

✓ All 4 phases complete
✓ IMPLEMENTATION_SUMMARY.md created
```

### Example Behavior: Phase Completion (Bad ❌)
```
✓ Read IMPLEMENTATION_PLAN.md
✓ Created todos for Phase 1

[Starting Phase 1: Foundation]
✓ Task 1.1: Create src/auth/google-strategy.ts (completed)
✓ Task 1.2: Update src/auth/passport.ts (completed)

✓ Implementation complete! ✓
→ STATUS.md updated

(RESULT: WRONG - only completed Phase 1 of 4, stopped early)
```

**Why Bad:**
- ❌ Only implemented Phase 1 (MVP)
- ❌ Phases 2, 3, 4 not implemented
- ❌ Tests not written
- ❌ Documentation missing
- ❌ Claimed completion prematurely

### Example Output: IMPLEMENTATION_SUMMARY.md (Good ✅)
```markdown
# Implementation Summary

## For Review Phase

### What Was Built
OAuth2 authentication with Google provider, integrated with existing passport.js auth system.

### Files Created
- `src/auth/google-strategy.ts` - Google OAuth2 strategy implementation
- `src/types/auth.ts` - OAuth provider type definitions
- `tests/auth/oauth.test.ts` - OAuth flow tests (15 tests)
- `tests/mocks/oauth.ts` - OAuth provider mocks

### Files Modified
- `src/auth/passport.ts:50-70` - Registered Google strategy
- `src/routes/auth.ts:45-80` - Added /auth/google routes
- `src/middleware/session.ts:120-150` - Updated session for provider tracking
- `package.json:25` - Added passport-google-oauth20 dependency

### Implementation Approach
- Followed existing GitHub OAuth pattern for consistency
- Used passport.js OAuth2 strategy framework
- Integrated with existing session management
- Mocked OAuth providers for testing

### All Phases Completed
- Phase 1 (Foundation): ✓ - Google strategy created, passport configured
- Phase 2 (Routes): ✓ - Routes added, callbacks implemented
- Phase 3 (Testing): ✓ - Tests written, mocks created
- Phase 4 (Polish): ✓ - Documentation complete, tests passing

### Tests Created
- 15 tests covering OAuth flow, error handling, session management
- All tests passing (15/15)
- Coverage: 95% for new code

### Deviations from Plan
None - implementation followed plan exactly.

### Known Limitations
- OAuth tokens stored in session (acceptable for current scale)
- Token refresh not implemented (deferred to future iteration)

### Git Diff Summary
- Files changed: 7
- Lines added: 450
- Lines removed: 12
```

### Example Output: IMPLEMENTATION_SUMMARY.md (Bad ❌)
```markdown
# Implementation Summary

Added Google OAuth login.

Files:
- src/auth/google-strategy.ts
- src/routes/auth.ts

Tests: Some tests written.

Done!
```

**Why Bad:**
- ❌ Missing file details (line numbers, purposes)
- ❌ Phases not listed (can't verify all complete)
- ❌ Test details missing
- ❌ No deviations section
- ❌ No implementation approach
- ❌ Too brief for review phase

## Inputs
- `issue_name`: Kebab-case identifier
- `IMPLEMENTATION_PLAN.md`: Phased implementation strategy
- `PROJECT_SPEC.md`: Technical requirements
- `STATUS.md`: Current workflow status

## Outputs
- Implementation code (new/modified files)
- Tests (unit, integration, edge cases)
- Updated `STATUS.md` with progress

## Procedure

### 1. Pre-Implementation
Read all planning artifacts:
- `IMPLEMENTATION_PLAN.md` - step-by-step guide
- `PROJECT_SPEC.md` - technical requirements and design
- `STATUS.md` - current workflow status

Update STATUS.md:
```markdown
## Phase: Implementation 🔄
- **Current:** Phase [N] - [name]
- **Progress:** [X/Y phases done]
- **Status:** Starting implementation
```

### 2. Research Codebase
- Read existing files mentioned in plan
- Understand current architecture and patterns
- Identify integration points
- Note any deviations from plan that might be necessary

### 3. Create Todo List
Use TodoWrite with tasks from **ALL phases in IMPLEMENTATION_PLAN.md**. **Exactly ONE task in_progress at a time.**

**IMPORTANT:** The IMPLEMENTATION_PLAN.md contains multiple phases (e.g., Phase 1 MVP, Phase 2, Phase 3, etc.). You MUST create todos for tasks from ALL of these phases, not just Phase 1.

### 4. Execute Phase by Phase

**For each task:**
1. Mark as `in_progress`
2. Implement following PROJECT_SPEC.md design
3. Follow existing code style and patterns
4. Add inline docs for complex logic only
5. Add tests (unit, integration, edge cases)
6. Ensure tests pass
7. Mark as `completed` immediately

### 5. Implementation Strategy

**Phase-by-Phase Approach:**
- **YOU MUST COMPLETE ALL PHASES IN THE PLAN** - stopping early is a failure
- Complete one phase before moving to next
- Each phase should result in working, tested code
- Don't skip ahead even if tempted
- If you discover plan issues, note them but work within structure

**Phase Completion Requirements:**
- Read IMPLEMENTATION_PLAN.md and identify ALL phases (e.g., "Phase 1: MVP", "Phase 2: Enhanced Features", "Phase 3: Advanced Features")
- Create todos for tasks from EVERY phase
- Implement tasks from each phase sequentially
- Do NOT declare implementation complete until ALL phases are done
- Example: If plan has Phase 1, Phase 2, Phase 3 - you must complete all 3 phases, not just Phase 1

**When to Deviate from Plan:**
- Plan conflicts with existing architecture (unanticipated)
- You discover a better implementation approach
- External dependencies have changed
- Security or performance issues identified

**Always document deviations** in comments or IMPLEMENTATION_NOTES.md

**Handling Blockers:**
1. Mark current task as `in_progress` (not completed)
2. Document blocker clearly
3. Ask user for guidance
4. Create new todo for resolving blocker if needed

### 6. Code Quality Standards

**General:**
- Follow existing code style and patterns
- Use proper types (if TypeScript: strict mode)
- Implement error handling as specified in PROJECT_SPEC.md
- Add input validation where needed
- Keep functions small and focused
- Use meaningful variable and function names

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

### 7. Testing Standards

**Unit Tests:**
- Test each function/method independently
- Mock external dependencies
- Cover happy path and edge cases
- Use descriptive test names
- Aim for high code coverage on business logic

**Integration Tests:**
- Test component interactions
- Use realistic test data
- Verify end-to-end workflows
- Test error scenarios

**Test Organization:**
- Follow existing test structure
- Keep tests close to implementation files
- Use setup/teardown appropriately
- Make tests readable and maintainable

### 8. Validation

**Before declaring implementation complete, verify ALL phases are done:**

**Phase Completion Check:**
- [ ] Read IMPLEMENTATION_PLAN.md and count the total phases (e.g., 3 phases, 4 phases, etc.)
- [ ] Create todos for tasks from ALL phases (not just Phase 1)
- [ ] Complete implementation for Phase 1
- [ ] Complete implementation for Phase 2
- [ ] Complete implementation for Phase 3 (and so on...)
- [ ] All phases have their tasks marked as "completed"
- [ ] No tasks remain in "pending" or "in_progress" status

**If any phase is incomplete, you are NOT done. Continue implementing.**

**After all phases are complete:**
- Run full test suite
- Check TypeScript errors (if applicable)
- Verify acceptance criteria
- Test end-to-end
- Review for TODO comments

### 9. Communication Style

**Progress Updates:**
- Clearly state which phase and task you're working on
- Show code snippets for significant implementations
- Explain non-obvious decisions
- Report completion of each phase

**Code References:**
- Reference files with line numbers (e.g., `src/auth/oauth.ts:45`)
- Link related implementations
- Note dependencies between components

**Completion Summary** (5-8 bullets max):
- Total phases completed: [N/N phases from IMPLEMENTATION_PLAN.md]
- Phase 1 (MVP) ✓, Phase 2 ✓, Phase 3 ✓ (list all completed phases)
- Tasks completed: [N] tasks across all phases
- Files modified/created: [count]
- Test results (N passing, M failing)
- Deviations from plan (if any)
- Next: `/sdlc {issue_name} --from-review`

### 10. Create IMPLEMENTATION_SUMMARY.md

**CRITICAL: You MUST create this summary artifact for the review phase.**

Create a comprehensive summary document for the review phase:

```markdown
# Implementation Summary

**Generated:** {timestamp}
**Issue:** {issue_name}

## For Review Phase

### What Was Built
[Complete description of what was implemented]

### Files Created
[List all new files with purposes]
- `src/new-file.ts` - [purpose description]
- `src/another-file.ts` - [purpose description]
- `tests/test-file.test.ts` - [test coverage for X]

### Files Modified
[List all modifications with purposes]
- `src/existing.ts:50-100` - Added [feature description]
- `src/config.ts:200-250` - Updated [configuration]

### Implementation Approach
[How it was built, technical decisions made]
- Architecture pattern followed
- Key technical decisions
- Implementation order

### All Phases Completed
[Verify ALL phases from IMPLEMENTATION_PLAN.md are complete]
- Phase 1 (MVP): ✓ - [MVP features implemented]
- Phase 2: ✓ - [Phase 2 features implemented]
- Phase 3: ✓ - [Phase 3 features implemented]
- [Continue for all phases in plan]

### Tests Created
[List all test files with coverage]
- `tests/unit/test-x.test.ts` - [X unit tests, Y% coverage]
- `tests/integration/test-y.test.ts` - [Integration tests for Z]
- Total: [N] tests, [X% passing]

### Deviations from Plan
[Document any deviations from IMPLEMENTATION_PLAN.md]
- [If no deviations, state: "None - implementation followed plan exactly"]
- [If deviations, explain why and what changed]

### Known Limitations
[Any known limitations or future work]
- [Limitation 1] - [impact]
- [Limitation 2] - [impact]

### Git Diff Summary
[Summary of changes]
- Files changed: [N]
- Lines added: [N]
- Lines removed: [N]

---
*This is a comprehensive summary for the review phase. Full implementation details in IMPLEMENTATION_PLAN.md.*
```

**Quality Check for Summary:**
- Lists ALL files created and modified
- Confirms ALL phases from plan are complete
- Tests are documented
- Deviations are explained (or "None")
- Limitations are documented

### 11. Update STATUS.md

```markdown
## Progress
- [x] Research | [x] Planning | [x] Implementation | [ ] Review

## Phase: Implementation ✓
- **Phases Completed:** [N/N phases from plan]
- **Phase 1:** ✓ [MVP features]
- **Phase 2:** ✓ [Enhanced features]
- **Phase 3:** ✓ [Advanced features]
- **Files:** [list key files]
- **Tests:** [N tests, passing/failing]
- **Deviations:** [any changes from plan, or "None"]
- **Next:** `/sdlc {issue_name} --phase review`
```

## Error Handling

### Known Error Scenarios

1. **Missing Planning Artifacts**
   - Check for PLAN_SUMMARY.md, IMPLEMENTATION_PLAN.md, PROJECT_SPEC.md
   - Error: "Missing required planning artifact: {filename}"
   - Recovery: Ask user to run planning phase first

2. **File Write Failures**
   - Check directory permissions
   - Verify file paths are valid
   - Error: "Cannot write file: {path}"
   - Recovery: Fix permissions, retry

3. **Test Failures**
   - Analyze test output for specific failures
   - Fix implementation or update tests as appropriate
   - Error: "Tests failing: {test_names}"
   - Recovery: Fix failing tests, ensure all pass

4. **Type Errors (TypeScript)**
   - Review type mismatches
   - Fix type annotations
   - Error: "Type check failed"
   - Recovery: Fix type errors, run type check again

5. **Build Failures**
   - Analyze build output for errors
   - Fix syntax, imports, dependencies
   - Error: "Build failed"
   - Recovery: Fix build issues, retry

### Error Recovery

- **Before implementation**: Verify all planning artifacts exist
- **During implementation**: If a file cannot be created, log error and continue
- **Test failures**: Fix implementation, don't skip tests
- **Type errors**: Fix immediately, don't accumulate
- **Blockers**: Document clearly, ask user for guidance

### Exit Conditions

**Success:**
- ALL phases from IMPLEMENTATION_PLAN.md are complete
- All code is written
- All tests pass
- Type checks pass (if TypeScript)
- IMPLEMENTATION_SUMMARY.md created
- STATUS.md updated

**Failure (Retryable):**
- File write permission errors (fix permissions, retry)
- Test failures (fix and retry)
- Type errors (fix and retry)

**Failure (Non-retryable):**
- Missing planning artifacts
- Plan is ambiguous or incomplete
- User clarification required
- Cannot proceed without architectural decision

### Validation Before Completion

Before marking implementation complete, verify:
- [ ] Read IMPLEMENTATION_PLAN.md and counted total phases
- [ ] ALL phases from plan are implemented (Phase 1, Phase 2, Phase 3, etc.)
- [ ] All tasks in all phases are marked "completed"
- [ ] No tasks remain in "pending" or "in_progress"
- [ ] All tests pass
- [ ] Type check passes (if TypeScript)
- [ ] Build succeeds
- [ ] IMPLEMENTATION_SUMMARY.md created with all sections
- [ ] All files created are listed in summary
- [ ] All files modified are listed in summary
- [ ] Deviations from plan are documented
- [ ] STATUS.md updated

## Important Reminders

- **CRITICAL: Implement ALL phases in IMPLEMENTATION_PLAN.md - stopping early is a failure**
- ALWAYS read IMPLEMENTATION_PLAN.md and PROJECT_SPEC.md first
- Count the total phases in the plan before starting
- Use TodoWrite tool to track ALL tasks from ALL phases
- Mark todos as completed immediately after finishing each task
- Don't skip testing - it's part of each phase
- Follow the plan structure unless there's a good reason to deviate
- Communicate clearly about progress and blockers
- Write production-quality code from the start
- Keep commits atomic if user requests git commits

## Quality Checks
- [ ] All planning artifacts read
- [ ] IMPLEMENTATION_PLAN.md read and all phases identified
- [ ] Todo list created with tasks from ALL phases (not just Phase 1)
- [ ] Exactly ONE task in_progress at any time
- [ ] All phases implemented (Phase 1, Phase 2, Phase 3, etc.)
- [ ] All tasks marked completed
- [ ] No tasks remaining in "pending" or "in_progress"
- [ ] Tests written for each phase
- [ ] Acceptance criteria verified
- [ ] IMPLEMENTATION_SUMMARY.md created with all sections
- [ ] STATUS.md updated
- [ ] Progress updates communicated
- [ ] Code references include line numbers
- [ ] Error scenarios handled

## Error Handling
- **Blocker**: Document clearly, ask user
- **Plan conflict**: Note deviation, continue
- **Test failures**: Create fix todos
- **External deps**: Document and adapt
