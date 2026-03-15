# Research: add-code-intelligence-layer

**What:** Upgrade the hierarchical context engine with symbol index, dependency graph, reranking, context pack builder, and session cache
**When:** 2026-03-15

---

## Summary

- **Risk:** Medium (design complexity, not implementation risk)
- **Approach:** Extend existing `researching-code` skill with 3 new steps; extend `repo-map` with symbol index output; add session cache as lightweight file storage
- **Effort:** Moderate — 3 files modified, 0–1 files created, all Markdown prompt engineering

---

## 1. Codebase Analysis

### Files to Modify

| File | Purpose | Change |
|------|---------|--------|
| `.claude/skills/researching-code/SKILL.md` | Research skill with Level 1→2 context loading | **Major**: Add Steps 0b (dependency graph), 0c (reranking), 0d (context pack builder); enhance Step 0 with symbol index awareness |
| `.claude/commands/repo-map.md` | Standalone repo map generator | **Moderate**: Extend symbol extraction to output structured symbol index (not just inline names) |
| `.claude/ARCHITECTURE.md` | System architecture documentation | **Moderate**: Add Code Intelligence Layer section, update Context Engine diagram |
| `.claude/commands/discover.md` | Discovery phase (auto-generates repo map) | **Minor**: Cross-reference updated repo-map format; ensure discovery embeds symbol index |
| `.claude/skills/implementing-code/SKILL.md` | Implementation skill | **Minor**: Add optional dependency graph query in Step 1 for impact analysis |

### Files Potentially Created

| File | Purpose | Rationale |
|------|---------|-----------|
| `.claude/skills/researching-code/SCHEMAS.md` | Reference schemas for symbol index + dependency graph | Optional — may inline in SKILL.md if compact enough |

### Existing Patterns to Follow

**1. Hierarchical context loading (researching-code Step 0):**
```
Level 1: Read repo map from 01_DISCOVERY.md → identify candidate files
Level 2: search_code MCP or Grep/Read → fetch detail for candidates
```
New components slot between Level 1 and Level 2 (symbol index, dependency graph) and after Level 2 (reranking, context pack builder).

**2. Graceful degradation (from retrieval integration):**
- Check if tool/data is available → use if present → skip silently if not
- Every new component must degrade to existing Glob/Grep behavior

**3. Step 0 pattern (from semantic retrieval integration):**
- Step 0 runs before the main research steps
- Enhances subsequent steps with better-targeted context
- Never blocks if unavailable

**4. Token budget constraint:**
- Repo map: ≤2K tokens
- Symbol index: must fit within the same or adjacent budget
- Context packs: configurable per-task budget (recommend ≤4K tokens for assembled context)

---

## 2. Architecture Context

### Where the New Components Fit

```
/discover (Phase 1)
    ├── Step 2: Detect stack
    ├── Step 3: Generate repo map + symbol index ← EXTENDED
    ├── Step 4: Create 01_DISCOVERY.md (includes map + symbol index)
    │
    ▼
/research (Phase 2) — researching-code skill
    ├── Step 0a: Level 1 — Read repo map + symbol index     ← EXISTING (enhanced)
    ├── Step 0b: Build dependency graph for candidates        ← NEW
    ├── Step 0c: Level 2 — Targeted search (MCP or Grep)    ← EXISTING
    ├── Step 0d: Rerank results against task description      ← NEW
    ├── Step 0e: Build context pack (seed + deps + tests)     ← NEW
    ├── Step 1: Think deeply (now with richer context)
    ├── Step 2: Quick discovery (guided by context pack)
    │
    ▼
/implement (Phase 5)
    ├── Step 1: Read plan + optionally query dependency graph for impact ← MINOR
```

### Data Flow: New Components

```
01_DISCOVERY.md
    ├── ## Repository Map (file tree + inline symbols)        [existing]
    └── ## Symbol Index (structured: name, type, file, line)  [new]
            │
            ▼
    Step 0a: Parse symbol index → identify relevant symbols
            │
            ▼
    Step 0b: For each relevant file, Grep import/export statements
             → Build local dependency graph (imports, exports, test files)
            │
            ▼
    Step 0c: search_code MCP (or Grep) → raw results
            │
            ▼
    Step 0d: Score each result against:
             (a) task keyword overlap (BM25-style)
             (b) dependency proximity (graph distance from seed files)
             (c) file-type priority (source > test > config > doc)
             → Re-order results by composite score
            │
            ▼
    Step 0e: Take top-N files from reranked results
             → Expand via dependency graph: add imported modules, callers, test files
             → Trim to token budget (≤4K tokens of assembled context)
             → This is the "context pack" used by Steps 1-2
```

### Session Cache Flow

```
Session Start
    │
    ├── Check .claude/planning/{issue}/01_DISCOVERY.md
    │   ├── Has ## Repository Map? → Load (skip regeneration)
    │   └── Has ## Symbol Index? → Load (skip re-extraction)
    │
    └── No discovery doc? → Generate on-the-fly (existing fallback)

Between Phases (within same session)
    │
    └── Dependency graph from Step 0b is held in-context
        (no explicit file cache needed — Claude Code's context window IS the cache)
```

**Key insight: Claude Code's context window is the session cache.** Data computed in `/research` is available in subsequent tool calls within the same session. We don't need a file-based cache for intra-session state. The `01_DISCOVERY.md` serves as the cross-session cache (repo map + symbol index persist between conversations).

---

## 3. Dependency Analysis

### Internal Dependencies

| Module | Dependency Type | Notes |
|--------|----------------|-------|
| `researching-code/SKILL.md` | **Major modification** | Core integration point for all 5 components |
| `repo-map.md` | **Moderate modification** | Gains symbol index output section |
| `discover.md` | **Minor modification** | Cross-reference updated repo-map format |
| `implementing-code/SKILL.md` | **Minor modification** | Optional dependency graph query |
| `ARCHITECTURE.md` | **Documentation** | New Code Intelligence Layer section |
| `01_DISCOVERY.md` template | **Extended** | Gains `## Symbol Index` section |

### External Dependencies

| Dependency | Status | Required? |
|------------|--------|-----------|
| `claude-context` MCP (`search_code`) | Configured in settings.json | **No** — Level 2 falls back to Grep |
| Glob/Grep/Read (Claude Code built-in) | Always available | **Yes** — powers symbol extraction, dependency graph building |
| Ollama + Milvus (via claude-context) | Optional | **No** — only needed for MCP search path |

### No New External Dependencies Required

All 5 new components are implemented as **prompt instructions** that use existing Claude Code built-in tools (Glob, Grep, Read). The symbol index is populated by Grep. The dependency graph is built by Grep on import statements. Reranking is a scoring procedure in the prompt. The context pack builder is an assembly procedure. The session cache leverages `01_DISCOVERY.md` (cross-session) and Claude Code's context window (intra-session).

---

## 4. Integration Points

### Systems Affected

| System | Impact | Backward Compatible? |
|--------|--------|---------------------|
| `researching-code` Step 0 | Extended from 2 sub-steps to 5 | **Yes** — new steps only run if data available; existing Level 1→2 unchanged |
| `repo-map.md` output format | Gains structured symbol index section | **Yes** — existing compact tree format preserved; new section appended |
| `01_DISCOVERY.md` template | Gains `## Symbol Index` section | **Yes** — old discoveries without it still work |
| `implementing-code` Step 1 | Optional dependency graph query | **Yes** — purely additive |
| `ARCHITECTURE.md` | Documentation update | N/A |

### No Breaking Changes

Every integration is additive. The core invariant holds: **if no symbol index exists, if no dependency graph is built, if reranking data is insufficient → fall back to existing Level 1 repo map + Level 2 search/Grep behavior.**

---

## 5. Risk Assessment

| Risk | Severity | Likelihood | Mitigation |
|------|----------|------------|------------|
| **Symbol index bloats 01_DISCOVERY.md beyond token budget** | Medium | Medium | Define a symbol budget (≤1K tokens / ~4K chars). For large repos, index only primary source directories. Progressive truncation tiers (same as repo map). |
| **Dependency graph extraction via Grep is language-dependent** | Medium | High | Define import patterns per language (JS: `import.*from`, Python: `^import\|^from.*import`, Go: import block). Accept imperfect extraction — the graph is a heuristic, not a compiler. |
| **Reranking adds complexity without measurable improvement** | Low | Medium | Keep scoring simple: 3 factors (keyword overlap, graph proximity, file-type priority). No ML, no embeddings. If the procedure is more than 10 lines in the prompt, it's too complex. |
| **Steps 0b-0e make researching-code too long/complex** | Medium | Medium | Each new step is 5-10 lines of instruction. Use conditional execution: "If symbol index available, do X; otherwise skip." Total skill stays under 200 lines. |
| **Context pack over-expands (includes too many files)** | Low | Medium | Hard cap: context pack ≤ 8 files. Expansion budget: seed files + max 3 dependency expansions + max 2 test files. |
| **Dependency graph is misleading for dynamic languages** | Low | High | Label the graph as "static import analysis — may miss dynamic requires/imports." Acceptable for the 80% case. |

---

## 6. Prior Art & Ecosystem Research

### Symbol Indexing Approaches

| Tool | Approach | Key Insight |
|------|----------|-------------|
| **Aider** | Tree-sitter AST → symbols with full signatures | Signatures > names for context; but requires Tree-sitter runtime |
| **ctags/etags** | Regex-based symbol extraction | Grep patterns per language work for 80% of cases without AST |
| **LSP (Language Server Protocol)** | Full semantic analysis → symbols, references, definitions | Gold standard but requires per-language server; too heavy for prompts |
| **Our repo-map** | Grep with language patterns → top-level symbols | Already working; extend to output structured format (name, type, file, line) |

**Takeaway:** Grep-based symbol extraction is the right level of fidelity for prompt-engineering. We don't need AST precision — we need "good enough" symbols to guide the LLM's search.

### Dependency Graph Approaches

| Tool | Approach | Key Insight |
|------|----------|-------------|
| **Aider** | Tree-sitter reference graph → PageRank | Graph centrality identifies "important" files; but requires full parse |
| **madge** (JS/TS) | Static import analysis → dependency tree | Import statement parsing is sufficient for module-level deps |
| **pydeps** (Python) | AST import analysis → module graph | Same pattern — import statements are the primary dependency signal |
| **Code Pathfinder MCP** | AST call graphs for Python/Go | Full call graph is too expensive; module-level imports are the 80/20 |

**Takeaway:** Module-level import/export analysis via Grep is the right scope. Function-level call graphs are too expensive for a prompt-based system and yield diminishing returns. The LLM can infer function-level dependencies from reading the imported modules.

### Reranking Approaches

| Tool | Approach | Key Insight |
|------|----------|-------------|
| **Cohere Reranker** | Cross-encoder model scoring query vs. document | ML reranking is overkill for our case (prompt-based, no model) |
| **Aider** | PageRank on reference graph | Structural importance is a valid ranking signal |
| **RAG systems** | BM25 + vector score fusion (RRF) | Reciprocal Rank Fusion is simple and effective for combining scores |
| **Sourcegraph** | Heuristic scoring (file path match, symbol type, recency) | Simple heuristics beat complex models when the candidate set is small (<50 files) |

**Takeaway:** For our use case (candidate set of 5-15 files from Level 2), a 3-factor heuristic score is sufficient: (1) keyword overlap with task description, (2) dependency graph distance from seed files, (3) file-type priority. No ML needed.

### Context Pack / Context Assembly

| Tool | Approach | Key Insight |
|------|----------|-------------|
| **Aider** | Repo map + active files + related files | "Related files" via reference graph is the key expansion |
| **Continue.dev** | Context providers pipeline (file, codebase, web, terminal) | Pipeline pattern: each provider contributes context items, then assembled |
| **Cursor** | Embedding similarity + file recency + manual pins | Recency is a valid signal but not available in our prompt-only system |
| **RepoGraph** | k-hop ego-graph from seed files | k-hop expansion is the right model: expand N hops from seed files in dependency graph |

**Takeaway:** 1-hop expansion from seed files in the dependency graph (imports + test files) is the right balance. 2-hop risks context explosion. The context pack builder assembles: seed files + 1-hop imports + corresponding test files, trimmed to token budget.

### Session Caching

**Key insight from our architecture:** Claude Code's context window IS the session cache. Data computed in early phases (repo map, symbol index, dependency graph) persists in the conversation context for later phases. Cross-session persistence is handled by `01_DISCOVERY.md` (already exists). **No new file-based cache mechanism is needed.**

This simplifies the design significantly — the "session cache" component becomes: (1) embed structured data in `01_DISCOVERY.md` for cross-session persistence, and (2) rely on Claude Code's context window for intra-session persistence. No `.claude/cache/` directory needed.

---

## 7. Recommendations

### Suggested Technical Approach

**Extend existing files with 5 lightweight procedures — no new infrastructure:**

1. **Symbol Index** → Extend `repo-map.md` to output a structured `## Symbol Index` section in addition to the compact tree. Format: `| Symbol | Type | File | Line |` table, capped at ≤1K tokens. Embed in `01_DISCOVERY.md` alongside the repo map.

2. **Dependency Graph** → New Step 0b in `researching-code`: for each candidate file from the repo map, Grep for import/export patterns (per-language). Build a mental model of which files depend on which. Format: brief inline list, NOT a separate artifact. The LLM holds the graph in context — it doesn't need to be written down.

3. **Reranking** → New Step 0d in `researching-code`: after Level 2 retrieval, score each result on 3 factors:
   - Keyword overlap (how many task keywords appear in the file/symbol names)
   - Dependency proximity (is this file imported by or importing a seed file?)
   - File-type priority (source=3, test=2, config=1, doc=0)
   - Re-order and take top-N (default N=8)

4. **Context Pack Builder** → New Step 0e in `researching-code`: take the top-N reranked files, expand 1-hop via dependency graph (add imported modules + test files), trim to ≤8 files total. Read these files (or key sections) — this is the assembled context pack for Steps 1-2.

5. **Session Cache** → No new mechanism. `01_DISCOVERY.md` persists the repo map + symbol index across sessions. Claude Code's context window persists the dependency graph and context pack within a session.

### Alternative Approaches Considered

| Alternative | Pros | Cons | Verdict |
|-------------|------|------|---------|
| **New `.claude/cache/` directory** for session state | Explicit, inspectable | Extra file I/O; extra cleanup; Claude Code context window already does this | **Rejected** — unnecessary complexity |
| **Full AST parsing via MCP** for symbol index | More accurate symbols | Requires claude-context MCP configured; breaks "works without MCP" | **Rejected** — Grep is good enough |
| **ML-based reranking** (embedding similarity) | More accurate ranking | Requires embedding infrastructure; overkill for 5-15 candidate files | **Rejected** — heuristic scoring sufficient |
| **Separate `DEPENDENCY_GRAPH.md` artifact** | Persistent, inspectable | Violates "minimal artifacts" principle; graph changes with every research run | **Rejected** — keep in-context only |

### Key Decisions to Make

1. **Symbol index token budget**: Recommend ≤1K tokens (on top of ≤2K repo map = ≤3K total for structural context). For repos with >200 symbols, truncate to primary source directories.

2. **Import pattern coverage**: Start with 4 language families:
   - JS/TS: `^import .+ from ['"](.+)['"]` and `require\(['"](.+)['"]\)`
   - Python: `^import (.+)` and `^from (.+) import`
   - Go: `"(.+)"` inside import blocks
   - PHP: `^use (.+);`
   - Generic fallback: `^import|^require|^use|^from .+ import`

3. **Context pack size cap**: Recommend 8 files max (seed + expansions). Configurable via prompt instruction — no hard-coded limit, but default guidance.

4. **Reranking threshold**: Don't rerank if ≤5 results (just use them all). Only rerank when Level 2 returns >5 candidates.

### Unknowns Resolved

- **Session cache design**: Resolved — no new file cache needed; use `01_DISCOVERY.md` + context window.
- **Symbol index storage**: Resolved — embed in `01_DISCOVERY.md` as a structured table.
- **Dependency graph persistence**: Resolved — in-context only (not written to file).

### Open Questions

1. **Should the symbol index be a Markdown table or a fenced code block?** Table is more readable but uses more tokens. Fenced block with `name:type:file:line` per line is more compact. Recommend: compact fenced format for large repos, table for small repos.

2. **Should Step 0b-0e be conditional on repo size?** For small repos (<30 files), the full pipeline may be overkill — the LLM can just read all relevant files. Recommend: skip dependency graph and reranking for repos with <50 source files; go directly from repo map to file reading.
