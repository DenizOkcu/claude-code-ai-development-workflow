---
name: firecrawl/setup
description: Interactive setup wizard for Firecrawl MCP integration. Configures self-hosted or cloud Firecrawl for web scraping and crawling.
model: sonnet
---

# Firecrawl MCP Setup Wizard

You are guiding the user through setting up the **Firecrawl MCP** integration, which gives Claude Code powerful web scraping, crawling, and content extraction capabilities — especially useful as a fallback when the built-in `WebFetch` tool fails on complex pages (JavaScript-rendered, anti-bot protected, etc.).

## Overview

Firecrawl MCP provides:
- **Scrape** — Extract clean markdown/structured data from any URL
- **Crawl** — Recursively crawl entire websites with depth control
- **Search** — Web search with content extraction in one step
- **Map** — Discover all URLs on a website
- **Extract** — LLM-powered structured data extraction from pages

## Step 1: Check Existing Configuration

First, check if firecrawl is already configured:

```bash
cat .claude/settings.json
```

If `firecrawl` already exists in `mcpServers`, inform the user:

> Firecrawl MCP is already configured in this project. Run `/firecrawl/setup` again to reconfigure, or use `/firecrawl` to start scraping.

If not configured, proceed to Step 2.

## Step 2: Ask About Hosting Preference

Present these options clearly and **wait for the user's response**:

> ### Firecrawl MCP Setup
>
> **How would you like to run Firecrawl?**
>
> 1. **Self-hosted** (recommended for this project) — Run Firecrawl locally via Docker. Full control, no API limits, no external dependencies. Requires Docker.
> 2. **Self-hosted via npx** — Run the MCP server via npx, pointing to your self-hosted Firecrawl instance.
> 3. **Cloud API** — Use Firecrawl's hosted API at `api.firecrawl.dev`. Requires an API key (free tier: 500 credits).
>
> Which option? (1-3)

## Step 3: Collect Configuration Details

### Option 1: Self-hosted via Docker

Guide the user to start a self-hosted Firecrawl instance:

> **Self-Hosted Firecrawl Setup**
>
> You'll need to run the Firecrawl service first. If you haven't already:
>
> ```bash
> git clone https://github.com/firecrawl/firecrawl.git
> cd firecrawl
> # Copy and configure environment
> cp .env.example .env
> # Edit .env — set your own FIRECRAWL_API_KEY (any string you choose for self-hosted)
> docker compose up -d
> ```
>
> Once running, provide:
> 1. **Firecrawl API URL** — Default: `http://localhost:3002` (or your custom URL)
> 2. **API Key** — The key you set in `.env` (for self-hosted, this is your own chosen key)

### Option 2: Self-hosted via npx

> **Self-Hosted Firecrawl + npx MCP**
>
> This runs the MCP server locally via npx, connecting to your self-hosted Firecrawl instance.
>
> Please provide:
> 1. **Firecrawl API URL** — Your self-hosted instance URL (e.g., `http://localhost:3002`)
> 2. **API Key** — The key configured in your Firecrawl instance

### Option 3: Cloud API

> **Firecrawl Cloud Setup**
>
> 1. Sign up at [firecrawl.dev](https://firecrawl.dev) to get an API key
> 2. Provide your **API Key** (starts with `fc-`)
>
> Free tier includes 500 credits. No self-hosted infrastructure needed.

**IMPORTANT**: Never store API keys directly in `.claude/settings.json` or any committed file. Instead:
- Store the API key as an environment variable `FIRECRAWL_API_KEY`
- Reference it in the configuration
- Suggest adding it to a `.env` file (which must be in `.gitignore`)

## Step 4: Apply Configuration

Based on user choices, update `.claude/settings.json` to add the firecrawl MCP server.

### Option 1: Self-hosted Docker (with npx MCP client)

```json
{
  "mcpServers": {
    "firecrawl": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_URL": "<user-provided-url>",
        "FIRECRAWL_API_KEY": "<user-provided-key>"
      }
    }
  }
}
```

### Option 2: Self-hosted via npx

```json
{
  "mcpServers": {
    "firecrawl": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_URL": "<user-provided-url>",
        "FIRECRAWL_API_KEY": "<user-provided-key>"
      }
    }
  }
}
```

### Option 3: Cloud API

```json
{
  "mcpServers": {
    "firecrawl": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_KEY": "<user-provided-key>"
      }
    }
  }
}
```

Note: When `FIRECRAWL_API_URL` is omitted, the MCP server defaults to Firecrawl's cloud API.

## Step 5: Verify Setup

After applying configuration:

1. Inform the user they need to **restart Claude Code** for MCP changes to take effect
2. After restart, they can verify by running `/firecrawl test https://example.com`

> ### Setup Complete!
>
> Firecrawl MCP has been configured in `.claude/settings.json`.
>
> **Next steps:**
> 1. Restart Claude Code for the MCP server to load
> 2. Run `/firecrawl test https://example.com` to verify
>
> **Available tools (after restart):**
> - `firecrawl_scrape` — Scrape a single URL to clean markdown
> - `firecrawl_crawl` — Recursively crawl a website
> - `firecrawl_search` — Web search + content extraction
> - `firecrawl_map` — Discover all URLs on a site
> - `firecrawl_extract` — LLM-powered structured data extraction
>
> **Usage:** Run `/firecrawl [request]` or use Firecrawl tools directly when `WebFetch` fails on complex pages.

## Error Handling

- If `npx` is not available: suggest installing Node.js 18+
- If Docker is not available for self-hosted: suggest installing Docker Desktop or using cloud API
- If Firecrawl API URL is unreachable: check if the Docker container is running (`docker ps | grep firecrawl`)
- If API key is invalid: for self-hosted, verify the key matches `.env`; for cloud, regenerate at firecrawl.dev
- If rate limited (cloud): suggest upgrading plan or switching to self-hosted
