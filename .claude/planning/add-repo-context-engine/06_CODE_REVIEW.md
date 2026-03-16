# Code Review: add-repo-context-engine

**When:** 2026-03-15
**Reviewer:** Automated review (senior engineer simulation)

---

## Automated Checks

| Check | Status | Notes |
|-------|--------|-------|
| YAML Frontmatter | ✅ pass | All 3 modified files + 1 new file have valid frontmatter |
| Markdown Structure | ✅ pass | All files have correct heading hierarchy |
| Pattern Consistency | ✅ pass | Command format (frontmatter → instructions → quality gates) followed |
| Backward Compatibility | ✅ pass | All changes are additive; no existing sections removed |

*No runtime code — linting, type checking, tests, and build checks are N/A for this prompt-engineering project.*

---

## Manual Code Review

### File 1: `.claude/commands/repo-map.md` (NEW)

| # | Severity | Finding |
|---|----------|---------|
| 1 | 💡 Praise | Excellent progressive truncation strategy (4 tiers: <100, 100-200, 200-500, >500 files). Well-reasoned thresholds that degrade gracefully. |
| 2 | 💡 Praise | Path-scoped mode (`/repo-map src/auth`) is a smart power-user feature that bypasses truncation — good design. |
| 3 | 🔵 Suggestion | `discover.md:89` — The exclusion list in Step 3 of discover duplicates the list in `repo-map.md:30-34`. If someone adds a pattern to one, they'll likely forget the other. Consider adding a note: "See `/repo-map` for the canonical exclusion list." |
| 4 | 🔵 Suggestion | `repo-map.md:23` — The generic fallback pattern includes `pub fn` which is Rust-specific. Not harmful (won't match in non-Rust repos) but slightly impure for a "generic" pattern. |
| 5 | 💡 Praise | The `>8 symbols → show first 6 + N more` rule prevents verbose maps for large files. Good information density control. |

### File 2: `.claude/commands/discover.md` (MODIFIED)

| # | Severity | Finding |
|---|----------|---------|
| 6 | 💡 Praise | Clean step renumbering (1→2→**3**→4→5→6→7). No gaps, no confusing 2.5 numbering. |
| 7 | 💡 Praise | `## Repository Map` section placement is correct: after `## Detected Tech Stack`, before `### Missing Quality Tooling Recommendations`. Data flows naturally top-to-bottom. |
| 8 | 🔵 Suggestion | `discover.md:175` — The placeholder `{paste the compact repo map generated in Step 3 here}` is clear but could be more explicit: "Insert the formatted repo map output from Step 3." Minor wording preference. |
| 9 | 💡 Praise | Quality gate addition (`Repository map generated and embedded in 01_DISCOVERY.md (≤2K tokens)`) is specific and measurable — exactly what quality gates should be. |

### File 3: `.claude/skills/researching-code/SKILL.md` (MODIFIED)

| # | Severity | Finding |
|---|----------|---------|
| 10 | 💡 Praise | The Level 1→Level 2 escalation is clearly explained. The "map exists" vs "no discovery doc" branching is well-specified with graceful fallback. |
| 11 | 💡 Praise | "Mentally note the repo structure and candidate files — you don't need to write this down" — smart constraint that prevents the LLM from over-documenting during research. |
| 12 | 💡 Praise | Cross-referencing MCP results with map candidates ("both structural relevance and semantic relevance") is a solid information fusion technique. |
| 13 | 🔵 Suggestion | `SKILL.md:33` — The on-the-fly map fallback exclusion list (`node_modules/`, `vendor/`, etc.) is a shorter subset of the canonical list in `repo-map.md`. This is fine for a "quick" map but could lead to minor inconsistency. Consider referencing the full list from `/repo-map`. |

### File 4: `.claude/ARCHITECTURE.md` (MODIFIED)

| # | Severity | Finding |
|---|----------|---------|
| 14 | 💡 Praise | The ASCII diagram for the two-level context engine is clear and compact. Shows the data flow and relationship between levels well. |
| 15 | 💡 Praise | "Inspired by" attribution to Aider with differentiation note ("simpler but always available") is good context for future readers. |
| 16 | 💡 Praise | Integration point update (`researching-code` skill line 168) accurately reflects the new scoped query behavior. |

---

## Plan Compliance

- [x] Phase 1 acceptance criteria met: `/repo-map` command exists, follows format, covers 6 languages, compact format
- [x] Phase 2 acceptance criteria met: `/discover` auto-generates map, embedded in `01_DISCOVERY.md`, existing output unchanged, step numbering clean
- [x] Phase 3 acceptance criteria met: `researching-code` reads map first, graceful fallback when no map, MCP still used when available, Quality Check updated, ARCHITECTURE.md documented
- [x] No deviations from plan

---

## CLAUDE.md Consistency Check

| Item | Status | Notes |
|------|--------|-------|
| `/repo-map` listed in CLAUDE.md commands | ⚠ Not yet | CLAUDE.md doesn't reference `/repo-map` in the Quick Start or Expert Commands sections. Non-blocking — CLAUDE.md is large and should be updated as part of deploy/retro. |
| Workflow section references repo map | ⚠ Not yet | Same — the workflow section doesn't mention repo map as part of `/discover`. |

---

## Review Summary

| Category | Count |
|----------|-------|
| 🔴 Critical | 0 |
| 🟡 Important | 0 |
| 🔵 Suggestions | 4 |
| 💡 Praise | 10 |

## Verdict: ✓ APPROVED

All planned features implemented correctly. No blocking issues found. The hierarchical context engine is well-designed with proper graceful degradation, clean integration into existing commands/skills, and accurate architecture documentation.

### Recommendations (non-blocking)

1. **Exclusion list consolidation** (findings #3, #13): Consider adding a cross-reference note in `discover.md` Step 3 and `researching-code` Step 0 pointing to `/repo-map` as the canonical exclusion list. Prevents drift over time.
2. **CLAUDE.md update**: Add `/repo-map` to the Quick Reference Commands section and the workflow documentation during the deploy or retro phase.
3. **Generic pattern cleanup** (finding #4): Optional — remove `pub fn` from the generic fallback pattern since Rust has its own dedicated pattern.
