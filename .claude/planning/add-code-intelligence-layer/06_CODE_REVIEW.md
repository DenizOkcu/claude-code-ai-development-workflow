# Code Review: add-code-intelligence-layer

**Reviewer:** Claude (simulated senior engineer)
**Date:** 2026-03-15
**Scope:** 5 modified Markdown files implementing the Code Intelligence Layer

---

## 1. Automated Checks

| Check | Result | Notes |
|-------|--------|-------|
| Lint | N/A | Prompt-engineering project — no linter configured |
| Type check | N/A | No runtime code |
| Test suite | N/A | No automated tests — quality via skill checklists |
| Build | N/A | No build step |
| Line count: `researching-code/SKILL.md` | **PASS** (190 lines) | Under 200-line constraint |
| Token budget: symbol index spec | **PASS** | ≤1K tokens stated, ~80 entries max |
| Token budget: total structural | **PASS** | ≤3K tokens (2K map + 1K index) |

---

## 2. Manual Code Review

### File 1: `.claude/skills/researching-code/SKILL.md` (190 lines)

| Line(s) | Severity | Finding |
|---------|----------|---------|
| 6 | 💡 Praise | Version bumped from 1.0.0 → 2.0.0. Correct semver for a breaking change in skill structure. |
| 23-25 | 💡 Praise | Excellent renaming from "Hierarchical Context Loading" to "Code Intelligence Pipeline" — clearly signals the upgraded capability. |
| 41 | 💡 Praise | Small-repo bypass gate is well-placed and clearly worded. The < 50 files threshold is reasonable. |
| 43-57 | 💡 Praise | Dependency graph step is concise. Import pattern table covers 5 language families + generic fallback. Graceful degradation is explicit. |
| 47-53 | 🔵 Suggestion | The import pattern table in the skill combines PHP and Rust into one row (`PHP/Rust`), while the `03_PROJECT_SPEC.md` and `03_ARCHITECTURE.md` list them separately. Cosmetic inconsistency — no functional impact, but worth noting for cross-reference accuracy. |
| 64-75 | 💡 Praise | Reranking rubric is clear and well-structured. The dual activation conditions (≥50 files AND >5 candidates) prevent unnecessary work. Graceful 2-factor fallback when graph is missing is a smart design. |
| 77-88 | 💡 Praise | Context pack builder is well-specified: hard cap of 8 files, clear priority order for dropping (tests first, then lowest-scored imports), progressive read depth. |
| 99-112 | 💡 Praise | Steps 1 and 2 correctly reference the context pack. Step 2 is renamed to "Targeted Analysis" and removes the broad Grep examples — good alignment with the new pipeline-first approach. |
| 116 | 🟡 Important | **Output path inconsistency:** Step 3 says `docs/{issue_name}/RESEARCH.md` and Step 4 says `docs/{issue_name}/STATUS.md`. But the SDLC workflow uses `.claude/planning/{issue-name}/02_CODE_RESEARCH.md` and `.claude/planning/{issue-name}/00_STATUS.md`. This pre-existing inconsistency was inherited from the original skill, not introduced by this change — but it should be noted. |
| 171-182 | 💡 Praise | Quality check is comprehensive — 10 items covering all 5 pipeline stages plus the 3 research questions. |
| 184-190 | 💡 Praise | Common Issues section is updated to reference the pipeline and context pack cap. The "Over-engineering small repos" warning is practical. |

### File 2: `.claude/commands/repo-map.md` (167 lines)

| Line(s) | Severity | Finding |
|---------|----------|---------|
| 117-158 | 💡 Praise | Step 7 (Symbol Index) is well-structured. Type vocabulary table is complete, format spec matches `03_PROJECT_SPEC.md`, truncation tiers mirror Step 5. |
| 119 | 💡 Praise | "Reuse the same Grep results from Step 3" — correctly avoids duplicate tool calls. |
| 130 | 🔵 Suggestion | The type vocabulary includes `impl` (for Rust `impl` blocks) which is reasonable but wasn't in the original `03_PROJECT_SPEC.md` type vocabulary. Minor addition — acceptable since it follows from Rust support in the existing symbol extraction patterns. |
| 155-158 | 💡 Praise | Rules are clear: skip test/config files, skip uncleanable symbols. Good noise reduction. |
| 160-167 | 💡 Praise | Quality gates updated to include symbol index budget check. |

### File 3: `.claude/commands/discover.md` (260 lines)

| Line(s) | Severity | Finding |
|---------|----------|---------|
| 183-187 | 💡 Praise | Symbol Index section added in the right location — after Repository Map, before Missing Quality Tooling. Clean placement. |
| 185 | 🟡 Important | **Missing generation instruction in Step 3:** The `01_DISCOVERY.md` template (Step 5) includes a `## Symbol Index` placeholder saying "see /repo-map Step 7". However, `discover.md` Step 3 (lines 73-107) generates the repo map but does NOT include an instruction to also generate the symbol index. The LLM executing `/discover` may produce the map but miss the index unless it reads ahead to Step 5's template and infers the need. **Fix:** Add a note to Step 3 (or a brief Step 3b) explicitly instructing to generate the symbol index alongside the map using the type vocabulary from `/repo-map` Step 7. |
| 250 | 💡 Praise | Quality gates updated to include symbol index ≤1K tokens check. |

### File 4: `.claude/skills/implementing-code/SKILL.md` (167 lines)

| Line(s) | Severity | Finding |
|---------|----------|---------|
| 31 | 💡 Praise | Impact Analysis paragraph is concise (2 sentences) and correctly conditional ("if dependency graph available"). References the right location (RESEARCH.md "Key Dependencies" section). |

### File 5: `.claude/ARCHITECTURE.md` (393 lines)

| Line(s) | Severity | Finding |
|---------|----------|---------|
| 175-229 | 💡 Praise | Complete replacement of the Context Engine section. The 5-level pipeline diagram is clear and well-labeled. Scaling behavior table covers all 4 tiers. Session persistence documentation is accurate. |
| 212 | 💡 Praise | Data flow description matches the actual step labels in `researching-code/SKILL.md` (0a → 0b → 0c → 0d → 0e → 1 → 2). Good cross-reference accuracy. |

---

## 3. Requirements Compliance

| ID | Requirement | Status | Evidence |
|----|-------------|--------|----------|
| TR-1 | Symbol index schema + population | **MET** | `repo-map.md` Step 7 defines type vocabulary, format, truncation; `discover.md` template embeds it |
| TR-2 | Dependency graph schema + build | **MET** | `researching-code` Step 0b with import pattern table + test file detection |
| TR-3 | Reranking procedure | **MET** | Step 0d with 3-factor scoring (40/35/25), activation conditions, graceful degradation |
| TR-4 | Context pack builder | **MET** | Step 0e with 8-file cap, expansion procedure, progressive read depth |
| TR-5 | Session persistence | **MET** | Cross-session via `01_DISCOVERY.md`; intra-session via context window; no file cache |
| TR-6 | Backward compatibility | **MET** | Every step has graceful degradation; small-repo bypass; "if exists" checks throughout |
| TR-7 | Documentation | **MET** | `ARCHITECTURE.md` fully updated with Code Intelligence Layer |

| NFR | Target | Status |
|-----|--------|--------|
| Token overhead ≤ 3K | ≤ 3K (2K map + 1K index) | **MET** |
| Skill file ≤ 200 lines | 190 lines | **MET** |
| Small repo bypass | < 50 files skips graph + reranking | **MET** |

---

## 4. ADR Compliance

| ADR | Decision | Implementation Match? |
|-----|----------|----------------------|
| ADR-001 (prompt-based) | All 5 components as prompt instructions | **YES** — no new MCP server, no runtime code, all in Markdown |
| ADR-002 (context window as cache) | No file-based session cache | **YES** — no `.claude/cache/` directory, no new artifacts for intermediate state |

---

## 5. Test Quality Review

| Aspect | Assessment |
|--------|-----------|
| Critical paths | Quality check in skill covers all 5 pipeline stages |
| Edge cases | Small-repo bypass, missing discovery doc, missing symbol index, ≤5 candidates — all handled |
| Error paths | Graceful degradation specified for every step (Steps 0b, 0d, 0e) |
| Regression risk | Low — all changes are additive; existing behavior is the fallback |

---

## Review Summary

| Category | Count |
|----------|-------|
| 🔴 Critical | 0 |
| 🟡 Important | 2 |
| 🔵 Suggestions | 2 |
| 💡 Praise | 16 |

## Verdict: ⚠ NEEDS_FIX

### Required Changes

1. **🟡 `discover.md` Step 3 — Missing symbol index generation instruction** (`discover.md:73-107`)
   Step 3 generates the repo map but does not instruct the LLM to also generate the symbol index. The template in Step 5 has a placeholder, but the actual generation step is missing. Add a note to Step 3 (after the formatting rules, before Step 4) instructing: "Also generate a symbol index using the same Grep results — see `/repo-map` Step 7 for format and type vocabulary."

2. **🟡 `researching-code/SKILL.md` Step 3/4 — Pre-existing output path inconsistency** (`researching-code/SKILL.md:116,162`)
   The RESEARCH.md template references `docs/{issue_name}/` but the SDLC workflow stores artifacts in `.claude/planning/{issue-name}/`. This is a pre-existing issue not introduced by this change. **Non-blocking for this PR** but should be tracked for a follow-up fix.

### Non-Blocking Recommendations

1. **🔵 PHP/Rust row in import pattern table** — Consider splitting into separate rows for consistency with `03_PROJECT_SPEC.md` and `03_ARCHITECTURE.md`.
2. **🔵 `impl` type in symbol index vocabulary** — Add to `03_PROJECT_SPEC.md` type vocabulary list for completeness (it's in `repo-map.md` but not in the spec).
