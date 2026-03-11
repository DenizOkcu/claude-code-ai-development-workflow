# Research: add-memory-improve-skills

**What:** Add persistent memory system and upgrade skills to official Anthropic format
**When:** 2026-02-28

---

## Summary

- **Risk:** Medium
- **Approach:** Migrate skills to folder-based format; build memory layer on auto-memory directory
- **Effort:** Significant (5 skills to restructure + memory system + cross-reference updates)

---

## 1. Codebase Analysis

### Files to Touch (Skills Migration)

**Skill files to convert (delete old, create new folder structure):**
- `.claude/skills/code-research.md` -> `.claude/skills/code-research/SKILL.md`
- `.claude/skills/solution-planning.md` -> `.claude/skills/solution-planning/SKILL.md`
- `.claude/skills/code-implementation.md` -> `.claude/skills/code-implementation/SKILL.md`
- `.claude/skills/code-review.md` -> `.claude/skills/code-review/SKILL.md`
- `.claude/skills/review-fix.md` -> `.claude/skills/review-fix/SKILL.md`

**Cross-reference files that mention skill names (MUST update):**
- `.claude/agents/sdlc-orchestrator.md` -- lines 31, 46, 63, 83, 96 (references `researching-code`, `planning-solutions`, etc.)
- `.claude/ARCHITECTURE.md` -- lines 77-81 (skills table)
- `.claude/commands/COMMAND_USAGE.md` -- lines 96-100 (skill list)
- `docs/integration-plan.md` -- lines 22, 70, 137, 221-225 (skill file paths)

**Documentation to update:**
- `.claude/ARCHITECTURE.md` -- add memory layer, update skill format description
- `.claude/STATE_MANAGEMENT.md` -- outdated (describes old 8+ artifact system), needs reconciliation
- `README.md` -- patch with new skill format and memory info

**Memory system files to create (NEW):**
- `~/.claude/projects/{hash}/memory/MEMORY.md` -- auto-loaded index
- `~/.claude/projects/{hash}/memory/patterns.md` -- stable patterns
- `~/.claude/projects/{hash}/memory/decisions.md` -- architectural decisions
- `~/.claude/projects/{hash}/memory/learnings.md` -- lessons from /retro

### Patterns to Follow

**Current skill YAML frontmatter pattern (already exists!):**
```yaml
---
name: researching-code
description: "Understand minimum codebase context..."
model: sonnet
---
```

**Official Anthropic spec requires:**
```yaml
---
name: skill-name-in-kebab-case
description: What it does. Use when [trigger conditions]. [Key capabilities].
---
```

**Key differences from official spec:**
1. `model: sonnet` is NOT a standard field -- should be removed or moved to `metadata:`
2. Files must be named exactly `SKILL.md` (case-sensitive) inside a kebab-case folder
3. `description` must include trigger phrases (WHAT + WHEN)
4. No XML tags allowed
5. Optional fields: `license`, `compatibility`, `allowed-tools`, `metadata`

**Existing naming convention:** Skill names use verb-gerund pattern (`researching-code`, `implementing-code`). These are valid kebab-case and CAN be kept as folder names.

### Key Dependencies

- Claude Code skill loading mechanism (reads `SKILL.md` from folders in `.claude/skills/`)
- SDLC Orchestrator Agent (references skills by `name` field)
- `/retro` command (currently writes learnings to `CLAUDE.md` -- should also write to memory)

---

## 2. Architecture Context

### How Skills Fit in the System

```
User -> /sdlc command -> SDLC Orchestrator Agent
                              |
                              v
                    Skills (capabilities layer):
                    ├── researching-code
                    ├── planning-solutions
                    ├── implementing-code
                    ├── reviewing-code
                    └── fixing-review-issues (review-fix)
                              |
                              v
                    Artifacts (data layer):
                    docs/{issue-name}/ (5 files per issue)
```

### Skill Name Resolution

The orchestrator references skills by their `name` field from YAML frontmatter:
- `sdlc-orchestrator.md:31` -> `**Skill:** \`researching-code\``
- `sdlc-orchestrator.md:46` -> `**Skill:** \`planning-solutions\``
- `sdlc-orchestrator.md:63` -> `**Skill:** \`implementing-code\``
- `sdlc-orchestrator.md:83` -> `**Skill:** \`reviewing-code\``
- `sdlc-orchestrator.md:96` -> `**Skill:** \`fixing-review-issues\``

**CRITICAL FINDING:** In Claude Code, the skill `name` field in frontmatter should match the folder name. Current skill names DON'T match file names:
| Skill `name` | Current File | Target Folder |
|--------------|-------------|---------------|
| `researching-code` | `code-research.md` | `code-research/SKILL.md` OR `researching-code/SKILL.md` |
| `planning-solutions` | `solution-planning.md` | `solution-planning/SKILL.md` OR `planning-solutions/SKILL.md` |
| `implementing-code` | `code-implementation.md` | `code-implementation/SKILL.md` OR `implementing-code/SKILL.md` |
| `reviewing-code` | `code-review.md` | `code-review/SKILL.md` OR `reviewing-code/SKILL.md` |
| `fixing-review-issues` | `review-fix.md` | `review-fix/SKILL.md` OR `fixing-review-issues/SKILL.md` |

**Decision needed:** Should folder names match the `name` field (e.g., `researching-code/`) or the current file names (e.g., `code-research/`)?

### Memory System Architecture

**Current "memory" mechanism:**
- `/retro` command appends learnings to `CLAUDE.md` `## Learnings` section
- This persists in the repo, visible to all users
- No structured per-project memory across sessions

**Target memory architecture:**
```
Auto-memory (per-project, auto-loaded):
~/.claude/projects/{hash}/memory/
├── MEMORY.md              (index, always in context, max 200 lines)
├── patterns.md            (stable patterns from multiple sessions)
├── decisions.md           (architectural decisions)
└── learnings.md           (lessons from /retro)

Repo-level learnings (shared, in-repo):
CLAUDE.md -> ## Learnings section (already works via /retro)
```

**Integration points:**
- `/retro` should write to BOTH `CLAUDE.md` (repo-shared) AND `memory/learnings.md` (project-personal)
- Auto-memory `MEMORY.md` is always loaded into conversation context (first 200 lines)
- Topic files are loaded on demand when referenced from MEMORY.md

---

## 3. Dependency Analysis

### Internal Dependencies

| Component | Depends On | Impact of Change |
|-----------|-----------|-----------------|
| `/sdlc` command | Orchestrator agent | None (doesn't reference skills directly) |
| Orchestrator agent | Skill `name` fields | Must match new folder/name convention |
| ARCHITECTURE.md | Skill names | Update documentation table |
| COMMAND_USAGE.md | Skill names | Update architecture diagram |
| `/retro` command | CLAUDE.md | Extend to also write to auto-memory |
| `docs/integration-plan.md` | Skill file paths | Update paths |
| STATE_MANAGEMENT.md | Old artifact model | Outdated -- reconcile or remove |

### External Dependencies

| Dependency | Status | Risk |
|-----------|--------|------|
| Claude Code skill loading | Active, well-documented | Low -- well-defined spec |
| Auto-memory directory | Built-in feature | Low -- already works |
| YAML frontmatter parsing | Built-in | Low -- standard format |

### Compatibility Concerns

- `model: sonnet` field in current skills is NOT part of official spec
  - In Claude Code, the session model is used (not skill-specified)
  - This field should be removed to comply with spec
  - Alternative: move to `metadata.model` if we want to preserve the hint

---

## 4. Integration Points

### Systems Affected

1. **SDLC Orchestrator Agent** (`sdlc-orchestrator.md`)
   - References skills by name at 5 locations
   - Must be updated if skill names change
   - **If folder names match current `name` fields: NO changes needed to orchestrator**

2. **Architecture Documentation** (`ARCHITECTURE.md`)
   - Skills table at lines 77-81
   - Visual diagram at lines 28-37
   - Needs memory layer added

3. **Command Usage Docs** (`COMMAND_USAGE.md`)
   - Skills list at lines 96-100
   - Architecture diagram

4. **Retro Command** (`retro.md`)
   - Currently writes to `CLAUDE.md`
   - Should be extended to write to auto-memory

5. **Integration Plan Doc** (`docs/integration-plan.md`)
   - References old skill file paths
   - Can be updated or archived (it's historical documentation)

### What Won't Be Affected

- All 30+ command files (`.claude/commands/*.md`) -- they don't reference skills directly
- The `/sdlc` command -- it invokes the orchestrator, not skills
- `settings.json` -- no skill references
- `USER_COMMUNICATION.md` -- no skill references
- `TROUBLESHOOTING.md` -- no skill references
- Example files and templates

---

## 5. Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| Skill name mismatch breaks orchestrator | High | Keep `name` fields identical; only change folder structure |
| `SKILL.md` not recognized by Claude Code | Medium | Test with one skill first before converting all 5 |
| `model: sonnet` removal changes behavior | Low | Claude Code uses session model regardless; field is likely ignored |
| Progressive disclosure makes content harder to find | Low | Keep SKILL.md self-contained; use references/ only for truly supplementary content |
| Memory system bloats context | Low | Enforce 200-line MEMORY.md limit; topic files loaded on-demand only |
| STATE_MANAGEMENT.md conflicts with current architecture | Medium | Reconcile or archive; it describes the OLD 8+ artifact model |

### Critical Risk: Skill Name/Folder Name Alignment

The official spec says: "The folder itself must use kebab-case" and "name field: kebab-case only, should match folder name."

**Current `name` fields vs. current file names don't match:**
- `researching-code` vs `code-research.md`
- `planning-solutions` vs `solution-planning.md`
- `implementing-code` vs `code-implementation.md`
- `reviewing-code` vs `code-review.md`
- `fixing-review-issues` vs `review-fix.md`

**Safest approach:** Use the current `name` fields as folder names (since the orchestrator already references those names). This means:
- `researching-code/SKILL.md` (not `code-research/SKILL.md`)
- `planning-solutions/SKILL.md` (not `solution-planning/SKILL.md`)
- etc.

This way the orchestrator references DON'T need to change.

---

## 6. Prior Art & Ecosystem Research

### Anthropic's Official Skill Format (from reference PDFs)

**Structure:**
```
your-skill-name/
├── SKILL.md            # Required. The brain of your skill.
├── scripts/            # Optional. Python, Bash, etc.
├── references/         # Optional. Docs, guides, examples.
└── assets/             # Optional. Templates, fonts, icons.
```

**Required YAML frontmatter:**
```yaml
---
name: your-skill-name
description: What it does. Use when user asks to [specific phrases].
---
```

**Best practices from the guide:**
1. Description must explain WHAT and WHEN (under 1024 chars)
2. Include trigger phrases users would say
3. Keep SKILL.md under 5,000 words
4. Move detailed docs to `references/`
5. No XML tags anywhere
6. Instructions should be specific and actionable, not vague
7. Include error handling sections
8. Use `## Important` or `## Critical` headers for key instructions

**Skill categories that apply to this project:**
- **Category 2: Workflow Automation** -- ordered steps, validation gates, refinement loops
- This is the primary category for all 5 SDLC skills

**Relevant patterns:**
- **Pattern 1: Sequential Workflow Orchestration** -- explicit step ordering, dependencies, rollback
- **Pattern 3: Iterative Refinement** -- quality criteria, validation scripts, stopping conditions

### Memory Patterns in Claude Code

**Auto-memory system (built-in):**
- Directory: `~/.claude/projects/{project-hash}/memory/`
- `MEMORY.md` auto-loaded into context (first 200 lines)
- Topic files linked from MEMORY.md, loaded on demand
- Designed for: stable patterns, architectural decisions, user preferences, debugging insights

**What to save (from Claude Code docs):**
- Stable patterns confirmed across multiple interactions
- Key architectural decisions
- User preferences for workflow and tools
- Solutions to recurring problems

**What NOT to save:**
- Session-specific context
- Incomplete/unverified information
- Anything duplicating CLAUDE.md instructions

### Anti-Patterns to Avoid

1. **Over-splitting skills** -- Don't create too many small files. If SKILL.md is under 5,000 words, `references/` may not be needed
2. **Vague descriptions** -- "Helps with projects" won't trigger. Need specific phrases
3. **Instructions buried** -- Put critical instructions at the top, not the bottom
4. **Model laziness** -- Add "Take your time, quality is important" type encouragements

---

## 7. Recommendations

### Suggested Technical Approach

**Phase 1: Migrate skills to folder format (low risk)**
1. Create folder structure: `researching-code/SKILL.md`, etc.
2. Move content from flat `.md` files into `SKILL.md` files
3. Upgrade YAML frontmatter to official spec (remove `model: sonnet`, improve `description`)
4. Delete old flat `.md` files
5. Test each skill loads correctly

**Phase 2: Improve skill content (medium effort)**
1. Rewrite `description` fields following `[What] + [When] + [Capabilities]` pattern
2. Evaluate which skills need `references/` (most are under 5,000 words already)
3. Add error handling sections where missing
4. Add explicit trigger phrases to descriptions

**Phase 3: Implement memory system (additive, low risk)**
1. Create structured `MEMORY.md` template
2. Create topic files (patterns.md, decisions.md, learnings.md)
3. Update `/retro` command to write to auto-memory in addition to CLAUDE.md
4. Update ARCHITECTURE.md with memory layer

**Phase 4: Documentation and cleanup**
1. Update ARCHITECTURE.md (skills format + memory layer)
2. Update COMMAND_USAGE.md
3. Reconcile or archive STATE_MANAGEMENT.md (outdated)
4. Patch README.md

### Key Decisions Needed

1. **Folder naming:** Use current `name` fields (`researching-code/`) or current file prefixes (`code-research/`)?
   - **Recommendation:** Use `name` fields as folder names to avoid orchestrator changes

2. **`model: sonnet` field:** Remove, move to `metadata:`, or keep?
   - **Recommendation:** Remove. Claude Code uses session model. If users want a hint, add to `metadata.preferred_model`

3. **Progressive disclosure depth:** How much content goes to `references/`?
   - **Recommendation:** Keep SKILL.md self-contained for now (all are under 5,000 words). Only create `references/` for the templates and examples currently inline

4. **Memory scope:** What goes in auto-memory vs CLAUDE.md Learnings?
   - **Recommendation:** CLAUDE.md Learnings = repo-shared, actionable lessons. Auto-memory = personal project context, patterns, preferences

5. **STATE_MANAGEMENT.md disposition:** Update, archive, or delete?
   - **Recommendation:** Archive to `docs/archive/` -- it documents the old architecture and has historical value, but conflicts with the current 5-artifact system

### Unknowns That Need Resolution Before Design

1. Does Claude Code's skill loader require `name` field to match folder name exactly? (The official docs say "should match" but is it enforced?)
2. Does the `model: sonnet` field cause issues or is it silently ignored?
3. Can `references/` files be loaded by the skill body via simple markdown links, or do they need special syntax?

---

## Open Questions

1. **Folder name convention?**
   - Options: Match `name` field (`researching-code/`) vs match semantic intent (`code-research/`)
   - Recommendation: Match `name` field to minimize orchestrator changes

2. **Should we rename skill `name` fields?**
   - Current: verb-gerund (`researching-code`, `implementing-code`)
   - Alternative: noun-based (`code-research`, `code-implementation`)
   - Recommendation: Keep current names. They work and changing them requires updating 4+ files

3. **Memory system bootstrap:** Should we pre-populate MEMORY.md with current project knowledge?
   - Recommendation: Yes, seed with key patterns from ARCHITECTURE.md and existing learnings from CLAUDE.md
