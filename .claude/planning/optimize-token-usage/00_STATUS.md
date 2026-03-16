# Status: optimize-token-usage

**Risk:** Low | **Updated:** 2026-03-15

## Summary
Reduce always-on token cost by restructuring CLAUDE.md files. Separate behavioral rules (always-on) from reference material (on-demand).

## Progress
- [x] Discovery - N/A (direct analysis, no formal discovery)
- [x] Research - Completed (line count analysis, identified 1,108-line duplication)
- [x] Design - N/A (straightforward file restructuring)
- [x] Planning - N/A
- [x] Implementation - Completed (6 files modified, 2 created)
- [x] Review - Completed (README + commit verified)
- [x] Security - N/A (prompt-only, no runtime code)
- [x] Deploy - Completed (pushed to main: 39d2da5)
- [x] Observe - N/A
- [x] Retro - Completed

## Key Decisions
- Project CLAUDE.md: rules only (~91 lines). No learnings history, no command catalog, no cheat sheets.
- Global ~/CLAUDE.md: shared guidelines only (~98 lines). No project-specific content.
- .claude/LEARNINGS.md: full learnings archive (on-demand).
- .claude/QUICK_REFERENCE.md: tool cheat sheets (on-demand).
- /retro rotation rule: max 2 most recent blocks in CLAUDE.md; older blocks live in LEARNINGS.md only.

## Artifacts
- `.claude/LEARNINGS.md` — created
- `.claude/QUICK_REFERENCE.md` — created
- `.claude/commands/retro.md` — updated (writes to LEARNINGS.md + abbreviated CLAUDE.md)
- `CLAUDE.md` — reduced 555→91 lines
- `~/CLAUDE.md` — reduced 552→98 lines (not tracked in git)
- `README.md` — updated (Memory & Token Optimization section, file tree, phase table)

## Status: ✅ WORKFLOW COMPLETE
