# AI-Powered Software Development Lifecycle (SDLC) — Extended Edition

> A comprehensive, 10-phase slash-command workflow for Claude Code that covers the **entire** software development lifecycle — from discovery through post-deployment observability.

Built by synthesizing [DenizOkcu/claude-code-ai-development-workflow](https://github.com/DenizOkcu/claude-code-ai-development-workflow) (structured Research → Plan → Execute → Review commands) with [OmarKAly22/llm-knowledge-hub](https://github.com/OmarKAly22/llm-knowledge-hub) (LLM best practices, prompt engineering, agentic AI patterns, RAG, security, and evaluation), then extending both with DevOps, observability, knowledge-capture, and multi-agent orchestration phases.

---

## Why This Exists

Most AI-assisted coding workflows stop at "write code → review code." Real software delivery has **10+ distinct activities** that benefit from structured AI assistance. This project fills the gaps:

| Gap in Existing Workflows | How This Project Addresses It |
|---|---|
| No threat modeling or security audit phase | `/security-audit` command with OWASP, STRIDE, dependency scanning |
| No architecture decision records | `/design-system` produces ADRs + system diagrams |
| No performance/load testing phase | `/perf-test` generates benchmarks, profiles, and load scripts |
| No deployment automation guidance | `/deploy-plan` creates rollout strategy + rollback playbook |
| No post-deploy observability | `/observe` sets up logging, metrics, alerts, and dashboards |
| No knowledge capture / retrospective | `/retro` generates lessons-learned docs and updates CLAUDE.md |
| No multi-feature orchestration | Parallel issue tracking via `STATUS.md` per feature |
| No LLM/AI-specific development patterns | `/ai-integrate` for prompt engineering, RAG, eval, and guardrails |

---

## Quick Start

```bash
# Copy this entire .claude/ directory into your project root.
cp -r .claude/ /path/to/your/project/.claude/

# Start with discovery on a new feature:
/discover Add real-time collaborative editing to the document editor

# This generates an issue name (e.g., "add-realtime-collab") and kicks off the 10-phase workflow.
```

---

## The 10-Phase Workflow

```
┌─────────────┐    ┌──────────────┐    ┌────────────────┐    ┌──────────────┐
│ 1. DISCOVER  │───▶│ 2. RESEARCH  │───▶│ 3. DESIGN      │───▶│ 4. PLAN      │
│ /discover    │    │ /research    │    │ /design-system │    │ /plan        │
└─────────────┘    └──────────────┘    └────────────────┘    └──────────────┘
                                                                     │
       ┌─────────────────────────────────────────────────────────────┘
       ▼
┌──────────────┐    ┌────────────────┐    ┌──────────────┐    ┌──────────────┐
│ 5. IMPLEMENT │───▶│ 6. REVIEW      │───▶│ 7. SECURITY  │───▶│ 8. DEPLOY    │
│ /implement   │    │ /review        │    │ /security    │    │ /deploy-plan │
└──────────────┘    └────────────────┘    └──────────────┘    └──────────────┘
                                                                     │
       ┌─────────────────────────────────────────────────────────────┘
       ▼
┌──────────────┐    ┌──────────────┐
│ 9. OBSERVE   │───▶│ 10. RETRO    │
│ /observe     │    │ /retro       │
└──────────────┘    └──────────────┘
```

### Phase Summaries

| # | Phase | Command | Artifacts Produced |
|---|-------|---------|-------------------|
| 1 | **Discover** | `/discover [description]` | Issue name, `DISCOVERY.md`, `STATUS.md` |
| 2 | **Research** | `/research {issue}` | `CODE_RESEARCH.md`, updated `STATUS.md` |
| 3 | **Design** | `/design-system {issue}` | `ARCHITECTURE.md`, `ADR-*.md`, `PROJECT_SPEC.md` |
| 4 | **Plan** | `/plan {issue}` | `IMPLEMENTATION_PLAN.md`, test strategy |
| 5 | **Implement** | `/implement {issue}` | Source code, tests, updated `STATUS.md` |
| 6 | **Review** | `/review {issue}` | `CODE_REVIEW.md`, approval/rejection status |
| 7 | **Security** | `/security {issue}` | `SECURITY_AUDIT.md`, dependency report |
| 8 | **Deploy** | `/deploy-plan {issue}` | `DEPLOY_PLAN.md`, rollback playbook |
| 9 | **Observe** | `/observe {issue}` | `OBSERVABILITY.md`, alert definitions |
| 10 | **Retro** | `/retro {issue}` | `RETROSPECTIVE.md`, CLAUDE.md updates |

---

## Bonus Commands

| Command | Purpose |
|---------|---------|
| `/ai-integrate {issue}` | Add LLM/AI capabilities — prompt design, RAG, eval, guardrails |
| `/perf-test {issue}` | Performance testing — benchmarks, profiling, load tests |
| `/hotfix [description]` | Compressed emergency workflow (research → fix → review → deploy) |

## Language & Cloud Expert Commands

Auto-detected during `/discover` based on your project's tech stack:

| Command | Focus |
|---------|-------|
| `/language/typescript-pro [desc]` | Strict types, generics, branded types, discriminated unions |
| `/language/javascript-react-pro [desc]` | ES2024+, React 19, Server Components, accessibility, performance |
| `/language/php-pro [desc]` | PHP 8.2+, strict types, enums, readonly, Laravel/Symfony patterns |
| `/language/python-pro [desc]` | Python 3.11+, typing, dataclasses, Protocols, FastAPI/Django, Ruff + mypy |
| `/language/terraform-pro [desc]` | Modules, `for_each`, validation, state isolation, security, tflint/trivy |
| `/language/aws-pro [desc]` | Well-Architected, service selection, IAM, VPC, cost optimization |
| `/language/azure-pro [desc]` | Well-Architected, Managed Identity, Bicep, Entra ID, Defender |
| `/language/gcp-pro [desc]` | Architecture Framework, Workload Identity, Cloud Run, SRE practices |
| `/language/ansible-pro [desc]` | Roles, idempotency, Vault encryption, Molecule testing, dynamic inventory |
| `/language/kubernetes-pro [desc]` | Deployments, RBAC, NetworkPolicies, probes, security contexts, GitOps |
| `/language/openshift-pro [desc]` | Routes, SCCs, BuildConfigs, ImageStreams, Operators, User Workload Monitoring |
| `/language/software-engineer-pro [desc]` | **Fallback** — SOLID, clean architecture, API design, testing, refactoring (any language) |
| `/language/cloud-engineer-pro [desc]` | **Fallback** — Provider-agnostic networking, IAM, IaC, observability, DR, cost control |

## Quality Commands

Run anytime — auto-detect your stack and apply appropriate tooling:

| Command | Purpose |
|---------|---------|
| `/quality/code-audit [scope]` | Full code quality analysis — static analysis, metrics, architecture review |
| `/quality/test-strategy [scope]` | Design test pyramid, generate configs (Vitest/Pytest/PHPUnit), scaffold examples |
| `/quality/lint-setup [scope]` | Configure linter, formatter, pre-commit hooks, editor config |
| `/quality/dependency-check` | Vulnerability scan, outdated packages, license audit, bundle bloat |

---

## File Organization

```
your-project/
├── .claude/
│   ├── commands/                    # Slash commands (the workflow engine)
│   │   ├── discover.md              # Phase 1: Discovery & scoping
│   │   ├── research.md              # Phase 2: Codebase & ecosystem research
│   │   ├── design-system.md         # Phase 3: Architecture & system design
│   │   ├── plan.md                  # Phase 4: Implementation planning
│   │   ├── implement.md             # Phase 5: Code implementation
│   │   ├── review.md                # Phase 6: Code review & QA
│   │   ├── security.md              # Phase 7: Security audit
│   │   ├── deploy-plan.md           # Phase 8: Deployment strategy
│   │   ├── observe.md               # Phase 9: Observability setup
│   │   ├── retro.md                 # Phase 10: Retrospective
│   │   ├── ai-integrate.md          # Bonus: LLM/AI integration
│   │   ├── perf-test.md             # Bonus: Performance testing
│   │   ├── hotfix.md                # Bonus: Emergency hotfix workflow
│   │   ├── language/
│   │   │   ├── typescript-pro.md    # TypeScript expert mode
│   │   │   ├── javascript-react-pro.md  # JavaScript + React expert mode
│   │   │   ├── php-pro.md          # PHP expert mode
│   │   │   ├── python-pro.md       # Python expert mode
│   │   │   ├── terraform-pro.md    # Terraform / IaC expert mode
│   │   │   ├── aws-pro.md          # AWS architecture expert mode
│   │   │   ├── azure-pro.md        # Azure architecture expert mode
│   │   │   ├── gcp-pro.md          # GCP architecture expert mode
│   │   │   ├── ansible-pro.md      # Ansible automation expert mode
│   │   │   ├── kubernetes-pro.md   # Kubernetes workloads expert mode
│   │   │   ├── openshift-pro.md    # OpenShift enterprise expert mode
│   │   │   ├── software-engineer-pro.md  # Fallback: any language
│   │   │   └── cloud-engineer-pro.md     # Fallback: any cloud/infra
│   │   ├── quality/
│   │   │   ├── code-audit.md        # Full code quality analysis
│   │   │   ├── test-strategy.md     # Test pyramid setup & config
│   │   │   ├── lint-setup.md        # Linter, formatter, hooks setup
│   │   │   └── dependency-check.md  # Vulnerability & license audit
│   │   └── devops/
│   │       └── ci-pipeline.md       # CI/CD pipeline generation
│   ├── planning/                    # Auto-generated per issue
│   │   └── {issue-name}/
│   │       ├── STATUS.md            # Central progress dashboard
│   │       ├── DISCOVERY.md
│   │       ├── CODE_RESEARCH.md
│   │       ├── ARCHITECTURE.md
│   │       ├── ADR-001-*.md
│   │       ├── PROJECT_SPEC.md
│   │       ├── IMPLEMENTATION_PLAN.md
│   │       ├── CODE_REVIEW.md
│   │       ├── SECURITY_AUDIT.md
│   │       ├── DEPLOY_PLAN.md
│   │       ├── OBSERVABILITY.md
│   │       └── RETROSPECTIVE.md
│   ├── agents/                      # Multi-agent orchestration
│   │   └── sdlc-orchestrator.md    # Autonomous SDLC agent (Research→Plan→Implement→Review)
│   ├── skills/                      # Folder-based skills (Anthropic official format)
│   │   ├── researching-code/
│   │   │   └── SKILL.md            # Codebase research skill (model: opus)
│   │   ├── planning-solutions/
│   │   │   └── SKILL.md            # Implementation planning skill (model: opus)
│   │   ├── implementing-code/
│   │   │   └── SKILL.md            # Code implementation skill (model: opus)
│   │   ├── reviewing-code/
│   │   │   └── SKILL.md            # Code review skill (model: sonnet)
│   │   └── review-fix/
│   │       └── SKILL.md            # Review fix skill (model: sonnet)
│   └── settings.json                # Claude Code project settings
├── CLAUDE.md                        # Project-level AI instructions
└── docs/
    └── guides/
        ├── ai-integration-guide.md  # How to add LLM features
        └── sdlc-reference.md        # Full workflow reference
```

---

## Skills Format

Skills follow the [Anthropic official skill specification](https://docs.anthropic.com). Each skill is a folder under `.claude/skills/` with a `SKILL.md` file:

```
.claude/skills/{skill-name}/
├── SKILL.md            # Required: YAML frontmatter + instructions
└── references/         # Optional: supplementary docs (loaded on demand)
```

**SKILL.md structure:**
```yaml
---
name: skill-name          # kebab-case, matches folder name
description: "What it does. Use when [trigger]. [Capabilities]."
model: opus               # optional: sonnet, opus, haiku, inherit
metadata:
  version: 1.0.0
  category: workflow-automation
---
# Skill Title
## Goal
## Instructions
## Output Format
## Quality Check
## Common Issues
```

Skills are loaded progressively: YAML frontmatter is always in context (~100 tokens), the full SKILL.md body loads only when the skill triggers (~2K tokens), and `references/` files load on demand.

---

## Model Routing

Each phase uses a cost-appropriate model via the `model:` field in YAML frontmatter. Deep reasoning phases get Opus; checklist/template phases get Sonnet (~5x cheaper per token).

| Phase | Command | Model | Rationale |
|-------|---------|-------|-----------|
| 1. Discover | `/discover` | sonnet | Stack detection, checklist scanning |
| 2. Research | `/research` | opus | Deep architectural reasoning |
| 3. Design | `/design-system` | opus | Architecture decisions, ADRs |
| 4. Plan | `/plan` | opus | Phase sequencing, acceptance criteria |
| 5. Implement | `/implement` | opus | Multi-file code generation, testing |
| 6. Review | `/review` | sonnet | Checklist verification, pattern matching |
| 7. Security | `/security` | sonnet | OWASP/STRIDE checklists |
| 8. Deploy | `/deploy-plan` | sonnet | Document generation from template |
| 9. Observe | `/observe` | sonnet | Document generation from template |
| 10. Retro | `/retro` | sonnet | Summarization, knowledge extraction |

**Cost impact:** 6/10 phases on Sonnet saves ~40-60% per full SDLC run compared to running everything on Opus, with no quality loss on the checklist/template phases.

The `model:` field is officially supported in both commands and skills. Valid values: `sonnet`, `opus`, `haiku`, `inherit`.

---

## Memory System

The workflow includes a two-tier memory system for cross-session knowledge retention:

| Tier | Location | Scope | How It Works |
|------|----------|-------|-------------|
| **Tier 1: Repo-shared** | `CLAUDE.md ## Learnings` | Team-visible, versioned in git | `/retro` appends specific, actionable learnings |
| **Tier 2: Project-personal** | `~/.claude/projects/{hash}/memory/` | Per-user, auto-loaded | MEMORY.md (index, always loaded) + topic files (on demand) |

**Auto-memory files:**
- `MEMORY.md` — Index with quick reference, links to topic files (max 200 lines, auto-loaded every session)
- `patterns.md` — Stable workflow patterns confirmed across sessions
- `decisions.md` — Key architectural decisions
- `learnings.md` — Detailed lessons from `/retro`

The `/retro` command writes to both tiers automatically.

---

## STATUS.md — Your Progress Dashboard

Every command reads and updates `STATUS.md`. It is the single source of truth.

```markdown
# Status: add-realtime-collab

**Risk:** High | **Updated:** 2026-02-21 3:00 PM

## Progress
- [x] Discovery - Completed (scope: WebSocket-based OT)
- [x] Research - Completed (identified conflict resolution patterns)
- [x] Design - Completed (ADR-001: chose CRDT over OT)
- [x] Planning - Completed (4 phases, 12 tasks)
- [~] Implementation - In Progress (Phase 2/4)
- [ ] Review - Not started
- [ ] Security - Not started
- [ ] Deploy - Not started
- [ ] Observe - Not started
- [ ] Retro - Not started

## Key Decisions
- ADR-001: CRDT over OT for conflict resolution (latency vs complexity tradeoff)
- ADR-002: WebSocket with fallback to SSE for transport

## Artifacts
- DISCOVERY.md, CODE_RESEARCH.md, ARCHITECTURE.md
- ADR-001-conflict-resolution.md, ADR-002-transport.md
- PROJECT_SPEC.md, IMPLEMENTATION_PLAN.md
```

---

## Stack Auto-Detection

The `/discover` command automatically scans your project to detect:

**Languages:** TypeScript, JavaScript, PHP, Python, Rust, Go, Ruby — by checking for `tsconfig.json`, `package.json`, `composer.json`, `pyproject.toml`, etc.

**Frameworks:** Next.js, Nuxt, Angular, Vue, Django, FastAPI, Flask, Laravel, Symfony — by inspecting dependency manifests.

**Cloud/Infra:** AWS, Azure, GCP, Terraform, Docker, Kubernetes — by scanning for `.tf` files, `*.bicep`, CDK configs, Dockerfiles, etc.

**Quality Tooling:** ESLint, Prettier, Vitest/Jest, PHPStan, Ruff, pre-commit hooks, CI/CD pipelines — reports what's configured and what's missing.

This detection feeds into `DISCOVERY.md` and `STATUS.md`, so every subsequent phase knows which expert commands (`/language/*-pro`) and quality commands (`/quality/*`) are relevant. If quality tooling gaps are found, the discovery phase recommends fixing them before proceeding to implementation.

---

## Complete Workflow Example

```bash
# Phase 1: Discover — define scope and generate issue name
/discover Add JWT authentication with refresh tokens and role-based access control

# Output: issue name "add-jwt-rbac", DISCOVERY.md, STATUS.md
# STATUS: [x] Discovery | [ ] Research | ...

# Phase 2: Research — deep-dive into codebase and ecosystem
/research add-jwt-rbac

# Output: CODE_RESEARCH.md (existing auth patterns, deps, risks)
# STATUS: [x] Discovery | [x] Research | [ ] Design | ...

# Phase 3: Design — architecture, ADRs, system spec
/design-system add-jwt-rbac

# Output: ARCHITECTURE.md, ADR-001-token-strategy.md, PROJECT_SPEC.md
# STATUS: [x] Discovery | [x] Research | [x] Design | ...

# Phase 4: Plan — detailed implementation plan with phases and tasks
/plan add-jwt-rbac

# Output: IMPLEMENTATION_PLAN.md (4 phases, test strategy)
# STATUS: [x] Discovery | [x] Research | [x] Design | [x] Planning | ...

# Phase 5: Implement — write code phase by phase
/implement add-jwt-rbac

# STATUS: [~] Implementation (Phase 2/4) → [x] Implementation (12 files, 47 tests)

# Phase 6: Review — comprehensive code review and QA
/review add-jwt-rbac

# Output: CODE_REVIEW.md
# STATUS: [x] Review - ✓ APPROVED

# Phase 7: Security audit
/security add-jwt-rbac

# Output: SECURITY_AUDIT.md (threat model, dependency scan, OWASP checklist)
# STATUS: [x] Security - ✓ PASSED (0 critical, 2 low findings)

# Phase 8: Deploy
/deploy-plan add-jwt-rbac

# Output: DEPLOY_PLAN.md (rollout strategy, feature flags, rollback)

# Phase 9: Observe
/observe add-jwt-rbac

# Output: OBSERVABILITY.md (metrics, alerts, dashboard specs)

# Phase 10: Retro
/retro add-jwt-rbac

# Output: RETROSPECTIVE.md, updates to CLAUDE.md with learnings
# STATUS: ALL PHASES COMPLETE ✓
```

---

## When to Skip Phases

Not every change needs all 10 phases. Use judgment:

| Change Type | Recommended Phases |
|---|---|
| Typo fix | Just fix it directly |
| Small bug fix | Research → Implement → Review |
| Medium feature | Discover → Research → Plan → Implement → Review |
| Large feature | All 10 phases |
| Security-critical | All 10 phases, emphasize Security + Observe |
| Hotfix/emergency | `/hotfix` (compressed: Research → Fix → Review → Deploy) |
| AI/LLM feature | Add `/ai-integrate` between Design and Plan |

---

## Parallel Feature Development

```bash
# Start two features simultaneously
/discover Add OAuth2 authentication        # → generates: add-oauth-auth
/discover Fix memory leak in data pipeline  # → generates: fix-data-pipeline-leak

# Each gets its own directory and STATUS.md
.claude/planning/add-oauth-auth/STATUS.md
.claude/planning/fix-data-pipeline-leak/STATUS.md

# Continue each workflow independently
/research add-oauth-auth
/research fix-data-pipeline-leak
```

---

## CLAUDE.md — Project Intelligence

The `CLAUDE.md` file at your project root is a living document. The `/retro` command appends lessons learned automatically:

```markdown
# Project: MyApp

## Architecture
- Next.js 15 App Router + TypeScript
- Prisma + PostgreSQL
- Tailwind CSS + shadcn/ui

## Commands
- `npm run dev` — development server
- `npm run build` — production build
- `npm run test` — test suite (vitest)
- `npm run lint` — ESLint

## Conventions
- Use `interface` over `type` for object shapes
- Error boundaries on all async components
- All API routes return `{ data, error, meta }` envelope

## Learnings (auto-updated by /retro)
- 2026-02-15: Always invalidate JWT on password change (add-jwt-rbac)
- 2026-02-10: Use connection pooling for WebSocket scaling (add-realtime-collab)
```

---

## Credits

This project synthesizes and extends:

- **[claude-code-ai-development-workflow](https://github.com/DenizOkcu/claude-code-ai-development-workflow)** by DenizOkcu — the original 4-phase slash command workflow (Research → Plan → Execute → Review)
- **[llm-knowledge-hub](https://github.com/OmarKAly22/llm-knowledge-hub)** by OmarKAly22 — comprehensive LLM development guides, agentic AI patterns, RAG, security, evaluation, and best practices

Extended with: Discovery, Architecture/ADR, Security Audit, Deployment, Observability, Retrospective phases, AI/LLM integration commands, performance testing, hotfix workflow, multi-agent orchestration patterns, and self-improving CLAUDE.md via automated retrospectives.

## License

MIT
