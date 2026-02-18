---
name: researching-code
description: "Understand minimum codebase context needed for planning. Use before implementation or when analyzing unfamiliar code areas. Triggers: research, investigate codebase, analyze architecture, find patterns."
model: sonnet
---

# Code Research

**Mindset:** Understand just enough to plan effectively. Skip comprehensive documentation.

## Goal

Find the minimum context needed to answer:
1. What files will this touch?
2. What patterns should we follow?
3. What are the main risks?

## Extended Thinking

Before creating artifacts, think deeply about:
- What are the hidden dependencies?
- What could break if we change X?
- What's the simplest approach?

Use phrases like "think deeper", "think about edge cases" to trigger extended thinking.

## Inputs
- `issue_name`: Kebab-case identifier
- `feature_description`: What to build

## Output
- `docs/{issue_name}/RESEARCH.md`
- `docs/{issue_name}/STATUS.md` (updated)

## Procedure

### 1. Quick Discovery (5-15 files max)

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

### 2. Create RESEARCH.md

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

### 3. Update STATUS.md

```markdown
# Status: {issue_name}

**Risk:** {level} | **Updated:** {timestamp}

## Progress
- [~] Research | [ ] Planning | [ ] Implementation | [ ] Review

## Phase: Research
- **Finding:** {key finding}
- **Risk:** {level}
- **Next:** Planning

## Artifacts
- RESEARCH.md ✓
```

## What NOT to Do

- Don't document the entire architecture
- Don't trace full data flows
- Don't analyze git history (rarely needed)
- Don't create 30+ file analyses
- Don't write comprehensive documentation

## Quality Check

Before marking complete:
- [ ] Answered: What files to touch?
- [ ] Answered: What patterns to follow?
- [ ] Answered: What are the risks?
- [ ] RESEARCH.md created (not CODE_RESEARCH.md + RESEARCH_SUMMARY.md)
- [ ] STATUS.md updated
