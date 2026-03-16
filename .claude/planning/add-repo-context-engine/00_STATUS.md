# Status: add-repo-context-engine

**Risk:** Low | **Updated:** 2026-03-15
**Stack:** Claude Code skills/commands (Markdown + YAML) + MCP (claude-context optional)

## Progress
- [x] Discovery — Completed
- [x] Research — Completed
- [ ] Design — Skipped (M-size, design captured in research + plan)
- [x] Planning — Completed (3 phases)
- [x] Implementation — Completed (1 file created, 3 files modified)
  - Phase 1: ✓ Complete — `/repo-map` command created
  - Phase 2: ✓ Complete — `/discover` integration (Step 3 + template + output + quality gate)
  - Phase 3: ✓ Complete — `researching-code` Level 1→2 + ARCHITECTURE.md
- [x] Review — APPROVED (0 blocking, 4 suggestions, 10 praise)
- [x] Security (7a) — PASSED (0 critical/high/medium, 1 low accepted risk)
- [x] Deploy — Plan created (Big Bang, single commit)
- [ ] Observe — Skipped (prompt-only, no runtime metrics)
- [x] Retro — Completed

## Status: ✅ WORKFLOW COMPLETE

## Detected Stack
Claude Code skills/commands (Markdown prompt files) + `@zilliz/claude-context-mcp` (optional MCP)

## Key Decisions
1. **Repo map embedded in `/discover` (not separate artifact)** — user feedback: users forget to run standalone commands, so auto-generate during discovery
2. **Glob + Grep approach (not MCP-dependent)** — must work in repos without `claude-context` MCP configured
3. **~2K token budget for repo map** — aligns with Aider's proven sweet spot (1K–2K tokens)
4. **Level 1→Level 2 escalation in `researching-code`** — read map from `01_DISCOVERY.md`, identify candidate files, then fetch detail
5. **3-phase implementation**: standalone command → discover integration → research skill upgrade

## Files Changed
- **Created:** `.claude/commands/repo-map.md`
- **Modified:** `.claude/commands/discover.md` (Step 3 + template + output + quality gate)
- **Modified:** `.claude/skills/researching-code/SKILL.md` (Step 0 → hierarchical context loading)
- **Modified:** `.claude/ARCHITECTURE.md` (Context Engine section)

## Artifacts
- 01_DISCOVERY.md
- 02_CODE_RESEARCH.md
- 04_IMPLEMENTATION_PLAN.md
- 06_CODE_REVIEW.md
- 07a_SECURITY_AUDIT.md
- 09_DEPLOY_PLAN.md
- 11_RETROSPECTIVE.md
