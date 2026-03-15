# ADR-002: Use Context Window as Session Cache Instead of File-Based Cache

## Status: Accepted
## Date: 2026-03-15

## Context

The code intelligence layer produces intermediate state (dependency graph, reranked results, context pack) that needs to persist within a research session but not across sessions. Options:
- (A) Use Claude Code's context window as the implicit session cache
- (B) Write intermediate state to `.claude/cache/{issue}/` as Markdown files
- (C) Write to planning artifacts (e.g., `02b_DEPENDENCY_GRAPH.md`)

## Decision

Use Claude Code's **context window as the session cache** for ephemeral state (dependency graph, reranked list, context pack). Use `01_DISCOVERY.md` for cross-session persistence (repo map, symbol index).

## Alternatives Considered

| Option | Pros | Cons |
|--------|------|------|
| **A: Context window** (chosen) | Zero file I/O; no cleanup needed; no new artifacts; fastest | Lost when session ends; not inspectable by user; relies on context window size |
| **B: `.claude/cache/` directory** | Inspectable; survives session restarts; explicit | Extra file I/O every research run; requires cleanup logic; cache invalidation is hard; new directory to manage |
| **C: Planning artifacts** | Visible in git; part of audit trail | Violates "minimal artifacts" (5 files); clutters planning directory; data changes every run but would be committed |

## Consequences

- **Positive:** Zero additional files or directories. No cache invalidation logic. No cleanup.
- **Positive:** Aligns with design principle "Trust Claude Code" — the context window is Claude Code's built-in state management.
- **Negative:** Dependency graph is lost between sessions. Mitigation: it's cheap to rebuild (3-5 Grep calls) and changes per research run anyway.
- **Negative:** User can't inspect intermediate reasoning. Mitigation: the final output (RESEARCH.md) captures the conclusions; the intermediate steps are reasoning, not artifacts.
- **Risk:** If context window is compressed (long conversations), intermediate state may be lost mid-session. Mitigation: the context pack (8 files) is the critical output — even if graph details are compressed, the file list persists as the immediate input to Steps 1-2.

## References
- ARCHITECTURE.md: "Trust Claude Code — Claude Code handles context management (200K window)"
- Learning from optimize-token-usage: "'Always-on' vs 'on-demand' is the key split"
