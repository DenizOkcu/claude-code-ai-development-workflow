# AI Integration Analysis: add-memory-improve-skills

**When:** 2026-02-28
**Author:** Claude Code (automated)

---

## 1. Use Case Analysis

This project is itself an AI/LLM integration — it defines how Claude Code (an LLM-powered agent) behaves during software development. The "application" is a prompt engineering system where:

- **Skills** = system prompts that shape Claude's behavior per SDLC phase
- **Memory** = persistent context injection (a lightweight RAG without embeddings)
- **Orchestrator** = an agent workflow with phase gates and iteration limits

| Dimension | Value |
|-----------|-------|
| **Task types** | Code generation, code review, planning, research, knowledge capture |
| **Latency** | Interactive (< 30s per skill invocation via Claude Code CLI) |
| **Accuracy** | High — SDLC artifacts must be actionable, not hallucinated |
| **Volume** | Low — 1-10 skill invocations per developer session |
| **Cost sensitivity** | Medium — uses Opus (most expensive tier) by user preference |

---

## 2. Model Selection

| Criterion | Requirement | Current Selection |
|-----------|-------------|-------------------|
| Task complexity | Multi-file code generation, architectural reasoning | **Claude Opus 4.6** (user's `settings.json`: `"model": "opus"`) |
| Latency | Interactive CLI, < 30s per response | Opus is slower but acceptable for SDLC tasks |
| Cost | Developer tooling, not production API | Opus pricing acceptable for low-volume developer use |
| Privacy | Code stays local (CLI tool, not cloud API) | Claude Code runs locally, sends code to Anthropic API |
| Context window | Skills + memory + codebase files | 200K tokens — skills consume ~3,500 tokens total overhead |

### Model Routing — IMPLEMENTED

The `model:` field in YAML frontmatter is officially supported by Claude Code for both skills and commands. Each phase now declares its preferred model:

| Phase | Command | Skill | Model | Rationale |
|-------|---------|-------|-------|-----------|
| 1. Discover | `discover.md` | — | sonnet | Stack detection, checklist scanning |
| 2. Research | `research.md` | `researching-code` | opus | Deep architectural reasoning |
| 3. Design | `design-system.md` | — | opus | Architecture decisions, ADRs |
| 4. Plan | `plan.md` | `planning-solutions` | opus | Phase sequencing, acceptance criteria |
| 5. Implement | `implement.md` | `implementing-code` | opus | Multi-file code generation, testing |
| 6. Review | `review.md` | `reviewing-code` | sonnet | Checklist verification, pattern matching |
| 7. Security | `security.md` | — | sonnet | OWASP/STRIDE checklists |
| 8. Deploy | `deploy-plan.md` | — | sonnet | Document generation from template |
| 9. Observe | `observe.md` | — | sonnet | Document generation from template |
| 10. Retro | `retro.md` | — | sonnet | Summarization, knowledge extraction |
| Orchestrator | `sdlc.md` | — | sonnet | Routing logic, not deep reasoning |
| Fix loop | — | `review-fix` | sonnet | Targeted fixes from explicit instructions |

**Cost impact:** 6/10 phases on Sonnet (~5x cheaper per token than Opus). The 4 Opus phases are where reasoning quality directly impacts output quality.

**Note:** Earlier in this project, `model: sonnet` was removed from skill frontmatter based on incomplete information. The field IS in the official Claude Code spec and has been re-added with phase-appropriate values.

---

## 3. Prompt Engineering Analysis

### 3a. Skill Prompt Architecture

Each skill follows a standardized prompt template:

```
┌─────────────────────────────────────────────┐
│ YAML Frontmatter (Level 1 - always loaded)  │
│ ┌─────────────────────────────────────────┐ │
│ │ name: kebab-case identifier             │ │
│ │ description: [What] + [When] + [Caps]   │ │
│ │ model: opus | sonnet | haiku            │ │
│ │ metadata: version, category             │ │
│ └─────────────────────────────────────────┘ │
├─────────────────────────────────────────────┤
│ SKILL.md Body (Level 2 - on skill match)    │
│ ┌─────────────────────────────────────────┐ │
│ │ Mindset    → Sets behavioral frame      │ │
│ │ Goal       → Defines success criteria   │ │
│ │ Instructions → Step-by-step procedure   │ │
│ │ Output     → Artifact template          │ │
│ │ Quality    → Self-verification gates    │ │
│ │ Common     → Anti-pattern prevention    │ │
│ └─────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

### 3b. Prompt Patterns Used

| Pattern | Where Used | Purpose |
|---------|-----------|---------|
| **Role framing** ("Mindset") | All 5 skills | Sets cognitive stance before task execution |
| **Chain of thought** | researching-code Step 1 | "Think deeply about hidden dependencies" triggers extended thinking |
| **Structured output** | All skills (Output Format) | Markdown templates ensure consistent artifact format |
| **Self-verification** | All skills (Quality Check) | Checklist prevents premature completion |
| **Anti-pattern lists** | All skills (Common Issues) | Explicitly names failure modes to avoid |
| **Counting constraint** | implementing-code | "Count phases explicitly: I see {N} phases" prevents early stopping |
| **Iteration limit** | review-fix | "Max 3 iterations" prevents infinite fix loops |
| **Minimal output** | researching-code | "5-15 files max" constrains exploration scope |

### 3c. Prompt Design Per Feature

#### Feature: researching-code

**System Prompt (Mindset):**
> Understand just enough to plan effectively. Skip comprehensive documentation.

**User Prompt Template:**
```
Research the codebase for issue: {{issue_name}}
Answer: 1) What files will this touch? 2) What patterns to follow? 3) What risks?
```

**Output Format:** RESEARCH.md with Summary, Files to Touch, Patterns, Risks, Open Questions

**Edge Cases:**
- Empty/new codebase → Skill still produces valid RESEARCH.md with "no existing patterns" noted
- Massive codebase → "5-15 files max" constraint prevents runaway exploration
- Ambiguous issue → Open Questions section captures uncertainties

#### Feature: implementing-code

**System Prompt (Mindset):**
> Build working software that solves the problem. The plan is a guide, not a contract.

**User Prompt Template:**
```
Implement all phases from PLAN.md for issue: {{issue_name}}
Count phases explicitly. Validate each phase. Document deviations.
```

**Output Format:** IMPLEMENTATION.md with Changes, Phases Completed, Test Results, Deviations

**Edge Cases:**
- Plan conflicts with reality → "Deviate Wisely" section authorizes divergence with documentation
- Tests fail mid-phase → "Silent failures" anti-pattern says keep implementing, don't stop
- Single-phase plan → Works fine, skill counts "1 phase" and completes

#### Feature: Memory System (context injection)

**System Prompt (implicit — auto-loaded):**
MEMORY.md is injected at session start as supplementary context.

**Template:**
```markdown
# Project Memory
## Quick Reference
- {{key facts about the project}}
## Session Notes
- {{rolling log of recent work}}
```

**Output Format:** N/A — consumed as input context, not generated

**Edge Cases:**
- Empty memory (new project) → No error, Claude Code operates without it
- Memory exceeds 200 lines → Truncated at line 200 by Claude Code
- Stale/wrong memory → Supplementary only; CLAUDE.md is authoritative. User can delete and regenerate.
- Contradictory information → CLAUDE.md (Tier 1) takes precedence over memory (Tier 2)

---

## 4. RAG Architecture

This project implements a **lightweight RAG system without embeddings**:

### Data Sources

| Source | Loaded When | Size Limit | Purpose |
|--------|------------|------------|---------|
| `CLAUDE.md` | Every session (always) | No hard limit | Authoritative project instructions + learnings |
| `MEMORY.md` | Every session (always) | 200 lines | Quick reference, session notes, topic links |
| `patterns.md` | On demand (referenced from MEMORY.md) | No limit | Stable workflow patterns |
| `decisions.md` | On demand (referenced from MEMORY.md) | No limit | Architectural decision log |
| `learnings.md` | On demand (referenced from MEMORY.md) | No limit | Accumulated project insights |
| Skill YAML frontmatter | Every session (always) | ~100 tokens per skill | Skill matching/routing |
| SKILL.md body | On skill activation | ~2,000 tokens per skill | Full skill instructions |

### Retrieval Strategy

```
Session Start
    │
    ├── Load CLAUDE.md (always)           ← Tier 1: authoritative
    ├── Load MEMORY.md (always)           ← Tier 2: supplementary context
    └── Load 5x skill frontmatter (always) ← Skill routing
         │
User Prompt
    │
    ├── Match skill by description        ← Semantic matching (Claude's built-in)
    │   └── Load matched SKILL.md body    ← Full instructions on demand
    │
    └── If skill references memory topic  ← On-demand retrieval
        └── Read patterns.md / decisions.md / learnings.md
```

### Chunking Strategy

| Content | Chunk Strategy | Rationale |
|---------|---------------|-----------|
| MEMORY.md | Single chunk (max 200 lines) | Small enough to load whole |
| Topic files | Single chunk each | Loaded whole on demand |
| CLAUDE.md | Single chunk | Auto-loaded by Claude Code |
| SKILL.md | Split: frontmatter (always) + body (on match) | Progressive disclosure |

### Key Difference from Traditional RAG

- **No embedding model** — Claude Code's built-in understanding matches skills by description text
- **No vector store** — Files on local filesystem, loaded by path
- **No similarity search** — Exact file references, not fuzzy retrieval
- **Deterministic retrieval** — Same input always loads same files (no relevance ranking)
- **Write-back capability** — `/retro` writes new knowledge back to memory (traditional RAG is read-only)

---

## 5. Guardrails & Safety

### Input Validation

| Guard | Implementation | Status |
|-------|---------------|--------|
| Skill name injection | Names validated: `/^[a-z][a-z0-9-]*$/`, no `claude`/`anthropic` | Active |
| YAML injection | No executable code in YAML values, no `!!python/object` | Verified in security audit |
| XML injection | No XML tags in frontmatter | Verified in security audit |
| Path traversal | No absolute filesystem paths in repo files | Verified in security audit |
| Secret leakage | No credentials, API keys, PII in any skill or memory file | Verified in security audit |
| Context overflow | MEMORY.md capped at 200 lines; skills under 5,000 words each | Enforced by design |

### Output Validation

| Guard | Implementation | Status |
|-------|---------------|--------|
| Artifact schema | Each skill defines exact output format (RESEARCH.md, PLAN.md, etc.) | Active |
| Self-verification | Quality Check section in every skill — checklist must pass | Active |
| Review gate | `reviewing-code` skill produces APPROVED/NEEDS_FIX verdict | Active |
| Iteration limit | `review-fix` max 3 iterations, then escalates to human | Active |
| Phase counting | `implementing-code` explicitly counts phases to prevent early stopping | Active |

### Fallback Behavior

| Failure Mode | Fallback |
|-------------|----------|
| Skill doesn't activate | User can invoke manually via `/implement`, `/review`, etc. |
| Memory file missing | Claude Code operates without it (memory is supplementary) |
| Memory file corrupted | Delete `memory/` directory, start fresh. CLAUDE.md preserves core learnings. |
| Review loop stuck (3 iterations) | Escalate to user with clear explanation of blockers |
| Orchestrator gate fails | Stops workflow, reports which gate failed and why |

### Prompt Injection Protection

| Vector | Protection |
|--------|-----------|
| Skill content injection | Skills are git-tracked; changes visible in version control |
| Memory poisoning | Memory is supplementary, not authoritative. CLAUDE.md is the source of truth. |
| User input in skill templates | Skills use `{{variables}}` from STATUS.md/PLAN.md (internal, not user-facing) |
| Cross-skill interference | Skills are stateless — no skill can modify another skill's behavior |

---

## 6. Evaluation & Testing

### Current Eval Strategy

| Metric | Method | Target | Current |
|--------|--------|--------|---------|
| Skill activation accuracy | Manual: invoke 10 relevant prompts | ≥ 90% (9/10) | Not yet measured (post-deploy) |
| Artifact quality | Manual: review generated RESEARCH.md, PLAN.md, etc. | Actionable, not hallucinated | Verified during this SDLC run |
| Memory persistence | Manual: write via /retro, check in new session | 100% retention | Verified during implementation |
| Context overhead | Token count: frontmatter + MEMORY.md + 1 active skill | < 4,000 tokens | ~3,500 tokens (PASS) |
| Phase completion | Run /sdlc on test issue | All phases complete | Verified during this SDLC run |
| Anti-pattern prevention | Check: does implementing-code complete ALL phases? | 100% | Verified (4/4 phases this run) |

### Recommended Eval Dataset (Future)

| Category | Count | Examples |
|----------|-------|---------|
| Happy path | 6 | Standard feature, bug fix, refactor, config change, doc update, migration |
| Edge cases | 2 | Empty codebase, single-file change |
| Adversarial | 1 | Ambiguous/contradictory requirements |
| Regression | 1 | Re-run previous SDLC issue, verify same quality |
| **Total** | **10** | Minimum viable eval set |

### Key Metrics to Track Over Time

1. **Skill trigger accuracy** — Does the right skill activate for natural language prompts?
2. **Phase gate pass rate** — How often do phases pass gates on first attempt?
3. **Review iteration count** — Average fix iterations before APPROVED (target: ≤ 1.5)
4. **Memory usefulness** — Does memory context actually improve outputs? (subjective, track via retro)

---

## 7. Cost Optimization

### Current State

| Component | Token Cost | Frequency |
|-----------|-----------|-----------|
| CLAUDE.md loading | ~5,000 tokens | Every session |
| MEMORY.md loading | ~1,000 tokens | Every session |
| 5x skill frontmatter | ~500 tokens | Every session |
| 1x active SKILL.md body | ~2,000 tokens | Per skill activation |
| Codebase file reads | Variable (500-5,000 per file) | Per skill execution |
| **Overhead per session** | **~8,500 tokens** | Fixed cost |

### Optimization Strategies

| Strategy | Applicable? | Notes |
|----------|------------|-------|
| **Semantic caching** | No | Each SDLC run is unique (different issue, different codebase state) |
| **Prompt compression** | Partially | Skills already optimized (max 541 words). MEMORY.md pruning prevents growth. |
| **Model routing** | Active | Implemented via `model:` frontmatter field. Opus for research/design/plan/implement; Sonnet for discover/review/security/deploy/observe/retro. |
| **Batch processing** | No | Interactive CLI tool, not batch API |
| **Token budget** | Active | MEMORY.md 200-line cap, skill body < 5,000 words, research limited to 5-15 files |

### Cost-Saving Recommendations

1. **Model routing (implemented):** 6/10 phases route to Sonnet (~5x cheaper). Research/design/plan/implement stay on Opus for reasoning quality. Estimated ~40-60% cost reduction on a full SDLC run.
2. **Memory pruning discipline:** Run `/retro` pruning when MEMORY.md Session Notes exceeds 150 lines. Each extra line is context consumed every session.
3. **Skill description quality:** Better descriptions → better auto-activation → fewer manual retries → fewer wasted tokens.

---

## 8. Architecture Summary

```
┌────────────────────────────────────────────────────────────┐
│                    CONTEXT INJECTION                        │
│                                                            │
│  ┌──────────┐  ┌──────────┐  ┌─────────────────────────┐  │
│  │CLAUDE.md │  │MEMORY.md │  │ 5x Skill Frontmatter    │  │
│  │(Tier 1)  │  │(Tier 2)  │  │ (name + description)    │  │
│  │~5K tokens│  │~1K tokens│  │ ~500 tokens total       │  │
│  └──────────┘  └──────────┘  └─────────────────────────┘  │
│       │              │              │                       │
│       └──────────────┴──────────────┘                       │
│                      │                                      │
│              Claude Code Session                            │
│              (200K context window)                           │
│                      │                                      │
│              User Prompt                                    │
│                      │                                      │
│              ┌───────▼───────┐                              │
│              │ Skill Match   │  ← By description text       │
│              │ (auto or /cmd)│                               │
│              └───────┬───────┘                              │
│                      │                                      │
│              ┌───────▼───────┐                              │
│              │ SKILL.md Body │  ← ~2K tokens on demand      │
│              │ (full prompt) │                               │
│              └───────┬───────┘                              │
│                      │                                      │
│              ┌───────▼───────┐                              │
│              │ Execute Skill │                               │
│              │ (read files,  │                               │
│              │  write code,  │                               │
│              │  run tests)   │                               │
│              └───────┬───────┘                              │
│                      │                                      │
│              ┌───────▼───────┐                              │
│              │ Produce       │                               │
│              │ Artifact      │  → RESEARCH.md, PLAN.md, etc.│
│              └───────┬───────┘                              │
│                      │                                      │
│              ┌───────▼───────┐                              │
│              │ /retro        │  ← Write-back to memory      │
│              │ (knowledge    │  → learnings.md, patterns.md │
│              │  capture)     │  → MEMORY.md, CLAUDE.md      │
│              └───────────────┘                              │
└────────────────────────────────────────────────────────────┘
```

### Key AI Integration Patterns

1. **Progressive Disclosure** — Load minimal context (frontmatter) always; full context (SKILL.md body) only when needed. Saves ~8,000 tokens per unused skill per session.

2. **Write-Back Memory** — Unlike traditional RAG (read-only retrieval), `/retro` writes confirmed learnings back to the knowledge base. This creates a self-improving system.

3. **Two-Tier Authority** — CLAUDE.md (git-tracked, team-shared) is authoritative; auto-memory (local, personal) is supplementary. Prevents memory poisoning from overriding team conventions.

4. **Self-Verification Prompting** — Every skill includes a Quality Check that acts as a self-evaluation step. The LLM verifies its own output before declaring completion.

5. **Bounded Iteration** — Review-fix loop is capped at 3 iterations. This prevents the common LLM failure mode of infinite self-correction loops.

6. **Explicit Counting** — "Count phases explicitly: I see {N} phases" is a prompt engineering technique that prevents the LLM from losing track of multi-step plans.

7. **Model Routing** — Each skill/command declares its preferred model via `model:` frontmatter. Deep reasoning phases use Opus; checklist/template phases use Sonnet. This saves ~40-60% on a full SDLC run without sacrificing quality where it matters.
