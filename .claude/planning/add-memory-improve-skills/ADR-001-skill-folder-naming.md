# ADR-001: Skill Folder Naming Convention

## Status: Accepted
## Date: 2026-02-28

## Context

The project has 5 skills as flat `.md` files in `.claude/skills/`. Each has a YAML `name` field that doesn't match its filename:

| `name` field (in frontmatter) | Current filename |
|------------------------------|------------------|
| `researching-code` | `code-research.md` |
| `planning-solutions` | `solution-planning.md` |
| `implementing-code` | `code-implementation.md` |
| `reviewing-code` | `code-review.md` |
| `fixing-review-issues` | `review-fix.md` |

Anthropic's official skill spec requires: "The folder itself must use kebab-case" and "name field should match folder name." We must choose a folder naming convention.

The SDLC Orchestrator agent references skills by their `name` field at 5 locations. The ARCHITECTURE.md and COMMAND_USAGE.md also reference these names.

## Decision

**Use the current `name` fields as folder names:**
- `researching-code/SKILL.md`
- `planning-solutions/SKILL.md`
- `implementing-code/SKILL.md`
- `reviewing-code/SKILL.md`
- `review-fix/SKILL.md` (shortened from `fixing-review-issues` for folder name; `name` field updated to match)

**Exception:** `fixing-review-issues` is renamed to `review-fix` for both folder and `name` field. The current name is verbose (22 chars) and the orchestrator reference is a documentation-only reference, not a programmatic one. Updating 3 lines in the orchestrator is acceptable.

## Alternatives Considered

| Option | Pros | Cons |
|--------|------|------|
| **A: Use `name` fields as-is** (`researching-code/`, `planning-solutions/`, etc.) | Zero changes to orchestrator; names already in use | `fixing-review-issues/` is very long; verb-gerund pattern is unusual for folder names |
| **B: Use current filenames** (`code-research/`, `solution-planning/`, etc.) | Shorter, noun-based; matches common conventions | Requires updating `name` fields in all 5 skills + 4 cross-reference files |
| **C: Hybrid** (use `name` fields but normalize `fixing-review-issues` to `review-fix`) | Minimal changes (only 1 skill rename); cleaner folder names | Small rename in orchestrator + docs |

## Consequences

- **Positive:** Minimal cross-reference updates (only `review-fix` rename touches 3 files)
- **Positive:** Folder names match `name` fields (Anthropic spec compliance)
- **Positive:** Existing orchestrator references for 4/5 skills remain unchanged
- **Negative:** Verb-gerund folder names (`researching-code/`) are unconventional but functional
- **Risks:** If Claude Code enforces exact `name`-to-folder matching, the rename of `fixing-review-issues` to `review-fix` is required
