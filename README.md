# stockinsights.ai Plugins for Claude Code and Codex

stockinsights.ai Claude Code and Codex plugins add MCP-powered equity research workflows for public companies. The marketplace includes separate plugins for India and US coverage so users can install only the region they need.

## Prerequisites

- **Claude Code or Codex** - Install or upgrade your agent before installing these plugins.
- **stockinsights.ai account** - You need an account that can authenticate with the stockinsights.ai MCP server.

## Installation

Install the plugin for the coding agent you use. Claude Code is the preferred flow, and Codex is supported with the same shared plugin folders and MCP configuration.

### Claude Code

Run these commands inside Claude Code:

```text
/plugin marketplace add stockinsights-ai/plugins
```

Then install the region you need:

```text
# India
/plugin install stockinsights-in@stockinsights-ai

# US
/plugin install stockinsights-us@stockinsights-ai
```

If you do not see the `/plugin` command in Claude Code, upgrade Claude Code and retry.

### Codex

Run these commands in your terminal:

```text
codex plugin marketplace add stockinsights-ai/plugins
```

Then install the region you need:

```bash
# India
codex plugin install stockinsights-in

# US
codex plugin install stockinsights-us
```

If `codex plugin marketplace add` returns a marketplace command not found error, upgrade Codex and retry.

## Configuration

No API key environment variable is required. Each plugin connects to a hosted stockinsights.ai MCP server configured in its `.mcp.json` file and uses OAuth on first connection.

When your coding agent first connects to the MCP server, complete the browser-based stockinsights.ai authorization flow. The agent manages the OAuth token after authentication.

## Getting Started

Ask your coding agent to run the setup check for the region you installed:

```text
Run /stockinsights.ai India setup
Run /stockinsights.ai US setup
```

The setup skill verifies the agent is running, checks the MCP configuration, confirms OAuth authentication, and tests the announcements feed tool.

## Plugins

| Plugin             | Region | MCP test tool            | Example                                       |
| ------------------ | ------ | ------------------------ | --------------------------------------------- |
| `stockinsights-in` | India  | `get_announcement_feed`  | Research TCS using stockinsights.ai India MCP |
| `stockinsights-us` | US     | `get_announcements_feed` | Research AAPL using stockinsights.ai US MCP   |

## Data Sources

- **stockinsights.ai India MCP** - Indian company filings, corporate announcements, semantic search, keyword search, and AI insight summaries.
- **stockinsights.ai US MCP** - US company filings, company documents, semantic search, keyword search, and AI insight summaries.

Results are returned directly in your agent conversation. Treat outputs as research material, not investment advice.
