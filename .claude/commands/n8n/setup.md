---
name: n8n/setup
description: Interactive setup wizard for n8n-MCP integration. Configures hosted or self-hosted n8n-MCP with basic or full capabilities.
model: sonnet
---

# n8n-MCP Setup Wizard

You are guiding the user through setting up the **n8n-MCP** integration, which connects Claude Code to n8n's workflow automation platform via the Model Context Protocol.

## Overview

n8n-MCP provides AI-assisted access to:
- **Basic mode**: n8n documentation, 1,084+ node library, 2,709 workflow templates
- **Full mode**: Everything above + direct management of a live n8n instance (CRUD workflows, trigger executions, version control)

## Step 1: Check Existing Configuration

First, check if n8n-mcp is already configured:

```bash
# Check project settings
cat .claude/settings.json
```

If `n8n-mcp` already exists in `mcpServers`, inform the user:

> n8n-MCP is already configured in this project. Run `/n8n/setup` again to reconfigure, or use `/n8n` to start working with n8n.

If not configured, proceed to Step 2.

## Step 2: Ask About Hosting Preference

Present these options clearly and **wait for the user's response**:

> ### n8n-MCP Setup
>
> **How would you like to run n8n-MCP?**
>
> 1. **Hosted service** (zero setup) — Uses dashboard.n8n-mcp.com. Free tier: 100 tool calls/day. No local infrastructure needed.
> 2. **Self-hosted via npx** (recommended) — Runs locally via Node.js. No Docker required. ~5 second setup.
> 3. **Self-hosted via Docker** — Runs in a container. Requires Docker installed. ~280MB image.
> 4. **Local development** — Clone the repo and build from source. For contributors or custom modifications.
>
> Which option? (1-4)

## Step 3: Ask About Capabilities

After the user chooses hosting, ask:

> **What capabilities do you need?**
>
> 1. **Basic** (documentation only) — Search nodes, browse templates, validate workflows. No n8n instance required.
> 2. **Full** (instance management) — Everything in Basic + create/edit/delete/trigger workflows on your n8n instance. Requires n8n API URL and API key.
>
> Which option? (1-2)

## Step 4: Collect Credentials (Full mode only)

If the user chose Full capabilities:

> **n8n Instance Details**
>
> I need two pieces of information:
> 1. **n8n API URL** — Your n8n instance URL (e.g., `https://your-instance.app.n8n.cloud` or `http://localhost:5678`)
> 2. **n8n API Key** — Generate one in n8n: Settings → API → Create API Key
>
> Please provide your n8n API URL and API key.

**IMPORTANT**: Never store API keys directly in `.claude/settings.json` or any committed file. Instead:
- Store the API key as an environment variable `N8N_API_KEY`
- Reference it in the configuration
- Suggest adding it to a `.env` file (which must be in `.gitignore`)

## Step 5: Apply Configuration

Based on user choices, update `.claude/settings.json` to add the n8n-mcp server.

### Option 1: Hosted Service

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "type": "url",
      "url": "https://mcp.n8n-mcp.com/mcp"
    }
  }
}
```

Note: The user will need to sign up at dashboard.n8n-mcp.com and configure their API token separately.

### Option 2: npx (Basic)

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["n8n-mcp"],
      "env": {
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true"
      }
    }
  }
}
```

### Option 2: npx (Full)

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["n8n-mcp"],
      "env": {
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true",
        "N8N_API_URL": "<user-provided-url>",
        "N8N_API_KEY": "<user-provided-key>"
      }
    }
  }
}
```

### Option 3: Docker (Basic)

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm", "--init",
        "-e", "MCP_MODE=stdio",
        "-e", "LOG_LEVEL=error",
        "-e", "DISABLE_CONSOLE_OUTPUT=true",
        "ghcr.io/czlonkowski/n8n-mcp:latest"
      ]
    }
  }
}
```

### Option 3: Docker (Full)

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm", "--init",
        "-e", "MCP_MODE=stdio",
        "-e", "LOG_LEVEL=error",
        "-e", "DISABLE_CONSOLE_OUTPUT=true",
        "-e", "N8N_API_URL=<user-provided-url>",
        "-e", "N8N_API_KEY=<user-provided-key>",
        "ghcr.io/czlonkowski/n8n-mcp:latest"
      ]
    }
  }
}
```

### Option 4: Local Development

Guide the user:

```bash
git clone https://github.com/czlonkowski/n8n-mcp.git
cd n8n-mcp
npm install
npm run build
npm run rebuild
```

Then configure settings.json to point to the local build:

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "node",
      "args": ["<path-to-clone>/dist/index.js"],
      "env": {
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true"
      }
    }
  }
}
```

## Step 6: Security Configuration (Optional)

Ask the user:

> **Optional: Privacy & Telemetry**
>
> n8n-MCP collects anonymous usage telemetry by default. Would you like to disable it? (y/n)

If yes, add `"N8N_MCP_TELEMETRY_DISABLED": "true"` to the env block.

## Step 7: Verify Setup

After applying configuration:

1. Inform the user they need to **restart Claude Code** for MCP changes to take effect
2. After restart, they can verify by running `/n8n` which will attempt to use the n8n-mcp tools

> ### Setup Complete!
>
> n8n-MCP has been configured in `.claude/settings.json`.
>
> **Next steps:**
> 1. Restart Claude Code for the MCP server to load
> 2. Run `/n8n` to start working with n8n workflows
>
> **Available tools (after restart):**
> - `search_nodes` — Search n8n's 1,084+ node library
> - `get_node` — Get detailed node documentation
> - `search_templates` — Browse 2,709 workflow templates
> - `validate_workflow` — Validate workflow configurations
> {if full mode:}
> - `list_workflows` — List workflows on your instance
> - `create_workflow` — Create new workflows
> - `update_workflow` — Modify existing workflows
> - `trigger_workflow` — Execute workflows
> - And more...

## Error Handling

- If `npx` is not available: suggest installing Node.js 18+
- If Docker is not available: suggest installing Docker Desktop or switching to npx
- If n8n API URL is unreachable: suggest checking the URL and network connectivity
- If API key is invalid: guide user to regenerate in n8n Settings → API
