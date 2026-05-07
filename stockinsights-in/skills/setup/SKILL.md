---
name: setup
description: Verify the StockInsights India MCP connection and API key setup before running Indian equity research queries.
---

# StockInsights India Setup

Walk the user through verifying their StockInsights India plugin setup. Be concise and practical.

## Step 1: Verify Codex

Confirm Codex is running. If the user is seeing this skill, Codex is available.

## Step 2: Check MCP Configuration

This plugin connects to the StockInsights India MCP server configured in `stockinsights-in/.mcp.json`.

Read `.mcp.json` if available and confirm:

- MCP server name: `stockinsights-in`
- API key env var: `STOCKINSIGHTS_AI_IN_API_KEY`
- MCP URL: the `url` value in `.mcp.json`

If the MCP server is unavailable, tell the user to verify `.mcp.json`, set `STOCKINSIGHTS_AI_IN_API_KEY`, and restart Codex after plugin installation.

## Step 3: Verify MCP Connection

Run a quick test by calling the India MCP tool `get_announcement_feed`.

Show the user whether the tool returned data. If it returns data, tell them setup is working. If it fails, report the error and suggest checking:

- `STOCKINSIGHTS_AI_IN_API_KEY` is set in the Codex runtime environment
- The configured MCP URL is reachable
- Codex has been restarted after plugin installation or env var changes

## Step 4: Suggested First Queries

Suggest one or two follow-up prompts:

- Research TCS using StockInsights India MCP.
- Summarize recent INFY corporate announcements.
- Search HDFCBANK filings for credit growth commentary.
