# ADR-002: Two-Tier Memory System Design

## Status: Accepted
## Date: 2026-02-28

## Context

The SDLC workflow generates valuable knowledge during retrospectives (`/retro`) and across sessions, but nothing persists beyond the `CLAUDE.md ## Learnings` section. Each new Claude Code conversation starts with no memory of previous sessions' patterns, decisions, or debugging insights.

Claude Code provides a built-in auto-memory feature: a `memory/` directory under `~/.claude/projects/{project-hash}/` where `MEMORY.md` is auto-loaded into every conversation (first 200 lines), with topic files loaded on demand.

## Decision

**Implement a two-tier memory system:**

**Tier 1 -- Repo-shared learnings (existing, in git):**
- `CLAUDE.md ## Learnings` section
- Specific, actionable lessons written by `/retro`
- Visible to all team members who use the repo
- Permanent, versioned in git

**Tier 2 -- Project-personal memory (new, auto-memory directory):**
- `~/.claude/projects/{hash}/memory/MEMORY.md` (index, max 200 lines, auto-loaded)
- `memory/patterns.md` (stable workflow patterns)
- `memory/decisions.md` (architectural decisions)
- `memory/learnings.md` (accumulated lessons, detailed)

**Integration:**
- `/retro` writes to BOTH tiers
- New sessions get CLAUDE.md (repo) + MEMORY.md (personal) automatically
- Topic files (patterns.md, decisions.md, learnings.md) loaded only when referenced

## Alternatives Considered

| Option | Pros | Cons |
|--------|------|------|
| **A: CLAUDE.md only** (current) | Simple; shared via git | No personal context; learnings section grows unbounded; no structured topics |
| **B: Auto-memory only** | Per-user; structured; auto-loaded | Not shared via git; lost if user switches machines; duplicates CLAUDE.md role |
| **C: Two-tier (chosen)** | Best of both; shared + personal; structured topics | Two places to write; slightly more complex /retro logic |
| **D: Dedicated memory skill** | Self-contained; could auto-trigger | Over-engineered for this use case; adds complexity |

## Consequences

- **Positive:** Cross-session knowledge retention without sacrificing team sharing
- **Positive:** MEMORY.md auto-loaded means zero friction -- knowledge is just "there"
- **Positive:** Topic files (patterns, decisions, learnings) provide structured access
- **Positive:** CLAUDE.md learnings remain the primary shared mechanism; auto-memory supplements
- **Negative:** `/retro` command becomes slightly more complex (writes to two places)
- **Negative:** Users must understand two memory locations
- **Risks:** MEMORY.md could bloat if not pruned; mitigated by 200-line hard cap with pruning strategy

## References

- Claude Code auto-memory documentation
- Anthropic's "The Complete Guide to Building Skills for Claude" (2026), Chapter 5: Patterns
