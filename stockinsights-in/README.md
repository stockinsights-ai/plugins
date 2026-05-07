# StockInsights India

Codex plugin for Indian equity research with StockInsights.ai.

This plugin follows the same shape as the Cloudflare plugin example: a Codex manifest, local assets, and a remote HTTP MCP server configuration. Skills are intentionally omitted for now.

## MCP Server

| Server | Purpose |
|--------|---------|
| stockinsights-in | Access StockInsights.ai filings search, semantic search, corporate announcements, and AI insight summaries for Indian public companies |

The plugin currently points to:

```text
https://stockinsights-ai-feat-mcp-f1f2f2b.zuplo.app/mcp
```

Set `STOCKINSIGHTS_AI_IN_API_KEY` in the Codex runtime environment. Update [`.mcp.json`](./.mcp.json) if the production MCP server is deployed at a different URL.

## Example Prompts

- Research TCS using StockInsights India MCP.
- Summarize recent INFY corporate announcements.
- Search HDFCBANK filings for credit growth commentary.
