# Discovery: add-repo-context-engine

## Summary
Design and implement a hierarchical repository context system that first exposes a compact repo map to the LLM (Level 1), then retrieves detailed context packs only for files identified as relevant (Level 2). The goal is to dramatically reduce token usage and improve code reasoning accuracy across large repositories by avoiding full-codebase loading. This augments the existing semantic retrieval layer (claude-context MCP) rather than replacing it.

## Problem Statement
The current retrieval system (`@zilliz/claude-context-mcp`) provides semantic search over code chunks but lacks a structural overview. For tasks that require understanding repository topology (e.g., "what is the entry point?", "which modules own this feature?"), the LLM must either blindly search or read many files. This wastes tokens and degrades reasoning quality on large repos. A hierarchical approach — repo map first, detail on demand — mirrors how expert developers reason about unfamiliar codebases.

## Success Criteria
- [ ] A `repo-map` skill/command generates a compact structural overview (file tree + key symbols) for any repo, fitting within ~2K tokens
- [ ] The `researching-code` skill gains a "Level 1 → Level 2" escalation step: use repo map to identify candidate files, then fetch context packs for those files only
- [ ] Context pack retrieval is integrated into the existing `search_code` / `index_codebase` MCP tooling (or a new companion tool)
- [ ] The repo map is generated without requiring a pre-built index (graceful: works in repos where retrieval MCP is not configured)
- [ ] Token budget is predictable: repo map ≤ 2K tokens; context pack per file ≤ 500 tokens

## Scope

### In Scope
- `repo-map` command: generates hierarchical structural overview (file tree + top-level symbols per file)
- Level 1→Level 2 escalation pattern in `researching-code` skill
- Context pack format specification (what to include per file: imports, exports, class/function signatures, docstrings — no implementations)
- Integration with existing retrieval MCP as Level 2 when available
- Graceful fallback via Glob + Grep when MCP is not configured

### Out of Scope
- Building a new MCP server (adopt or extend existing tools)
- IDE/editor integrations
- Real-time file watching / incremental map updates
- Changes to planning artifact format (STATUS.md, RESEARCH.md, etc.)

## Stakeholders
- Users: Developers using `/research` and `/implement` on large codebases
- Teams: Anyone using this SDLC workflow system
- Systems: `researching-code` skill, `implementing-code` skill, `@zilliz/claude-context-mcp`

## Risk Assessment
**Level:** Low
**Justification:** This is purely additive — new skill + new step in existing skill. No existing commands or artifacts are removed. Graceful degradation means no breakage if new feature isn't configured.

## Dependencies
- Existing: `@zilliz/claude-context-mcp` (optional, Level 2 enhancement)
- Existing: `researching-code` skill (will gain new step)
- New: `repo-map` command (`.claude/commands/repo-map.md`)
- Possibly new: `context-pack` command or integrated into `repo-map`

## Estimated Complexity
**Size:** M
**Reasoning:** No new MCP server or runtime code needed. Implementation is entirely in Markdown prompt files. Two files need to be created (`repo-map.md` command), one file needs modification (`researching-code` skill). The main design challenge is the repo map format and the Level 1→Level 2 escalation logic.

---

## What's Already Implemented

### Existing Retrieval (Level 2 equivalent)
The `@zilliz/claude-context-mcp` MCP provides:
- `search_code` — hybrid BM25 + vector search over AST-indexed code chunks
- `index_codebase` — builds/increments the semantic index
- Tree-sitter parsing → semantic chunks (functions, classes, methods)
- Step 0 in `researching-code` already queries this before Glob/Grep

This is already a strong Level 2 (detail retrieval for specific queries). What's **missing** is Level 1: a structural overview that helps identify *which* areas to query.

### What's Missing
| Component | Status |
|-----------|--------|
| Repo map generation (structural overview) | ❌ Not implemented |
| Level 1 → Level 2 escalation pattern | ❌ Not in researching-code skill |
| Context pack format spec | ❌ Not defined |
| `/repo-map` command | ❌ Not present |

---

## Detected Tech Stack

### Languages & Frameworks
| Technology | Version | Expert Command |
|------------|---------|----------------|
| Markdown + YAML frontmatter | — | `/language/software-engineer-pro` |
| Claude Code skills/commands (prompt engineering) | — | `/language/software-engineer-pro` |

### Infrastructure
| Technology | Expert Command |
|------------|----------------|
| Claude Code CLI (skills, commands, agents) | — |
| MCP (Model Context Protocol) servers | — |

### Quality Tooling
| Tool | Status |
|------|--------|
| Linter | ✗ Not applicable (Markdown project) |
| Formatter | ✗ Not applicable |
| Test Runner | ✗ Not applicable |
| CI/CD | ✗ Not configured |
| Pre-commit Hooks | ✗ Not configured |

### Missing Quality Tooling Recommendations
- This is a prompt-engineering project — traditional linting doesn't apply
- Quality is enforced via skill Quality Check checklists (already present)
- Consider `/quality/code-audit` if you add scripting components

### Applicable Expert Commands
- `/language/software-engineer-pro` — design patterns, API design, system thinking
