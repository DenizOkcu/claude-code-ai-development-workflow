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

### Step 0: Hierarchical Context Loading

Load context in two levels — structural overview first, then targeted detail.

**Level 1 — Repo Map (structural overview):**

Check if `01_DISCOVERY.md` exists for this issue (in `.claude/planning/{issue-name}/`) and contains a `## Repository Map` section.

- **If the map exists:** Read it. Use it to identify which directories and files are likely relevant to the feature. Note the key entry points and the overall structure.
- **If no discovery doc exists** (user jumped straight to `/research`): Generate a quick repo map on-the-fly using Glob + Grep:
  1. Run `Glob` for source files (exclude `node_modules/`, `vendor/`, `dist/`, `build/`, `.git/`, `*.lock`, `*.min.js`)
  2. Auto-detect the primary language from file extensions
  3. Run `Grep` with the appropriate language pattern to extract top-level symbols (classes, functions, exports)
  4. Mentally note the repo structure and candidate files — you don't need to write this down, just use it to guide your searches

**Level 2 — Targeted Detail (for candidate files only):**

From the repo map, identify 5-15 candidate files most relevant to the feature. Then:

- **If `search_code` MCP is available:** Call `search_code` with the project path and a natural language description of the feature. Cross-reference results with your candidate list from the map. This gives you both structural relevance (from the map) and semantic relevance (from the search).
- **If MCP is not available:** Use `Grep` for feature-related terms scoped to the candidate directories identified from the map, then `Read` the top matches.

The key insight: the map tells you *where* to look, the detail search tells you *what's there*. Don't search the entire repo when the map narrows it down.

### Step 1: Think Deeply Before Searching

Before creating artifacts, think deeply about:
- What are the hidden dependencies?
- What could break if we change X?
- What's the simplest approach?

Use phrases like "think deeper", "think about edge cases" to trigger extended thinking.

### Step 2: Quick Discovery (5-15 files max)

Use the repo map from Step 0 to narrow your Glob/Grep searches to relevant directories. Don't search the entire repo when the map tells you where to look.

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

- [ ] Used repo map (Level 1) to identify candidate files?
- [ ] Used targeted search (Level 2) for candidate files only?
- [ ] Answered: What files to touch?
- [ ] Answered: What patterns to follow?
- [ ] Answered: What are the risks?
- [ ] RESEARCH.md created (not CODE_RESEARCH.md + RESEARCH_SUMMARY.md)
- [ ] STATUS.md updated

## Common Issues

- **Over-researching:** Don't document the entire architecture. Don't trace full data flows. Don't analyze git history (rarely needed). Don't create 30+ file analyses.
- **Wrong artifact name:** Create RESEARCH.md, not CODE_RESEARCH.md + RESEARCH_SUMMARY.md.
- **Too many files:** Limit discovery to 5-15 files maximum.
- **Ignoring the repo map:** Don't skip the map and go straight to broad Grep. The map exists to narrow your search — use it to identify candidate directories before searching.
