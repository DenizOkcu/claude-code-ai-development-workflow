# Discovery: add-memory-improve-skills

## Summary
Expand the existing Claude Code SDLC workflow to add a persistent memory system and upgrade the current skills from informal `.md` files to the official Anthropic skill format (folder-based with `SKILL.md` + YAML frontmatter + `references/`). This brings the project in line with Anthropic's "The Complete Guide to Building Skills for Claude" (2026) and enables cross-session knowledge retention.

## Problem Statement
**Two core gaps exist:**

1. **No memory system** -- The workflow generates valuable knowledge during retrospectives (`/retro`) and across sessions, but nothing persists. Each new conversation starts from scratch. The auto-memory directory exists but has no structured content.

2. **Skills use an informal format** -- The 5 existing skills (`.claude/skills/*.md`) don't follow Anthropic's official skill specification. They lack YAML frontmatter with proper `name`/`description` fields, aren't packaged as folders with `SKILL.md`, and don't use progressive disclosure (frontmatter -> body -> references). This means:
   - Skills may not trigger automatically based on descriptions
   - No support for `references/` directory for detailed documentation
   - Missing metadata fields (`license`, `compatibility`, `allowed-tools`)
   - Can't be distributed/shared as proper skill packages

**Who is affected:** Anyone using this SDLC workflow template (the repo is a shareable workflow definition).

## Success Criteria
- [ ] All 5 existing skills converted to official folder-based format with `SKILL.md` + YAML frontmatter
- [ ] Each skill has a clear `description` field explaining WHAT it does and WHEN to trigger (under 1024 chars)
- [ ] Skills use progressive disclosure: core instructions in SKILL.md (under 5,000 words), detailed docs in `references/`
- [ ] A memory system is implemented that captures learnings from `/retro` and feeds them back into future sessions
- [ ] MEMORY.md (auto-memory) has structured content with links to topic files
- [ ] Documentation (README.md) updated to reflect the new skill format and memory capabilities
- [ ] Existing commands (`.claude/commands/`) continue to work unchanged
- [ ] No regression in the SDLC orchestrator workflow

## Scope
### In Scope
- Convert 5 skills from `.claude/skills/*.md` to `.claude/skills/{name}/SKILL.md` folder format
- Add proper YAML frontmatter (`name`, `description`, `metadata`) to each skill
- Implement progressive disclosure: move detailed content to `references/` subdirectories
- Create a structured memory system using the auto-memory directory
- Add a memory management mechanism (how learnings are saved, organized, and retrieved)
- Update ARCHITECTURE.md to document the new skill format and memory layer
- Update README.md (patch, not rewrite)

### Out of Scope
- Creating entirely new skills beyond the existing 5
- MCP server integrations
- Changes to the `/discover`, `/research`, `/plan`, etc. command files
- CI/CD pipeline or automated testing infrastructure
- API-based skill distribution (skills via API `/v1/skills`)
- Changes to the SDLC orchestrator agent logic

## Stakeholders
- Users: Developers and teams adopting this SDLC workflow template
- Teams: Anyone who clones this repo to bootstrap their Claude Code workflows
- Systems: Claude Code (CLI), Claude.ai (if skills are uploaded)

## Risk Assessment
**Level:** Medium
**Justification:** Renaming/restructuring skills could break existing references in the orchestrator agent and commands. The memory system is additive (low risk), but the skill format migration requires verifying all cross-references still resolve. No application code is at stake -- all files are Markdown/YAML configuration.

## Dependencies
- Anthropic's official skill specification (documented in the reference PDFs)
- Claude Code's skill loading mechanism (progressive disclosure, YAML frontmatter parsing)
- Existing SDLC orchestrator agent (references skills by name)

## Estimated Complexity
**Size:** L
**Reasoning:**
- 5 skills to restructure (each needs careful content split between SKILL.md and references/)
- Memory system design and implementation from scratch
- Cross-reference updates across orchestrator, commands, and architecture docs
- Testing that the restructured skills still trigger correctly
- Documentation updates

## Detected Tech Stack

### Languages & Frameworks
| Technology | Version | Expert Command |
|------------|---------|----------------|
| Markdown | N/A | N/A |
| YAML (frontmatter) | N/A | N/A |
| Claude Code Skills | 2026 Spec | N/A |
| Claude Code Commands | N/A | N/A |

### Infrastructure
| Technology | Expert Command |
|------------|----------------|
| Git | N/A |
| Claude Code CLI | N/A |

### Quality Tooling
| Tool | Status |
|------|--------|
| Linter | N/A (Markdown project) |
| Formatter | N/A (Markdown project) |
| Test Runner | N/A (Markdown project) |
| CI/CD | N/A |
| Pre-commit Hooks | N/A |

### Missing Quality Tooling Recommendations
This is a pure Markdown/YAML configuration project (Claude Code workflow definitions). Traditional linters, test runners, and CI/CD don't apply. Quality is ensured through:
- Skill validation: YAML frontmatter correctness, description quality, trigger testing
- Manual testing: running skills in Claude Code and verifying correct activation
- The `/review` command in the workflow itself serves as the quality gate

### Fallback Expert Commands
- **`/language/software-engineer-pro`** -- For general patterns around modularity, progressive disclosure, and clean architecture applied to the skill/command structure

## Reference Documents Analyzed

### Anthropic Official Guide (33 pages)
- **File structure:** `your-skill-name/SKILL.md` + `scripts/` + `references/` + `assets/`
- **YAML frontmatter:** Required fields: `name` (kebab-case), `description` (what + when, under 1024 chars)
- **Optional fields:** `license`, `compatibility`, `allowed-tools`, `metadata` (author, version, mcp-server, category, tags)
- **Progressive disclosure:** Level 1 (frontmatter, always loaded) -> Level 2 (SKILL.md body, loaded on match) -> Level 3 (references/, loaded on demand)
- **3 skill categories:** Document & Asset Creation, Workflow Automation, MCP Enhancement
- **5 patterns:** Sequential Workflow, Multi-MCP Coordination, Iterative Refinement, Context-Aware Tool Selection, Domain-Specific Intelligence
- **Testing:** Triggering tests (90% auto-activation target), Functional tests, Performance comparison
- **Size limit:** Keep SKILL.md under 5,000 words; use `references/` for detailed docs
- **Security:** No XML tags in frontmatter, no "claude"/"anthropic" in skill names, no code execution in YAML

### Medium Article Summary (by Amanraj)
- Distills the official guide into actionable steps
- Emphasizes description quality as the #1 factor for skill effectiveness
- Pre-upload checklist: folder kebab-case, SKILL.md exact spelling, `---` delimiters, description explains WHAT and WHEN
- Pattern: `[What it does] + [When to use it] + [Key capabilities]`

## Current vs. Target State

### Current Skills (informal format)
```
.claude/skills/
├── code-research.md       (flat file, no frontmatter)
├── solution-planning.md   (flat file, no frontmatter)
├── code-implementation.md (flat file, no frontmatter)
├── code-review.md         (flat file, no frontmatter)
└── review-fix.md          (flat file, no frontmatter)
```

### Target Skills (official format)
```
.claude/skills/
├── code-research/
│   ├── SKILL.md           (YAML frontmatter + core instructions)
│   └── references/
│       └── research-patterns.md
├── solution-planning/
│   ├── SKILL.md
│   └── references/
│       └── planning-templates.md
├── code-implementation/
│   ├── SKILL.md
│   └── references/
│       └── implementation-guidelines.md
├── code-review/
│   ├── SKILL.md
│   └── references/
│       └── review-checklist.md
└── review-fix/
    ├── SKILL.md
    └── references/
        └── fix-patterns.md
```

### Memory System (new)
```
~/.claude/projects/{project-hash}/memory/
├── MEMORY.md              (index, auto-loaded, kept under 200 lines)
├── patterns.md            (stable patterns confirmed across sessions)
├── decisions.md           (key architectural decisions)
├── learnings.md           (lessons from /retro, debugging insights)
└── preferences.md         (user workflow preferences)
```
