# Implementation Plan: add-semantic-retrieval

## Overview
- **Total Phases:** 4
- **Estimated Effort:** M (mostly Markdown authoring + config; no custom code to build)
- **Dependencies:** None blocking — all phases are additive to existing workflow
- **Feature Flag:** Implicit — retrieval is active only if `claude-context` exists in `.claude/settings.json`

## Phase 1: Setup Wizard & Command Handler

### Objective
Create the `/retrieval/setup` interactive wizard and `/retrieval` command handler, following the exact pattern established by n8n and firecrawl integrations. After this phase, users can set up and manage retrieval via commands.

### Tasks
- [ ] Task 1.1: Create `.claude/commands/retrieval/setup.md` — Interactive setup wizard with 3 paths: local (Ollama + Docker Milvus), cloud (Zilliz Cloud), and OpenAI-compatible (LM Studio, etc.)
- [ ] Task 1.2: Create `.claude/commands/retrieval.md` — Command handler that checks MCP config, lists available tools, and routes user requests (search, index, status, clear)
- [ ] Task 1.3: Add `.gitignore` entry for any local index artifacts if needed

### Tests
- [ ] Manual: Run `/retrieval/setup` — verify it checks existing config, presents options, and generates correct JSON for each path
- [ ] Manual: Run `/retrieval` without config — verify it shows "not configured" message with `/retrieval/setup` suggestion
- [ ] Manual: Verify generated settings.json JSON is valid and contains all required env vars from 03_PROJECT_SPEC.md

### Acceptance Criteria
- `/retrieval/setup` presents 3 hosting options matching the n8n/firecrawl wizard pattern
- `/retrieval/setup` outputs correct `settings.json` JSON block for local (Ollama + Milvus) path including all env vars: `EMBEDDING_PROVIDER`, `EMBEDDING_MODEL`, `EMBEDDING_DIMENSION`, `EMBEDDING_BATCH_SIZE`, `OLLAMA_HOST`, `OLLAMA_NUM_PARALLEL`, `MILVUS_ADDRESS`, `SPLITTER_TYPE`, `HYBRID_MODE`, `CUSTOM_IGNORE_PATTERNS`
- `/retrieval` command handler checks for `claude-context` in `mcpServers` and stops if not found
- `/retrieval` documents all 4 MCP tools: `search_code`, `index_codebase`, `get_indexing_status`, `clear_index`
- Both files use YAML frontmatter with `name`, `description`, `model: sonnet`

### Rollback
- `rm .claude/commands/retrieval.md .claude/commands/retrieval/setup.md`
- No other files affected

---

## Phase 2: Skill Integration — Research

### Objective
Add a **Step 0: Semantic Retrieval** to the `researching-code` skill so it automatically queries the code index before manual Glob/Grep exploration. This is the primary integration point — the highest-value change.

### Tasks
- [ ] Task 2.1: Modify `.claude/skills/researching-code/SKILL.md` — Add "Step 0: Semantic Retrieval (if available)" before the existing Step 1, with instructions to call `search_code` and use results to guide subsequent Glob/Grep
- [ ] Task 2.2: Add graceful degradation logic in Step 0 — if `claude-context` MCP is not configured or returns an error, skip silently and proceed to Step 1
- [ ] Task 2.3: Update the "Quality Check" section to include a new checkbox: "Checked semantic retrieval (if available)?"

### Tests
- [ ] Manual: Run `/research {issue}` with retrieval configured — verify RESEARCH.md mentions retrieval results
- [ ] Manual: Run `/research {issue}` without retrieval configured — verify workflow completes normally using only Glob/Grep
- [ ] Manual: Verify Step 0 instructions are clear enough that the agent calls `search_code` with the right query format

### Acceptance Criteria
- `researching-code` SKILL.md has a Step 0 that calls `search_code` before Step 1
- Step 0 is explicitly marked as optional: "If `claude-context` MCP is not configured or returns an error, skip this step"
- Existing Steps 1-4 are unchanged (renumbered to 1-4, not 2-5 — keep original numbering)
- Quality Check includes retrieval checkbox
- Skill still works identically when retrieval is not configured

### Rollback
- `git checkout .claude/skills/researching-code/SKILL.md`

---

## Phase 3: Skill Integration — Implementation

### Objective
Add optional retrieval to the `implementing-code` skill so it can discover related files (tests, dependencies, similar patterns) before making changes.

### Tasks
- [ ] Task 3.1: Modify `.claude/skills/implementing-code/SKILL.md` — Add a note in "Step 1: Read the Plan" to optionally query `search_code` for related files when starting each phase
- [ ] Task 3.2: Add a "Pattern Discovery" tip in the skill: suggest using `search_code` to find similar implementations before writing new code
- [ ] Task 3.3: Update the "Quality Check" section with: "Queried for related files (if retrieval available)?"

### Tests
- [ ] Manual: Run `/implement {issue}` with retrieval configured — verify the agent uses retrieval to find related files
- [ ] Manual: Run `/implement {issue}` without retrieval — verify workflow completes normally
- [ ] Manual: Verify the skill modifications are minimal and don't change the core implementation flow

### Acceptance Criteria
- `implementing-code` SKILL.md mentions retrieval as an optional enhancement in Step 1
- The addition is concise (< 10 lines of new content) — does not inflate the skill
- Core implementation flow (Phase by Phase, Deviate Wisely, Full Validation) is untouched
- Skill works identically when retrieval is not configured

### Rollback
- `git checkout .claude/skills/implementing-code/SKILL.md`

---

## Phase 4: Documentation & Architecture Update

### Objective
Update project documentation to reflect the new retrieval layer: ARCHITECTURE.md, CLAUDE.md quick reference, and settings.json with the MCP server entry (commented out as a template).

### Tasks
- [ ] Task 4.1: Update `.claude/ARCHITECTURE.md` — Add "Retrieval Layer" section describing claude-context MCP integration, positioning it between Capabilities and Security layers
- [ ] Task 4.2: Update `CLAUDE.md` — Add `/retrieval` and `/retrieval/setup` to the Expert Commands section under a new "Code Retrieval" subsection
- [ ] Task 4.3: Update `CLAUDE.md` — Add retrieval setup reference in the Quick Start section as an optional enhancement
- [ ] Task 4.4: Add `Bash(npx @zilliz/claude-context-mcp*)` to `.claude/settings.json` permissions.allow list so the MCP server can start without user approval
- [ ] Task 4.5: Update `README.md` — Add a "Semantic Code Retrieval" section describing the capability and setup (patch, don't rewrite)

### Tests
- [ ] Manual: Read updated ARCHITECTURE.md — verify retrieval layer is accurately described and diagram is consistent
- [ ] Manual: Verify CLAUDE.md commands section includes `/retrieval` and `/retrieval/setup`
- [ ] Manual: Verify README.md patch adds retrieval without removing existing content
- [ ] Manual: Verify settings.json permissions allow npx execution of the package

### Acceptance Criteria
- ARCHITECTURE.md includes retrieval layer with component responsibilities, data flow, and relationship to existing layers
- CLAUDE.md Expert Commands lists `/retrieval` and `/retrieval/setup` with descriptions
- README.md has a new section explaining semantic retrieval (what it does, how to set up)
- settings.json permissions include the npx command
- All documentation accurately reflects ADR-001 and ADR-002 decisions

### Rollback
- `git checkout .claude/ARCHITECTURE.md CLAUDE.md README.md .claude/settings.json`

---

## Test Strategy

### Manual Testing (primary — this is a Markdown/YAML project)

All testing is manual since the deliverables are Markdown command files and skill modifications. The "code under test" is the behavior these Markdown instructions produce when Claude Code executes them.

| Test | Method | Pass Criteria |
|------|--------|---------------|
| Setup wizard flow (local) | Run `/retrieval/setup`, choose local | Outputs correct JSON with Ollama + Milvus env vars |
| Setup wizard flow (cloud) | Run `/retrieval/setup`, choose cloud | Outputs correct JSON with Zilliz Cloud env vars |
| Command handler (not configured) | Run `/retrieval` without MCP config | Shows "not configured" + suggests `/retrieval/setup` |
| Command handler (configured) | Run `/retrieval search "auth middleware"` | Routes to `search_code` tool correctly |
| Research skill (with retrieval) | Run `/research` on indexed repo | RESEARCH.md shows files found via semantic search |
| Research skill (without retrieval) | Run `/research` without config | Workflow completes normally, no errors |
| Implement skill (with retrieval) | Run `/implement` on indexed repo | Agent queries for related files before coding |
| Graceful degradation | Stop Milvus, run `/research` | Falls back to Glob/Grep, no crash |

### Integration Testing

| Test | Method | Pass Criteria |
|------|--------|---------------|
| Full pipeline | Set up Ollama + Milvus + config → index → search | `search_code` returns relevant results |
| Incremental index | Index, modify file, re-index | Second index is faster, only changed file processed |
| Multi-project | Index two different projects | Each has independent collection, searches don't cross-contaminate |

### Performance Validation

| Metric | Target | How to Measure |
|--------|--------|---------------|
| Search latency | < 500ms | Time `search_code` call in Claude Code |
| Index time (this repo, ~100 files) | < 30s | Time `index_codebase` call |
| MCP server startup | < 5s | Time from Claude Code launch to first tool response |
