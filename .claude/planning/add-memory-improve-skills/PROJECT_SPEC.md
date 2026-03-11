# Project Spec: add-memory-improve-skills

## Technical Requirements

_Derived from DISCOVERY.md success criteria._

### TR-1: Skill Folder Migration
- Each of the 5 skills must exist as `.claude/skills/{name}/SKILL.md`
- Folder names must be kebab-case and match the YAML `name` field
- Old flat `.md` files must be deleted after migration
- Target folder names: `researching-code/`, `planning-solutions/`, `implementing-code/`, `reviewing-code/`, `review-fix/`

### TR-2: YAML Frontmatter Compliance
- Required fields: `name`, `description`
- `description` must follow `[What] + [When] + [Capabilities]` pattern
- `description` must be under 1,024 characters
- `model: sonnet` field removed (not in official spec)
- Optional `metadata` block with `version` and `category`
- No XML tags anywhere in frontmatter

### TR-3: Skill Content Standardization
- Each SKILL.md must follow the standardized internal structure:
  - Mindset, Goal, Instructions (numbered steps), Output Format, Quality Check, Common Issues
- All skills must remain under 5,000 words
- `references/` directories created only if content exceeds limits (not required initially)

### TR-4: Memory System — Auto-Memory Directory
- Create structured `MEMORY.md` in `~/.claude/projects/{hash}/memory/`
- Create topic files: `patterns.md`, `decisions.md`, `learnings.md`
- `MEMORY.md` must stay under 200 lines (auto-loaded cap)
- Topic files have no size limit (loaded on demand only)

### TR-5: Memory Integration with /retro
- `/retro` command must write to BOTH:
  - `CLAUDE.md ## Learnings` (existing, repo-shared)
  - Auto-memory `learnings.md` (new, project-personal)
- `/retro` must update `MEMORY.md` index when adding new entries

### TR-6: Cross-Reference Integrity
- All 4 files that reference skill names must be updated:
  - `.claude/ARCHITECTURE.md`
  - `.claude/commands/COMMAND_USAGE.md`
  - `docs/integration-plan.md`
  - `.claude/agents/sdlc-orchestrator.md` (only if `fixing-review-issues` -> `review-fix` rename)
- No broken references after migration

### TR-7: Documentation Updates
- `README.md` patched (not rewritten) with skill format + memory info
- `.claude/ARCHITECTURE.md` updated with memory layer + skill format
- `.claude/STATE_MANAGEMENT.md` archived to `docs/archive/`

### TR-8: Zero Regression
- All existing commands must continue to work unchanged
- SDLC orchestrator workflow must function end-to-end
- No skill loading failures after migration

---

## Non-Functional Requirements

| ID | Requirement | Target | Measurement |
|----|-------------|--------|-------------|
| NFR-1 | Context overhead from skills | < 4,000 tokens total | Count: 5 frontmatters (~500) + 1 active SKILL.md body (~2,000) + MEMORY.md (~1,000) |
| NFR-2 | MEMORY.md size | ≤ 200 lines | Line count of MEMORY.md file |
| NFR-3 | Skill description length | ≤ 1,024 chars each | Character count of `description` field |
| NFR-4 | SKILL.md word count | ≤ 5,000 words each | Word count of SKILL.md body (excluding frontmatter) |
| NFR-5 | Skill trigger accuracy | 90%+ auto-activation | Manual test: invoke 10 relevant prompts, verify skill triggers ≥9 times |
| NFR-6 | Migration completeness | 100% of skills converted | All 5 folders exist with valid SKILL.md; all 5 old files deleted |
| NFR-7 | Cross-reference integrity | 0 broken references | Grep for old file paths returns 0 results |

---

## Interface Contracts

### Skill YAML Frontmatter Schema

```yaml
# === Required ===
name: string            # kebab-case, must match folder name
                        # Pattern: /^[a-z][a-z0-9-]*$/
                        # Max length: 50 chars
description: string     # Pattern: "[What]. Use when [trigger]. [Capabilities]."
                        # Max length: 1024 chars

# === Optional ===
metadata:
  version: string       # semver, e.g. "1.0.0"
  category: string      # enum: "workflow-automation" | "document-creation" | "mcp-enhancement"
  author: string        # optional attribution
```

### Skill Folder Structure Contract

```
.claude/skills/{name}/
├── SKILL.md            # REQUIRED. Case-sensitive. Contains frontmatter + instructions.
├── references/         # OPTIONAL. Supplementary docs loaded on demand.
│   └── *.md
├── scripts/            # OPTIONAL. Not used in this project.
└── assets/             # OPTIONAL. Not used in this project.
```

### SKILL.md Body Structure Contract

```markdown
---
{YAML frontmatter}
---

# {Skill Title}

**Mindset:** {one-line philosophy}

## Goal
{What this skill accomplishes — 1-3 sentences}

## Instructions

### Step 1: {Action verb}
{Specific, actionable instructions}

### Step 2: {Action verb}
{...}

## Output Format
{Template for the artifact this skill produces}

## Quality Check
{Checklist — items that must be true before marking complete}

## Common Issues
{Error handling, troubleshooting, edge cases}
```

### Memory File Contracts

**MEMORY.md** (auto-loaded, max 200 lines):
```markdown
# Project Memory

## Quick Reference
- {key fact 1}
- {key fact 2}

## Key Patterns
See [patterns.md](patterns.md) for stable workflow patterns.

## Architectural Decisions
See [decisions.md](decisions.md) for key technical decisions.

## Lessons Learned
See [learnings.md](learnings.md) for accumulated project insights.

## Session Notes
{Rolling notes — oldest entries pruned first when approaching 200 lines}
```

**patterns.md** (on-demand, no size limit):
```markdown
# Workflow Patterns

## Pattern: {name}
- **Context:** {when this applies}
- **Solution:** {what to do}
- **Confirmed:** {date, issue name}
```

**decisions.md** (on-demand, no size limit):
```markdown
# Architectural Decisions

## Decision: {title}
- **Date:** {date}
- **Context:** {why}
- **Decision:** {what}
- **Status:** Active | Superseded
```

**learnings.md** (on-demand, no size limit):
```markdown
# Lessons Learned

## {date} — {issue-name}
- {specific, actionable learning}
- {specific, actionable learning}
```

### /retro Memory Write Contract

When `/retro` completes, it must:
1. Append to `CLAUDE.md ## Learnings` (existing behavior, unchanged)
2. Append to `memory/learnings.md` with date + issue name header
3. Update `memory/MEMORY.md` Session Notes section (add brief entry)
4. If a new stable pattern is identified, append to `memory/patterns.md`
5. If a new architectural decision is captured, append to `memory/decisions.md`

---

## Testing Requirements

### Test Category 1: Skill Structure Validation

| Test | Method | Pass Criteria |
|------|--------|---------------|
| Folder exists for each skill | `ls .claude/skills/*/SKILL.md` | Returns exactly 5 paths |
| Old flat files deleted | `ls .claude/skills/*.md` | Returns 0 files |
| YAML frontmatter parses | Read each SKILL.md, verify `---` delimiters | All 5 parse without error |
| `name` matches folder | Compare YAML `name` to parent folder name | All 5 match |
| `description` under 1024 chars | Character count | All 5 ≤ 1024 |
| Body under 5000 words | Word count (excluding frontmatter) | All 5 ≤ 5000 |
| No `model:` field in frontmatter | Grep for `model:` in SKILL.md files | 0 results |

### Test Category 2: Cross-Reference Integrity

| Test | Method | Pass Criteria |
|------|--------|---------------|
| No references to old file paths | Grep for `.claude/skills/*.md` (not `*/SKILL.md`) | 0 results outside archive |
| Orchestrator references valid | Read sdlc-orchestrator.md, check skill names | All 5 names match folder names |
| `fixing-review-issues` renamed | Grep across repo | 0 occurrences (replaced by `review-fix`) |
| ARCHITECTURE.md updated | Read and verify skill table | Lists 5 skills with correct format |
| COMMAND_USAGE.md updated | Read and verify skill list | Lists 5 skills with correct names |

### Test Category 3: Memory System

| Test | Method | Pass Criteria |
|------|--------|---------------|
| MEMORY.md exists | `cat ~/.claude/projects/{hash}/memory/MEMORY.md` | File exists with structured content |
| MEMORY.md under 200 lines | `wc -l MEMORY.md` | ≤ 200 |
| Topic files exist | Check patterns.md, decisions.md, learnings.md | All 3 exist |
| /retro writes to memory | Run /retro, check learnings.md | New entry appended |

### Test Category 4: Functional (End-to-End)

| Test | Method | Pass Criteria |
|------|--------|---------------|
| `/sdlc` workflow completes | Run full /sdlc on a test issue | All 5 phases complete without error |
| Skills trigger from descriptions | Use natural language matching skill descriptions | ≥ 4 of 5 skills auto-activate |
| Memory persists across sessions | Write via /retro, start new session, check MEMORY.md | Entries visible in new session |

---

## Migration Plan

### Phase 1: Skill Folder Structure (non-destructive)
1. Create 5 new folders under `.claude/skills/`
2. Copy content from old `.md` files into `SKILL.md` in each folder
3. Upgrade YAML frontmatter (remove `model:`, improve `description`)
4. Standardize SKILL.md body structure
5. **Verification:** All 5 SKILL.md files parse correctly

### Phase 2: Delete Old Files + Update References
1. Delete 5 old flat `.md` files from `.claude/skills/`
2. Update `fixing-review-issues` -> `review-fix` in orchestrator (3 references)
3. Update ARCHITECTURE.md skills table + add memory layer
4. Update COMMAND_USAGE.md skill list
5. Update docs/integration-plan.md file paths
6. Archive STATE_MANAGEMENT.md to docs/archive/
7. **Verification:** Grep for old paths returns 0 results

### Phase 3: Memory System (additive)
1. Create `MEMORY.md` with structured template
2. Create `patterns.md`, `decisions.md`, `learnings.md` templates
3. Seed MEMORY.md with project knowledge from ARCHITECTURE.md
4. Update `/retro` command to write to auto-memory
5. **Verification:** MEMORY.md loaded in new session

### Phase 4: Documentation
1. Patch README.md with skill format + memory section
2. Final cross-reference audit
3. **Verification:** All docs consistent

### Ordering Rationale
- Phase 1 before Phase 2: Create new before deleting old (safe rollback)
- Phase 2 before Phase 3: Fix skill references before adding memory (clean baseline)
- Phase 3 independent of Phase 2: Memory system is purely additive
- Phase 4 last: Documentation reflects final state

---

## Feature Flag Strategy

N/A — This is a configuration project (Markdown/YAML). There are no feature flags, runtime toggles, or gradual rollouts. Changes take effect immediately when files are committed.

**Progressive deployment alternative:**
- Migrate one skill first (e.g., `researching-code`) and verify it works
- Then migrate remaining 4 skills
- Then add memory system
- This phased approach serves the same purpose as feature flags

---

## Rollback Plan

### Skill Migration Rollback

**Trigger:** Skills fail to load after migration, or orchestrator references break.

**Steps:**
1. `git revert HEAD` (or `git checkout HEAD~1 -- .claude/skills/`) to restore old flat files
2. If new folders were created, `rm -rf .claude/skills/*/` (folder-based skills)
3. Restore orchestrator if modified: `git checkout HEAD~1 -- .claude/agents/sdlc-orchestrator.md`
4. Verify: Run `/sdlc` on test issue to confirm restoration

**Time to rollback:** < 2 minutes (single git revert)

### Memory System Rollback

**Trigger:** Memory system causes context bloat or unexpected behavior.

**Steps:**
1. Delete memory files: `rm -rf ~/.claude/projects/{hash}/memory/`
2. Revert `/retro` command: `git checkout HEAD~1 -- .claude/commands/retro.md`
3. CLAUDE.md learnings unaffected (primary tier intact)

**Time to rollback:** < 1 minute (delete directory + git checkout)

### Documentation Rollback

**Trigger:** Documentation changes introduce confusion.

**Steps:**
1. `git revert` the documentation commit(s)
2. No runtime impact — documentation is advisory only

**Key principle:** Each phase can be rolled back independently. The memory system (Phase 3) has zero coupling to the skill migration (Phases 1-2), so rolling back one doesn't require rolling back the other.
