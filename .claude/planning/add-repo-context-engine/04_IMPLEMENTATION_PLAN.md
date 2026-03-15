# Implementation Plan: add-repo-context-engine

## Overview
- **Total Phases:** 3
- **Estimated Effort:** M
- **Dependencies:** None (purely additive, no blockers)
- **Feature Flag:** N/A (implicit: repo map section present in 01_DISCOVERY.md = feature active)

---

## Phase 1: Create Standalone `/repo-map` Command

### Objective
Create the core repo map generation logic as a standalone command. This defines the format, token budget, and language-specific symbol extraction patterns. Later phases will integrate it into `/discover` and `/research`.

### Tasks
- [ ] Task 1.1: Create `.claude/commands/repo-map.md` with YAML frontmatter (`model: sonnet`) — `.claude/commands/repo-map.md`
- [ ] Task 1.2: Define the repo map generation instructions — uses `Glob` for file tree (exclude `node_modules/`, `dist/`, `build/`, `.git/`, `*.lock`, `*.min.js`, `vendor/`, `__pycache__/`, `.terraform/`, `.next/`) — `.claude/commands/repo-map.md`
- [ ] Task 1.3: Define language-specific symbol extraction patterns via `Grep` — `.claude/commands/repo-map.md`:
  - JS/TS: `^export (function|class|const|type|interface)\s+\w+`
  - Python: `^(class |def |async def )\w+`
  - Go: `^(func |type \w+ (struct|interface))`
  - PHP: `^(class |function |interface |trait )\w+`
  - Rust: `^(pub )?(fn |struct |enum |trait |impl )\w+`
  - Generic fallback: `^(class |function |def |export )\w+`
- [ ] Task 1.4: Define compact output format — `.claude/commands/repo-map.md`:
  ```
  ## Repository Map (~{N} files, {detected_language})

  src/
    auth/
      middleware.ts — AuthMiddleware, validateToken(), refreshSession()
      types.ts — User, Session, AuthConfig
    api/
      routes.ts — createRouter(), healthCheck()
      handlers.ts — handleLogin(), handleLogout()
  tests/
    auth.test.ts — (test file)
  ```
- [ ] Task 1.5: Define token budget management — `.claude/commands/repo-map.md`:
  - Target: ≤2K tokens for the repo map
  - For repos with >200 source files: prioritize `src/`, `lib/`, `app/`, `pkg/` directories; summarize others as `{dir}/ — {N} files`
  - For repos with >500 source files: directory-level summary only (no per-file symbols)
- [ ] Task 1.6: Add Quality Gates section — `.claude/commands/repo-map.md`

### Acceptance Criteria
- `/repo-map` command exists and follows the command format pattern (YAML frontmatter, instructions, quality gates)
- Instructions cover: file tree generation, symbol extraction, format specification, token budget, large repo handling
- Language patterns cover the 6 most common stacks (JS/TS, Python, Go, PHP, Rust, generic)
- Output format is compact (file tree + inline symbols, not verbose)

### Rollback
- Delete `.claude/commands/repo-map.md` — no other files affected

---

## Phase 2: Integrate Repo Map into `/discover`

### Objective
Add repo map generation as an automatic step in the discovery phase. The map is embedded in `01_DISCOVERY.md` so it's always available for downstream phases. This addresses the user requirement: "users may forget to run it."

### Tasks
- [ ] Task 2.1: Add `#### 2.5. Generate Repository Map` step to `discover.md`, between Step 2 (Detect Stack) and Step 3 (Create Planning Directory) — `.claude/commands/discover.md`
  - Instructions: After detecting the stack, use the detected language to select the appropriate Grep pattern from the repo-map command
  - Run Glob to build file tree (respecting the same exclusion list from Phase 1)
  - Run Grep with detected-language patterns to extract top-level symbols
  - Format as compact tree with inline symbols
  - Apply token budget: truncate if >2K tokens
- [ ] Task 2.2: Add `## Repository Map` section to the `01_DISCOVERY.md` template in `discover.md` — `.claude/commands/discover.md`
  - Place after `## Detected Tech Stack` and before `### Missing Quality Tooling Recommendations`
  - Content: the generated repo map from Step 2.5
  - Include note: "Generated automatically. Run `/repo-map` to refresh."
- [ ] Task 2.3: Add repo map to the `#### 6. Output to User` section — `.claude/commands/discover.md`
  - Add item 2.5: **Repository map** (compact structural overview)
- [ ] Task 2.4: Update Quality Gates to include repo map — `.claude/commands/discover.md`
  - Add: "Repository map generated and embedded in 01_DISCOVERY.md"

### Acceptance Criteria
- Running `/discover` now auto-generates a repo map and embeds it in `01_DISCOVERY.md`
- The repo map appears in the discovery output to the user
- Existing discovery output (issue name, stack, quality tooling, expert commands) is unchanged
- Step numbering is clean (2.5 between 2 and 3, or renumber to 3 with subsequent steps shifted)

### Rollback
- Revert changes to `.claude/commands/discover.md` — the file is version-controlled

---

## Phase 3: Level 1→Level 2 Escalation in `researching-code`

### Objective
Modify the research skill to use the repo map (Level 1) as a navigation aid before performing targeted searches (Level 2). This replaces the current flat "search everything" approach with a hierarchical one.

### Tasks
- [ ] Task 3.1: Replace `### Step 0: Semantic Retrieval (if available)` with `### Step 0: Hierarchical Context Loading` — `.claude/skills/researching-code/SKILL.md`
  - **Level 1 (Repo Map):** Check if `01_DISCOVERY.md` exists for this issue and contains a `## Repository Map` section. If yes, read it to understand the repo structure and identify candidate files/modules relevant to the feature. If no discovery doc exists (user jumped straight to `/research`), generate a quick map using the same Glob + Grep approach from `/repo-map`.
  - **Level 2 (Targeted Detail):** For the candidate files identified from the map:
    - If `search_code` MCP is available: query it with the feature description, filtered to candidate paths
    - If MCP is not available: use `Grep` for relevant terms within candidate files, then `Read` the top matches
  - This replaces the current "query MCP with broad description" approach
- [ ] Task 3.2: Update `### Step 2: Quick Discovery` to reference the repo map — `.claude/skills/researching-code/SKILL.md`
  - Add: "Use the repo map from Step 0 to narrow your Glob/Grep searches to relevant directories. Don't search the entire repo when the map tells you where to look."
- [ ] Task 3.3: Update `## Quality Check` to include repo map — `.claude/skills/researching-code/SKILL.md`
  - Change: `- [ ] Checked semantic retrieval (if available)?` → `- [ ] Used repo map (Level 1) to identify candidate files?`
  - Add: `- [ ] Used targeted search (Level 2) for candidate files only?`
- [ ] Task 3.4: Update `## Common Issues` — `.claude/skills/researching-code/SKILL.md`
  - Add: "**Ignoring the repo map:** Don't skip the map and go straight to broad Grep. The map exists to narrow your search."
- [ ] Task 3.5: Update `.claude/ARCHITECTURE.md` — add Context Engine description — `.claude/ARCHITECTURE.md`
  - Add a `### Context Engine (Hierarchical)` subsection under the existing Retrieval Layer, describing:
    - Level 1: Repo map (generated by `/discover`, stored in `01_DISCOVERY.md`)
    - Level 2: Targeted retrieval (MCP `search_code` or Glob/Grep, scoped to candidate files)
    - Data flow: discover → 01_DISCOVERY.md → researching-code → targeted search

### Acceptance Criteria
- `researching-code` skill reads the repo map before doing any searches
- If no repo map exists, it generates one on-the-fly (graceful fallback)
- MCP semantic search is still used when available, but scoped to candidate files from the map
- The Quality Check reflects the new hierarchical approach
- ARCHITECTURE.md documents the Context Engine

### Rollback
- Revert changes to `.claude/skills/researching-code/SKILL.md` and `.claude/ARCHITECTURE.md`

---

## Test Strategy

### Validation Approach

Since this is a prompt-engineering project (Markdown files, no runtime code), traditional unit/integration/E2E tests don't apply. Instead, validation is done via:

### Manual Validation (per phase)

**Phase 1 — `/repo-map` command:**
- [ ] Run `/repo-map` on this project (claude-code-ai-development-workflow) — verify it produces a compact map
- [ ] Run `/repo-map` on a medium-sized TypeScript project — verify JS/TS patterns work
- [ ] Run `/repo-map` on a Python project — verify Python patterns work
- [ ] Verify map stays under ~2K tokens for repos with <200 source files
- [ ] Verify large repo handling: map truncates gracefully for repos with >200 source files

**Phase 2 — `/discover` integration:**
- [ ] Run `/discover` on a fresh project — verify repo map appears in `01_DISCOVERY.md`
- [ ] Verify existing discovery output (stack detection, quality tooling) is unchanged
- [ ] Verify repo map section is between Detected Tech Stack and Missing Quality Tooling

**Phase 3 — `researching-code` Level 1→2:**
- [ ] Run `/research {issue}` on a project with `01_DISCOVERY.md` — verify it reads the map first
- [ ] Run `/research {issue}` on a project WITHOUT discovery — verify it generates a map on-the-fly
- [ ] Verify MCP `search_code` is still used when available, but scoped to candidate paths
- [ ] Verify the research output references specific files identified from the map

### Quality Checks (Structural)
- [ ] All modified files retain valid YAML frontmatter
- [ ] No existing command/skill behavior is broken (backward compatible)
- [ ] ARCHITECTURE.md accurately reflects the new Context Engine layer
- [ ] Token budget instructions are clear and actionable
