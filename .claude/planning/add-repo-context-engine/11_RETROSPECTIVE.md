# Retrospective: add-repo-context-engine

## Summary
- **Started:** 2026-03-15
- **Completed:** 2026-03-15
- **Complexity:** M (estimated) vs M (actual) — accurate
- **Files Changed:** 4 (1 created, 3 modified)
- **Tests Added:** N/A (prompt-engineering project)
- **Phases Completed:** 8/10 (Design and Observe skipped as appropriate)

---

## What Went Well

- **User requirement shaped the architecture** — the explicit "users forget to run it" feedback directly determined the most important design decision: embed the map in `/discover` rather than as a standalone step. This avoided a classic discoverability trap.
- **Prior art research paid off** — the ecosystem research on Aider, Cursor, Continue.dev, and RepoGraph took ~30 minutes but saved significant trial-and-error on format, token budget, and symbol extraction approach. The 2K token budget was independently validated by 3 different tools before being written.
- **Glob + Grep over MCP was the right call** — choosing always-available tooling over richer-but-optional MCP makes the feature work on day 1 in any repo, not just indexed ones.
- **3-phase implementation was clean** — building the standalone command first, then integrating it, avoided circular dependencies and gave a clear rollback boundary at each phase.
- **Review caught useful patterns** — the exclusion list duplication issue (finding #3, #13) was a real latent maintenance risk. Surfacing it early during review is better than discovering drift 6 months later.

---

## What Could Be Improved

- **Design phase was skipped** — for an M-sized change this was fine, but the architecture decisions (Level 1/Level 2 split, embedded vs. separate artifact) could have had a formal ADR. The research doc captured this informally, but a `03_ARCHITECTURE.md` would have made the decision record more discoverable.
- **Exclusion list exists in two places** — `repo-map.md` and `discover.md` both define the exclusion list independently. The code review flagged this but it wasn't fixed during implementation. Should be resolved before the first time someone adds a new pattern to one but not the other.
- **Grep patterns weren't tested against real repos** — the language-specific Grep patterns were derived from prior art research but not empirically validated on real codebases. They may over-match (too noisy) or under-match (missing symbols) in edge cases.

---

## Surprises / Unknowns Encountered

- **The "generic fallback" pattern had a Rust-specific term** (`pub fn`) — caught during review. Minor, but shows that pattern composition from language-specific sets needs careful review.
- **CLAUDE.md wasn't updated during implementation** — the deploy plan correctly called this out as a pre-deploy task, but it wasn't caught as a phase 3 acceptance criterion. For workflow tooling projects, CLAUDE.md updates should be part of the implementation phase, not deferred to deploy.

---

## Key Technical Learnings

- **For prompt-engineering projects, the "test" is the prompt itself** — quality gates and quality checks in the skill/command files serve the same function as unit tests: they define what correct behavior looks like and force the author to be explicit about edge cases.
- **Token budget tiering (4 tiers by file count) is more useful than a hard cutoff** — a hard "stop at 2K tokens" loses structural information unpredictably. Progressive degradation (symbols → files → directories → summary) preserves the most useful information at each tier.
- **Hierarchical context loading (Level 1 structural → Level 2 semantic) mirrors how expert developers reason** — they don't search randomly; they first understand the module structure, then dive into relevant areas. Encoding this pattern explicitly into the research skill improves reproducibility.
- **Prompt injection via filesystem content is a low but real risk for any LLM-reads-filesystem feature** — worth documenting and accepting explicitly, not ignoring. The STRIDE analysis surfaced this cleanly.

---

## Process Learnings

- **Design phase can be skipped for M-sized prompt-only changes** — the research + plan combination captured all necessary design decisions. The `/design-system` phase adds value for complex architectural decisions involving multiple systems or data models; it's overkill for pure prompt file changes.
- **Observe phase is not applicable for prompt-only changes** — there are no metrics, dashboards, or alerting to configure. Skip confidently for this category of work.
- **Security phase (7a) is still valuable even for zero-runtime changes** — the STRIDE analysis caught the prompt injection risk and the `01_DISCOVERY.md` tampering risk. Both are low severity but worth documenting as accepted risks. Don't skip security just because there's no code.
- **Deploy plan for prompt-only changes is very lightweight** — Big Bang single commit with a git revert rollback plan is always correct. The value of the deploy plan is the CLAUDE.md update checklist, not the rollout strategy.

---

## Patterns to Reuse

- **Pattern: Embed mandatory tools in existing phase entry points rather than as optional standalone commands**
  - Where: Any feature that users might "forget" to run
  - Why: Discoverability and adoption; users follow the happy path, not optional extras

- **Pattern: Progressive truncation with named tiers for LLM output budget management**
  - Where: Any command that generates variable-length structured output
  - Why: Predictable token usage, graceful degradation, no surprising cutoffs

- **Pattern: Level 1 (structural) → Level 2 (semantic) context loading**
  - Where: Any research or investigation task over an unfamiliar codebase
  - Why: Reduces token waste, mirrors expert developer reasoning, works without MCP

---

## Anti-Patterns to Avoid

- **Anti-pattern: Duplicating exclusion lists across related files**
  - Why: They drift. Someone adds `.turbo/` to `repo-map.md` but forgets `discover.md`.
  - Instead: Reference the canonical source, or deduplicate into a single location

- **Anti-pattern: Deferring CLAUDE.md updates to the deploy phase**
  - Why: They get forgotten or deprioritized
  - Instead: Include CLAUDE.md update as an explicit task in Phase 3 (implementation) for any workflow tooling change

---

## Metrics
- Estimation accuracy: M vs M — accurate
- Review iterations: 1 (APPROVED on first pass)
- Security findings: 0 critical/high/medium, 1 low (accepted)
- Phases skipped appropriately: Design, Observe (2/10)
