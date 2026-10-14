# stockinsights.ai India

Evidence-backed equity research on Indian listed companies, powered by the stockinsights.ai India MCP server.

## Setup

```
/plugin install stockinsights-india-plugin --marketplace stockinsights-ai/plugins
```

Install the plugin. On first use, the `stockinsights-india` MCP server (`https://api-in.stockinsights.ai/mcp`) opens a browser window for stockinsights.ai sign-in (OAuth). If tools are unavailable later, re-authorise the connector.

## Skills

| Skill | Use |
| --- | --- |
| `stock-research` | Answers questions about Indian listed companies. It covers financial statements and ratios, valuation and market data, screening and peers, shareholding, filings and earnings calls, corporate announcements, and results dates. Reported figures include their source period and units; filing claims include available source links. |

The skill triggers automatically on questions such as:

- "What was TCS's revenue and net margin over the last 8 quarters?"
- "Which large-cap banks have the highest ROE?"
- "What did Infosys management say about deal wins last quarter?"
- "Latest announcements for Tata Motors"

## Data covered

- Company profiles, market data, and industry classification
- XBRL income statements, balance sheets, cash flows, and segment revenue
- Shareholding patterns
- Earnings-call transcripts, annual reports, investor presentations, and quarterly results
- Corporate announcements
- Results calendar

## Data and privacy

The plugin runs no local code and bundles no scripts or binaries. It contains skill instructions and one remote MCP server, `stockinsights-india` (`https://api-in.stockinsights.ai/mcp`, HTTPS). Your research queries (company names, filters, and search text) are sent to that server to retrieve data, and results are returned to Claude. Authentication is OAuth in the browser; the plugin never asks for or stores credentials, and no API keys are read from your environment. A stockinsights.ai account is required. The plugin is read-only and does not place trades or orders.

Outputs are research aids, not investment advice.

## Support and legal

- Website: https://www.stockinsights.ai
- Support: https://www.stockinsights.ai/contact-us
- Privacy policy: https://www.stockinsights.ai/privacy
- Terms of service: https://www.stockinsights.ai/terms
- Source: https://github.com/stockinsights-ai/plugins
- License: MIT (see `LICENSE`)
