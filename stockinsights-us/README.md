# StockInsights US

Codex plugin for US equity research with StockInsights.ai.

This plugin follows the same shape as the Cloudflare plugin example: a Codex manifest, local assets, and a remote HTTP MCP server configuration. Skills are intentionally omitted for now.

## MCP Server

| Server | Purpose |
|--------|---------|
| stockinsights-us | Access StockInsights.ai filings search, semantic search, company documents, and AI insight summaries for US public companies |

The plugin currently points to:

```text
https://stockinsights-ai-us-feat-mcp-e21e62f.zuplo.app/mcp
```

Set `STOCKINSIGHTS_AI_US_API_KEY` in the Codex runtime environment. Update [`.mcp.json`](./.mcp.json) if the production MCP server is deployed at a different URL.

## Example Prompts

- Research AAPL using StockInsights US MCP.
- Search MSFT filings for AI capex.
- Summarize recent NVDA filings.
