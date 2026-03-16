# Architecture: add-memory-improve-skills

## System Context

```
┌─────────────────────────────────────────────────────────────────┐
│                     CLAUDE CODE SESSION                          │
│                                                                  │
│  Auto-loaded on every conversation:                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────────────┐      │
│  │CLAUDE.md │  │MEMORY.md │  │Skill YAML frontmatter    │      │
│  │(repo)    │  │(project) │  │(all skills, Level 1)     │      │
│  └──────────┘  └──────────┘  └──────────────────────────┘      │
│                                                                  │
│  Loaded when skill triggers (Level 2):                          │
│  ┌─────────────────────────────────────────┐                    │
│  │ SKILL.md body (full instructions)        │                    │
│  └─────────────────────────────────────────┘                    │
│                                                                  │
│  Loaded on demand (Level 3):                                    │
│  ┌──────────────────────┐  ┌──────────────────────┐            │
│  │ references/*.md       │  │ memory/topic-files   │            │
│  └──────────────────────┘  └──────────────────────┘            │
└─────────────────────────────────────────────────────────────────┘
```

This shows the three-level progressive disclosure model:
1. **Always loaded:** CLAUDE.md, MEMORY.md (first 200 lines), skill YAML frontmatter
2. **On skill match:** SKILL.md body loaded into context
3. **On demand:** `references/` files, memory topic files

---

## Component Design

### Component 1: Skills (restructured)

**Responsibility:** Provide procedural instructions for each SDLC phase.

**Current state:** Flat `.md` files in `.claude/skills/`
**Target state:** Folder-based with `SKILL.md` + optional `references/`

```
.claude/skills/
├── researching-code/
│   └── SKILL.md
├── planning-solutions/
│   └── SKILL.md
├── implementing-code/
│   └── SKILL.md
├── reviewing-code/
│   └── SKILL.md
└── review-fix/
    └── SKILL.md
```

**Design decisions:**
- Folder names = current `name` fields (preserves orchestrator compatibility)
- No `references/` initially (all skills under 5,000 words)
- No `scripts/` or `assets/` (not needed for workflow automation skills)
- `model` field removed from frontmatter (not in official spec)
- `description` fields rewritten to official pattern: `[What] + [When] + [Capabilities]`
- `metadata` added with version and category

**SKILL.md internal structure (standardized across all 5):**
```markdown
---
name: {skill-name}
description: {What it does}. Use when {trigger conditions}. {Key capabilities}.
metadata:
  version: 1.0.0
  category: workflow-automation
---

# {Skill Title}

**Mindset:** {one-line philosophy}

## Goal
{What this skill accomplishes}

## Instructions

### Step 1: {Action}
{Specific, actionable instructions}

### Step 2: {Action}
{...}

## Output Format
{Template for the artifact this skill produces}

## Quality Check
{Checklist before marking complete}

## Common Issues
{Error handling and troubleshooting}
```

### Component 2: Memory System (new)

**Responsibility:** Persist cross-session knowledge at two levels.

**Architecture:**
```
Memory Layer (two tiers):

Tier 1 - Repo-shared (in git, visible to all users):
  CLAUDE.md ## Learnings section
  └── Specific, actionable lessons from /retro
  └── Referenced by date and issue name

Tier 2 - Project-personal (auto-memory, per user):
  ~/.claude/projects/{hash}/memory/
  ├── MEMORY.md        (index, max 200 lines, always loaded)
  ├── patterns.md      (stable workflow patterns)
  ├── decisions.md     (architectural decisions from this project)
  └── learnings.md     (accumulated lessons, detailed)
```

**Data flow:**
```
/retro command completes
    ├── Writes to CLAUDE.md ## Learnings (repo-shared)
    └── Writes to auto-memory:
        ├── MEMORY.md (update index with new entry)
        └── learnings.md (append detailed lesson)

New session starts
    ├── CLAUDE.md loaded (repo instructions + learnings)
    └── MEMORY.md loaded (first 200 lines, project context)
        └── References patterns.md, decisions.md, learnings.md (on demand)
```

**MEMORY.md structure:**
```markdown
# Project Memory

## Quick Reference
- SDLC workflow: 10-phase, 5 core skills
- Architecture: commands -> orchestrator -> skills -> artifacts
- Skill format: folder-based with SKILL.md + YAML frontmatter

## Key Patterns
See [patterns.md](patterns.md) for stable workflow patterns.

## Architectural Decisions
See [decisions.md](decisions.md) for key technical decisions.

## Lessons Learned
See [learnings.md](learnings.md) for accumulated project insights.

## Session Notes
{Brief notes from recent sessions - oldest entries pruned first}
```

### Component 3: Cross-reference Updates (modified)

**Files that need updates when skills are restructured:**

| File | What Changes | Scope |
|------|-------------|-------|
| `.claude/ARCHITECTURE.md` | Skills table, visual diagram, add memory layer | Significant |
| `.claude/commands/COMMAND_USAGE.md` | Skills list in architecture section | Minor |
| `docs/integration-plan.md` | Skill file path references | Minor (historical doc) |
| `.claude/STATE_MANAGEMENT.md` | Archive to `docs/archive/` | Remove from active |
| `README.md` | Add skills format + memory section | Patch only |

**Files that DON'T change (verified):**
- `.claude/agents/sdlc-orchestrator.md` -- references skill `name` fields which stay the same
- `.claude/commands/sdlc.md` -- invokes orchestrator, not skills
- All 30+ command files -- no skill references
- `settings.json` -- no skill references

### Component 4: /retro Command Enhancement (modified)

**Current behavior:** Writes learnings to `CLAUDE.md ## Learnings`
**New behavior:** Also writes to auto-memory directory

**Change scope:** Add 1 section to `retro.md` command instructing it to update memory files.

---

## Data Model

### Skill YAML Frontmatter Schema

```yaml
# Required fields
name: string          # kebab-case, matches folder name
description: string   # Under 1024 chars, includes WHAT + WHEN + capabilities

# Optional fields
metadata:
  version: string     # semver (e.g., "1.0.0")
  category: string    # "workflow-automation"
  author: string      # optional
```

### Memory File Schema

**MEMORY.md** (max 200 lines):
```
Line 1-5:     # Header + Quick Reference
Line 6-30:    ## Key Patterns (summary with links)
Line 31-60:   ## Decisions (summary with links)
Line 61-100:  ## Lessons Learned (summary with links)
Line 101-150: ## Session Notes (rolling, oldest pruned)
Line 151-200: (buffer for growth)
```

**patterns.md** (no size limit, loaded on demand):
```markdown
# Workflow Patterns

## Pattern: {name}
- **Context:** {when this applies}
- **Solution:** {what to do}
- **Confirmed:** {date, issue name}
```

**decisions.md** (no size limit, loaded on demand):
```markdown
# Architectural Decisions

## Decision: {title}
- **Date:** {date}
- **Context:** {why}
- **Decision:** {what}
- **Status:** Active | Superseded
```

**learnings.md** (no size limit, loaded on demand):
```markdown
# Lessons Learned

## {date} - {issue-name}
- {specific, actionable learning}
- {specific, actionable learning}
```

---

## API Design

N/A -- This is a Markdown/YAML configuration project. No HTTP APIs, REST endpoints, or programmatic interfaces. The "API" is the file structure convention that Claude Code reads.

**Skill "interface contract":**
- Input: YAML frontmatter triggers skill loading
- Output: Markdown artifacts in `docs/{issue-name}/` or `.claude/planning/{issue-name}/`

**Memory "interface contract":**
- Input: `/retro` writes to memory directory
- Output: Auto-loaded MEMORY.md provides context in new sessions

---

## State Management

### Skill State
Skills are stateless procedures. State flows through artifacts:
```
RESEARCH.md -> PLAN.md -> IMPLEMENTATION.md -> REVIEW.md
```
No change to this model. Skills restructuring is purely structural.

### Memory State
Memory is append-mostly with periodic pruning:
```
/retro writes -> learnings.md (append)
              -> MEMORY.md (update index, prune if over 200 lines)
              -> patterns.md (append if new pattern confirmed)
              -> decisions.md (append if new decision)
```

**Pruning strategy:** When MEMORY.md approaches 200 lines, prune oldest "Session Notes" entries first. Key Patterns, Decisions, and Lessons references are kept.

---

## Error Handling Strategy

| Error | Recovery | User Impact |
|-------|----------|-------------|
| SKILL.md not found in folder | Claude Code falls back to no skill | Skill doesn't trigger; user sees no special behavior |
| Invalid YAML frontmatter | Skill won't load; Claude Code logs warning | User can check SKILL.md format |
| MEMORY.md over 200 lines | Lines after 200 truncated automatically | Oldest session notes lost; key refs preserved at top |
| Memory directory doesn't exist | Create on first `/retro` run | Transparent to user |
| `/retro` fails to write memory | CLAUDE.md learnings still work (primary) | Memory is supplementary; no data loss |

**Key principle:** Memory is supplementary. If it fails, the repo-level CLAUDE.md learnings still work. No critical workflow depends on auto-memory.

---

## Performance Considerations

| Concern | Mitigation |
|---------|-----------|
| MEMORY.md loaded every session | Max 200 lines (~4KB). Negligible overhead |
| 5 skill frontmatters loaded every session | Each ~100 chars description. Total ~500 chars. Negligible |
| SKILL.md bodies loaded on trigger | Only one skill body loaded at a time. Max ~200 lines |
| `references/` files | Loaded on demand only. Not created initially |

**Token budget estimate:**
- MEMORY.md: ~1,000 tokens (200 lines)
- 5 skill frontmatters: ~500 tokens total
- Active SKILL.md body: ~2,000 tokens (200 lines)
- **Total overhead: ~3,500 tokens** (well within 200K context)

---

## Scalability Notes

| Scale | Impact | Notes |
|-------|--------|-------|
| 10 skills | Frontmatter grows to ~1,000 tokens | Still negligible |
| 50 skills | Frontmatter grows to ~5,000 tokens | Consider selective enablement |
| Large memory files | MEMORY.md capped at 200 lines | Topic files grow but are on-demand |
| Many /retro entries | learnings.md grows unbounded | Consider periodic archival (manual) |
| Multi-user teams | Each user has own auto-memory dir | CLAUDE.md learnings shared via git |

**Recommendation from Anthropic guide:** If you have more than 20-50 skills simultaneously, consider skill "packs" for selective enablement. Not a concern for this project (5 skills).
