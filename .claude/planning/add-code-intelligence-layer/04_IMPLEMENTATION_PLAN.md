# Implementation Plan: add-code-intelligence-layer

## Overview
- **Total Phases:** 4
- **Estimated Effort:** M (all Markdown prompt changes — no runtime code)
- **Dependencies:** None (all additive to existing files)
- **Feature Flag:** Implicit — components activate when data is present (symbol index in `01_DISCOVERY.md`, >5 candidates for reranking, etc.)

## Files Summary

| File | Phase | Change Type |
|------|-------|-------------|
| `.claude/commands/repo-map.md` | 1 | Moderate — add Step 7 (Symbol Index generation) |
| `.claude/commands/discover.md` | 1 | Minor — cross-reference symbol index in discovery template |
| `.claude/skills/researching-code/SKILL.md` | 2, 3 | Major — extend Step 0 with sub-steps 0b through 0e |
| `.claude/skills/implementing-code/SKILL.md` | 3 | Minor — add optional dependency graph note in Step 1 |
| `.claude/ARCHITECTURE.md` | 4 | Moderate — update Context Engine section |

---

## Phase 1: Symbol Index Generation

### Objective
Extend `repo-map.md` to output a structured `## Symbol Index` section alongside the existing compact tree. Update `discover.md` to embed this index in `01_DISCOVERY.md`.

### Tasks
- [ ] Task 1.1: Add **Step 7: Generate Symbol Index** to `repo-map.md` — `.claude/commands/repo-map.md`
  - After Step 6 (Output), add a new step that formats Grep-extracted symbols into the compact `type:name:file:line` format
  - Reuse the same Grep results from Step 3 (no additional tool calls)
  - Apply progressive truncation tiers matching the repo map (same 4-tier system)
  - Include the type vocabulary mapping: `function`→`func`, `class`→`class`, `interface`→`iface`, `type`→`type`, `const`→`const`, `export`→`export`, `enum`→`enum`, `trait`→`trait`, `struct`→`struct`
- [ ] Task 1.2: Add symbol index output format to Step 6 in `repo-map.md` — `.claude/commands/repo-map.md`
  - After the repo map tree output and the Files/Language/Entry points summary, add the `## Symbol Index` block
  - Format: fenced code block with one `type:name:file:line` entry per line
  - Budget note: ≤1K tokens (~80 entries max)
- [ ] Task 1.3: Update `discover.md` template for `01_DISCOVERY.md` — `.claude/commands/discover.md`
  - In Step 5 (Create `01_DISCOVERY.md`), add `## Symbol Index` section after `## Repository Map`
  - Add note: `> Generated alongside repo map. Run /repo-map to refresh.`
- [ ] Task 1.4: Add token budget guidance for symbol index to `repo-map.md` — `.claude/commands/repo-map.md`
  - Same 4 tiers as Step 5 token budget: <100 files (all symbols), 100-200 (primary dirs), 200-500 (top 50 files), >500 (top 30 + "Partial index" note)

### Validation
- [ ] `repo-map.md` contains a complete Step 7 with type vocabulary, format spec, and truncation tiers
- [ ] `discover.md` template includes `## Symbol Index` section in `01_DISCOVERY.md`
- [ ] Symbol index format is `type:name:file:line` (compact, one per line)
- [ ] Token budget ≤1K is stated explicitly
- [ ] No changes to existing Steps 1-6 behavior in `repo-map.md`

### Rollback
Revert `repo-map.md` and `discover.md` to their pre-change versions. Existing `01_DISCOVERY.md` files without `## Symbol Index` continue to work.

---

## Phase 2: Dependency Graph + Reranking in Research Skill

### Objective
Extend `researching-code/SKILL.md` Step 0 with three new sub-steps: dependency graph building (0b), reranking (0d), and a small-repo bypass gate.

### Tasks
- [ ] Task 2.1: Add **small-repo bypass gate** at the start of Step 0 — `.claude/skills/researching-code/SKILL.md`
  - After Level 1 (repo map reading), check source file count from the repo map
  - If < 50 source files: skip Steps 0b and 0d; go directly from Level 1 → Level 2 → Step 1
  - Note: "For small repos, the repo map provides sufficient guidance. The full intelligence pipeline activates for repos with 50+ source files."
- [ ] Task 2.2: Add **Step 0b: Build Dependency Graph** — `.claude/skills/researching-code/SKILL.md`
  - Insert between existing Level 1 and Level 2
  - Instruction: For each candidate file (5-15 from Level 1), Grep for import statements using the per-language pattern table
  - Include the import pattern table (JS/TS, Python, Go, PHP, Rust, Generic)
  - Include test file detection patterns
  - Instruction: Build a mental adjacency list — `file → [imports, imported-by, tested-by]`
  - Graceful degradation: "If import patterns don't match the language, skip this step and proceed to Level 2."
- [ ] Task 2.3: Add **Step 0d: Rerank Results** after Level 2 — `.claude/skills/researching-code/SKILL.md`
  - Activation: "If Level 2 returned > 5 candidate files, apply reranking. Otherwise, use all candidates."
  - Include the 3-factor scoring rubric:
    - Keyword overlap (40%): task keywords found in file path + symbol names
    - Dependency proximity (35%): import distance from seed files (3/2/1/0)
    - File-type priority (25%): source=3, test=2, config=1, doc=0
  - Instruction: Re-order candidates by composite score, take top-8
  - Graceful degradation: "If no dependency graph available, score using keyword + file-type only (2 factors)."
- [ ] Task 2.4: Update **Quality Check** section — `.claude/skills/researching-code/SKILL.md`
  - Add: `- [ ] Built dependency graph for candidate files (if repo ≥ 50 files)?`
  - Add: `- [ ] Reranked results when > 5 candidates returned?`

### Validation
- [ ] Step 0 now has sub-steps: 0a (Level 1 + symbol index), 0b (dependency graph), Level 2 (existing), 0d (reranking)
- [ ] Small-repo bypass gate is the first check after Level 1
- [ ] Import pattern table covers 6 language families
- [ ] Test file detection patterns included
- [ ] Reranking rubric has exact weights (40/35/25)
- [ ] Both new steps have graceful degradation instructions
- [ ] Quality Check has 2 new items
- [ ] Skill file stays under 200 lines

### Rollback
Revert `researching-code/SKILL.md` to pre-change version. The original Level 1→Level 2 behavior is fully restored.

---

## Phase 3: Context Pack Builder + Implementing-Code Integration

### Objective
Add the context pack builder (Step 0e) to `researching-code` and add an optional dependency graph note to `implementing-code`.

### Tasks
- [ ] Task 3.1: Add **Step 0e: Assemble Context Pack** — `.claude/skills/researching-code/SKILL.md`
  - Insert after Step 0d (reranking)
  - Procedure:
    1. Start with top-5 reranked seed files
    2. For each seed, check dependency graph: add up to 2 direct imports not already in pack
    3. Add corresponding test files for seed files (if they exist)
    4. Hard cap: ≤ 8 files total. If over, drop test files first, then lowest-scored imports
  - Progressive read depth guidance:
    - Small files (< 100 lines): read entirely
    - Medium files (100-300 lines): read imports + exported symbols + relevant functions
    - Large files (> 300 lines): read only sections matching task keywords
  - Graceful degradation: "If no dependency graph or reranking was performed (small repo), use Level 2 candidates directly as the context pack."
- [ ] Task 3.2: Update **Step 1** and **Step 2** references — `.claude/skills/researching-code/SKILL.md`
  - Step 1 ("Think Deeply"): Add note: "Use the context pack from Step 0e as your primary reference. The pack contains the most relevant files — start your analysis there."
  - Step 2 ("Quick Discovery"): Update: "The context pack from Step 0e replaces broad searching. Read the pack files. Only use additional Grep if the pack doesn't answer one of the three questions."
- [ ] Task 3.3: Update **Quality Check** — `.claude/skills/researching-code/SKILL.md`
  - Add: `- [ ] Assembled context pack (≤ 8 files) before deep analysis?`
- [ ] Task 3.4: Add optional dependency note to **Step 1** in `implementing-code` — `.claude/skills/implementing-code/SKILL.md`
  - After "Pattern Discovery" paragraph, add: "**Impact Analysis (if dependency graph available):** If the research phase built a dependency graph (visible in RESEARCH.md's 'Key Dependencies' section), use it to identify callers and downstream consumers of files you're modifying. This helps catch unintended breakage."
  - Keep it brief — 2-3 sentences max

### Validation
- [ ] Step 0e has complete context pack assembly procedure with hard cap of 8 files
- [ ] Progressive read depth guidance covers 3 tiers (small/medium/large)
- [ ] Steps 1 and 2 reference the context pack
- [ ] Quality Check has the new context pack item
- [ ] `implementing-code/SKILL.md` has a brief dependency note (not a full procedure)
- [ ] `researching-code/SKILL.md` stays under 200 lines
- [ ] Full pipeline flow reads naturally: 0a → 0b → Level 2 → 0d → 0e → 1 → 2 → 3 → 4

### Rollback
Revert `researching-code/SKILL.md` and `implementing-code/SKILL.md` to pre-change versions.

---

## Phase 4: Architecture Documentation + CLAUDE.md Update

### Objective
Update `ARCHITECTURE.md` to document the Code Intelligence Layer and update `CLAUDE.md` if any behavioral rule changes are needed.

### Tasks
- [ ] Task 4.1: Update **Context Engine (Hierarchical)** section in `ARCHITECTURE.md` — `.claude/ARCHITECTURE.md`
  - Rename section to: `### Context Engine (Code Intelligence Layer)`
  - Replace the existing 2-level diagram with the upgraded pipeline:
    ```
    Level 1: Repo Map + Symbol Index (≤3K tokens)
    Level 1b: Dependency Graph (in-context, import/export tracing)
    Level 2: Targeted Detail (search_code MCP or Grep/Read)
    Level 2b: Reranking (3-factor heuristic scoring)
    Level 3: Context Pack (≤8 files, seed + deps + tests)
    ```
  - Update the data flow description
  - Add scaling behavior table (small/medium/large/very large repos)
- [ ] Task 4.2: Add **Reranking** details to the Context Engine section — `.claude/ARCHITECTURE.md`
  - Brief description of the 3-factor heuristic (keyword, proximity, file-type)
  - Note: "Activates only when >5 candidates; mental procedure, not code"
- [ ] Task 4.3: Add **Context Pack Builder** details — `.claude/ARCHITECTURE.md`
  - Brief description of the assembly procedure (seeds + 1-hop expansion + test files)
  - Hard cap note (≤8 files)
  - Progressive read depth tiers
- [ ] Task 4.4: Verify `CLAUDE.md` — `.claude/../../CLAUDE.md` (project root)
  - Check if any new behavioral rules are needed. Expected: no changes, since all new behavior is in skill files.
  - If the intelligence layer introduces a new session-start check (it doesn't — symbol index is implicit), it would go here.

### Validation
- [ ] `ARCHITECTURE.md` Context Engine section reflects the full 5-component pipeline
- [ ] Scaling behavior table covers 4 repo size tiers
- [ ] Data flow description matches the actual Step 0 sub-steps in `researching-code`
- [ ] No stale references to the old 2-level-only system
- [ ] `CLAUDE.md` verified — no changes needed (or minimal update made)

### Rollback
Revert `ARCHITECTURE.md` to pre-change version. Documentation-only — no functional impact.

---

## Test Strategy

### Validation Approach (Prompt-Engineering Project)

No automated test suite. Quality validated through:

| Method | Phase | What It Verifies |
|--------|-------|-----------------|
| **Quality Check checklists** | 2, 3 | Each skill execution self-verifies via embedded checklist |
| **Line count check** | 2, 3 | `researching-code/SKILL.md` stays under 200 lines |
| **Token budget check** | 1 | Symbol index output ≤ 4K chars (~1K tokens) |
| **Regression dry run** | All | Run `/research` on a repo WITHOUT `01_DISCOVERY.md` — verify existing behavior unchanged |
| **Small repo dry run** | 2 | Run `/research` on a repo with < 50 files — verify dep graph + reranking skipped |
| **Full pipeline dry run** | All | Run `/discover` + `/research` on a medium repo (50-200 files) and verify all 5 components activate |
| **Cross-reference check** | 4 | `ARCHITECTURE.md` diagrams match actual skill step numbers and names |

### Key Quality Invariants

1. **Backward compatibility:** Removing `## Symbol Index` from any `01_DISCOVERY.md` must not break `/research`
2. **Graceful degradation:** Every new sub-step has an explicit "skip if unavailable" instruction
3. **File size constraint:** `researching-code/SKILL.md` ≤ 200 lines after all modifications
4. **Token budget:** Repo map (≤2K) + symbol index (≤1K) = ≤3K tokens total structural overhead
5. **Context pack cap:** Hard limit of 8 files — never exceeded regardless of repo size
