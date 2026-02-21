## Phase 10: Retrospective & Knowledge Capture

You are entering the **Retro** phase for issue: `$ARGUMENTS`

### Pre-Conditions
- Read `STATUS.md` — review the entire journey
- Read all artifacts in `.claude/planning/$ARGUMENTS/`
- Read project root `CLAUDE.md` for current learnings

### Instructions

Capture lessons learned and improve future workflows. Create `.claude/planning/$ARGUMENTS/RETROSPECTIVE.md` AND update the project's `CLAUDE.md`.

#### 1. `RETROSPECTIVE.md`

```markdown
# Retrospective: {issue-name}

## Summary
- **Started:** {date}
- **Completed:** {date}
- **Complexity:** {actual vs estimated}
- **Files Changed:** {count}
- **Tests Added:** {count}
- **Phases Completed:** {N}/10

## What Went Well
- ...
- ...

## What Could Be Improved
- ...
- ...

## Surprises / Unknowns Encountered
- ...
- ...

## Key Technical Learnings
- Learning 1: {specific, actionable insight}
- Learning 2: ...

## Process Learnings
- Were any phases unnecessary for this type of change?
- Were any phases missing or insufficient?
- What would you do differently next time?

## Patterns to Reuse
- Pattern: {description}
  - Where: {context}
  - Why: {benefit}

## Anti-Patterns to Avoid
- Anti-pattern: {description}
  - Why: {consequence}
  - Instead: {better approach}

## Metrics
- Estimation accuracy: {estimated vs actual effort}
- Test coverage delta: {before vs after}
- Review iterations: {count}
- Security findings: {count and severity}
```

#### 2. Update `CLAUDE.md`

Append **specific, actionable learnings** to the `## Learnings` section in the project root `CLAUDE.md`. Only add insights that will help future development:

**Good learnings:**
- `2026-02-21: Always use parameterized queries for user input in the reports module (add-report-export)`
- `2026-02-21: WebSocket reconnection needs exponential backoff with jitter (add-realtime-collab)`

**Bad learnings (too vague):**
- `Write good tests` (not actionable)
- `Be careful with security` (not specific)

#### 3. Update `STATUS.md`

Mark ALL phases as complete:

```markdown
## Progress
- [x] Discovery - Completed
- [x] Research - Completed
- [x] Design - Completed
- [x] Planning - Completed
- [x] Implementation - Completed
- [x] Review - Completed ✓ APPROVED
- [x] Security - Completed ✓ PASSED
- [x] Deploy - Completed (planned)
- [x] Observe - Completed (configured)
- [x] Retro - Completed

## Status: ✅ WORKFLOW COMPLETE
```

### Post-Actions
- Update `STATUS.md`: mark Retro as completed, set status to WORKFLOW COMPLETE
- Update `CLAUDE.md` with learnings
- Summarize the entire workflow journey to the user
- Suggest: commit, push, and deploy

### Quality Gates
- Retrospective covers all sections (not just "it went fine")
- At least 2 learnings added to CLAUDE.md
- Learnings are specific and reference the issue name
- Process learnings address whether phase selection was appropriate
- STATUS.md shows complete journey
