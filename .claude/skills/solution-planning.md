---
name: planning-solutions
description: "Transform requirements into actionable implementation plans and technical specifications. Use when creating phased implementation strategies or designing technical solutions. Triggers: plan, design solution, create specification, implementation strategy, technical design."
model: sonnet
temperature: 0
---

# Solution Planning

Create implementation plans and technical specifications from requirements.

## LLM Parameters

**For consistent, deterministic outputs:**
- **Temperature:** 0 (deterministic, same inputs → same outputs)
- **Max Tokens:** 8000 (allows detailed plans and specs)
- **Top-P:** 1.0 (use full distribution)
- **Frequency Penalty:** 0 (no penalty for repetition)
- **Presence Penalty:** 0 (no penalty for new topics)

**Seed Strategy:** Use `issue_name` as seed for consistency

**Why These Parameters:**
- Temperature 0 ensures reproducible plans
- Higher token limit accommodates both IMPLEMENTATION_PLAN.md and PROJECT_SPEC.md
- No penalties allow comprehensive phase breakdowns
- Consistent plans enable reliable implementation

## Few-Shot Example

### Example Input
```yaml
issue_name: "add-oauth-login"
CODE_RESEARCH.md: [exists with findings]
feature_description: "Add OAuth2 login with Google and GitHub providers"
```

### Example Output: IMPLEMENTATION_PLAN.md (Good ✅)
```markdown
### Overview
- Goal: Add OAuth2 authentication with Google and GitHub providers
- Success criteria:
  - Users can authenticate via Google OAuth2
  - Users can authenticate via GitHub OAuth
  - Session management supports multiple providers
  - OAuth tokens stored securely
  - Logout works for both providers

### Phases

**Phase 1: Foundation** (Complexity: Low)
- `src/auth/google-strategy.ts:1-60` - Create Google OAuth2 strategy
- `src/auth/passport.ts:50-70` - Register Google strategy with passport
- `package.json:1` - Add passport-google-oauth20 dependency

**Phase 2: Routes & Controllers** (Complexity: Medium)
- `src/routes/auth.ts:45-80` - Add Google OAuth routes (/auth/google, /auth/google/callback)
- `src/controllers/auth.ts:100-150` - Add Google OAuth callback handler
- `src/middleware/session.ts:120-150` - Update session for provider tracking

**Phase 3: Integration & Testing** (Complexity: Medium)
- `src/types/auth.ts:20-40` - Add OAuth provider types
- `tests/auth/oauth.test.ts:1-100` - Test OAuth flows
- `tests/mocks/oauth.ts:1-50` - Mock OAuth providers

**Phase 4: Polish & Documentation** (Complexity: Low)
- `docs/auth/oauth.md:1-100` - Document OAuth setup
- `.env.example:5-6` - Add OAuth env var examples
- Verify all tests pass

### Testing
- Unit: OAuth strategy configuration, callback handlers
- Integration: Full OAuth flow with mocks
- Edge cases: OAuth failure, token expiry, session overflow

### Estimates
| Phase | Effort |
| ----- | ------ |
| 1     | 30min  |
| 2     | 1hr    |
| 3     | 1.5hr  |
| 4     | 30min  |
| Total | 3.5hr  |
```

### Example Output: IMPLEMENTATION_PLAN.md (Bad ❌)
```markdown
# OAuth Implementation Plan

## Overview
Add Google OAuth login.

## Steps
1. Install passport-google-oauth20
2. Create Google strategy
3. Add routes
4. Test it

## Testing
Write tests for OAuth.
```

**Why Bad:**
- No success criteria
- No file:line references
- No complexity estimates
- Phases not well-defined
- Missing time estimates
- Tasks not actionable
- No edge case coverage

## Inputs
- `issue_name`: Kebab-case identifier
- `CODE_RESEARCH.md`: Research findings (optional but recommended)
- `feature_description`: Requirements description

## Outputs
- `docs/{issue_name}/IMPLEMENTATION_PLAN.md` (phased strategy)
- `docs/{issue_name}/PROJECT_SPEC.md` (technical spec)
- `docs/{issue_name}/STATUS.md` (updated)

## Procedure

### 1. Read Context First
- Read `CODE_RESEARCH.md` if exists (contains vital context)
- Read `STATUS.md` if exists to understand current progress
- Read relevant codebase files before planning

### 2. Analyze Requirements
Parse feature description or derive from research:
- Functional requirements (what it does, 3-5 bullets)
- Non-functional requirements (performance, security, compatibility)
- Success criteria (3-5 specific items)

**Ask clarifying questions if:**
- Requirements are vague
- Critical information is missing
- Multiple valid approaches exist

### 3. Research Existing Patterns
- Understand existing architecture and patterns
- Identify integration points
- Maintain consistency with code style
- Consider what similar features exist

### 4. Design Architecture

**Determine:**
- Pattern: layered, event-driven, etc.
- Components and their roles
- Data flow (brief description or diagram)

**Propose concrete solutions:**
- Specific file paths and function names
- Code patterns to follow
- Interfaces and types (if TypeScript)

**Design key types/interfaces** (if typed language):
```typescript
// Only novel/complex types
interface FooBar {
  // Type definition
}
```

### 5. Break Into Phases

Create 3-4 sequential phases. Each phase includes:
- Complexity level (Low/Med/High)
- Specific tasks with file:line references
- Expected deliverables

**Example phase structure:**
- Phase 1: Foundation/setup tasks
- Phase 2: Core functionality
- Phase 3: Integration and polish
- Phase 4: Testing and validation

**Think modularly:**
- Break large features into smaller, testable components
- Each phase should result in working code
- Adapt phases to project needs (not always 4)

### 6. Consider Edge Cases

Think about:
- Failure scenarios
- Error handling requirements
- Input validation needs
- Boundary conditions
- Race conditions (if async)

### 7. Define Error Handling
- Validation approach
- Error scenarios (top 3-5)
- User feedback mechanism
- Recovery strategies

### 8. Create IMPLEMENTATION_PLAN.md

```markdown
### Overview
- Goal (1 sentence)
- Success criteria (3-5 bullets)

### Phases

**Phase 1: [Name]** (Complexity: Low/Med/High)
- Task 1: `file/path.ts:lines` - specific action
- Task 2: `file/path.ts:lines` - specific action

**Phase 2: [Name]** (Complexity: Low/Med/High)
- Task 1: ...

[3-4 phases total]

### Testing
- Unit: [key functions to test]
- Integration: [workflows to test]
- Edge cases: [critical scenarios]

### Estimates
| Phase | Effort |
| ----- | ------ |
| 1     | 30min  |
| 2     | 1hr    |
| Total | 1.5hr  |
```

### 9. Create PROJECT_SPEC.md

```markdown
### Requirements
- Functional: [what it does, 3-5 bullets]
- Non-functional: [constraints]

### Technical Design

**Architecture:**
- Pattern: [layered, event-driven, etc.]
- Components: [list with brief roles]
- Data flow: [brief description or diagram]

**Key Types/Interfaces:** (if applicable)
```typescript
interface FooBar {
  // Only novel/complex types
}
```

**Files to create/modify:**
```
src/foo/bar.ts - New: implement X
src/baz/qux.ts:50-100 - Modify: update Y
```

### Error Handling
- Validation: [strategy]
- Error scenarios: [top 3-5]
- User feedback: [approach]

### Configuration
- Env vars: [if any]
- Config files: [if any]
- External deps: [if any]
```

### 10. Update STATUS.md

```markdown
## Progress
- [x] Research | [x] Planning | [ ] Implementation | [ ] Review

## Phase: Planning ✓
- **Phases:** [4 phases, Est: 2hr]
- **Complexity:** [Simple/Medium/Complex]
- **Key Decisions:** [top 2-3 bullets]
- **Next:** `/sdlc {issue_name} --from-implement`
```

### 11. Create PLAN_SUMMARY.md

**CRITICAL: You MUST create this summary artifact for the implementation phase.**

Create a comprehensive summary document that guides the implementation phase:

```markdown
# Plan Summary

**Generated:** {timestamp}
**Issue:** {issue_name}

## For Implementation Phase

### What to Build
[Complete feature description with acceptance criteria]

### Architecture Overview
[Complete architecture description]
- Pattern: [layered, event-driven, etc.]
- Components: [list with roles]
- Data flow: [description]

### Implementation Phases

**Phase 1: [Name]** (Complexity: Low/Med/High)
- `path/to/file.ts:lines` - Specific task description
- `path/to/file.ts:lines` - Specific task description

**Phase 2: [Name]** (Complexity: Low/Med/High)
- [Continue with all phases from IMPLEMENTATION_PLAN.md]

**Phase 3: [Name]** (Complexity: Low/Med/High)
- [Continue]

**Phase 4: [Name]** (Complexity: Low/Med/High)
- [Continue]

### File-by-File Breakdown

**Files to Create:**
```
src/new-component.ts - NEW: Implement [description]
  - Lines 1-50: Imports and type definitions
  - Lines 51-100: Main component class
  - Lines 101-150: Helper methods
  - Lines 151-200: Export statements
```

**Files to Modify:**
```
src/existing.ts:50-100 - MODIFY: Add [feature]
  - Lines 50-75: New method implementation
  - Lines 76-100: Integration with existing code

src/config.ts:200-250 - MODIFY: Update configuration
  - Lines 200-225: Add new config values
  - Lines 226-250: Update validation logic
```

### Testing Requirements

**Unit Tests:**
- [key functions to test]
- [test coverage requirements]

**Integration Tests:**
- [workflows to test]
- [API endpoints to test]

**Edge Cases:**
- [critical scenarios to test]
- [boundary conditions]

### Edge Cases to Handle
[All edge cases with mitigation strategies]

### Dependencies
[Necessary dependencies and versions]

### Implementation Notes
[Any additional context for implementer]
- [Important conventions to follow]
- [Integration gotchas]
- [Performance considerations]

---
*This is a comprehensive summary for the implementation phase. See IMPLEMENTATION_PLAN.md and PROJECT_SPEC.md for full details.*
```

**Quality Check for Summary:**
- Contains all required sections
- File references include line ranges
- All phases from IMPLEMENTATION_PLAN.md are included
- Dependencies are listed with versions
- Testing requirements are complete
- Edge cases are documented

### 12. Present Summary
Present brief summary of both documents with:
- Number of phases
- Estimated effort
- Key decisions
- Next step guidance

## Error Handling

### Known Error Scenarios

1. **Missing CODE_RESEARCH.md**
   - Check if research was conducted
   - Warning: "No CODE_RESEARCH.md found - proceeding without research context"
   - Recommend running research phase first

2. **Ambiguous Requirements**
   - Identify vague or conflicting requirements
   - Ask clarifying questions via AskUserQuestion
   - Don't guess - clarify before proceeding

3. **Unable to Read Context Files**
   - Verify file paths exist
   - Check file permissions
   - Error: "Cannot read required file: {path}"

4. **No Existing Patterns Found**
   - Document lack of patterns
   - Propose industry-standard alternatives
   - Warning: "No existing patterns found - using standard approach"

### Error Recovery

- **Before planning**: Verify RESEARCH_SUMMARY.md exists
- **During planning**: If context is missing, note as assumption
- **Architecture unknown**: Research similar projects for patterns
- **Dependencies unclear**: Mark as TODO for implementation to resolve

### Exit Conditions

**Success:**
- IMPLEMENTATION_PLAN.md created with 3-4 phases
- PROJECT_SPEC.md created with all 4 sections
- PLAN_SUMMARY.md created with all sections
- STATUS.md updated
- All requirements addressed

**Failure (Retryable):**
- File write permission issues
- Temporary context unavailability

**Failure (Non-retryable):**
- Requirements are too ambiguous to plan
- User clarification needed
- Issue directory doesn't exist

### Validation Before Completion

Before marking planning complete, verify:
- [ ] IMPLEMENTATION_PLAN.md has Overview, Phases, Testing sections
- [ ] Each phase has file:line references for tasks
- [ ] PROJECT_SPEC.md has Requirements, Technical Design, Error Handling sections
- [ ] PLAN_SUMMARY.md has all required sections
- [ ] Success criteria documented
- [ ] Error handling strategy defined
- [ ] Edge cases considered
- [ ] STATUS.md updated
- [ ] All phases have specific, actionable tasks

## Important Approach Guidelines

- Ask clarifying questions if requirements are vague or missing critical information
- Research codebase to understand existing patterns and architecture
- Identify integration points with existing code
- Propose concrete solutions with specific file paths, function names, and code patterns
- Consider TypeScript best practices: strict typing, generics, proper interfaces
- Think about edge cases and failure scenarios
- Prioritize maintainability and code quality
- Consider testing from the start, not as an afterthought
- Document assumptions clearly
- Flag unknowns that need user input
- Think modularly - break large features into smaller, testable components

## Quality Checks
- IMPLEMENTATION_PLAN.md created with 3-4 phases
- Each phase has file:line references for tasks
- PROJECT_SPEC.md created with all 4 sections
- PLAN_SUMMARY.md created with all sections
- Success criteria documented
- Error handling strategy defined
- Edge cases considered
- Integration points identified
- Concrete solutions proposed
- STATUS.md updated
- Summary presented to user
- Error scenarios handled
