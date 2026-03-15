# Project Spec: add-semantic-retrieval

## Technical Requirements

Derived from 01_DISCOVERY.md success criteria:

| ID | Requirement | Acceptance Criteria |
|----|-------------|-------------------|
| TR-1 | Repository indexing into queryable store | `index_codebase` creates Milvus collection with AST chunks + embeddings; `get_indexing_status` returns `indexed: true` |
| TR-2 | Natural language code search | `search_code` returns ranked results with file paths, scores, and code snippets for any natural language query |
| TR-3 | Research skill auto-retrieval | `researching-code` SKILL.md includes a step to call `search_code` before manual Glob/Grep |
| TR-4 | Incremental index updates | Second `index_codebase` call after modifying 1 file re-indexes only that file (Merkle tree) |
| TR-5 | Local-first operation | Full pipeline works with Ollama + Docker Milvus, no internet required |
| TR-6 | Setup wizard | `/retrieval/setup` guides user through Ollama, Milvus, and MCP configuration |

## Non-Functional Requirements

| NFR | Target | Measurement |
|-----|--------|-------------|
| Search latency | < 500ms per query | Time from `search_code` call to results returned |
| Index time (incremental) | < 2 min for 10 changed files | Time for `index_codebase` with Merkle tree optimization |
| Index time (full, <5K files) | < 5 min | First `index_codebase` call |
| Memory overhead (Milvus) | < 500MB | Docker stats |
| Memory overhead (Ollama) | < 1.5GB | `ollama ps` |
| Disk per project | < 500MB for 5K-file codebase | Milvus volume size |
| Graceful degradation | 0 workflow breakage when retrieval unavailable | All skills work without retrieval MCP; error messages guide user to fix |
| Startup time | < 5s for MCP server ready | Time from Claude Code launch to first tool available |

## Interface Contracts

### MCP Server Configuration (settings.json)

```json
{
  "mcpServers": {
    "claude-context": {
      "command": "npx",
      "args": ["-y", "@zilliz/claude-context-mcp@0.1.6"],
      "env": {
        "EMBEDDING_PROVIDER": "Ollama",
        "EMBEDDING_MODEL": "nomic-embed-text",
        "EMBEDDING_DIMENSION": "768",
        "EMBEDDING_BATCH_SIZE": "5",
        "OLLAMA_HOST": "http://127.0.0.1:11434",
        "OLLAMA_NUM_PARALLEL": "1",
        "MILVUS_ADDRESS": "127.0.0.1:19530",
        "SPLITTER_TYPE": "ast",
        "HYBRID_MODE": "true",
        "CUSTOM_IGNORE_PATTERNS": "node_modules/**,.git/**,vendor/**,dist/**,build/**,.next/**,__pycache__/**,*.pyc,.terraform/**"
      }
    }
  }
}
```

### MCP Tool Interfaces

```typescript
// search_code — Primary search tool
interface SearchCodeInput {
  path: string;                // Absolute path to indexed codebase
  query: string;               // Natural language query
  limit?: number;              // Default: 10
  extensionFilter?: string[];  // e.g., [".ts", ".py"]
}

interface SearchCodeResult {
  filePath: string;
  content: string;
  score: number;
  startLine: number;
  endLine: number;
  language: string;
  symbolName?: string;
}

// index_codebase — Build/update index
interface IndexCodebaseInput {
  path: string;
  ignorePatterns?: string[];
  force?: boolean;             // Default: false
  splitter?: "ast" | "langchain"; // Default: "ast"
  customExtensions?: string[];
}

// get_indexing_status — Check index health
interface IndexingStatusInput {
  path: string;
}

interface IndexingStatusResult {
  indexed: boolean;
  progress?: number;           // 0-100
  lastIndexed?: string;        // ISO 8601
  fileCount?: number;
  chunkCount?: number;
}

// clear_index — Remove index
interface ClearIndexInput {
  path: string;
}
```

### Skill Integration Interface

The `researching-code` skill gains a new Step 0 that follows this pattern:

```markdown
### Step 0: Semantic Retrieval (if available)

If the `claude-context` MCP server is configured, query it before manual search:

1. Call `search_code` with a natural language description of what you're looking for
2. Review the ranked results — note the top 5 file paths and their relevance
3. Use these results to inform your Glob/Grep searches in Step 2

If `claude-context` is not configured or returns an error, skip this step
and proceed with Step 1 as before. Retrieval is an enhancement, not a requirement.
```

## Testing Requirements

| Test Category | What to Test | How |
|---------------|-------------|-----|
| **Setup wizard** | Walks through all 3 setup paths (local, cloud, skip) | Manual walkthrough |
| **MCP configuration** | settings.json is valid after setup; Claude Code loads server | Restart Claude Code, verify tools available |
| **Indexing** | `index_codebase` on a sample repo produces correct chunk count | Call tool, verify `get_indexing_status` |
| **Search quality** | `search_code("authentication middleware")` returns auth-related files | Call tool on known codebase, verify results contain expected files |
| **Incremental update** | Modify 1 file, re-index, verify only that file processed | Compare indexing duration: full vs incremental |
| **Graceful degradation** | Skills work when Milvus/Ollama are down | Stop Docker, run `/research`, verify Glob/Grep still works |
| **Skill integration** | `researching-code` uses retrieval when available | Run `/research` with retrieval configured, check RESEARCH.md references retrieval results |

## Migration Plan

No data migration needed — this is a net-new capability. Existing workflows are unchanged.

### Rollout Steps
1. Add configuration files (commands, setup wizard)
2. Update skills with optional retrieval step
3. Update ARCHITECTURE.md and CLAUDE.md
4. User runs `/retrieval/setup` to opt in

### Feature Flag Strategy

**Implicit feature flag via MCP configuration**: The retrieval layer is active if and only if `claude-context` appears in `.claude/settings.json` under `mcpServers`. Skills check for tool availability before attempting retrieval calls. No code-level feature flags needed.

- **Not configured** → Skills behave exactly as before (Glob/Grep/Read only)
- **Configured but services down** → Skills log a warning and fall back to Glob/Grep/Read
- **Configured and running** → Skills use retrieval first, then Glob/Grep/Read for deeper exploration

## Rollback Plan

### If retrieval causes issues:

1. **Quick disable** (seconds): Remove `claude-context` from `.claude/settings.json`, restart Claude Code
2. **Revert skill changes** (seconds): `git checkout .claude/skills/researching-code/SKILL.md .claude/skills/implementing-code/SKILL.md`
3. **Clean up Docker** (optional): `docker stop milvus && docker rm milvus`
4. **Remove command files** (optional): `rm -rf .claude/commands/retrieval/ .claude/commands/retrieval.md`

### If index is corrupted:
1. Call `clear_index` tool with the project path
2. Or: `docker exec milvus milvus run clean` to wipe all collections
3. Re-run `index_codebase` to rebuild from scratch

### Impact of rollback:
- Zero impact on existing workflow — retrieval is purely additive
- No data loss — no existing files or indexes are modified by the feature
- Skills revert to their pre-retrieval behavior (Glob/Grep/Read only)
