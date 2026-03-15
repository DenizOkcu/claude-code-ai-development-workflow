---
name: retrieval/setup
description: Interactive setup wizard for semantic code retrieval via claude-context MCP. Configures Ollama + Milvus (local) or Zilliz Cloud for codebase indexing and search.
model: sonnet
---

# Semantic Code Retrieval Setup Wizard

You are guiding the user through setting up the **claude-context MCP** integration, which gives Claude Code semantic code search capabilities — hybrid BM25 + vector search over an AST-indexed codebase.

## Overview

claude-context MCP provides:
- **Index** — Parse your codebase with Tree-sitter AST, generate embeddings, store in a vector database
- **Search** — Hybrid keyword (BM25) + semantic (vector) search over code chunks
- **Incremental updates** — Merkle tree change detection re-indexes only modified files

## Step 1: Check Existing Configuration

First, check if claude-context is already configured:

```bash
cat .claude/settings.json
```

If `claude-context` already exists in `mcpServers`, inform the user:

> claude-context MCP is already configured in this project. Run `/retrieval/setup` again to reconfigure, or use `/retrieval` to search and index.

If not configured, proceed to Step 2.

## Step 2: Ask About Setup Preference

Present these options clearly and **wait for the user's response**:

> ### Semantic Code Retrieval Setup
>
> **How would you like to run semantic retrieval?**
>
> 1. **Fully local** (recommended) — Ollama for embeddings + Docker Milvus for vector storage. No cloud services, no API keys. Requires Docker and Ollama installed.
> 2. **Zilliz Cloud + Ollama** — Local embeddings via Ollama, cloud-managed vector storage via Zilliz Cloud. Requires Ollama + Zilliz Cloud account.
> 3. **Zilliz Cloud + OpenAI** — Cloud embeddings + cloud storage. Easiest setup, but requires API keys and internet. No local infrastructure needed.
> 4. **OpenAI-compatible local** — LM Studio or similar local inference server + Docker Milvus. For users who prefer a different local embedding runtime.
>
> Which option? (1-4)

## Step 3: Collect Configuration Details

### Option 1: Fully Local (Ollama + Docker Milvus)

Guide the user through prerequisites:

> **Local Setup Prerequisites**
>
> **1. Ollama** (embedding model runtime)
> ```bash
> # Install Ollama if not already installed
> # macOS: brew install ollama
> # Linux: curl -fsSL https://ollama.com/install.sh | sh
>
> # Start Ollama and pull the embedding model
> ollama serve   # if not already running
> ollama pull nomic-embed-text
> ```
>
> **2. Milvus** (vector database)
> ```bash
> docker run -d --name milvus \
>   -p 127.0.0.1:19530:19530 -p 127.0.0.1:9091:9091 \
>   -v milvus-data:/var/lib/milvus \
>   milvusdb/milvus:latest milvus run standalone
> ```
>
> Confirm both are running:
> - Ollama: `ollama list` should show `nomic-embed-text`
> - Milvus: `docker ps | grep milvus` should show the container

### Option 2: Zilliz Cloud + Ollama

> **Zilliz Cloud + Ollama Setup**
>
> **1. Ollama** (same as Option 1 above)
>
> **2. Zilliz Cloud** — Sign up at [cloud.zilliz.com](https://cloud.zilliz.com):
>    - Create a free cluster
>    - Note your **Public Endpoint** (e.g., `https://xxx.api.gcp-us-west1.zillizcloud.com`)
>    - Create an **API Key**
>
> Please provide:
> 1. **Zilliz Cloud Endpoint** — Your cluster's public endpoint
> 2. **Zilliz Cloud API Key** — Your cluster's API key
>
> **IMPORTANT**: Store the API key as environment variable `MILVUS_TOKEN`, not directly in settings.json.

### Option 3: Zilliz Cloud + OpenAI

> **Zilliz Cloud + OpenAI Setup**
>
> Please provide:
> 1. **OpenAI API Key** — From [platform.openai.com](https://platform.openai.com)
> 2. **Zilliz Cloud Endpoint** — Your cluster's public endpoint
> 3. **Zilliz Cloud API Key** — Your cluster's API key
>
> **IMPORTANT**: Store API keys as environment variables (`OPENAI_API_KEY`, `MILVUS_TOKEN`), not directly in settings.json.

### Option 4: OpenAI-Compatible Local

> **Local Inference Server + Docker Milvus**
>
> This works with LM Studio, LocalAI, or any OpenAI-compatible API.
>
> **1. Start your local inference server** with an embedding model loaded
> **2. Milvus** (same Docker setup as Option 1)
>
> Please provide:
> 1. **API Base URL** — e.g., `http://localhost:1234/v1`
> 2. **Model name** — The embedding model name in your server

## Step 4: Apply Configuration

Based on user choices, update `.claude/settings.json` to add the claude-context MCP server. **Merge with existing config — do not replace the Shannon or other MCP servers.**

### Option 1: Fully Local

```json
{
  "mcpServers": {
    "claude-context": {
      "command": "npx",
      "args": ["-y", "@zilliz/claude-context-mcp@latest"],
      "env": {
        "EMBEDDING_PROVIDER": "Ollama",
        "EMBEDDING_MODEL": "nomic-embed-text",
        "EMBEDDING_DIMENSION": "768",
        "EMBEDDING_BATCH_SIZE": "5",
        "OLLAMA_HOST": "http://127.0.0.1:11434",
        "OLLAMA_NUM_PARALLEL": "1",
        "MILVUS_ADDRESS": "127.0.0.1:19530",
        "SPLITTER_TYPE": "ast",
        "HYBRID_MODE": "true",
        "CUSTOM_IGNORE_PATTERNS": "node_modules/**,.git/**,vendor/**,dist/**,build/**,.next/**,__pycache__/**,*.pyc,.terraform/**"
      }
    }
  }
}
```

### Option 2: Zilliz Cloud + Ollama

```json
{
  "mcpServers": {
    "claude-context": {
      "command": "npx",
      "args": ["-y", "@zilliz/claude-context-mcp@latest"],
      "env": {
        "EMBEDDING_PROVIDER": "Ollama",
        "EMBEDDING_MODEL": "nomic-embed-text",
        "EMBEDDING_DIMENSION": "768",
        "EMBEDDING_BATCH_SIZE": "5",
        "OLLAMA_HOST": "http://127.0.0.1:11434",
        "OLLAMA_NUM_PARALLEL": "1",
        "MILVUS_ADDRESS": "<user-provided-zilliz-endpoint>",
        "MILVUS_TOKEN": "${MILVUS_TOKEN}",
        "SPLITTER_TYPE": "ast",
        "HYBRID_MODE": "true",
        "CUSTOM_IGNORE_PATTERNS": "node_modules/**,.git/**,vendor/**,dist/**,build/**,.next/**,__pycache__/**,*.pyc,.terraform/**"
      }
    }
  }
}
```

### Option 3: Zilliz Cloud + OpenAI

```json
{
  "mcpServers": {
    "claude-context": {
      "command": "npx",
      "args": ["-y", "@zilliz/claude-context-mcp@latest"],
      "env": {
        "OPENAI_API_KEY": "${OPENAI_API_KEY}",
        "MILVUS_ADDRESS": "<user-provided-zilliz-endpoint>",
        "MILVUS_TOKEN": "${MILVUS_TOKEN}",
        "SPLITTER_TYPE": "ast",
        "HYBRID_MODE": "true",
        "CUSTOM_IGNORE_PATTERNS": "node_modules/**,.git/**,vendor/**,dist/**,build/**,.next/**,__pycache__/**,*.pyc,.terraform/**"
      }
    }
  }
}
```

### Option 4: OpenAI-Compatible Local

```json
{
  "mcpServers": {
    "claude-context": {
      "command": "npx",
      "args": ["-y", "@zilliz/claude-context-mcp@latest"],
      "env": {
        "EMBEDDING_PROVIDER": "OpenAI",
        "OPENAI_API_KEY": "local",
        "OPENAI_BASE_URL": "<user-provided-base-url>",
        "EMBEDDING_MODEL": "<user-provided-model-name>",
        "MILVUS_ADDRESS": "127.0.0.1:19530",
        "SPLITTER_TYPE": "ast",
        "HYBRID_MODE": "true",
        "CUSTOM_IGNORE_PATTERNS": "node_modules/**,.git/**,vendor/**,dist/**,build/**,.next/**,__pycache__/**,*.pyc,.terraform/**"
      }
    }
  }
}
```

## Step 5: Verify Setup

After applying configuration:

1. Inform the user they need to **restart Claude Code** for MCP changes to take effect
2. After restart, they can verify by running `/retrieval status`

> ### Setup Complete!
>
> claude-context MCP has been configured in `.claude/settings.json`.
>
> **Next steps:**
> 1. **Restart Claude Code** for the MCP server to load
> 2. Run `/retrieval index` to build the search index for this project
> 3. Run `/retrieval search "your query"` to test semantic search
>
> **Available tools (after restart):**
> - `search_code` — Hybrid BM25 + semantic search over indexed code
> - `index_codebase` — Build or update the code index (AST + embeddings)
> - `get_indexing_status` — Check index health and progress
> - `clear_index` — Remove the index for a codebase
>
> **Usage:** Run `/retrieval [request]` to search, index, or manage. The `/research` and `/implement` skills will also use retrieval automatically when available.

## Error Handling

- If `npx` is not available: suggest installing Node.js 20+ (`node --version` must be >= 20.0.0)
- If Docker is not available for Milvus: suggest installing Docker Desktop or using Zilliz Cloud (Option 2/3)
- If Ollama is not installed: suggest installing via `brew install ollama` (macOS) or `curl -fsSL https://ollama.com/install.sh | sh` (Linux)
- If `nomic-embed-text` not pulled: run `ollama pull nomic-embed-text`
- If Milvus container not running: `docker start milvus`
- If Ollama not running: `ollama serve`
- If embedding dimension errors occur: ensure `EMBEDDING_DIMENSION=768` is set (workaround for Ollama batch issue)
