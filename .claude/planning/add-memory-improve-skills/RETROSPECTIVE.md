# Retrospective: add-memory-improve-skills

## Summary
- **Started:** 2026-02-28
- **Completed:** 2026-02-28
- **Complexity:** Estimated M, Actual M (accurate)
- **Files Changed:** 21 modified/created, 6 deleted
- **Tests Added:** 0 (pure Markdown/YAML — no test framework; verified via 13 automated grep/wc checks)
- **Phases Completed:** 10/10

## What Went Well

1. **Non-destructive migration pattern.** Creating new folder-based skills alongside old flat files before deleting allowed safe rollback at any point. Phase 1 (create new) was fully verified before Phase 2 (delete old + update refs).

2. **Cross-reference audit caught everything.** Systematic grep-based auditing after each phase found all 4 files referencing old skill names, including 7 stale paths in `docs/integration-plan.md` that would have been missed with a less thorough approach.

3. **Memory system seeded with real knowledge.** Instead of empty templates, the memory files were populated with actual patterns, decisions, and learnings from this very implementation — making them immediately useful for the next session.

4. **Full SDLC workflow worked end-to-end.** All 10 phases executed smoothly. The structured approach caught issues that ad-hoc implementation would have missed (e.g., the ASCII diagram multi-line rename).

5. **Deploy plan uncovered ~/.claude preservation requirements.** The user's feedback to audit `~/.claude` for plugins, MCP servers, and existing learnings added real value — INFRA-555 retro learnings were migrated into the new memory system.

6. **Model routing discovery.** The AI integration analysis revealed that `model:` IS supported in skill/command frontmatter — correcting an earlier assumption. This enabled immediate implementation of cost-saving model routing.

## What Could Be Improved

1. **Earlier verification of the `model:` field spec.** The initial implementation removed `model: sonnet` from skills based on incomplete information. The field IS officially supported. A more thorough spec check during Research/Design would have caught this.

2. **Context window management.** Two Agent tool calls hit rate limits during the planning phase, requiring fallback to direct Read calls. For large file sets, prefer batched Read calls over parallel Explore agents.

3. **Observe phase was skipped.** The workflow jumped from Deploy to Retro without creating OBSERVABILITY.md. For a Markdown/YAML project this is arguably N/A, but it should have been explicitly marked as skipped with justification.

4. **The `/sdlc` command model routing interaction is unclear.** When `/sdlc` (model: sonnet) orchestrates phases that have their own model: field, the precedence behavior hasn't been tested. This needs validation post-deploy.

## Surprises / Unknowns Encountered

1. **`model:` field is in the official Claude Code spec.** The Anthropic documentation for skills includes `model` as an optional frontmatter field with values `sonnet`, `opus`, `haiku`, `inherit`. This was not discovered during the initial Research phase.

2. **ASCII art diagrams break replace-all renames.** The orchestrator's visual diagram split `fixing-review-issues` across two lines (`│fixing-review │` / `│-issues       │`), which `replace_all` couldn't match. Required manual Edit.

3. **Global `~/.claude/CLAUDE.md` had retro learnings not in the repo CLAUDE.md.** The INFRA-555 learnings existed in the user's global CLAUDE.md but not in the repo's CLAUDE.md. This asymmetry needed explicit migration during the deploy phase.

4. **Jira API token in plaintext in `claude_desktop_config.json`.** Discovered during the deploy phase's `~/.claude` inventory. Not in scope for this issue but flagged for future remediation.

## Key Technical Learnings

1. **The `model:` frontmatter field IS officially supported in Claude Code skills and commands.** Values: `sonnet`, `opus`, `haiku`, `inherit`. Use this for cost-optimized model routing per phase (add-memory-improve-skills).

2. **Claude Code commands and skills in `.claude/commands/` and `.claude/skills/` are functionally equivalent.** Both support the same YAML frontmatter fields. A command at `.claude/commands/review.md` and a skill at `.claude/skills/review/SKILL.md` both create `/review` (add-memory-improve-skills).

3. **Always audit `~/.claude/` before deploying changes that touch the user's Claude Code environment.** Check for: `settings.json` (permissions, model preference), `plugins/` (installed plugins), `claude_desktop_config.json` (MCP servers), existing memory files. These are user-specific and must be preserved (add-memory-improve-skills).

4. **Non-destructive migration: create alongside, verify, then delete.** For any file restructuring, always create the new structure first, verify it works, then remove the old. This provides a safe rollback at every step (add-memory-improve-skills).

5. **Cross-reference audits must include docs/ and archive paths.** `docs/integration-plan.md` had 7 stale skill file paths that would have broken documentation. Always grep the full repo, not just `.claude/` (add-memory-improve-skills).

## Process Learnings

- **Were any phases unnecessary?** The Observe phase (Phase 9) was effectively N/A for a Markdown/YAML project with no runtime components. For config-only changes, consider merging Observe into Deploy or skipping with explicit justification.

- **Were any phases missing?** The AI Integration analysis (`/ai-integrate`) was a bonus phase that added genuine value — it prompted the model routing discovery and implementation. Consider making it a standard phase for projects that are themselves AI/LLM systems.

- **What would you do differently?**
  1. Run a more thorough spec check during Research to catch supported YAML fields (would have found `model:` earlier)
  2. Batch Read calls instead of parallel Explore agents for large file sets (avoids rate limits)
  3. Include `~/.claude` environment audit as a standard Deploy pre-check, not just when the user asks

## Patterns to Reuse

- **Pattern: Model Routing via Frontmatter**
  - Where: Any multi-phase workflow with varying complexity per phase
  - Why: Deep reasoning phases (research, plan, implement) get Opus; checklist/template phases (review, security, retro) get Sonnet. ~40-60% cost reduction with no quality loss where it matters.

- **Pattern: ~/.claude Environment Inventory**
  - Where: Any deploy phase that touches Claude Code configuration
  - Why: Prevents accidental deletion of user-specific settings, plugins, MCP servers, and existing memory. Make it a standard deploy pre-check.

- **Pattern: Retro Learning Migration**
  - Where: When initializing a memory system on a project with existing learnings
  - Why: Historical learnings (from global CLAUDE.md or prior sessions) should be migrated into the new memory structure to avoid knowledge loss.

## Anti-Patterns to Avoid

- **Anti-pattern: Removing YAML fields based on incomplete spec knowledge**
  - Why: We removed `model: sonnet` from skills, then had to re-add it. Wasted effort and required a second pass.
  - Instead: Verify against the official Claude Code documentation before making spec compliance decisions. Use `/claude-code-guide` or web search to check.

- **Anti-pattern: Using parallel Explore agents for simple file reads**
  - Why: Two agents hit rate limits simultaneously, delaying the workflow.
  - Instead: Use direct Read calls for known file sets. Reserve Explore agents for open-ended codebase discovery.

## Metrics
- Estimation accuracy: M estimated, M actual (accurate)
- Test coverage delta: N/A (no test framework; manual verification via 13 automated grep/wc checks)
- Review iterations: 0 (APPROVED on first review, 0 blocking issues)
- Security findings: 0 critical, 0 high, 0 medium, 0 low (PASSED)
- Model routing: 6/10 phases on Sonnet, 4/10 on Opus (~40-60% estimated cost savings)
