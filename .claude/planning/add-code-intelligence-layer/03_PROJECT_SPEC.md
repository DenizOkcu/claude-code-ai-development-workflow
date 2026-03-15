# Project Spec: add-code-intelligence-layer

## Technical Requirements

Derived from `01_DISCOVERY.md` success criteria:

| ID | Requirement | Acceptance Criteria |
|----|-------------|---------------------|
| TR-1 | Symbol index schema + population procedure | `repo-map.md` outputs `## Symbol Index` section in compact `type:name:file:line` format; `discover.md` embeds it in `01_DISCOVERY.md` |
| TR-2 | Dependency graph schema + build procedure | `researching-code` Step 0b builds import/export adjacency list in-context for candidate files using per-language Grep patterns |
| TR-3 | Reranking procedure | `researching-code` Step 0d scores results on 3 factors (keyword 40%, proximity 35%, file-type 25%) and re-orders; activates only when > 5 candidates |
| TR-4 | Context pack builder | `researching-code` Step 0e assembles ≤ 8 files: top-N seeds + 1-hop imports + test files; applies progressive read depth |
| TR-5 | Session persistence | Cross-session: symbol index in `01_DISCOVERY.md`. Intra-session: context window. No new file cache. |
| TR-6 | Backward compatibility | All new steps degrade gracefully; existing Level 1→Level 2 behavior unchanged when intelligence data is unavailable |
| TR-7 | Documentation | `ARCHITECTURE.md` updated with Code Intelligence Layer section |

## Non-Functional Requirements

| NFR | Target | Measurement |
|-----|--------|-------------|
| **Token overhead** | ≤ 3K tokens for structural context (repo map + symbol index) | Character count of ## Repository Map + ## Symbol Index sections in 01_DISCOVERY.md ÷ 3.5 |
| **Tool call efficiency** | ≤ 13 tool calls in Step 0 pipeline (a through e) | Count of Glob/Grep/Read/search_code calls during research |
| **Context pack precision** | ≥ 60% of context pack files are referenced in final RESEARCH.md | Count files in pack that appear in "Files to Touch" section |
| **Graceful degradation** | 100% of existing workflows unaffected | Run `/research` on a repo without `01_DISCOVERY.md` — same behavior as before |
| **Skill file size** | `researching-code/SKILL.md` stays under 200 lines | Line count after modification |
| **Small repo bypass** | Full pipeline skipped for repos < 50 source files | No dependency graph or reranking steps executed |

## Interface Contracts

### Symbol Index Format (in 01_DISCOVERY.md)

```markdown
## Symbol Index

<!-- type:name:file:line — one per line, sorted by file path -->
func:calculateTotal:src/billing/calculator.ts:15
class:PaymentService:src/billing/service.ts:8
iface:PaymentConfig:src/billing/types.ts:3
func:validateCard:src/billing/validator.ts:22
export:DEFAULT_CURRENCY:src/billing/constants.ts:1
```

**Type vocabulary:** `func`, `class`, `iface`, `type`, `const`, `export`, `enum`, `trait`, `struct`

**Budget:** ≤ 1K tokens (~80 entries max)

### Dependency Graph (in-context, not written)

Mental model held by the LLM:

```
src/billing/service.ts
  imports: [src/billing/calculator.ts, src/billing/types.ts, src/billing/validator.ts]
  imported-by: [src/api/routes.ts]
  tested-by: [tests/billing/service.test.ts]
```

### Context Pack (in-context, not written)

Mental model held by the LLM:

```
Context Pack (7 files):
  1. src/billing/service.ts — seed, score 0.92
  2. src/billing/calculator.ts — seed, score 0.87
  3. src/billing/types.ts — import of #1, score 0.85
  4. src/billing/validator.ts — import of #1, score 0.78
  5. src/api/routes.ts — imports #1, score 0.72
  6. tests/billing/service.test.ts — test for #1
  7. tests/billing/calculator.test.ts — test for #2
```

### Reranking Rubric (embedded in skill)

```
Scoring per candidate file:
  keyword_score = (matching_keywords / total_keywords) × 3    # 0-3 range
  proximity_score = 3 if direct import of seed,
                    2 if imports a seed,
                    1 if shares dependency,
                    0 otherwise                                # 0-3 range
  filetype_score = 3 if source, 2 if test, 1 if config, 0 if doc  # 0-3 range

  composite = (keyword × 0.4) + (proximity × 0.35) + (filetype × 0.25)
```

### Import Patterns (per-language Grep patterns)

```
JS/TS:     ^import .+ from ['"](.+)['"]
           require\(['"](.+)['"]\)
Python:    ^import (\S+)
           ^from (\S+) import
Go:        "([^"]+)" (inside import blocks)
PHP:       ^use (.+);
Rust:      ^use (.+);
Generic:   ^import|^require|^use|^from .+ import
```

### Test File Detection Patterns

```
{name}.test.{ext}           # JS/TS: calculator.test.ts
{name}.spec.{ext}           # JS/TS alt: calculator.spec.ts
test_{name}.{ext}           # Python: test_calculator.py
{name}_test.{ext}           # Go: calculator_test.go
__tests__/{name}.{ext}      # Jest convention
tests/{name}.{ext}          # Generic test directory
```

## Testing Requirements

This is a prompt-engineering project — no automated test suite. Quality is validated by:

| Validation Method | What It Checks |
|-------------------|----------------|
| **Quality Check in skill** | Checklist at end of `researching-code/SKILL.md` verifying each step was executed |
| **Manual dry run** | Run `/discover` + `/research` on a real repo and verify: (a) symbol index generated, (b) dependency graph built, (c) reranking applied, (d) context pack assembled, (e) RESEARCH.md references context pack files |
| **Regression check** | Run `/research` on a repo WITHOUT `01_DISCOVERY.md` — verify existing Level 1→2 behavior unchanged |
| **Small repo check** | Run on a repo with < 50 files — verify pipeline skips dep graph + reranking |
| **Token budget check** | Verify `## Symbol Index` section is ≤ 4K characters (~1K tokens) |

## Migration Plan

No data migration needed. Changes are purely additive to existing Markdown files.

**Existing `01_DISCOVERY.md` files** (from prior `/discover` runs): Will not have a `## Symbol Index` section. This is handled by graceful degradation — if the section is missing, `researching-code` proceeds without it (existing behavior).

**Re-discovery:** Users can re-run `/discover` to regenerate `01_DISCOVERY.md` with the new symbol index section.

## Feature Flag Strategy

**Implicit feature flag via data presence** (same pattern as claude-context MCP):

- Symbol index is active if `## Symbol Index` section exists in `01_DISCOVERY.md`
- Dependency graph is active if symbol index + import patterns are available
- Reranking is active if > 5 candidates are returned from Level 2
- Context pack builder is active if reranking produced results

No explicit on/off switch. No configuration. No setup wizard. The features activate organically when data is available.

## Rollback Plan

**If the changes cause issues:**

1. **Revert `researching-code/SKILL.md`** to the version before this change (git revert). This restores the original Step 0 with Level 1→Level 2 only. Total rollback time: < 1 minute.

2. **Revert `repo-map.md`** to remove the symbol index output section. Existing repo maps continue to work (they just won't have the new section).

3. **Revert `ARCHITECTURE.md`** documentation changes.

4. **No data cleanup needed.** Existing `01_DISCOVERY.md` files with `## Symbol Index` sections are harmless — they just won't be consumed by the old skill version.

**Partial rollback:** Individual components can be removed independently by editing the skill file. For example, remove only the reranking step while keeping the symbol index and dependency graph.
