# Retrospective: add-semantic-retrieval

## Summary
- **Started:** 2026-03-15
- **Completed:** 2026-03-15
- **Complexity:** Estimated L — actual M (lighter than anticipated because we adopted a pre-built package)
- **Files Changed:** 6 modified + 2 new command files + 10 planning artifacts
- **Tests Added:** Manual test plan (no automated tests — Markdown/YAML project)
- **Phases Completed:** 9/10 (Discovery, Research, Design, Planning, Implementation, Review, Security, Deploy, AI Integration, Retro — Observe skipped: no runtime metrics to instrument)

---

## What Went Well

- **Adopt > Build paid off immediately.** Discovering `@zilliz/claude-context-mcp` during research saved an estimated 2-4 weeks of custom TypeScript development. The package already implemented Tree-sitter AST chunking, hybrid BM25+vector search, Merkle tree incremental indexing, and MCP protocol — all the hard parts. We delivered the same capability in a single session.

- **The full SDLC produced exactly the right artifacts.** The research phase surfaced Milvus Lite's Node.js gap (critical for the architecture decision), the design phase produced a precise security-relevant ADR-002 (Docker port binding), and the security audit caught the `0.0.0.0` binding issue that would otherwise have been in the wild.

- **Graceful degradation was designed in from the start.** Because the Discovery clearly stated "existing workflow must be unaffected," every skill modification included skip-if-unavailable logic. This made the security audit and review trivially clean.

- **Session-start suggestion** (added post-security, pre-retro) closes the discoverability gap — users who have the toolkit but haven't set up retrieval will be nudged on first session.

- **The implicit feature flag** (MCP config presence) meant zero opt-out friction. Teams can adopt the toolkit update without being forced to set up Ollama + Milvus.

---

## What Could Be Improved

- **Milvus Lite gap was a late discovery.** The initial design assumed Milvus Lite (in-process SQLite) would work with the Node.js SDK, which would have eliminated the Docker requirement. This unknown wasn't surfaced until a research subagent investigated. A pre-design spike on "does this package support embedded mode?" would have saved a planning assumption change.

- **The `@latest` version tag in setup wizard** (security finding F-01) wasn't caught until the security phase. A simple convention — always pin MCP package versions in setup wizards — would prevent this class of issue earlier.

- **No Observe phase was meaningful here.** For a local Markdown toolkit with no runtime metrics, the Observe phase has nothing to instrument. Future retros on this type of project should skip Observe explicitly at the Planning stage to avoid it sitting as an incomplete checkbox.

---

## Surprises / Unknowns Encountered

- **Milvus Lite is Python-only.** The Node.js `@zilliz/milvus2-sdk-node` does not support the in-process SQLite mode. This forced Docker as the minimum for local operation (ADR-002). Documented as a known gap in the Node.js SDK.

- **Ollama has a batch dimension bug** (issue #235) where embedding dimension detection fails after the first batch. Workaround: always explicitly set `EMBEDDING_DIMENSION=768`. This is a quirk that would have caused silent failures in production without the research phase digging into known issues.

- **The package uses `@latest` by default** in its own documentation examples. We followed that convention in the setup wizard, which then became a Low security finding. The right default is to pin.

- **`0.0.0.0` vs `127.0.0.1` Docker port binding** is easy to miss in a `docker run` command. Milvus's official docs use `0.0.0.0` (or omit the binding), making it network-accessible by default. Caught in STRIDE analysis (network listener threat).

---

## Key Technical Learnings

- **When adopting an npm MCP package, always check if it supports embedded/in-process modes.** The Node.js ecosystem often lacks SQLite-embedded alternatives that exist in Python. This affects local-first architecture options significantly.

- **Pin MCP server versions in setup wizard config blocks** (`@0.1.6`, not `@latest`). `@latest` is convenient for users who want updates but creates supply chain risk. The setup wizard should pin, with a comment explaining how to upgrade.

- **Docker `docker run -p PORT:PORT` binds to `0.0.0.0` by default.** Always use `127.0.0.1:PORT:PORT` for developer tooling that doesn't need network access. Add this as a default pattern for any setup wizard that includes Docker commands.

- **Ollama embedding bugs require explicit dimension configuration.** Don't rely on auto-detection when using Ollama for embeddings in MCP servers. Set `EMBEDDING_DIMENSION` explicitly and keep `EMBEDDING_BATCH_SIZE` low (5) for stability.

- **For adopted MCP packages, security analysis shifts to configuration and trust boundaries**, not code review. The key questions become: what does the package do with credentials? what ports does it open? what data does it transmit? This is faster and more accurate than code-level review for third-party packages.

---

## Process Learnings

- **The Design phase is where the critical decision happened** (adopt vs build, Milvus Lite gap). This justifies running a full Design phase even for "toolkit enhancement" issues. The research subagent finding on Milvus Lite directly shaped ADR-002.

- **The Security phase caught a real issue** (Milvus `0.0.0.0` binding) that wasn't in the code review. STRIDE analysis on network listeners is valuable even for local dev tooling. Worth doing for any feature that introduces new ports or external processes.

- **`/ai-integrate` was useful retrospectively** for documenting the embedding model selection rationale and evaluation strategy. It would have been more useful before the implementation phase to confirm the nomic-embed-text workarounds.

- **Observe should be explicitly skipped** during Planning for Markdown/YAML toolkit projects. Add a heuristic: if there are no runtime components (no server, no scheduled jobs, no API), mark Observe as N/A at Planning time.

---

## Patterns to Reuse

- **Pattern: Implicit feature flag via MCP config presence**
  - Where: Any optional MCP server integration
  - Why: Zero configuration needed to disable; feature is off by default; no code-level flag to manage

- **Pattern: Session-start check for optional tooling**
  - Where: Any enhancement that users benefit from but isn't auto-configured
  - Why: Ensures discoverability without forcing adoption; one-time suggestion per session avoids noise

- **Pattern: 4-option setup wizard (local / cloud+local / cloud / compatible-local)**
  - Where: Any MCP integration with multiple hosting choices
  - Why: Covers all user profiles (privacy-first, cloud-first, mixed); established by n8n + firecrawl + retrieval

---

## Anti-Patterns to Avoid

- **Anti-pattern: Using `@latest` in MCP setup wizard configs**
  - Why: Supply chain risk; a compromised update is fetched silently on next session start
  - Instead: Pin to specific version; add a comment with upgrade instructions

- **Anti-pattern: Assuming embedded/in-process modes exist across language ecosystems**
  - Why: Python libraries (chromadb, milvus-lite) often have no Node.js equivalent
  - Instead: Validate embedded mode availability for the target runtime during research phase

- **Anti-pattern: Docker port binding without explicit host address**
  - Why: `-p 19530:19530` binds to `0.0.0.0` (all interfaces), making services network-accessible
  - Instead: Always use `-p 127.0.0.1:19530:19530` for local dev tooling

---

## Metrics

| Metric | Value |
|--------|-------|
| Estimated complexity | L |
| Actual complexity | M (adoption reduced effort significantly) |
| Files created | 2 (commands) + 11 (planning artifacts) |
| Files modified | 6 |
| Review findings | 0 critical, 0 important, 4 suggestions |
| Security findings | 0 critical, 0 high, 0 medium, 2 low (1 fixed) |
| Review iterations | 1 (approved on first pass) |
| Phases completed | 9/10 (Observe N/A) |
