# Research: add-semantic-retrieval

**What:** Semantic code retrieval layer for Claude Code SDLC workflow
**When:** 2026-03-15

---

## 1. Codebase Analysis

### Relevant Files and Modules

| Path | Purpose | Impact |
|------|---------|--------|
| `.claude/skills/researching-code/SKILL.md` | Research skill — uses Glob/Grep/Read to find 5-15 files | **Primary integration point** — retrieval queries before manual search |
| `.claude/skills/implementing-code/SKILL.md` | Implementation skill — reads PLAN.md and builds code | Secondary — could query index for pattern discovery |
| `.claude/skills/planning-solutions/SKILL.md` | Planning skill — reads RESEARCH.md and defines phases | Secondary — could query index for impact analysis |
| `.claude/agents/sdlc-orchestrator.md` | Orchestrates Research → Plan → Implement → Review | Needs awareness of retrieval availability |
| `.claude/commands/research.md` | `/research` command — invokes researching-code skill | Entry point for retrieval integration |
| `.claude/commands/implement.md` | `/implement` command — invokes implementing-code skill | Secondary entry point |
| `.claude/settings.json` | MCP server configuration (currently: Shannon only) | **Must add retrieval MCP server here** |
| `.claude/scripts/shannon-mcp-wrapper.sh` | OAuth wrapper pattern for MCP servers | Template for a retrieval wrapper script |
| `.claude/ARCHITECTURE.md` | System architecture documentation | Must update with retrieval layer |
| `CLAUDE.md` | Project guidelines, loaded every session | May reference retrieval setup instructions |

### Existing Patterns That Must Be Followed

1. **MCP server integration pattern** (4-step):
   - Command handler (`.claude/commands/{tool}.md`) with pre-condition check
   - Setup wizard (`.claude/commands/{tool}/setup.md`) with hosting options
   - Optional wrapper script (`.claude/scripts/{tool}-mcp-wrapper.sh`)
   - Registration in `.claude/settings.json` under `mcpServers`

2. **Skill structure** (YAML frontmatter with `name`, `description`, `model`, `metadata`):
   - Mindset → Goal → Instructions → Output Format → Quality Check → Common Issues

3. **Lean philosophy**: 5 artifacts max per issue, minimal communication, trust Claude Code's context management

4. **Secrets management**: Environment variables for API keys, `~/.claude/credentials.json` for OAuth tokens, never hardcode in config

### Code Conventions

- **Markdown/YAML** for all skills and commands (no application code yet)
- **kebab-case** for file/directory names
- **Model routing** via `model:` frontmatter field (`opus` for deep reasoning, `sonnet` for checklists)
- **Commands** are thin wrappers — logic lives in skills and agents

### Current Test Patterns

None — this is a Markdown/YAML toolkit. New code components (Python/TypeScript MCP server) will need their own test framework.

---

## 2. Architecture Context

### How the Affected Area Fits

The retrieval layer sits between the **User Interface** (commands) and the **Capabilities Layer** (skills):

```
USER → /research → researching-code skill
                        │
                        ├── [NEW] Query retrieval MCP → ranked results
                        ├── Glob/Grep/Read (existing, still available)
                        └── Output: RESEARCH.md
```

### Current Context Data Flow

1. **Session start**: CLAUDE.md + MEMORY.md loaded automatically
2. **Research phase**: Agent uses Glob → Grep → Read in iterative loops
3. **Planning phase**: Agent reads RESEARCH.md for context
4. **Implementation phase**: Agent reads PLAN.md + RESEARCH.md + source files

**Gap**: Steps 2-4 rely on the agent knowing what to search for. No pre-computed index, no semantic similarity, no cross-file relationship awareness.

### Proposed Data Flow (with Retrieval)

1. **Index build** (one-time + incremental): Parse repo → tree-sitter chunks → embed → store in vector DB
2. **Research phase**: Agent calls retrieval MCP tool with natural language query → gets ranked file/function results → then uses Glob/Grep/Read for deeper inspection
3. **Implementation phase**: Agent queries index for "files similar to X" or "functions that call Y" before making changes

### API Boundaries

The retrieval layer communicates with Claude Code exclusively through **MCP tools**:

| Tool | Input | Output |
|------|-------|--------|
| `semantic_search` | Natural language query + optional filters | Ranked list of code chunks with file paths, scores, snippets |
| `index_status` | None | Index health: last indexed, file count, staleness |
| `reindex` | Optional file paths | Trigger incremental or full re-index |
| `find_related` | File path or symbol name | Related files/symbols by dependency or similarity |

---

## 3. Dependency Analysis

### Internal Dependencies

- `.claude/settings.json` — MCP server registration
- `.claude/commands/` — New command handler and setup wizard
- `.claude/skills/researching-code/SKILL.md` — Updated instructions to use retrieval first
- `.claude/ARCHITECTURE.md` — Updated with retrieval layer

### External Dependencies (Candidates)

| Dependency | Purpose | Maturity | License |
|------------|---------|----------|---------|
| **tree-sitter** + language-pack | AST-based code chunking | Stable, 165+ languages | MIT |
| **nomic-embed-text** (via Ollama) | Local embedding model | Stable, widely used | Apache-2.0 |
| **sqlite-vec** | Vector search (SQLite extension) | Active development, Mozilla-backed | MIT/Apache-2.0 |
| **Ollama** | Local model runtime | Stable, widely adopted | MIT |
| MCP SDK (`@modelcontextprotocol/sdk`) | MCP server framework | Official, stable | MIT |

### Alternative: Adopt Existing MCP Server

| Option | Pros | Cons |
|--------|------|------|
| **Zilliz Claude Context** | Hybrid BM25+vector, AST chunking, 20+ langs, incremental | Requires Milvus/Zilliz Cloud (not local-only) |
| **DeepContext** | Symbol-aware, offline, semantic graph | TypeScript + Python only |
| **Code Pathfinder** | Structural analysis (call graphs), 100% local | Python + Go only, no semantic search |

### Version Constraints

- Ollama: Any recent version (0.3+)
- sqlite-vec: 0.1+ (still pre-1.0 but stable API)
- tree-sitter: 0.20+ for latest grammar format
- Node.js 18+ for MCP server (matches Claude Code runtime)

---

## 4. Integration Points

### Systems Affected

| System | Change | Risk |
|--------|--------|------|
| `.claude/settings.json` | Add retrieval MCP server entry | Low — additive |
| `researching-code` skill | Add "query retrieval first" step | Low — existing steps unchanged |
| `implementing-code` skill | Optional "query for related code" step | Low — additive |
| `.claude/ARCHITECTURE.md` | Document retrieval layer | Low — documentation only |
| `CLAUDE.md` | Add setup instructions reference | Low — documentation only |
| New: `.claude/commands/retrieval/` | Command handler + setup wizard | None — new files |
| New: MCP server (Python or TypeScript) | The actual retrieval engine | Medium — new code component |

### MCP Tool Integration Flow

```
Claude Code Session
    │
    ├── Loads .claude/settings.json
    │       └── mcpServers.retrieval → starts MCP server process
    │
    ├── User runs /research {issue}
    │       └── researching-code skill
    │               ├── Step 0 (NEW): mcp__retrieval__semantic_search("auth middleware changes")
    │               │       └── Returns: [{file: "src/auth/middleware.ts", score: 0.92, chunk: "..."}]
    │               ├── Step 1: Think deeply (existing)
    │               ├── Step 2: Glob/Grep/Read (existing, now informed by retrieval)
    │               └── Step 3: Create RESEARCH.md
    │
    └── User runs /implement {issue}
            └── implementing-code skill
                    ├── Step 0 (NEW): mcp__retrieval__find_related("src/auth/middleware.ts")
                    │       └── Returns: [{file: "src/auth/tests/middleware.test.ts", relation: "test"}]
                    └── Steps 1-5 (existing)
```

### No Frontend/Backend Boundary

This is a CLI tool. The MCP server runs as a child process of Claude Code, communicating via stdio (standard MCP pattern).

---

## 5. Risk Assessment

| Risk | Severity | Likelihood | Mitigation |
|------|----------|------------|------------|
| **Embedding model quality varies by language** | Medium | Medium | Start with nomic-embed-text (general) + test on real repos; upgrade to nomic-embed-code when GGUF available in Ollama |
| **Index staleness after file edits** | Medium | High | Git diff-based incremental re-index on MCP server startup; `reindex` tool for manual trigger |
| **Added complexity violates lean philosophy** | Medium | Medium | Make retrieval 100% optional — workflow works without it; setup wizard pattern matches existing tools |
| **Ollama not installed on user's machine** | Low | Medium | Graceful degradation — setup wizard checks for Ollama; falls back to API-based embeddings or disables retrieval |
| **Large repos overwhelm local embedding** | Medium | Low | Batch indexing with progress reporting; configurable file exclusions (node_modules, vendor, etc.) |
| **sqlite-vec pre-1.0 API instability** | Low | Low | Pin version in package.json; API is small and stable |
| **MCP server startup latency** | Low | Medium | Keep index in SQLite file (instant load); lazy-load embedding model on first query |

---

## 6. Prior Art & Ecosystem Research

### How Similar Problems Are Solved

| Tool | Approach | Key Insight |
|------|----------|-------------|
| **Aider** | Tree-sitter AST → dependency graph → PageRank ranking | No embeddings needed for structural relevance; lightweight |
| **Cursor** | Merkle tree change detection → syntactic chunks → proprietary embeddings → Turbopuffer cloud | Merkle trees for efficient incremental indexing |
| **Continue.dev** | Tree-sitter chunks → embeddings (MiniLM default, Voyage/Ollama optional) → local index | Graceful fallback: whole file if small, chunks if large |
| **Greptile** | Per-function chunks → translate code to natural language → embed descriptions | Embedding NL descriptions > embedding raw code |
| **Sourcegraph** | SCIP compile-time indexing for precise navigation | Compile-time precision unmatched, but heavy setup |

### Best Practices Synthesized

1. **Chunking**: Tree-sitter AST-based, targeting 250-512 tokens per chunk, with file path + class context as metadata
2. **Embedding**: Code-specialized models (nomic-embed-code, voyage-code-3) significantly outperform general-purpose
3. **Search**: Hybrid BM25 + vector similarity consistently beats pure vector
4. **Indexing**: Merkle tree or file hash tracking for incremental updates
5. **Metadata**: Include file path, language, parent scope, imports in chunk metadata
6. **Overlap**: Not beneficial for code chunks (Jan 2026 study)
7. **Greptile insight**: Translate code to NL descriptions before embedding improves retrieval

### Anti-Patterns to Avoid

1. **Line-based chunking**: Breaks mid-function, dilutes embedding quality
2. **General-purpose embeddings only**: Miss code-specific semantics
3. **Full re-index on every change**: O(n) cost makes large repos impractical
4. **No metadata in embeddings**: File path and parent scope are critical for disambiguation
5. **Embedding only, no BM25**: Misses exact keyword matches that are often what the user wants
6. **Cloud dependency for core functionality**: Breaks offline-first requirement

---

## 7. Recommendations

### Recommended Approach: Build-or-Adopt MCP Server

**Option A (Recommended): Adopt + Customize Zilliz Claude Context**

The `@zilliz/claude-context-mcp` package already implements the full pipeline:
- Tree-sitter AST chunking (20+ languages)
- Hybrid BM25 + vector search
- Merkle tree incremental indexing
- Multiple embedding providers (OpenAI, Voyage, Gemini, **Ollama**)
- MCP protocol native

**Customization needed:**
- Configure for Ollama local embeddings (instead of cloud default)
- Add setup wizard (`.claude/commands/retrieval/setup.md`)
- Integrate retrieval step into `researching-code` skill
- Configure to use local sqlite-based storage (Milvus Lite mode)

**Option B: Build Lightweight Custom Server**

If Zilliz Claude Context is too heavy or its Milvus dependency is unwanted:
- TypeScript MCP server using `@modelcontextprotocol/sdk`
- tree-sitter via `tree-sitter-language-pack` (npm) for chunking
- sqlite-vec for vector storage (zero external processes)
- Ollama API for local embeddings
- Custom BM25 implementation (or `orama` for full-text search)

**Option C: Aider-Style Repo Map (No Embeddings)**

Lightest option — no embedding model needed:
- Tree-sitter AST extraction → dependency graph → PageRank ranking
- Returns structurally relevant files based on graph centrality
- No semantic search (keyword + structure only)
- Fastest to implement, lowest resource requirement

### Key Decisions Needed

| Decision | Options | Recommendation |
|----------|---------|----------------|
| **Build vs adopt** | A: Adopt Zilliz Claude Context / B: Build custom / C: Aider-style | A if Milvus Lite works locally; B if lightweight needed |
| **Implementation language** | TypeScript (match MCP SDK) / Python (tree-sitter + ML ecosystem) | TypeScript — matches MCP SDK, node runtime already required |
| **Embedding model** | nomic-embed-text (Ollama, general) / nomic-embed-code (specialized) / voyage-code-3 (API) | nomic-embed-text via Ollama (available now); upgrade path to nomic-embed-code |
| **Vector store** | sqlite-vec / ChromaDB / Milvus Lite | sqlite-vec (zero deps, single file) |
| **Search strategy** | Pure vector / Hybrid BM25+vector | Hybrid — consensus best practice |
| **Chunking strategy** | Line-based / AST-based (tree-sitter) | AST-based with metadata enrichment |

### Unknowns to Resolve Before Design

1. **Milvus Lite viability**: Does Zilliz Claude Context work with Milvus Lite (in-process) or does it require a full Milvus server? If in-process works → Option A is clearly best.
2. **Ollama availability assumption**: What % of target users have Ollama installed? Do we need a fallback that doesn't require an embedding runtime?
3. **Index storage location**: `.claude/index/` (per-project, gitignored) vs `~/.claude/retrieval/` (per-user, shared)?
4. **Re-index trigger**: On MCP server startup only, or also via git hooks / file watchers?
