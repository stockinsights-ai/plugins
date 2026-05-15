---
name: setup
description: Verify the stockinsights.ai India MCP connection and OAuth setup before running Indian equity research queries.
---

# stockinsights.ai India Setup

Walk the user through verifying their stockinsights.ai India plugin setup. Be concise and practical.

## Step 1: Verify Codex

Confirm Codex is running. If the user is seeing this skill, Codex is available.

## Step 2: Check MCP Configuration

This plugin connects to the stockinsights.ai India MCP server configured in `stockinsights-in/.mcp.json`.

Read `.mcp.json` if available and confirm:

- MCP server name: `stockinsights-in`
- Auth: OAuth on first connection
- MCP URL: the `url` value in `.mcp.json`

If the MCP server is unavailable, tell the user to verify `.mcp.json` and restart Codex after plugin installation.

## Step 3: Verify MCP Connection

Run a quick test by calling the India MCP tool `get_announcement_feed`.

Show the user whether the tool returned data. If OAuth is required, ask the user to complete the browser-based stockinsights.ai authorization flow. If it returns data, tell them setup is working. If it fails, report the error and suggest checking:

- The stockinsights.ai OAuth flow completed successfully
- The configured MCP URL is reachable
- Codex has been restarted after plugin installation

## Step 4: Suggested First Queries

Suggest one or two follow-up prompts:

- Research TCS using stockinsights.ai India MCP.
- Summarize recent INFY corporate announcements.
- Search HDFCBANK filings for credit growth commentary.
