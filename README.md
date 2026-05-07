# StockInsights.ai Plugins for Codex

StockInsights.ai Codex plugins add MCP-powered equity research workflows for public companies. The marketplace includes separate plugins for India and US coverage so users can install only the region they need.

## Prerequisites

- **Codex** - Install or upgrade Codex before installing these plugins.
- **StockInsights.ai account** - You need a StockInsights.ai API key for each region you use.

## Installation

```bash
# 1. Add the marketplace
codex plugin marketplace add stockinsights-ai/plugins

# 2. Install the India plugin
codex plugin install stockinsights-in

# 3. Install the US plugin
codex plugin install stockinsights-us
```

If `codex plugin marketplace add` returns a marketplace command not found error, upgrade Codex and retry the installation.

## Configuration

Each plugin reads its API key from the environment variable configured in its `.mcp.json` file.

```bash
# India
export STOCKINSIGHTS_AI_IN_API_KEY="..."

# US
export STOCKINSIGHTS_AI_US_API_KEY="..."
```

Check these files before running queries:

- [stockinsights-in/.mcp.json](./stockinsights-in/.mcp.json)
- [stockinsights-us/.mcp.json](./stockinsights-us/.mcp.json)

## Getting Started

Ask Codex to run the setup check for the region you installed:

```text
Run StockInsights India setup
Run StockInsights US setup
```

The setup skill verifies Codex is running, checks the MCP configuration, confirms the expected API key environment variable, and tests the announcements feed tool.

## Plugins

| Plugin | Region | MCP test tool | Example |
|--------|--------|---------------|---------|
| `stockinsights-in` | India | `get_announcement_feed` | Research TCS using StockInsights India MCP |
| `stockinsights-us` | US | `get_announcements_feed` | Research AAPL using StockInsights US MCP |

## Data Sources

- **StockInsights India MCP** - Indian company filings, corporate announcements, semantic search, keyword search, and AI insight summaries.
- **StockInsights US MCP** - US company filings, company documents, semantic search, keyword search, and AI insight summaries.

Results are returned directly in the Codex conversation. Treat outputs as research material, not investment advice.
