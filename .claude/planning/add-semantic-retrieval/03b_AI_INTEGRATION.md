# AI Integration: add-semantic-retrieval

**Date:** 2026-03-15
**LLM Component:** Embedding model for code search (inference only, not generative)

---

## 1. Use Case Analysis

| Dimension | Value |
|-----------|-------|
| **Task type** | RAG — embedding generation for semantic code retrieval |
| **Latency** | Near-real-time (< 5s per query); async acceptable for indexing |
| **Accuracy** | Retrieval should surface relevant files in top-5 results for > 80% of queries |
| **Volume** | Low — 5-20 search queries per development session; 1 index build per session |
| **Cost sensitivity** | Zero marginal cost (local Ollama); or ~$0.001/query if using cloud embeddings |

**Key distinction:** This feature uses an **embedding model** (not a generative LLM). The model converts code chunks into vectors for similarity search. There is no text generation, no prompt engineering for the embedding model itself, and no hallucination risk from the retrieval layer.

The generative LLM (Claude) consumes retrieval results as context — but that's the existing skill pipeline, not a new integration.

---

## 2. Model Selection

| Criterion | Requirement | Recommendation |
|-----------|-------------|----------------|
| Task | Code chunk → dense vector embedding | Embedding model, not generative |
| Latency | < 500ms per query; < 5min full index for 5K files | `nomic-embed-text` via Ollama (local) — ~50ms/chunk |
| Cost | Zero for local; < $0.01/query for cloud | Ollama = free; Voyage Code 3 = $0.00006/1K tokens |
| Privacy | Code must not leave the machine (default) | Ollama (local-first); cloud opt-in only |
| Context window | 250-512 tokens per chunk typical | 8192 tokens (nomic-embed-text) — more than sufficient |
| Dimensions | Enough for semantic discrimination | 768 dimensions (nomic-embed-text) |
| Code-specific | Should understand programming constructs | nomic-embed-text is general-purpose; upgrade to nomic-embed-code when available in Ollama |

### Model Comparison

| Model | Local? | Dims | Code Quality | Speed | Cost |
|-------|--------|------|-------------|-------|------|
| **nomic-embed-text** (chosen) | Yes (Ollama) | 768 | Good | Fast (~50ms/chunk) | Free |
| nomic-embed-code | Yes (HF/GGUF) | 768 | Best (SOTA on CodeSearchNet) | Moderate (7B params) | Free |
| voyage-code-3 | No (API) | 1024 | Excellent | Fast | $0.00006/1K tokens |
| text-embedding-3-small | No (API) | 1536 | Good | Fast | $0.00002/1K tokens |

**Decision:** `nomic-embed-text` via Ollama — best balance of local-first, zero-cost, and good-enough quality. Upgrade path to `nomic-embed-code` when Ollama supports it.

---

## 3. Prompt Engineering

### Not Applicable (Embedding Model)

This feature does **not** use prompt engineering in the traditional sense. The embedding model is a pure function: `text → vector`. There is no system prompt, no user prompt template, and no output format to design.

The only "prompt" consideration is the **query prefix** for nomic-embed-text:

```
# For search queries (not for documents):
"search_query: {user's natural language query}"

# For documents being indexed:
"search_document: {code chunk}"
```

This prefixing is handled internally by `@zilliz/claude-context-mcp` — no user or skill configuration needed.

### Agent-Side Context Usage

The LLM (Claude) receives retrieval results as structured data from the MCP tool. The skill instructions in `researching-code` Step 0 guide how Claude interprets results:

```markdown
1. Call `search_code` with a natural language description of the feature/issue
2. Review the ranked results — note the top 5 file paths, their relevance scores, and code snippets
3. Use these results to inform your Glob/Grep searches in Step 2
```

No prompt template needed — Claude uses the results as context for its existing research workflow.

---

## 4. RAG Architecture

### Data Sources
- All source code files in the repository (filtered by `CUSTOM_IGNORE_PATTERNS`)
- Supported languages: TypeScript, JavaScript, Python, Java, C++, C#, Go, Rust, PHP, Ruby, Swift, Kotlin, Scala, Markdown (14 via Tree-sitter)

### Chunking Strategy

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| Method | Tree-sitter AST (symbol boundaries) | Preserves semantic units (functions, classes, methods) |
| Target size | 250-512 tokens | Balances specificity and context |
| Overlap | None | Jan 2026 study found no benefit for code chunks |
| Fallback | LangChain recursive character splitting | For unsupported file types |
| Metadata | File path, language, symbol name, line numbers | Critical for disambiguation and navigation |

### Embedding Model
- **Model:** `nomic-embed-text` (768 dimensions, 8192 token context)
- **Provider:** Ollama (local HTTP API on `127.0.0.1:11434`)
- **Batch size:** 5 chunks (workaround for Ollama dimension detection bug #235)

### Vector Store
- **Technology:** Milvus standalone (Docker, port `127.0.0.1:19530`)
- **Index type:** HNSW (Hierarchical Navigable Small World) — ANN for fast search
- **Collection per project:** `code_chunks_{project_path_hash}`

### Retrieval Strategy

| Parameter | Value |
|-----------|-------|
| Search type | Hybrid: BM25 (keyword) + dense vector (semantic) |
| Top-K | 10 (default, configurable via `limit` parameter) |
| Reranking | Built into claude-context hybrid scoring |
| Extension filter | Optional `.ts`, `.py`, etc. via `extensionFilter` parameter |

### Context Assembly
Results are returned as a ranked array of `{filePath, content, score, startLine, endLine, language, symbolName}`. The agent (Claude) receives these via MCP tool response and decides which files to read in full.

**No context window stuffing** — retrieval returns chunks, not full files. The agent decides what to promote to full-file reads.

---

## 5. Guardrails & Safety

### Input Validation
| Check | Implementation |
|-------|---------------|
| Query length | Handled by Ollama (8192 token context); long queries are truncated |
| Path validation | `search_code` requires absolute path; MCP validates parameter types |
| Ignore patterns | `CUSTOM_IGNORE_PATTERNS` excludes `.git/**`, `node_modules/**`, etc. — prevents indexing secrets in `.git/config` |

### Output Validation
| Check | Implementation |
|-------|---------------|
| Result structure | MCP returns structured JSON; Claude parses it as tool output |
| No code execution | Retrieval returns text only — no executable content |
| No hallucination risk | Embeddings are mathematical transforms; retrieved chunks are exact source code |

### Rate Limiting
- Ollama: `OLLAMA_NUM_PARALLEL=1` limits concurrent requests
- Milvus: No rate limit needed (local, single-user)
- `EMBEDDING_BATCH_SIZE=5` limits memory usage during indexing

### Fallback Behavior

| Scenario | Fallback |
|----------|----------|
| Ollama not running | Skills skip retrieval, use Glob/Grep only |
| Milvus not running | Skills skip retrieval, use Glob/Grep only |
| Not indexed | `/retrieval` suggests running `index_codebase` |
| Search returns 0 results | Agent falls back to Glob/Grep |
| MCP server crash | Claude Code detects child process exit; skills use Glob/Grep |

**No error is ever user-blocking.** Every failure path degrades to the existing workflow.

### Prompt Injection Protection
- **Not applicable for embedding models** — embeddings are a mathematical transform, not instruction-following
- **MCP tool parameters** are structured JSON, not free-text prompts
- **Retrieved code chunks** are displayed as-is (source code); they cannot influence Claude's behavior beyond providing context (same risk as reading any file)

---

## 6. Evaluation & Testing

### Metrics

| Metric | Method | Target |
|--------|--------|--------|
| **Retrieval relevance** | Manual: query known codebase, check if expected files in top-5 | > 80% hit rate |
| **Search latency** | Time MCP tool response | < 500ms (p95) |
| **Index time (incremental)** | Time after modifying 1 file | < 30s |
| **Index time (full, <5K files)** | Time for initial `index_codebase` | < 5 min |
| **Token savings** | Compare context used with/without retrieval | > 30% reduction |
| **False positives** | Irrelevant results in top-10 | < 3 per query |

### Eval Dataset (manual, this repo)

| Query | Expected Top File(s) | Category |
|-------|---------------------|----------|
| "MCP server configuration" | `.claude/settings.json`, `setup.md` files | Happy path |
| "security audit STRIDE" | `.claude/commands/security.md`, `offensive-security/SKILL.md` | Happy path |
| "visual HTML diagram generation" | `visual-explainer/SKILL.md`, `generate-web-diagram.md` | Happy path |
| "how to resume an incomplete workflow" | `sdlc/continue.md`, `sdlc-orchestrator.md` | Happy path |
| "banana" | No relevant results expected | Edge case (irrelevant query) |
| "" (empty string) | Error or empty results | Edge case (empty input) |
| Very long query (>1000 chars) | Truncated, still returns results | Edge case (length) |

---

## 7. Cost Optimization

### Current Architecture (Already Optimized)

| Optimization | Implementation |
|--------------|---------------|
| **Zero marginal cost** | Ollama runs locally — no per-query API charges |
| **Incremental indexing** | Merkle tree detects changed files — avoids re-embedding unchanged code |
| **Batch size control** | `EMBEDDING_BATCH_SIZE=5` prevents memory spikes on large repos |
| **Chunk-level retrieval** | Returns code chunks, not whole files — saves Claude's context window tokens |
| **No caching needed** | Embeddings are stored persistently in Milvus; no re-computation on search |

### If Migrating to Cloud Embeddings

| Model | Cost per 1M tokens | Est. cost to index 5K files (~2M tokens) | Est. cost per search query |
|-------|--------------------|-----------------------------------------|---------------------------|
| nomic-embed-text (Ollama) | $0 | $0 | $0 |
| text-embedding-3-small | $0.02 | $0.04 | ~$0.00002 |
| voyage-code-3 | $0.06 | $0.12 | ~$0.00006 |

**Recommendation:** Stay local (Ollama) for development. Cloud embeddings only make sense for CI/CD pipelines or shared team indexes where Ollama isn't available.

### Model Routing (Future)

If nomic-embed-code becomes available in Ollama:
- Use `nomic-embed-code` for `.ts`, `.py`, `.go`, `.rs`, `.java` (code-heavy files)
- Use `nomic-embed-text` for `.md`, `.yaml`, `.json` (text-heavy files)
- Route by file extension at indexing time
