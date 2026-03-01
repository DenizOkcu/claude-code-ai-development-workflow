---
name: planning-solutions
description: "Creates phased implementation plans with clear scope and acceptance criteria. Use when defining what to build, sequencing work into phases, or establishing validation gates. Produces actionable plans with per-phase validation checklists."
model: opus
metadata:
  version: 1.0.0
  category: workflow-automation
---

# Solution Planning

**Mindset:** Define WHAT to build, not HOW to build every line. The plan is a guide, not a contract.

## Goal

Create a clear plan that answers:
1. What are we building? (scope)
2. What are the phases? (sequence)
3. How do we know it's done? (acceptance criteria)

## Instructions

### Step 1: Think Before Planning

Before planning, think harder about:
- What could go wrong?
- What's the simplest approach?
- How do we know when it's done?

### Step 2: Read Context

Read `RESEARCH.md` if it exists. Understand:
- Files to touch
- Patterns to follow
- Risks identified

### Step 3: Define Scope

**In Scope:** What we WILL do (3-5 items)
**Out of Scope:** What we WON'T do (important for boundaries)

### Step 4: Design Phases (2-4 phases)

Each phase should:
- Be completable in one sitting
- Result in working, testable code
- Build on previous phases

**Example structure:**
- Phase 1: Foundation (setup, core types)
- Phase 2: Core feature (main implementation)
- Phase 3: Polish (edge cases, tests, docs)

### Step 5: Define Acceptance Criteria

How do we know it's done? Specific, testable criteria:
- "User can authenticate via Google"
- "Failed login shows error message"
- "Tests cover happy path and error cases"

### Step 6: Define Validation per Phase

For EACH phase, define specific validation steps the implementer must run:

**Example:**
```markdown
### Phase 1: Foundation
Validation:
- [ ] `npm test` passes
- [ ] TypeScript compiles without errors
- [ ] New class is importable
- [ ] Basic unit test for constructor exists
```

This ensures the implementer knows exactly how to verify each phase is complete.

### Step 7: Create PLAN.md and Update STATUS.md

Write `docs/{issue_name}/PLAN.md` and update `docs/{issue_name}/STATUS.md`.

## Output Format

```markdown
# Plan: {issue_name}

**What:** {feature_description}
**When:** {timestamp}

---

## Scope

**Building:**
- {Feature 1}
- {Feature 2}

**NOT Building:**
- {Out of scope 1}

---

## Phases

### Phase 1: {Name}
**Goal:** {what this phase accomplishes}

Tasks:
- [ ] {task 1}
- [ ] {task 2}

Validation:
- [ ] {specific check}
- [ ] {specific check}

### Phase 2: {Name}
**Goal:** {what this phase accomplishes}

Tasks:
- [ ] {task 1}

Validation:
- [ ] {specific check}

---

## Acceptance Criteria

- [ ] {criterion 1}
- [ ] {criterion 2}
- [ ] All tests passing
- [ ] No regressions

---

## Technical Notes

**Approach:** {high-level approach}

**Files:**
- Create: `path/to/new-file.ts`
- Modify: `path/to/existing.ts`

---

## Risks & Mitigations

| Risk | Plan |
|------|------|
| {risk} | {mitigation} |
```

## Quality Check

- [ ] Clear scope defined?
- [ ] 2-4 phases with clear goals?
- [ ] **Validation checklist per phase defined?**
- [ ] Acceptance criteria testable?
- [ ] Risks identified?
- [ ] PLAN.md created (single file)?
- [ ] STATUS.md updated?

## Common Issues

- **Too detailed:** Don't specify line numbers (they become stale). Don't write pseudo-code for every function.
- **Too many phases:** Maximum 4 phases. Break further if needed in implementation.
- **Missing validation:** Every phase MUST have a validation checklist. This is critical for the implementer.
- **Duplicate artifacts:** Create PLAN.md only. Don't create separate PLAN + SPEC documents.
