# State Management Strategy (Revised)

## Core Principles

1. **Deterministic Workflow** - Same inputs → same outputs
2. **Precise Communication** - Artifacts must be comprehensive
3. **Predictable Results** - No ambiguity, clear expectations
4. **Context Boundaries** - Clear phase transitions with full information transfer

## Key Insight

**Artifacts ARE the communication mechanism between phases.** They must be comprehensive, not minimized.

- **Artifacts** = Persistent storage (comprehensive, detailed)
- **Context** = Ephemeral conversation (can be cleared between phases)
- **STATE.json** = Index/manifest (not replacement for artifacts)

## Architecture

### File-Based Artifacts (Persistent, Comprehensive)

```
docs/{issue-name}/
├── STATE.json              (index/manifest)
├── STATUS.md               (human-readable progress)
├── CODE_RESEARCH.md        (full research artifact)
├── RESEARCH_SUMMARY.md     (comprehensive summary for next phase)
├── IMPLEMENTATION_PLAN.md (full plan artifact)
├── PLAN_SUMMARY.md         (comprehensive summary for next phase)
├── PROJECT_SPEC.md         (full technical spec)
├── IMPLEMENTATION_SUMMARY.md (comprehensive summary for next phase)
└── CODE_REVIEW.md          (full review artifact)
```

### Artifact Types

**1. Full Artifacts** (for reference, auditing, human review)
- CODE_RESEARCH.md - Complete research findings
- IMPLEMENTATION_PLAN.md - Complete implementation plan
- PROJECT_SPEC.md - Complete technical specification
- CODE_REVIEW.md - Complete review findings

**2. Summary Artifacts** (for next phase input, comprehensive but focused)
- RESEARCH_SUMMARY.md - What planning needs to know
- PLAN_SUMMARY.md - What implementation needs to know
- IMPLEMENTATION_SUMMARY.md - What review needs to know

**3. State Files** (for coordination)
- STATE.json - Machine-readable index
- STATUS.md - Human-readable progress

## Phase Transition Pattern

### End of Phase N

**1. Create Full Artifact** (comprehensive, detailed)
```markdown
# CODE_RESEARCH.md

## Summary
- Risk: Medium
- Key findings...

### Integration Points
(detailed list with line refs)

### Technical Context
(complete technical details)

### Risks
(complete risk assessment)

### Key Files
(comprehensive file list)

### Open Questions
(all questions documented)
```

**2. Create Summary for Next Phase** (comprehensive, actionable)
```markdown
# RESEARCH_SUMMARY.md

## For Planning Phase

### What We're Building
[Clear goal description]

### Technical Constraints
[Complete technical context]

### Integration Points
- Files to modify: src/auth/oauth.ts:50-100 (add passport strategy)
- Files to create: src/middleware/auth.ts (new authentication middleware)
- Dependencies: passport-google-oauth20, express-session

### Risks to Mitigate
1. Existing session management conflicts
2. OAuth callback URL configuration
3. User model extension needed

### Recommendations
[Complete recommendations with rationale]

### Questions to Answer
[All open questions]
```

**3. Update STATE.json** (index/manifest)
```json
{
  "issue_name": "add-oauth-login",
  "current_phase": "planning",
  "artifacts": {
    "CODE_RESEARCH.md": "complete",
    "RESEARCH_SUMMARY.md": "complete"
  },
  "ready_for": {
    "planning": {
      "required_artifacts": ["CODE_RESEARCH.md", "RESEARCH_SUMMARY.md"],
      "context_clear": true
    }
  }
}
```

**4. Clear Conversation Context** (not files)
```python
# Clear only ephemeral conversation context
# Files remain on disk, can be loaded when needed
clear_conversation_context()
```

### Start of Phase N+1

**1. Load STATE.json** (read index)
```json
{
  "current_phase": "planning",
  "artifacts": {...},
  "ready_for": {
    "planning": {
      "required_artifacts": ["CODE_RESEARCH.md", "RESEARCH_SUMMARY.md"]
    }
  }
}
```

**2. Load Required Artifacts** (from files, not context)
```python
# Read files when needed (they're not in context)
research_summary = read_file("RESEARCH_SUMMARY.md")
code_research = read_file_if_needed("CODE_RESEARCH.md")  # for reference
```

**3. Execute Phase** (with loaded artifacts)
```python
# Plan based on comprehensive summary
# Reference full artifact when needed
create_plan(research_summary, code_research)
```

## Deterministic Information Flow

### Research → Planning

**Planning receives:**
- RESEARCH_SUMMARY.md (comprehensive summary)
- CODE_RESEARCH.md (full artifact, on-demand reference)

**Planning creates:**
- IMPLEMENTATION_PLAN.md (full plan)
- PROJECT_SPEC.md (full spec)
- PLAN_SUMMARY.md (comprehensive summary for implementation)

### Planning → Implementation

**Implementation receives:**
- PLAN_SUMMARY.md (comprehensive summary)
- IMPLEMENTATION_PLAN.md (full plan)
- PROJECT_SPEC.md (full spec)
- RESEARCH_SUMMARY.md (for context, on-demand)

**Implementation creates:**
- Implementation code (files)
- IMPLEMENTATION_SUMMARY.md (comprehensive summary for review)

### Implementation → Review

**Review receives:**
- IMPLEMENTATION_SUMMARY.md (comprehensive summary)
- Changed files (git diff)
- PLAN_SUMMARY.md (for context, on-demand)
- PROJECT_SPEC.md (for reference, on-demand)

**Review creates:**
- CODE_REVIEW.md (full review)

### Review → Fix Loop

**Fix receives:**
- CODE_REVIEW.md (issues to fix)
- Changed files (git diff)
- IMPLEMENTATION_SUMMARY.md (for context)

**Fix creates:**
- Fixed code
- Updated CODE_REVIEW.md

## Context Management

### What is Cleared

**Ephemeral conversation context:**
- Chat history from previous phases
- Agent memory of previous phase details
- Temporary working context

**NOT cleared (files remain):**
- All artifact files (on disk)
- STATE.json (on disk)
- STATUS.md (on disk)

### What is Loaded

**When phase starts:**
1. STATE.json (index - always loaded)
2. Required summary artifact (comprehensive summary)
3. Full artifacts (on-demand, as needed)

**Example:**
```python
# Phase start
state = load_json("STATE.json")
summary = load_file("PLAN_SUMMARY.md")  # Comprehensive summary

# During phase execution
full_plan = load_file_if_needed("IMPLEMENTATION_PLAN.md")  # On-demand
spec = load_file_if_needed("PROJECT_SPEC.md")  # On-demand
```

## Comprehensive Summary Structure

**RESEARCH_SUMMARY.md** (for Planning)
```markdown
# Research Summary

## Goal
[What we're building, clear and complete]

## Technical Context
[Complete technical landscape]

## Integration Strategy
[Detailed integration points with file:line refs]

## Risks and Mitigations
[Complete risk assessment with mitigation strategies]

## Recommendations
[Complete recommendations with rationale]

## Open Questions
[All questions that need answers]

## Files Referenced
- src/auth/oauth.ts:50-100 - Existing auth patterns
- src/routes/index.ts:200-250 - Route registration
```

**PLAN_SUMMARY.md** (for Implementation)
```markdown
# Plan Summary

## What to Build
[Clear, complete feature description]

## Architecture
[Complete architecture with components]

## Implementation Phases
[Detailed phase breakdown with specific tasks]

## File-by-File Breakdown
- src/auth/oauth.ts - NEW: Implement Google OAuth2 strategy
  - Lines 1-50: Import and configure passport
  - Lines 51-100: Define strategy callbacks
  - Lines 101-150: Export middleware

- src/routes/auth.ts - MODIFY: Add OAuth routes
  - Lines 50-100: Add /auth/google route
  - Lines 101-150: Add /auth/google/callback route

## Testing Requirements
[Complete testing specifications]

## Edge Cases
[All edge cases to handle]

## Dependencies
[All dependencies and versions]
```

**IMPLEMENTATION_SUMMARY.md** (for Review)
```markdown
# Implementation Summary

## What Was Built
[Complete description of implementation]

## Files Created
[List all new files with purposes]

## Files Modified
[List all modifications with purposes]

## Implementation Approach
[How it was built, decisions made]

## Tests Created
[All tests with coverage]

## Deviations from Plan
[Any deviations with reasons]

## Known Limitations
[Any known limitations or future work]
```

## Fix Loop Optimization

### What Fix Loop Loads

**Always loads:**
1. STATE.json (index)
2. CODE_REVIEW.md (issues to fix)
3. IMPLEMENTATION_SUMMARY.md (context)
4. Changed files (git diff)

**On-demand loads:**
- PLAN_SUMMARY.md (if need to understand original plan)
- PROJECT_SPEC.md (if need technical details)

**Does NOT load:**
- Full CODE_RESEARCH.md (not needed for fixes)
- Chat history from previous phases (cleared)

### Fix Loop Context Budget

**Per fix iteration:**
- STATE.json: ~200 tokens
- CODE_REVIEW.md: ~1K tokens (issues list)
- IMPLEMENTATION_SUMMARY.md: ~1K tokens (context)
- Changed files: ~1K tokens (git diff)
- **Total: ~3K tokens per iteration**

**After 3 iterations: ~3K tokens** (not 21K)

## Benefits

1. **Deterministic** - Full information transfer via artifacts
2. **Precise** - Comprehensive summaries provide all needed details
3. **Predictable** - Clear phase structure, expectable results
4. **Bounded context** - Ephemeral context cleared, files on-demand
5. **Efficient** - Load only what's needed, when needed
6. **Auditable** - Full artifacts preserved for review
7. **Debuggable** - Clear phase boundaries and state transitions

## Implementation Requirements

1. **Each phase creates TWO artifacts:**
   - Full artifact (comprehensive, for reference)
   - Summary artifact (comprehensive, for next phase)

2. **STATE.json tracks:**
   - Current phase
   - All artifacts created
   - What's ready for next phase

3. **Context clearing:**
   - Clear ephemeral conversation context
   - Keep all files on disk
   - Load files on-demand

4. **Deterministic loading:**
   - STATE.json → Required summaries → Full artifacts (on-demand)
   - Same inputs → same outputs
