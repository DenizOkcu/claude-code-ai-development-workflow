---
name: researching-code
description: "Investigates codebase to find minimum context needed for planning. Use when starting a new feature, analyzing unfamiliar code, or preparing for implementation. Identifies files to touch, patterns to follow, and risks to mitigate."
model: opus
metadata:
  version: 1.0.0
  category: workflow-automation
---

# Code Research

**Mindset:** Understand just enough to plan effectively. Skip comprehensive documentation.

## Goal

Find the minimum context needed to answer:
1. What files will this touch?
2. What patterns should we follow?
3. What are the main risks?

## Instructions

### Step 1: Think Deeply Before Searching

Before creating artifacts, think deeply about:
- What are the hidden dependencies?
- What could break if we change X?
- What's the simplest approach?

Use phrases like "think deeper", "think about edge cases" to trigger extended thinking.

### Step 2: Quick Discovery (5-15 files max)

Use Glob/Grep/Read to answer the three questions:

**What files will this touch?**
```bash
# Find related files
Grep pattern="related_term" output_mode="files_with_matches"
Glob pattern="**/*related*"
```

**What patterns should we follow?**
```bash
# Find similar implementations
Grep pattern="similar_feature" output_mode="files_with_matches"
Read file_path="path/to/similar/implementation"
```

**What are the main risks?**
- Check for tests in related areas
- Look for shared utilities/dependencies
- Identify tight coupling

### Step 3: Create RESEARCH.md

Write `docs/{issue_name}/RESEARCH.md` with this structure:

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

### Patterns to Follow
- `{pattern}` from `path/to/reference.ts`

### Key Dependencies
- `{package}` - existing | needed

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

### Step 4: Update STATUS.md

Update `docs/{issue_name}/STATUS.md` to reflect research completion.

## Output Format

- `docs/{issue_name}/RESEARCH.md`
- `docs/{issue_name}/STATUS.md` (updated)

## Quality Check

- [ ] Answered: What files to touch?
- [ ] Answered: What patterns to follow?
- [ ] Answered: What are the risks?
- [ ] RESEARCH.md created (not CODE_RESEARCH.md + RESEARCH_SUMMARY.md)
- [ ] STATUS.md updated

## Common Issues

- **Over-researching:** Don't document the entire architecture. Don't trace full data flows. Don't analyze git history (rarely needed). Don't create 30+ file analyses.
- **Wrong artifact name:** Create RESEARCH.md, not CODE_RESEARCH.md + RESEARCH_SUMMARY.md.
- **Too many files:** Limit discovery to 5-15 files maximum.
