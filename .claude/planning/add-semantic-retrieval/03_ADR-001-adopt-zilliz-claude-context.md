# ADR-001: Adopt Zilliz Claude Context as Retrieval Engine

## Status: Accepted
## Date: 2026-03-15

## Context

The SDLC workflow needs a semantic code retrieval layer to help agents find relevant code in large repositories. Three options were evaluated: (A) adopt `@zilliz/claude-context-mcp`, (B) build a custom TypeScript MCP server, (C) implement Aider-style repo mapping without embeddings.

The user explicitly decided to adopt Zilliz Claude Context.

## Decision

Adopt `@zilliz/claude-context-mcp@0.1.6` as the retrieval engine, configured with:
- **Embedding:** Ollama + `nomic-embed-text` (local, no cloud API required)
- **Storage:** Milvus running as a Docker container (port 19530)
- **Search:** Hybrid BM25 + dense vector (the package's default mode)
- **Chunking:** AST-based via Tree-sitter (the package's default splitter)

## Alternatives Considered

| Option | Pros | Cons |
|--------|------|------|
| **A: Zilliz Claude Context (chosen)** | Full pipeline (AST chunking, hybrid search, Merkle tree indexing, 14+ langs); MCP-native; actively maintained by Milvus team | Requires Docker Milvus (no in-process SQLite mode); Ollama batch bug (#235) needs workaround |
| **B: Custom TypeScript MCP server** | Full control; sqlite-vec = zero external processes; lighter weight | 2-4 weeks build time; reinvents AST chunking, hybrid search, incremental indexing |
| **C: Aider-style repo map** | No embedding model needed; lightest weight; instant startup | No semantic search (structural only); misses conceptually related code |

## Consequences

### Positive
- **Immediate value**: Working hybrid search with AST chunking, no custom code to build or maintain
- **Incremental indexing**: Merkle tree change detection is already implemented
- **Multi-provider flexibility**: Can switch from Ollama to Voyage/OpenAI by changing env vars
- **Community maintained**: Active project by Zilliz (Milvus creators), issues resolved promptly

### Negative
- **Docker dependency**: Milvus requires Docker — adds a prerequisite for users
- **Resource overhead**: Milvus (200-500MB RAM) + Ollama (1-1.5GB RAM) running simultaneously
- **Ollama batch bug**: Must set `EMBEDDING_BATCH_SIZE=5` and `EMBEDDING_DIMENSION=768` to work around issue #235
- **Node.js 20+ required**: Package requires Node.js >=20.0.0 and <24.0.0

### Risks
- **Milvus Lite gap**: No in-process mode for Node.js; Docker is the minimum for local operation
- **Package version < 1.0**: API may change; pin version in configuration
- **Ollama availability**: Not all users will have Ollama installed — setup wizard must handle this

## References
- [GitHub: zilliztech/claude-context](https://github.com/zilliztech/claude-context)
- [NPM: @zilliz/claude-context-mcp](https://www.npmjs.com/package/@zilliz/claude-context-mcp)
- [Issue #235: Ollama dimension detection bug](https://github.com/zilliztech/claude-context/issues/235)
- [Issue #354: Milvus Lite Node.js not supported](https://github.com/milvus-io/milvus-sdk-node/issues/354)
- Research: `.claude/planning/add-semantic-retrieval/02_CODE_RESEARCH.md`
