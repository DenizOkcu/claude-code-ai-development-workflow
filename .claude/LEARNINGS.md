# Learnings (auto-updated by /retro)
<!-- The /retro command appends lessons learned here automatically -->

### 2026-03-15 — add-repo-context-engine

- **Embed mandatory workflow tools in existing phase entry points, not as optional standalone commands.** Users follow the happy path — they won't run `/repo-map` manually, but they will run `/discover`. Integrate the tool into the phase they already use.
- **For prompt-engineering projects, CLAUDE.md updates belong in the implementation phase, not the deploy phase.** Deferring them means they get forgotten. Add "update CLAUDE.md" as an explicit task in the implementation acceptance criteria for any workflow tooling change.
- **Exclusion lists that exist in multiple prompt files will drift.** If two commands need the same exclusion list (e.g., `repo-map.md` and `discover.md`), add a cross-reference note to one pointing to the other as canonical. Prevents silent divergence over time.
- **Progressive truncation with named tiers (< 100 / 100–200 / 200–500 / > 500 files) is better than a hard token cutoff for LLM output budget management.** Hard cutoffs lose structural information unpredictably; tiered degradation (symbols → files → directories → summary) preserves the most useful information at each tier.
- **The Design phase can be skipped for M-sized prompt-only changes; the Observe phase is always skippable for prompt-only changes.** No metrics, dashboards, or alerting to configure. But the Security phase (7a) is still valuable even with no runtime code — STRIDE analysis reliably catches prompt injection and artifact tampering risks.

### 2026-03-15 — add-semantic-retrieval

- **Pin MCP server versions in setup wizard config blocks** (`@0.1.6`, not `@latest`). `@latest` is convenient but creates supply chain risk — a compromised update is silently fetched on next session start. The setup wizard should pin with a comment explaining how to upgrade.
- **Docker `-p PORT:PORT` binds to `0.0.0.0` by default** — network-accessible from all interfaces. Always use `-p 127.0.0.1:PORT:PORT` in setup wizard Docker commands for local dev tooling. Caught via STRIDE network-listener analysis.
- **Ollama embedding dimension auto-detection is unreliable for batch processing** (issue #235). Always set `EMBEDDING_DIMENSION` explicitly (e.g., `768` for nomic-embed-text) and keep `EMBEDDING_BATCH_SIZE=5` when configuring MCP servers with Ollama.
- **Milvus Lite (in-process SQLite) is Python-only** — the Node.js SDK (`@zilliz/milvus2-sdk-node`) does not support it. If an MCP package depends on Milvus in Node.js, Docker is the minimum for local operation. Validate embedded-mode availability per runtime during research.
- **The implicit feature flag pattern (MCP config presence) is the right default for optional enhancements.** No code-level flag needed; the feature is off unless the user runs `/setup`. Session-start checks close the discoverability gap without forcing adoption.

### 2026-02-28 — add-memory-improve-skills

- **The `model:` frontmatter field is officially supported in Claude Code skills and commands (values: `sonnet`, `opus`, `haiku`, `inherit`).** Use it for cost-optimized model routing: Opus for deep reasoning phases (research, design, plan, implement), Sonnet for checklist/template phases (discover, review, security, deploy, observe, retro). Saves ~40-60% on a full SDLC run.
- **Always audit `~/.claude/` before deploying changes that touch the Claude Code environment.** Preserve: `settings.json` (permissions, model), `plugins/` (installed plugins), `claude_desktop_config.json` (MCP servers), existing `memory/` files. These are user-specific and silently lost if overwritten.
- **Non-destructive migration: create alongside, verify, then delete.** For file restructuring, create new structure first, verify it works, then remove old files. Provides safe rollback at every step.
- **Cross-reference audits must include `docs/` and archive paths.** `docs/integration-plan.md` had 7 stale skill file paths. Always grep the full repo, not just `.claude/`.
- **Verify YAML field support against official docs before removing fields.** We removed `model: sonnet` from skills based on incomplete information, then had to re-add it. Check the Claude Code spec first.
