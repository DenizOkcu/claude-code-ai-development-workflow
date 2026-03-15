# Discovery: add-semantic-retrieval

## Summary
Design and implement a semantic code retrieval layer that allows workflow agents (researching-code, planning-solutions, implementing-code) to automatically retrieve the most relevant parts of a codebase before building prompts. This replaces the current approach of static context (CLAUDE.md) and manual Glob/Grep scanning with an intelligent, embeddings-backed retrieval system.

## Problem Statement
The current workflow relies on two context mechanisms:
1. **Static context**: CLAUDE.md loaded at session start (always present, but generic)
2. **Manual scanning**: Skills like `researching-code` use Glob/Grep/Read to find files (effective for small codebases, but O(n) with codebase size and depends on the agent knowing *what* to search for)

For larger codebases, this leads to:
- **Incomplete context**: The agent misses relevant files it didn't know to search for
- **Wasted context window**: Irrelevant files consume the 200K token budget
- **Slower research phases**: Multiple rounds of Glob/Grep needed to triangulate
- **No semantic understanding**: Keyword search misses conceptually related code (e.g., searching "auth" won't find a rate-limiter that gates authenticated endpoints)

## Success Criteria
- [ ] Repository can be indexed into a queryable knowledge store (embeddings + symbol index)
- [ ] Agents can query the index with natural language and get ranked file/function results
- [ ] Integration point exists in the research skill so it runs automatically before manual Glob/Grep
- [ ] Retrieval results measurably improve context relevance (fewer irrelevant files, more hits on first try)
- [ ] Index updates incrementally on file changes (not full re-index)
- [ ] Works offline / locally (no cloud dependency for core functionality)

## Scope
### In Scope
- Architecture for a repository knowledge index (embeddings + symbol/function index)
- Integration points in the SDLC workflow (primarily research and implement phases)
- CLI tooling or MCP server for indexing and querying
- Incremental index updates (git diff-based)
- Dependency graph extraction for understanding cross-file relationships

### Out of Scope
- Cloud-hosted retrieval service (local-first design)
- Replacing Claude Code's built-in context management
- Modifying Claude Code internals (we work through skills, commands, and MCP)
- Training custom embedding models (use existing models)
- Multi-repository federated search (single-repo focus first)

## Stakeholders
- Users: Developers using this SDLC workflow on medium-to-large codebases (1k+ files)
- Teams: Any team adopting the claude-code-ai-development-workflow toolkit
- Systems: Claude Code runtime, MCP protocol, local embedding inference

## Risk Assessment
**Level:** Medium
**Justification:** The core workflow works without this — it's an enhancement, not a fix. Main risks are: (1) embedding model quality/speed tradeoffs, (2) index staleness if incremental updates fail, (3) added complexity for a toolkit that prizes leanness. No risk to existing functionality since this is additive.

## Dependencies
- Embedding model runtime (e.g., `ollama` with `nomic-embed-text`, or `sentence-transformers` via Python)
- Vector store (e.g., `sqlite-vec`, `hnswlib`, or `chromadb`)
- Tree-sitter for language-aware symbol extraction
- MCP SDK for exposing retrieval as a tool to Claude Code

## Estimated Complexity
**Size:** L
**Reasoning:** Multiple components (indexer, vector store, query engine, MCP server, skill integration), requires understanding of embeddings, tree-sitter parsing, and MCP protocol. However, each component is well-understood individually — the challenge is clean integration with the existing lean architecture.

## Detected Tech Stack

### Languages & Frameworks
| Technology | Version | Expert Command |
|------------|---------|----------------|
| Markdown | — | — |
| YAML (frontmatter) | — | — |
| Shell (bash scripts) | — | — |
| Claude Code Skills/Commands | v1 (folder-based) | `/language/software-engineer-pro` |

### Infrastructure
| Technology | Expert Command |
|------------|----------------|
| Claude Code (CLI) | `/language/software-engineer-pro` |
| MCP Protocol | `/language/software-engineer-pro` |
| Git | — |

### Quality Tooling
| Tool | Status |
|------|--------|
| Linter | ✗ Missing (Markdown/YAML project) |
| Formatter | ✗ Missing |
| Test Runner | ✗ Missing |
| CI/CD | ✗ Missing |
| Pre-commit Hooks | ✗ Missing |

### Missing Quality Tooling Recommendations
This is a Markdown/YAML/Shell toolkit — traditional linting is low-value. However, if code components are added (Python indexer, TypeScript MCP server):
- Run `/quality/lint-setup` after choosing implementation language
- Run `/quality/test-strategy` to set up testing for the retrieval components

### Fallback Expert Commands
- **`/language/software-engineer-pro`** — Architecture, API design, testing patterns
- **`/language/python-pro`** — If Python chosen for indexer (tree-sitter, embeddings)
- **`/language/typescript-pro`** — If TypeScript chosen for MCP server
- **`/ai-integrate`** — Embedding model selection, RAG patterns
