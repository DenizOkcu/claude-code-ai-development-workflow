# Artifact Templates

**Purpose:** Starting templates for all workflow artifacts
**Usage:** Copy these templates as starting points, customize for your issue

---

## Table of Contents

1. [CODE_RESEARCH.md Template](#code_research-template)
2. [RESEARCH_SUMMARY.md Template](#research_summary-template)
3. [IMPLEMENTATION_PLAN.md Template](#implementation_plan-template)
4. [PROJECT_SPEC.md Template](#project_spec-template)
5. [PLAN_SUMMARY.md Template](#plan_summary-template)
6. [IMPLEMENTATION_SUMMARY.md Template](#implementation_summary-template)
7. [CODE_REVIEW.md Template](#code_review-template)
8. [STATUS.md Template](#status-template)
9. [STATE.json Template](#state-template)

---

<a name="code_research-template"></a>
## CODE_RESEARCH.md Template

```markdown
# Code Research

**Issue:** {issue-name}
**Feature Description:** {what we're building}
**Research Date:** {timestamp}

---

### Summary

- **Risk:** {High | Medium | Low}
- **Key Findings:**
  - {Finding 1}
  - {Finding 2}
  - {Finding 3}
- **Top Recommendations:**
  - {Recommendation 1}
  - {Recommendation 2}
  - {Recommendation 3}
- **Questions to Answer:**
  - {Question 1}
  - {Question 2}

---

### Integration Points

**Files to Modify:**
- `path/to/file.ts:line-range` - Description of change
- `path/to/file.ts:line-range` - Description of change

**Files to Create:**
- `path/to/new-file.ts` - Description of new file

**Reusable Patterns:**
- `path/to/reference.ts:line-range` - Description of pattern
- `path/to/reference.ts:line-range` - Description of pattern

**Dependencies:**
- {package-name} (new | existing)
- {package-name} (new | existing)

---

### Technical Context

- **Stack:** {Languages, frameworks}
- **Pattern:** {Architecture style, design patterns}
- **Communication:** {Async patterns, data flow}
- **Conventions:** {Code style, naming patterns}

---

### Risks

- **{High | Medium | Low}:** {Risk description}
  - Impact: {What could go wrong}
  - Mitigation: {How to address}

---

### Key Files

```
{Directory structure}
├── path/to/file.ts (lines) - Description
└── path/to/other.ts (lines) - Description
```

---

### Open Questions

1. {Question 1}
   - {Context}
   - {Options}
   - {Recommendation}

2. {Question 2}
   - {Context}
   - {Options}
   - {Recommendation}
```

---

<a name="research_summary-template"></a>
## RESEARCH_SUMMARY.md Template

```markdown
# Research Summary

**Generated:** {timestamp}
**Issue:** {issue-name}

---

## For Planning Phase

### What We're Building
{Clear goal description from feature_description}

### Key Findings
{Extract top 5-7 key findings from CODE_RESEARCH.md}
- {Finding 1}
- {Finding 2}
- {Finding 3}
- {Finding 4}
- {Finding 5}

### Integration Points
**Files to modify:**
- `path/to/file.ts:line-range` - Description
- `path/to/file.ts:line-range` - Description

**Files to create:**
- `path/to/file.ts` - Description

**Dependencies:**
- {package-name} - {version}
- {package-name} - {version}

### Technical Context
- **Stack:** {Technologies used}
- **Pattern:** {Architecture style}
- **Conventions:** {Coding standards}

### Risks and Mitigations
1. **{Risk Name}** - Severity: {High | Medium | Low}
   - Impact description
   - Mitigation strategy

2. **{Risk Name}** - Severity: {High | Medium | Low}
   - Impact description
   - Mitigation strategy

### Recommendations
1. {Recommendation} - {Reason}
2. {Recommendation} - {Reason}
3. {Recommendation} - {Reason}

### Questions to Answer
{All open questions that planning must address}
- {Question 1}
- {Question 2}
- {Question 3}

---

*This is a comprehensive summary for the planning phase. See CODE_RESEARCH.md for full details.*
```

---

<a name="implementation_plan-template"></a>
## IMPLEMENTATION_PLAN.md Template

```markdown
# Implementation Plan

**Issue:** {issue-name}
**Feature:** {Brief description}

---

### Overview

**Goal:** {What we're building}

**Success Criteria:**
- [ ] {Criterion 1}
- [ ] {Criterion 2}
- [ ] {Criterion 3}

**Out of Scope:**
- {What we're NOT doing}

---

### Phases

**Phase 1: {Name}** (Complexity: {Low | Medium | High}, Est: {time})
- `path/to/file.ts:line-range` - {Task description}
- `path/to/file.ts:line-range` - {Task description}

**Phase 2: {Name}** (Complexity: {Low | Medium | High}, Est: {time})
- `path/to/file.ts:line-range` - {Task description}
- `path/to/file.ts:line-range` - {Task description}

**Phase 3: {Name}** (Complexity: {Low | Medium | High}, Est: {time})
- `path/to/file.ts:line-range` - {Task description}
- `path/to/file.ts:line-range` - {Task description}

---

### Testing Strategy

**Unit Tests:**
- {What to test}

**Integration Tests:**
- {What to test}

**Edge Cases:**
- {Edge case 1}
- {Edge case 2}

---

### Estimates

| Phase | Complexity | Estimated Time |
| ----- | ---------- | -------------- |
| 1     | {Level}    | {Time}         |
| 2     | {Level}    | {Time}         |
| 3     | {Level}    | {Time}         |
| **Total** | | **{Total Time}** |

---

### Dependencies

**External Packages:**
```
{package-name}: {version}
```

**Environment Variables:**
```bash
{VAR_NAME}={description}
```
```

---

<a name="project_spec-template"></a>
## PROJECT_SPEC.md Template

```markdown
# Project Specification

**Issue:** {issue-name}

---

## Requirements

### Functional Requirements
1. {Requirement 1}
2. {Requirement 2}
3. {Requirement 3}

### Non-Functional Requirements
- {Performance requirements}
- {Security requirements}
- {Usability requirements}

### Acceptance Criteria
- [ ] {Criterion 1}
- [ ] {Criterion 2}
- [ ] {Criterion 3}

---

## Technical Design

### Architecture
{Architecture overview, diagrams, patterns}

### Data Models
{Data structures, types, interfaces}

### API Design
{Endpoints, methods, contracts}

### Error Handling
{Error scenarios, recovery strategies}
```

---

<a name="plan_summary-template"></a>
## PLAN_SUMMARY.md Template

```markdown
# Plan Summary

**Generated:** {timestamp}
**Issue:** {issue-name}

---

## For Implementation Phase

### What to Build
{Complete feature description with acceptance criteria}

### Architecture Overview
{Complete architecture description}
- Pattern: {layered, event-driven, etc.}
- Components: {list with roles}
- Data flow: {description}

### Implementation Phases

**Phase 1: {Name}** (Complexity: {Low | Med | High})
- `path/to/file.ts:lines` - {Specific task}
- `path/to/file.ts:lines` - {Specific task}

**Phase 2: {Name}** (Complexity: {Low | Med | High})
- `path/to/file.ts:lines` - {Specific task}
- `path/to/file.ts:lines` - {Specific task}

### File-by-File Breakdown

**Files to Create:**
```
path/to/file.ts - {Purpose}
  - Lines X-Y: {Description}
  - Lines A-B: {Description}
```

**Files to Modify:**
```
path/to/file.ts:X-Y - {Purpose}
  - Lines X-Y: {Description}
```

### Testing Requirements

**Unit Tests:**
- {Functions to test}
- {Coverage requirements}

**Integration Tests:**
- {Workflows to test}

**Edge Cases:**
- {Scenarios to test}

### Dependencies
{Necessary dependencies and versions}

### Implementation Notes
{Additional context for implementer}
```

---

<a name="implementation_summary-template"></a>
## IMPLEMENTATION_SUMMARY.md Template

```markdown
# Implementation Summary

**Generated:** {timestamp}
**Issue:** {issue-name}

---

## For Review Phase

### What Was Built
{Complete description of what was implemented}

### Files Created
- `path/to/file.ts` - {Purpose}
- `path/to/file.ts` - {Purpose}

### Files Modified
- `path/to/file.ts:line-range` - {Change description}
- `path/to/file.ts:line-range` - {Change description}

### Implementation Approach
{How it was built, technical decisions}
- Architecture pattern
- Key decisions
- Implementation order

### All Phases Completed
- Phase 1: ✓ - {Summary}
- Phase 2: ✓ - {Summary}
- Phase 3: ✓ - {Summary}

### Tests Created
- {Test files with coverage}
- Total: {N} tests, {X%} passing

### Deviations from Plan
{Or: None - implementation followed plan exactly}
- {Deviation 1} - {Why}
- {Deviation 2} - {Why}

### Known Limitations
{Any known limitations or future work}
- {Limitation 1} - {Impact}
- {Limitation 2} - {Impact}

### Git Diff Summary
- Files changed: {N}
- Lines added: {N}
- Lines removed: {N}
```

---

<a name="code_review-template"></a>
## CODE_REVIEW.md Template

```markdown
# Code Review

**Issue:** {issue-name}
**Review Date:** {timestamp}

---

### Summary

- **Status:** {APPROVED | APPROVED_WITH_NOTES | NEEDS_REVISION}
- **Risk:** {Low | Medium | High}

---

### Automated Checks

```
✓/✗ Linting: {Passed | Failed} ({N errors, N warnings})
✓/✗ Type Checking: {Passed | Failed} ({N errors})
✓/✗ Tests: {N/N} passing ({X%})
✓/✗ Build: {Passed | Failed}
✓/✗ Security: {Vulnerabilities found}
```

---

### Code Quality

| Area | Rating | Notes |
|------|--------|-------|
| Structure | {Excellent | Good | Fair | Poor} | {Notes} |
| Naming | {Excellent | Good | Fair | Poor} | {Notes} |
| Error Handling | {Excellent | Good | Fair | Poor} | {Notes} |
| Type Safety | {Excellent | Good | Fair | Poor} | {Notes} |
| Testing | {Excellent | Good | Fair | Poor} | {Notes} |

---

### Issues

**❌ CRITICAL:**
- `path/to/file.ts:line` - {Issue description}

**⚠ IMPORTANT:**
- `path/to/file.ts:line` - {Issue description}
- `path/to/file.ts:line` - {Issue description}

**💡 SUGGESTIONS:**
- `path/to/file.ts:line` - {Issue description}

---

### Compliance

- ✓/✗ All planned features implemented
- ✓/✗ Design followed
- Deviations: {List any deviations or "None"}

---

### Recommendations

1. {Recommendation for improvement}
2. {Recommendation for improvement}
3. {Recommendation for improvement}
```

---

<a name="status-template"></a>
## STATUS.md Template

```markdown
# Status: {issue-name}

**Risk:** {High | Medium | Low | Unknown} | **Updated:** {timestamp}
**Iteration:** {N}/3 (during review-fix loop)

---

## Progress

- {[-] | [x] | [~]} Research
- {[-] | [x] | [~]} Planning
- {[-] | [x] | [~]} Implementation
- {[-] | [x] | [~]} Review

---

## Phase: {Phase Name} {Emoji}

- **Status:** {pending | in_progress | complete | failed}
- **Context:** {context mode description}

---

## Artifacts

- {Emoji} {Artifact Name}
- {Emoji} {Artifact Name}
```

---

<a name="state-template"></a>
## STATE.json Template

```json
{
  "schema_version": "1.0.0",
  "issue_name": "your-issue-name",
  "current_phase": "start",
  "phase_status": "pending",
  "review_iteration": 0,
  "created_at": "2026-01-12T10:00:00Z",
  "updated_at": "2026-01-12T10:00:00Z",
  "artifacts": {},
  "metadata": {
    "risk_level": "Unknown",
    "description": "Your feature description"
  }
}
```

---

**Template Usage:**
1. Copy the relevant template
2. Replace {placeholders} with actual content
3. Follow the structure and format
4. Ensure all required sections are present
