# stockinsights.ai US

Evidence-backed equity research on US listed companies, powered by the stockinsights.ai US MCP server.

## Setup

```
/plugin install stockinsights-us --marketplace stockinsights-ai/plugins
```

Install the plugin. On first use, the `stockinsights-us` MCP server (`https://api-us.stockinsights.ai/mcp`) opens a browser window for stockinsights.ai sign-in (OAuth). If tools are unavailable later, re-authorise the connector.

## Skills

| Skill | Use |
| --- | --- |
| `stock-research` | Answers questions about US listed companies. It covers reported financial statements, company profiles and peers, SEC filings and earnings calls, and corporate disclosures. Every figure is cited to its source, with currency, units and the issuer's fiscal period stated. |
| `get-started` | Verifies the connection and sign-in, then suggests first questions. |

The research skill triggers automatically on questions such as:

- "What was Apple's revenue and diluted EPS over the last 3 fiscal years?"
- "Who are Tesla's industry peers?"
- "What did NVIDIA management say about data-center demand last quarter?"
- "Latest 8-K disclosures for Boeing"

## Data covered

- Company profiles, industry classification, and a stored market-cap snapshot
- XBRL income statements, balance sheets, and cash flows from 10-K, 10-Q, and 20-F filings
- Earnings-call transcripts, 10-K, 10-Q, and 20-F filings (search and read)
- 8-K/6-K corporate disclosures

Not available: live or historical prices, brokerage research, shareholding data, and a US earnings calendar.

## Data and privacy

The plugin runs no local code and bundles no scripts or binaries. It contains skill instructions and one remote MCP server, `stockinsights-us` (`https://api-us.stockinsights.ai/mcp`, HTTPS). Your research queries (company names, filters, and search text) are sent to that server to retrieve data, and results are returned to Claude. Authentication is OAuth in the browser; the plugin never asks for or stores credentials, and no API keys are read from your environment. A stockinsights.ai account is required. The plugin is read-only and does not place trades or orders.

Outputs are research aids, not investment advice.

## Support and legal

- Website: https://www.stockinsights.ai
- Support: https://www.stockinsights.ai/contact-us
- Privacy policy: https://www.stockinsights.ai/privacy
- Terms of service: https://www.stockinsights.ai/terms
- Source: https://github.com/stockinsights-ai/plugins
- License: MIT (see `LICENSE`)
