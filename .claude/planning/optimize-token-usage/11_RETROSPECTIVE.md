# Retrospective: optimize-token-usage

## Summary
- **Started:** 2026-03-15
- **Completed:** 2026-03-15
- **Complexity:** S — analysis + file restructuring (no code, pure prompt engineering)
- **Files Changed:** 6 modified, 2 created (`CLAUDE.md`, `~/CLAUDE.md`, `.claude/LEARNINGS.md`, `.claude/QUICK_REFERENCE.md`, `.claude/commands/retro.md`, `README.md`)
- **Tests Added:** 0 (no runtime code)
- **Phases Completed:** Research → Implement (no formal SDLC, direct implementation after analysis)

## What Went Well

- **Root cause was clear immediately.** A line count comparison revealed the problem: two nearly-identical 554-line CLAUDE.md files loaded every conversation. No ambiguity about what to fix.
- **Tiered architecture was the right mental model.** Separating "always-on rules" from "reference material" maps cleanly to how Claude Code actually loads files. The result (CLAUDE.md as rules, `.claude/LEARNINGS.md` as archive, `.claude/QUICK_REFERENCE.md` as cheat sheet) is intuitive and maintainable.
- **`/retro` update was a natural fit.** The retro command already wrote to CLAUDE.md — extending it to also write the full version to LEARNINGS.md required minimal changes to the command logic.
- **README update covered the new structure clearly.** Adding a dedicated "Memory & Token Optimization" section with a comparison table makes the architecture self-documenting.

## What Could Be Improved

- **No formal SDLC tracking for this change.** The work was done conversationally without a `00_STATUS.md`. For workflow-tooling changes, even small ones, a lightweight status file helps with traceability — especially because the `/retro` command itself relies on one.
- **~/CLAUDE.md changes are outside git.** The global `~/CLAUDE.md` reduction (~452 lines removed) is not tracked in this repo. Any collaborator who installs this workflow won't benefit from the slimmed global file unless they manually update their own.
- **The Behavioral Guidelines section in project CLAUDE.md still says "Learnings go to CLAUDE.md AND both CLAUDE.md files"** — imprecise after the restructuring. The intent is LEARNINGS.md (full) + CLAUDE.md (abbreviated 2 blocks).

## Surprises / Unknowns Encountered

- **Both CLAUDE.md files were nearly identical.** Expected some divergence (project-specific vs global), but 95% of content was duplicated verbatim. The global file had extra n8n learnings from other projects; the project file had semantic retrieval additions. Both loaded every conversation in this repo.
- **The slide deck needed no changes.** The three references to CLAUDE.md in the deck (`"writes lessons to CLAUDE.md"`) are high-level enough to remain accurate — the abbreviated learnings still go to CLAUDE.md.

## Key Technical Learnings

- **Measure token cost before optimizing.** Line count × ~3.5 gives a rough token estimate for markdown. The two CLAUDE.md files together were 1,108 lines ≈ 18K tokens, loaded on every conversation turn. That's the baseline to beat.
- **"Always-on" vs "on-demand" is the key split for CLAUDE.md content.** Rules that govern behavior belong always-on. Reference material (cheat sheets, historical learnings, command catalogs) should be on-demand — readable by commands when needed, not injected into every conversation.
- **Deduplication of global + project CLAUDE.md is a one-time win with compounding savings.** Every conversation in a repo that has both files loaded pays the duplication tax. Fixing it once reduces cost for all future sessions permanently.
- **The `/retro` rotation rule prevents CLAUDE.md bloat from recurring.** The constraint "keep only 2 most recent retro blocks" means the file never grows past ~120 lines in the Learnings section, regardless of how many retros are run.

## Process Learnings

- **The right workflow for prompt-only SDLC-tooling changes is: analyze → implement directly → retro.** Research and Design phases add overhead without value when the change is a file restructuring with no external dependencies. The Security phase (7a STRIDE) is still worth a mental pass even for prompt-only changes.
- **Token optimization analysis doesn't need a `/discover` phase** — a `wc -l` and a diff is sufficient scoping. The investigation took 3 tool calls; full SDLC would have taken 30+.
- **Retro should be run even for conversational (non-SDLC) changes.** Lessons from ad-hoc improvements are as valuable as lessons from formal SDLC runs. The `/retro` command should be invoked at the end of any session with meaningful changes.

## Patterns to Reuse

- **Pattern: Tiered CLAUDE.md (always-on + on-demand)**
  - Where: Any project using Claude Code with growing CLAUDE.md files
  - Why: Prevents token budget bleed from accumulating reference material and historical learnings. Keeps always-on context under ~300 tokens for project CLAUDE.md.

- **Pattern: `/retro` rotation rule for learnings**
  - Where: The retro command's "Update CLAUDE.md" step
  - Why: A hard cap of "2 most recent blocks" prevents unbounded growth without losing history (full archive stays in LEARNINGS.md).

- **Pattern: Separate global vs project CLAUDE.md responsibilities**
  - Where: Multi-repo environments with a `~/CLAUDE.md`
  - Why: Global file = universal behavioral rules (security, naming, repo type guidelines). Project file = project-specific workflow only. Zero overlap.

## Anti-Patterns to Avoid

- **Anti-pattern: Duplicating content between global and project CLAUDE.md**
  - Why: Both files are loaded every conversation in that project. Duplication doubles the token cost of every turn.
  - Instead: Global file owns shared rules; project file overrides/adds project-specific content only.

- **Anti-pattern: Appending full-detail learnings directly to CLAUDE.md**
  - Why: Each `/retro` run adds ~5-10 lines of dense text. After 10 retros, the Learnings section alone is 80+ lines loaded every conversation.
  - Instead: Full learnings → `.claude/LEARNINGS.md`. Abbreviated 1-liners → CLAUDE.md (max 2 blocks, rotate on each retro).

- **Anti-pattern: Keeping quick-reference command cheat sheets in CLAUDE.md**
  - Why: Terraform/kubectl/ansible/docker snippets are consulted rarely but loaded constantly. Pure waste.
  - Instead: Move to `.claude/QUICK_REFERENCE.md`; reference it with a one-liner pointer in CLAUDE.md.

## Metrics
- Estimation accuracy: ~10 min estimated, ~15 min actual (README update took longer than expected)
- Test coverage delta: N/A (no runtime code)
- Review iterations: 0 (no code review needed for prompt restructuring)
- Security findings: 0
- Token savings: ~17K tokens/conversation (1,108 lines → 189 lines across both CLAUDE.md files)
