---
name: planning-solutions
description: "Define WHAT to build with clear phases and acceptance criteria. Use when creating implementation strategies. Triggers: plan, design solution, implementation strategy."
model: sonnet
---

# Solution Planning

**Mindset:** Define WHAT to build, not HOW to build every line. The plan is a guide, not a contract.

## Extended Thinking

Before planning, think harder about:
- What could go wrong?
- What's the simplest approach?
- How do we know when it's done?

## Goal

Create a clear plan that answers:
1. What are we building? (scope)
2. What are the phases? (sequence)
3. How do we know it's done? (acceptance criteria)

## Inputs
- `issue_name`: Kebab-case identifier
- `feature_description`: What to build
- `RESEARCH.md`: Research findings (if exists)

## Output
- `docs/{issue_name}/PLAN.md`
- `docs/{issue_name}/STATUS.md` (updated)

## Procedure

### 1. Read Context

Read `RESEARCH.md` if it exists. Understand:
- Files to touch
- Patterns to follow
- Risks identified

### 2. Define Scope

**In Scope:** What we WILL do (3-5 items)
**Out of Scope:** What we WON'T do (important for boundaries)

### 3. Design Phases (2-4 phases)

Each phase should:
- Be completable in one sitting
- Result in working, testable code
- Build on previous phases

**Example structure:**
- Phase 1: Foundation (setup, core types)
- Phase 2: Core feature (main implementation)
- Phase 3: Polish (edge cases, tests, docs)

### 4. Define Acceptance Criteria

How do we know it's done? Specific, testable criteria:
- "User can authenticate via Google"
- "Failed login shows error message"
- "Tests cover happy path and error cases"

### 5. Define Validation per Phase

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

### 6. Create PLAN.md

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
- [ ] {specific check - e.g., "New module is importable"}

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

### 6. Update STATUS.md

```markdown
# Status: {issue_name}

**Risk:** {level} | **Updated:** {timestamp}

## Progress
- [x] Research | [~] Planning | [ ] Implementation | [ ] Review

## Phase: Planning
- **Phases:** {N}
- **Key Decisions:** {decision}
- **Next:** Implementation

## Artifacts
- RESEARCH.md ✓
- PLAN.md ✓
```

## What NOT to Do

- Don't specify line numbers (they become stale)
- Don't write pseudo-code for every function
- Don't create detailed API contracts upfront
- Don't make more than 4 phases
- Don't create separate PLAN + SPEC documents

## Good vs Bad Plans

**Good:**
```
Phase 1: Foundation
- Create GoogleOAuthStrategy class
- Register strategy with passport
- Add dependency to package.json
```

**Bad:**
```
Phase 1: Foundation (lines 1-50 of src/auth/google.ts)
- Lines 1-10: Import statements
- Lines 11-30: Class definition
- Lines 31-50: Constructor with OAuth config
```

## Quality Check

- [ ] Clear scope defined?
- [ ] 2-4 phases with clear goals?
- [ ] **Validation checklist per phase defined?**
- [ ] Acceptance criteria testable?
- [ ] Risks identified?
- [ ] PLAN.md created (single file)?
- [ ] STATUS.md updated?
