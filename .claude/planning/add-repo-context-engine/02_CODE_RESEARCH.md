# Code Research: add-repo-context-engine

**What:** Hierarchical repository context system — repo map (Level 1) + context packs (Level 2)
**When:** 2026-03-15

---

## 1. Codebase Analysis

### Files to Modify

| File | Purpose | Change |
|------|---------|--------|
| `.claude/commands/discover.md` | Phase 1: Discovery & Scoping | **Add Step 2.5**: auto-generate repo map after stack detection, embed in `01_DISCOVERY.md` |
| `.claude/skills/researching-code/SKILL.md` | Research skill | **Modify Step 0**: replace raw semantic search with Level 1→Level 2 escalation (repo map → targeted search) |
| `.claude/ARCHITECTURE.md` | System architecture doc | **Update**: add Context Engine section |

### Files to Create

| File | Purpose |
|------|---------|
| `.claude/commands/repo-map.md` | Standalone `/repo-map` command for on-demand map generation |

### Existing Patterns to Follow

**Command format** (from `discover.md`, `research.md`):
- YAML frontmatter: `model:` field (`sonnet` for lightweight, `opus` for deep reasoning)
- `## Phase N: Title` heading
- `### Pre-Conditions` → `### Instructions` → `### Post-Actions` → `### Quality Gates`
- `$ARGUMENTS` variable for user input

**Skill format** (from `researching-code/SKILL.md`):
- YAML frontmatter: `name`, `description`, `model`, `metadata`
- Body: Mindset → Goal → Instructions (numbered steps) → Output Format → Quality Check → Common Issues

**Integration pattern** (from `retrieval.md` integration into `researching-code`):
- Step 0 pattern: check if a tool is available → use it if so → fall back gracefully if not
- This is the exact pattern we should follow for repo map: check if cached/available → generate if not → use result to guide search

**Discovery phase pattern** (from `discover.md`):
- Step 1: Generate issue name
- Step 2: Detect tech stack (file-scanning with Glob)
- Step 3: Create planning directory
- Step 4: Create `01_DISCOVERY.md`
- Step 5: Create `00_STATUS.md`
- Step 6: Output to user
- **The new repo map step should go between Step 2 (stack detection) and Step 4 (create discovery doc)** — the map informs the discovery document

---

## 2. Architecture Context

### Where This Fits

```
USER INPUT
    │
    ▼
/discover (Phase 1)
    ├── Step 2: Detect stack
    ├── Step 2.5: Generate repo map ← NEW
    ├── Step 4: Create 01_DISCOVERY.md (now includes repo map)
    │
    ▼
/research (Phase 2)
    ├── Step 0: Level 1 — Read repo map from 01_DISCOVERY.md ← MODIFIED
    ├── Step 0b: Level 2 — Targeted search (MCP or Glob/Grep) ← NEW
    ├── Step 1: Think deeply
    ├── Step 2: Quick discovery (5-15 files)
    │
    ▼
/implement (Phase 5) — uses repo map from discovery for pattern discovery
```

### Data Flow

1. **`/discover`** scans the repo → generates a compact map → embeds it in `01_DISCOVERY.md` under a new `## Repository Map` section
2. **`/research`** reads `01_DISCOVERY.md` → uses the repo map to identify candidate files → queries MCP `search_code` or Glob/Grep for those specific files → reads only the relevant ones
3. **`/implement`** can reference the map from `01_DISCOVERY.md` for pattern discovery (already partially does this via MCP Step 0)

### Why Embed in Discovery (Not a Separate Artifact)

Per the user's feedback: users may forget to run `/repo-map` manually. By embedding it in `/discover` (which is the standard entry point), the map is always available. The standalone `/repo-map` command exists for on-demand refresh or use outside the SDLC workflow.

---

## 3. Dependency Analysis

### Internal Dependencies

| Module | Dependency Type | Notes |
|--------|----------------|-------|
| `discover.md` | **Modified** | Adds repo map generation step |
| `researching-code/SKILL.md` | **Modified** | Reads repo map from `01_DISCOVERY.md` |
| `implementing-code/SKILL.md` | **No change needed** | Already has MCP Step 0; can optionally reference map |
| `01_DISCOVERY.md` template | **Extended** | Gains `## Repository Map` section |
| `00_STATUS.md` template | **No change** | No new status tracking needed |

### External Dependencies

| Dependency | Status | Required? |
|------------|--------|-----------|
| Tree-sitter (via MCP) | Available if `claude-context` MCP configured | No — fallback uses Glob/Grep |
| `@zilliz/claude-context-mcp` | Optional, already integrated | No — repo map works without it |
| Claude Code built-in tools (Glob, Grep, Read) | Always available | Yes — primary repo map generation method |

### No New Dependencies Required

The repo map is generated entirely using Claude Code's built-in tools (Glob for file tree, Grep for symbol extraction, Read for signatures). No new MCP server, no new npm package, no new runtime.

---

## 4. Integration Points

### Systems Affected

| System | Impact | Backward Compatible? |
|--------|--------|---------------------|
| `/discover` command | Gains Step 2.5 (repo map gen) | Yes — existing output unchanged, new section added |
| `researching-code` skill | Step 0 becomes Level 1→2 | Yes — still falls back to raw search if no map |
| `01_DISCOVERY.md` template | New `## Repository Map` section | Yes — old discoveries without map still work |
| `/repo-map` command | New standalone command | Yes — purely additive |
| `ARCHITECTURE.md` | Documentation update | N/A |

### No Breaking Changes

Every integration is additive. Existing workflows that don't use `/discover` (e.g., jumping straight to `/research`) still work because `researching-code` falls back gracefully when no repo map exists.

---

## 5. Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| Repo map exceeds token budget on very large repos (10K+ files) | Medium | Use Aider-style approach: rank files by importance, truncate to budget. Cap at 2K tokens. |
| Symbol extraction via Grep is language-dependent | Low | Use language-agnostic patterns (class/function/def/export keywords). Accept imperfect extraction over complex parsing. |
| Discovery phase becomes slow on large repos | Low | Glob + Grep are fast (< 5s for most repos). Map generation is embarrassingly parallelizable. |
| Map becomes stale if repo changes between phases | Low | Map is regenerated per `/discover` run. `/repo-map` available for manual refresh. |
| Repo map format wastes tokens on uninformative files (configs, generated code) | Medium | Exclude common non-informative patterns: `*.lock`, `*.min.js`, `dist/`, `build/`, `node_modules/`, `.git/` |

---

## 6. Prior Art & Ecosystem Research

### Aider's Repo Map (Gold Standard)

Aider pioneered the repo map concept. Key design decisions:

- **Tree-sitter + PageRank**: Parses all files with Tree-sitter, builds a file→symbol→file reference graph, ranks with PageRank
- **Token budget**: Default 1,024 tokens (expands to ~8K when no files active)
- **Format**: Shows file paths with indented function/class signatures. Elided lines shown as `...`
- **Symbols included**: Class definitions, method/function signatures with full parameters, decorators. NOT bodies, NOT comments, NOT docstrings
- **3-level caching**: Disk cache (keyed by file mtime), in-memory map cache, in-memory tree cache

**What we can adopt**: The output format (file tree + signatures) and the token budget concept.
**What we skip**: PageRank ranking (requires a graph library), Tree-sitter parsing (requires a runtime). We use Glob + Grep instead — simpler, always available, good enough for an LLM to reason about.

### Continue.dev's Repo Map Provider

- Lists files and top-level class/function/method signatures
- Uses Tree-sitter (optional: `includeSignatures` flag)
- Simpler than Aider — no graph ranking

### Cursor's Approach

- Embedding-based RAG (not structural maps)
- Merkle tree for change detection
- Good for "find similar code" but lacks the structural overview that a map provides
- **We already have this via `claude-context` MCP** — our repo map complements it

### RepoGraph (Academic, 2024)

- Builds code graph with `invoke` and `contain` edge types
- k-hop ego-graphs limit context to ~2K tokens
- 29.67% resolve rate on SWE-bench
- Interesting but too complex for a prompt-only implementation

### Key Takeaways from Prior Art

1. **~1K–2K tokens is the sweet spot** for a repo-wide structural overview
2. **Signatures > bodies** — include what things are, not how they work
3. **File tree + symbols is the minimal viable format** — this is what all tools converge on
4. **Ranking matters** for large repos — not all files are equally informative
5. **The map is a navigation aid, not a replacement for reading code**

---

## 7. Recommendations

### Suggested Technical Approach

**Approach A (Recommended): Glob + Grep repo map embedded in `/discover`**

1. **In `/discover`**, after stack detection (Step 2):
   - Run `Glob` to build the file tree (respecting `.gitignore`, excluding `node_modules/`, `dist/`, etc.)
   - Run `Grep` with language-appropriate patterns to extract top-level symbols:
     - JS/TS: `export (function|class|const|type|interface)`
     - Python: `^(class |def |async def )`
     - Go: `^func |^type .+ (struct|interface)`
     - PHP: `^(class |function |interface |trait )`
     - Rust: `^(pub )?(fn |struct |enum |trait |impl )`
     - Generic fallback: `^(class |function |def |export )`
   - Format as a compact tree with symbols indented under their file
   - Embed in `01_DISCOVERY.md` under `## Repository Map`

2. **In `researching-code`**, modify Step 0:
   - Read `01_DISCOVERY.md` `## Repository Map` section (Level 1)
   - Identify candidate files from the map that are relevant to the feature
   - Use `search_code` MCP (if available) or `Grep`/`Read` (Level 2) to fetch detail for those specific files
   - Proceed to existing Step 1 (think deeply) with better-targeted context

3. **`/repo-map` standalone command**: same generation logic as discover Step 2.5, but outputs directly to the user (not embedded in a planning artifact). Useful for on-demand refresh or non-SDLC usage.

**Alternative B: MCP-dependent approach**
- Use `claude-context` MCP's Tree-sitter AST to generate the map
- Pro: Richer symbol extraction
- Con: Requires MCP to be configured; doesn't work in fresh repos
- **Rejected**: Violates the "works without MCP" requirement from Discovery

**Alternative C: Separate artifact (`REPO_MAP.md`)**
- Generate as standalone file alongside `01_DISCOVERY.md`
- Pro: Clean separation of concerns
- Con: Extra artifact; user must remember it exists; violates the "minimal artifacts" design principle
- **Rejected**: User explicitly requested embedding in `/discover` flow

### Key Decisions to Make

1. **Token budget**: Recommend 2K tokens max for the map. For repos exceeding this, truncate to most important directories (src/, lib/, app/) and note truncation.
2. **Symbol depth**: Top-level only (classes, functions, exports) vs. including method signatures inside classes. Recommend: include class methods for primary source directories, top-level only for everything else.
3. **File ranking for large repos**: Sort by directory structure (src/ first, tests/ last) or by file size (larger = more important)? Recommend: directory structure with common source dirs first.

### Unknowns to Resolve

- **Exact Grep patterns per language**: Need to test the regex patterns against real repos to ensure they capture the right symbols without too much noise. Can iterate during implementation.
- **Large monorepo handling**: For repos with 10K+ files, the file tree alone could exceed 2K tokens. May need a directory-level summary instead of file-level for deeply nested repos.
