# Security Audit: add-semantic-retrieval

**Auditor:** Claude (static analysis — OWASP, STRIDE)
**Date:** 2026-03-15
**Scope:** Semantic code retrieval via `@zilliz/claude-context-mcp`, Ollama, Docker Milvus

---

## Attack Surface Summary

This feature introduces **3 new trust boundaries** and **2 network listeners**:

```
Trust Boundaries:
  TB1: Claude Code ↔ claude-context MCP (stdio, same machine)
  TB2: claude-context MCP ↔ Ollama (HTTP, localhost:11434)
  TB3: claude-context MCP ↔ Milvus (gRPC, localhost:19530)

Network Listeners:
  NL1: Ollama on 127.0.0.1:11434 (HTTP API)
  NL2: Milvus on 0.0.0.0:19530 (gRPC) + 0.0.0.0:9091 (metrics)
```

**Key observation:** No custom code is introduced by this change — only Markdown configuration and instructions. The actual code executing is the adopted `@zilliz/claude-context-mcp` npm package plus Ollama and Milvus. Security analysis focuses on configuration, data flow, and trust assumptions.

---

## 1. Threat Modeling (STRIDE)

### Component: claude-context MCP Server (`@zilliz/claude-context-mcp`)

| Threat | Description | Mitigation | Status |
|--------|-------------|------------|--------|
| **S**poofing | A malicious npm package could impersonate `@zilliz/claude-context-mcp` via typosquatting | Package is published by `@zilliz` (verified org on npm). Using `npx -y @zilliz/claude-context-mcp@latest` fetches from official registry. | Mitigated |
| **T**ampering | MCP server runs via `npx` — fetches latest version on each launch, which could be compromised via supply chain attack | Pin to specific version (`@0.1.6`) in config rather than `@latest`. Setup wizard currently uses `@latest`. | See Finding F-01 |
| **R**epudiation | MCP tool calls are not logged beyond Claude Code's session | Claude Code logs tool calls in its conversation transcript. No persistent audit log, but acceptable for a local dev tool. | Accepted Risk |
| **I**nformation Disclosure | Code chunks + embeddings are sent to Ollama (local) and stored in Milvus (local). In cloud config (Options 2-3), embeddings are sent to Zilliz Cloud. | Local-first option keeps all data on-machine. Cloud options are clearly documented as requiring data transfer. Setup wizard warns about this. | Mitigated |
| **D**enial of Service | Large repository indexing can consume significant CPU/RAM (Ollama ~1.5GB, Milvus ~500MB) | `EMBEDDING_BATCH_SIZE=5` and `OLLAMA_NUM_PARALLEL=1` limit resource usage. Indexing is user-initiated, not automatic. | Mitigated |
| **E**levation of Privilege | MCP tools are pre-approved in `settings.json` (`mcp__claude-context__*`). If the MCP server is compromised, it could return malicious content. | MCP tools only return text data (code chunks, scores). They cannot execute commands, modify files, or call other tools. The agent decides what to do with results. | Mitigated |

### Component: Milvus Docker Container

| Threat | Description | Mitigation | Status |
|--------|-------------|------------|--------|
| **S**poofing | Another process could connect to Milvus on port 19530 | Milvus binds to `0.0.0.0:19530` by default — accessible from any network interface on the host. | See Finding F-02 |
| **T**ampering | An attacker on the same machine could modify index data in Milvus | Milvus standalone has no auth by default. Any process on the host (or network-reachable machine) can read/write collections. | See Finding F-02 |
| **I**nformation Disclosure | Indexed code chunks are stored unencrypted in Milvus | Data is stored in a Docker volume on the local filesystem. Acceptable for local dev — matches how the filesystem itself is unencrypted. | Accepted Risk |
| **D**enial of Service | Milvus consumes 200-500MB RAM persistently | Documented in architecture. User explicitly opts in via Docker run command. | Mitigated |

### Component: Ollama

| Threat | Description | Mitigation | Status |
|--------|-------------|------------|--------|
| **S**poofing | If Ollama binds to `0.0.0.0`, remote attackers could use it as a free inference endpoint | Ollama defaults to `127.0.0.1:11434` (localhost only). Config uses `http://127.0.0.1:11434`. | Mitigated |
| **I**nformation Disclosure | Code chunks sent to Ollama for embedding are processed locally | Ollama runs entirely on the local machine. No data leaves. | Mitigated |

---

## 2. OWASP Top 10 Checklist

| # | Category | Applicable? | Assessment |
|---|----------|-------------|------------|
| A01 | **Injection** | No | No user input is passed to SQL/shell. MCP tool parameters are structured JSON via the MCP protocol. The adopted package handles Milvus queries internally. |
| A02 | **Broken Authentication** | Partially | Milvus has no auth in standalone mode (F-02). Ollama has no auth. Acceptable for localhost dev tooling. Cloud options use `${ENV_VAR}` for API keys — never hardcoded. |
| A03 | **Sensitive Data Exposure** | Yes | Code is indexed and stored as embeddings + raw chunks in Milvus. In local mode, data stays on-machine. Cloud mode (Zilliz) transmits embeddings over HTTPS. Setup wizard warns about this. No PII-specific handling needed — this indexes code, not user data. |
| A04 | **XXE** | No | No XML processing. |
| A05 | **Broken Access Control** | No | No multi-user access. Single-developer local tool. |
| A06 | **Security Misconfiguration** | Yes | See F-02 (Milvus port binding). See F-01 (`@latest` version pinning). Otherwise, default configs are reasonable. |
| A07 | **XSS** | No | No web UI. Output is text returned to Claude Code CLI. |
| A08 | **Insecure Deserialization** | No | MCP protocol uses JSON messages over stdio. No object deserialization from untrusted sources. |
| A09 | **Known Vulnerabilities** | Partially | `@zilliz/claude-context-mcp` is pre-1.0 (v0.1.6). Known issue #235 (Ollama batch dimension bug) is mitigated via `EMBEDDING_DIMENSION=768`. Cannot run `npm audit` on an npx-launched package without a local install. |
| A10 | **Insufficient Logging** | Yes | MCP tool calls are logged in Claude Code's session transcript. No persistent security audit log, but appropriate for a local dev tool. |

---

## 3. Dependency Audit

### Direct Dependencies (adopted, not installed locally)

| Package | Version | Risk | Notes |
|---------|---------|------|-------|
| `@zilliz/claude-context-mcp` | 0.1.6 | Medium | Pre-1.0, active development. MIT licensed. Published by Zilliz (Milvus creators). 14+ transitive deps including `@zilliz/milvus2-sdk-node`, tree-sitter parsers, embedding providers. |
| `milvusdb/milvus` (Docker) | latest | Low | Mature OSS project by Zilliz/LF AI. Widely deployed. |
| `nomic-embed-text` (Ollama model) | latest | Low | Apache-2.0 licensed. Widely used embedding model. |

**Cannot run `npm audit`** — the package is launched via `npx` and not installed locally. Dependency vulnerabilities in `@zilliz/claude-context-mcp` are managed by the package maintainers.

### Supply Chain Risk

| Risk | Severity | Mitigation |
|------|----------|------------|
| npm package compromise | Medium | Using `@zilliz` scoped package from verified org. Consider pinning version. |
| Docker image compromise | Low | `milvusdb/milvus` is the official image from the Milvus project. Consider pinning image tag. |
| Ollama model tampering | Low | Models downloaded from ollama.com registry with checksum verification. |

---

## 4. Code-Level Security Review

| Check | Result | Notes |
|-------|--------|-------|
| Input validation | N/A | No custom code. MCP tool parameters validated by claude-context-mcp internally. |
| Output encoding | N/A | No web rendering. MCP returns structured JSON. |
| Authentication | See F-02 | Milvus has no auth in standalone mode. |
| Secrets not hardcoded | PASS | All API keys use `${ENV_VAR}` references. Setup wizard explicitly warns against hardcoding. |
| Error messages don't leak internals | PASS | Error handling in retrieval.md gives user-friendly messages without exposing stack traces or internal paths. |
| File access controls | PASS | `CUSTOM_IGNORE_PATTERNS` excludes `.git/**`, preventing indexing of git internals (which may contain auth tokens in config). |

---

## 5. Data Privacy

| Check | Result | Notes |
|-------|--------|-------|
| PII in indexed data | LOW RISK | This indexes source code, not user data. Source code may contain hardcoded secrets (a pre-existing problem, not introduced by this feature). |
| Data retention | PASS | Index persists in Milvus Docker volume. `clear_index` tool provides explicit deletion. `docker rm milvus` removes all data. |
| Data export/deletion | PASS | `clear_index` removes index data. Docker volume can be destroyed. |
| Data in transit | PASS | Local mode: all communication over localhost (stdio, HTTP, gRPC). Cloud mode: HTTPS to Zilliz Cloud. |

---

## 6. Findings

### F-01: `@latest` version tag in MCP config (Low)

**Location:** `setup.md` — all 4 configuration options use `@zilliz/claude-context-mcp@latest`
**Risk:** Supply chain — a compromised update could be auto-fetched on next Claude Code session start.
**Severity:** Low (mitigated by npm's verified org, but best practice is to pin)
**Remediation:** Change `@latest` to `@0.1.6` (or current stable version) in all config JSON blocks. Users can manually update by changing the version number.
**Accepted Risk Alternative:** Using `@latest` ensures users get bug fixes and security patches automatically. Trade-off between supply chain risk and update convenience.

### F-02: Milvus binds to 0.0.0.0 by default (Low)

**Location:** `setup.md:68` — `docker run -d --name milvus -p 19530:19530 -p 9091:9091`
**Risk:** Milvus is network-accessible from any interface, not just localhost. Any machine on the same network can read/write the code index. Port 9091 exposes Milvus metrics.
**Severity:** Low (developer workstation, typically behind NAT/firewall; no auth data at risk — only code chunks)
**Remediation:** Bind to localhost only: `-p 127.0.0.1:19530:19530 -p 127.0.0.1:9091:9091`
**Impact:** Prevents remote access to the Milvus instance while maintaining identical functionality.

---

## Security Audit Summary

| Severity | Count |
|----------|-------|
| Critical | 0 |
| High | 0 |
| Medium | 0 |
| Low | 2 |

## Verdict: PASSED

Both findings are Low severity with straightforward remediations. Neither represents a data loss, unauthorized access, or code execution risk in the context of a local development tool.

### Required Remediations
None required — both findings are Low severity.

### Recommended Remediations (non-blocking)
1. **F-01**: Consider pinning `@zilliz/claude-context-mcp@0.1.6` instead of `@latest` in setup wizard config blocks
2. **F-02**: Bind Milvus Docker ports to localhost (`127.0.0.1:19530:19530`) in setup wizard instructions

### Accepted Risks
1. Milvus standalone has no authentication (acceptable for local dev tooling)
2. Indexed code chunks stored unencrypted in Docker volume (matches filesystem security model)
3. No persistent security audit log for MCP tool calls (session transcript is sufficient for dev tooling)
4. Pre-1.0 package dependency (actively maintained by Zilliz)
