# ADR-001: Implement Code Intelligence as Prompt Instructions, Not Runtime Code

## Status: Accepted
## Date: 2026-03-15

## Context

The code intelligence layer needs 5 new capabilities: symbol index, dependency graph, reranking, context pack building, and session caching. Each could be implemented as:
- (A) Prompt instructions within existing Markdown skill files — the LLM executes the procedures using built-in tools
- (B) A new MCP server with runtime code (TypeScript/Python) that computes these artifacts
- (C) A hybrid: some components as prompts, others as runtime

## Decision

Implement all 5 components as **prompt instructions** within existing Markdown skill files. No new MCP server, no runtime code.

## Alternatives Considered

| Option | Pros | Cons |
|--------|------|------|
| **A: All prompt-based** (chosen) | Zero new dependencies; instant availability; works without setup; maintains "prompt engineering only" project nature; graceful degradation built-in | Less precise than AST parsing; depends on LLM reasoning quality; can't do true graph algorithms (PageRank) |
| **B: New MCP server** | Precise AST parsing; real graph algorithms; deterministic results; can cache on disk | New runtime dependency; setup required; breaks "works without MCP" principle; maintenance burden; violates project's prompt-only nature |
| **C: Hybrid** | Best of both — precision where it matters, simplicity where it doesn't | Complexity of two implementation styles; harder to maintain; unclear boundary between prompt and runtime responsibilities |

## Consequences

- **Positive:** No new setup, no new dependencies, no new failure modes. Works immediately on any repo. Maintains the project's identity as a prompt-engineering framework.
- **Positive:** Graceful degradation is trivial — if a prompt procedure doesn't have data, it skips the step.
- **Negative:** Symbol extraction via Grep is ~80% accurate (misses decorators, nested classes, dynamic exports). Acceptable for guiding LLM search.
- **Negative:** Dependency graph via Grep misses dynamic imports, re-exports, barrel files. Acceptable as a heuristic.
- **Risk:** If LLM reasoning quality degrades under heavy context, the reranking step may not be reliable. Mitigation: keep the scoring rubric simple (3 factors) and don't rerank small result sets (≤5).

## References
- Research finding: "Grep-based symbol extraction is the right level of fidelity for prompt-engineering"
- Prior art: ctags/etags use regex patterns successfully for decades
- Learning from add-semantic-retrieval: "The implicit feature flag pattern (MCP config presence) is the right default for optional enhancements"
