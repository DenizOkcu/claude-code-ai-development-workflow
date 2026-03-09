# Artifact Templates

Lean templates for SDLC artifacts. 5 files total per issue.

---

## RESEARCH.md

```markdown
# Research: {issue_name}

**What:** {feature_description}
**When:** {timestamp}

---

## Summary

- **Risk:** Low | Medium | High
- **Approach:** {Brief approach recommendation}
- **Effort:** Quick | Moderate | Significant

---

## What We Found

### Files to Touch
- `path/to/file.ts` - {why}
- `path/to/file.ts` - {why}

### Patterns to Follow
- `{pattern}` from `path/to/reference.ts`
- `{convention}` used in this codebase

### Key Dependencies
- `{package}` - existing | needed
- `{shared_module}` - existing

---

## Risks

| Risk | Severity | Mitigation |
|------|----------|------------|
| {risk} | Low/Med/High | {how to address} |

---

## Open Questions

1. **{Question}?**
   - Options: {A or B}
   - Recommendation: {A because...}
```

---

## PLAN.md

```markdown
# Plan: {issue_name}

**What:** {feature_description}
**When:** {timestamp}

---

## Scope

**Building:**
- {Feature 1}
- {Feature 2}
- {Feature 3}

**NOT Building:**
- {Out of scope 1}
- {Out of scope 2}

---

## Phases

### Phase 1: {Name}
**Goal:** {what this phase accomplishes}

Tasks:
- [ ] {task 1}
- [ ] {task 2}
- [ ] {task 3}

Validation:
- [ ] {specific check - e.g., "npm test passes"}
- [ ] {specific check - e.g., "TypeScript compiles"}

### Phase 2: {Name}
**Goal:** {what this phase accomplishes}

Tasks:
- [ ] {task 1}
- [ ] {task 2}

Validation:
- [ ] {specific check}
- [ ] {specific check}

### Phase 3: {Name}
**Goal:** {what this phase accomplishes}

Tasks:
- [ ] {task 1}
- [ ] {task 2}

Validation:
- [ ] {specific check}
- [ ] {specific check}

---

## Acceptance Criteria

- [ ] {criterion 1}
- [ ] {criterion 2}
- [ ] {criterion 3}
- [ ] All tests passing
- [ ] No regressions

---

## Technical Notes

**Approach:** {high-level approach}

**Files:**
- Create: `path/to/new-file.ts`
- Modify: `path/to/existing.ts`

**Dependencies:**
- {package} (new | existing)

---

## Risks & Mitigations

| Risk | Plan |
|------|------|
| {risk} | {mitigation} |
```

---

## IMPLEMENTATION.md

```markdown
# Implementation: {issue_name}

**What:** {feature_description}
**When:** {timestamp}

---

## Summary

Built {brief description of what was implemented}.
{N}/{N} phases completed.

---

## Changes

### Files Created
- `path/to/file.ts` - {purpose}
- `path/to/file.ts` - {purpose}

### Files Modified
- `path/to/file.ts` - {what changed}

---

## Phases Completed

- [x] Phase 1: {name} - {brief result}
- [x] Phase 2: {name} - {brief result}
- [x] Phase 3: {name} - {brief result}

---

## Test Results

- Tests: {N} total, {N} passing
- Coverage: {X}% for new code
- Type check: ✓
- Lint: ✓
- Build: ✓

---

## Deviations from Plan

{None | List deviations with reasons}

| Plan Said | What We Did | Why |
|-----------|-------------|-----|
| {original} | {actual} | {reason} |

---

## Known Limitations

- {limitation} (impact: {low/medium})
```

---

## REVIEW.md

```markdown
# Review: {issue_name}

**When:** {timestamp}

---

## Verdict

**Status:** APPROVED | NEEDS_FIX

---

## Automated Checks

| Check | Status | Notes |
|-------|--------|-------|
| Tests | ✓/✗ | {N} passing |
| Type Check | ✓/✗ | |
| Lint | ✓/✗ | |
| Build | ✓/✗ | |
| Security | ✓/✗ | |

---

## Plan Compliance

- [ ] Acceptance criteria met
- [ ] Planned features built
- [ ] Deviations documented

**Deviations:** {None | acceptable because...}

---

## Issues

### Blocking (must fix)
- `path/to/file.ts` - {issue}

### Non-Blocking (nice to have)
- `path/to/file.ts` - {suggestion}

---

## Decision

{APPROVED: Ready to ship | NEEDS_FIX: Blocking issues above}
```

---

## STATUS.md

```markdown
# Status: {issue_name}

**Risk:** {level} | **Updated:** {timestamp}

## Progress
- [ ] Research | [ ] Planning | [ ] Implementation | [ ] Review

## Phase: {current}
- **Status:** {pending|in_progress|complete|blocked}
- **Next:** {next phase}

## Artifacts
- RESEARCH.md
- PLAN.md
- IMPLEMENTATION.md
- REVIEW.md
```

---

## STATE.json (optional)

```json
{
  "issue_name": "add-oauth",
  "current_phase": "implementation",
  "phase_status": "in_progress",
  "review_iteration": 0,
  "created_at": "2026-01-12T10:00:00Z",
  "updated_at": "2026-01-12T11:30:00Z",
  "artifacts": ["RESEARCH.md", "PLAN.md"]
}
```

---

---

## PENTEST_REPORT.md

```markdown
# Penetration Test Report — {issue_name}

**Date**: {date}
**Tool**: Shannon Lite (autonomous AI pentester)
**Target**: {staging_url}
**Duration**: {duration}
**Shannon Workflow ID**: {workflow_id}

## Executive Summary

{2-3 sentence summary: number of critical/high findings, categories, overall risk}

## Confirmed Vulnerabilities

### [SEVERITY] Finding Title

**CVSS Score**: X.X | **CWE**: CWE-XXX | **OWASP**: A0X

**Description**: {concise description}
**Affected**: {file path or endpoint}

**Reproduction Steps**
1. {step}

**Proof of Concept**
```
{copy-paste exploit}
```

**Fix**: {specific code change}

## Unconfirmed / Informational

{Findings Shannon attempted but could not exploit}

## Correlation with Static Audit

| Static Finding | Dynamic Result |
|---|---|
| {finding} | CONFIRMED / NOT EXPLOITABLE / NEEDS MANUAL REVIEW |

## Metrics

- Critical: X | High: X | Medium: X | Low: X
```

---

## AI_THREAT_MODEL.md

```markdown
# AI Threat Model — {issue_name}

**Date**: {date}
**Models Used**: {list with versions}

## AI Components Inventory

| Component | Model | Integration Point | User Input Path |
|---|---|---|---|
| {component} | {model} | {endpoint} | {data flow} |

## Threat Summary

| Threat | Vector | Severity | Exploitable? |
|---|---|---|---|
| {threat} | Direct / Indirect | High | YES — PoC in §X |

## Mitigations

- Never interpolate raw user input into system prompts
- HTML-escape model output before rendering
- Restrict tool calls to an explicit allow-list
```

---

## HARDEN_PLAN.md

```markdown
# Hardening Plan — {issue_name}

**Source reports**: SECURITY_AUDIT.md | PENTEST_REPORT.md | AI_THREAT_MODEL.md
**Total findings**: X (Critical: X | High: X | Medium: X | Low: X)

## Fix Summary

| Priority | Finding | CWE | Affected File | Effort |
|---|---|---|---|---|
| P0 | {finding} | CWE-XXX | {path} | {hours} |

## P0 Fixes — Implement Immediately

### Fix 1: {title}
**Root cause**: {why}
**Before**: {vulnerable code}
**After**: {fixed code}
**Test**: {regression test}

## Verification Checklist

- [ ] All regression tests pass
- [ ] Re-run `/security` — static findings resolved
- [ ] Re-run `/security/pentest` — exploits closed
```

---

**Artifact Summary:**

| File | Purpose | Created By |
|------|---------|------------|
| RESEARCH.md | What we found | Research phase |
| PLAN.md | What we'll build | Planning phase |
| IMPLEMENTATION.md | What we built | Implementation phase |
| REVIEW.md | Is it ready? | Review phase |
| STATUS.md | Where are we? | All phases |
| SECURITY_AUDIT.md | Static security findings | Phase 7a |
| PENTEST_REPORT.md | Confirmed exploits with PoCs | Phase 7b |
| AI_THREAT_MODEL.md | LLM attack surface | Phase 7c |
| HARDEN_PLAN.md | Fix plan + patches | Phase 8 |
